# RV64I ISA

This is the step 2 of the book [_Writing a RISC-V Emulator from Scratch in 10 Steps_](./), whose goal is running [xv6](https://github.com/mit-pdos/xv6-riscv), a small Unix-like OS, in your emulator in the final step.

The source code is available at [d0iasm/rvemu-for-book/step2/](https://github.com/d0iasm/rvemu-for-book/tree/master/step2).

## Goal of This Page

In the end of this page, we can execute the sample file that calculates [a fibonacci number](https://en.wikipedia.org/wiki/Fibonacci_number) in our emulator. We will support RV64 ISAs, the base integer instruction set a 64-bit architecture, to calculate a fibonacci number.

Sample binary files are also available at [d0iasm/rvemu-for-book/step2/](https://github.com/d0iasm/rvemu-for-book/tree/master/step2). We successfully see the result of 10th fibonacci number when we execute the sample binary file `fib.text`.

```text
// fib.c contains the following C code and fib.text is the build result of it:
// int fib(int n);
// int main() {
//   return fib(10); // Calculate the 10th fibonacci number.
// }
// int fib(int n) {
//   if (n == 0 || n == 1)
//     return n;
//   else
//     return (fib(n-1) + fib(n-2));
// }

$ cargo run fib.text
...           
x12=0x0 x13=0x0 x14=0x1 x15=0x37 // x15 should contain 55 (= 10th fibonacci number).
```

## RV64I: Base Integer Instruction Set

RV64I is a base integer instruction set for the 64-bit architecture, which builds upon the RV32I variant. RV64I shares most of instructions with RV32I but the width of registers is different and there are a few additional instructions only in RV64I.

In this step, we're going to implement 47 instructions \(35 instructions from RV32I and 12 instructions from RV64I\). We've already implemented `add` and `addi` so we'll skip them. Also, we'll skip to implement `fence`, `ecall`, and `ebreak` for now. I'll cover `ecall` and `ebreak` in the following step and won't explain `fence`. The `fence` instruction is a type of barrier instruction to apply an ordering constraint on memory operations issued before and after it. We don't need it since our emulator is a single core system and doesn't reorder memory operations \(out-of-order execution\).

Fig 2.1 and Fig 2.2 are the list for RV32I and RV64I, respectively. We're going to implement all instructions in the figures.

![Fig 2.1 RV32I Base Instruction Set \(Source: RV32I Base Instruction Set table in Volume I: Unprivileged ISA\)](.gitbook/assets/rvemubook-rv32i.png)

![Fig 2.2 RV64I Base Instruction Set \(Source: RV64I Base Instruction Set table in Volume I: Unprivileged ISA\)](.gitbook/assets/screen-shot-2020-04-19-at-20.20.35.png)

## CPU Module

First, we're going to divide the implementation of CPU from the `main.rs` file. Rust provides a module system to split code in logical units and organize visibility. We're going to move the implementation of CPU to a new file `cpu.rs`.

In order to define a cpu module we need to `mod` keyword at the beginning of the main file. Also `use` keyword allows to use public items in the cpu module.

{% code title="src/main.rs" %}
```rust
// This declaration will look for a file named `cpu.rs` or `cpu/mod.rs` and will
// insert its contents inside a module named `cpu` under this scope.
mod cpu;

// Use all public structures, methods, and functions defined in the cpu module.
use crate::cpu::*;
```
{% endcode %}

{% code title="src/cpu.rs" %}
```rust
// `pub` keyword allows other modules use the `Cpu` structure and methods
// relating to it. 
pub struct Cpu {
    ...
}

impl Cpu {
    ...
}
```
{% endcode %}

### Fetch-decode-execute Cycle

The step 1 already mentioned the fetch-decode-execute cycle and we're going to implement it in the `main.rs`. An emulator is ideally an infinite loop and execute program infinitely unless something wrong happens or a user stops an emulator explicitly. However, we're going to stop an emulator implicitly when the program counter is 0 or over the length of memory, and an error happens during the execution.

{% code title="src/main.rs" %}
```rust
fn main() -> io::Result<()> {
    ...
    while cpu.pc < cpu.memory.len() as u64 {
        // 1. Fetch
        let inst = cpu.fetch();
        
        // 2. Add 4 to the program counter.
        cpu.pc += 4;

        // 3. Decode.
        // 4. Execute.
        match cpu.execute(inst) {
            // True if an error happens.
            true => break,
            false => {}
        };

        // This is a workaround for avoiding an infinite loop.
        if cpu.pc == 0 {
            break;
        }
    }
    ...
}
```
{% endcode %}

### Fetch Stage

The fetch stage is basically same with the previous step, but I prefer to create a new function to read 32-bit data from a memory because there are other instructions to read and write memory.

{% code title="src/cpu.rs" %}
```rust
impl Cpu {
    ...  
    fn read32(&self, addr: u64) -> u64 {
        let index = addr as usize;
        return (self.memory[index] as u64)
            | ((self.memory[index + 1] as u64) << 8)
            | ((self.memory[index + 2] as u64) << 16)
            | ((self.memory[index + 3] as u64) << 24);
    }
    ...    
    pub fn fetch(&self) -> u32 {
        return self.read32(self.pc) as u32;
    }
    ...
}
```
{% endcode %}

### Decode Stage

The decode stage is almost same with the previous step too and we'll add 2 new fields `funct3` and `funct7`. `funct3` is located from 12 to 14 bits and `funct7` is located from 25 to 31 bits as we can see in the Fig 2.1 and 2.2. These fields and opcode select the type of operation.

{% code title="src/cpu.rs" %}
```rust
impl Cpu {
    ... 
    fn execute(&mut self, inst: u32) {
        ...
        let funct3 = (inst & 0x00007000) >> 12;
        let funct7 = (inst & 0xfe000000) >> 25;
        ...
```
{% endcode %}

In RISC-V, there are many common positions in all formats, but decoding an immediate value is quite different depending on instructions, so we'll decode an immediate value in each operation.

For example, the immediate value in branch instructions is located in the place of `rd` and `funct7`. A branch instruction is a `if` statement in C to change the sequence of instruction execution depending on a condition, which includes `beq`, `bne`, `blt`, `bge`, `bltu`, and `bgeu`.

Decoding is performed by bitwise ANDs and bit shifts. The point to be noted is that an immediate value should be sign-extended. It means we need to fill in the upper bits with 1 when the significant bit is 1. In this implementation, filling in bits with 1 is performed by casting from a signed integer to an unsigned integer.

The way how to decode each instruction is listed in Fig 2.1 and Fig 2.2.

{% code title="src/cpu.rs" %}
```rust
impl Cpu {
    ... 
    fn execute(&mut self, inst: u32) {
        ...
        match opcode {
            0x63 => {
                // imm[12|10:5|4:1|11] = inst[31|30:25|11:8|7]
                let imm = (((inst & 0x80000000) as i32 as i64 >> 19) as u64)
                    | ((inst & 0x80) << 4)   // imm[11]
                    | ((inst >> 20) & 0x7e0) // imm[10:5]
                    | ((inst >> 7) & 0x1e);  // imm[4:1]
                    
                match funct3 {
                    ...
```
{% endcode %}

### Execute Stage

Each operation is performed in each `match` arm. For example, a branch instruction `beq`, which is one of the branch instructions, is executed when `opcode` is 0x63 and `funct3` is 0x0. `beq` sets the `pc` to the current `pc` plus the signed-extended offset if a value in `rs1` equals a value in `rs2`. The current `pc` means the position when CPU fetched an instruction from memory so we need to subtract 4 from `pc` because we added 4 after fetch.

{% code title="src/cpu.rs" %}
```rust
impl Cpu {
    ... 
    fn execute(&mut self, inst: u32) {
        ...
        match opcode {
            0x63 => {
                let imm = ...;

                match funct3 {
                    0x0 => {
                        // beq
                        if self.regs[rs1] == self.regs[rs2] {
                            self.pc = self.pc.wrapping_add(imm).wrapping_sub(4);
                        }
                    }
                ...
```
{% endcode %}

## Instructions List

The following table is the brief explanation for each instruction. The book won't describe the details of each instruction but will indicate points to be noted when you implement instructions. In addition, you can see the implementation in [d0iasm/rvemu-for-book/step2/src/cpu.rs](https://github.com/d0iasm/rvemu-for-book/blob/master/step2/src/cpu.rs) and description in _Chapter 2 RV32I Base Integer Instruction Set_ and _Chapter 5 RV64I Base Integer Instruction Set_ in [the unprivileged specification](https://riscv.org/specifications/isa-spec-pdf/).

**Points to be noted**:

* Arithmetic operations are done by wrapping\_\* functions to avoid an overflow.
* Sign-extended is done by casting from a smaller signed integer to a larger signed integer.
* The amount for 64-bit shift operations is encoded in the lower 6 bits in an immediate, and the amount for 32-bit shift operations is encoded in the lower 5 bits.

| Instruction | Pseudocode | Description |
| :--- | :--- | :--- |
| **lui** rd, imm | x\[rd\] = sext\(imm\[31:12\] &lt;&lt; 12\) | Load upper immediate value. |
| **auipc** rd, imm | x\[rd\] = pc + sext\(imm\[31:12\] &lt;&lt; 12\) | Add upper immediate value to PC. |
| **jal** rd, offset | x\[rd\] = pc + 4; pc += sext\(offset\) | Jump and link. |
| **jalr** rd, offset\(rs1\) | t = pc+4; pc = \(x\[rs1\] + sext\(offset\)&~1\); x\[rd\] = t | Jump and link register. |
| **beq** rs1, rs2, offset | if \(rs1 == rs2\) pc += sext\(offset\) | Branch if equal. |
| **bne** rs1, rs2, offset | if \(rs1 != rs2\) pc += sext\(offset\) | Branch if not equal. |
| **blt** rs1, rs2, offset | if \(rs1 &lt; rs2\) pc += sext\(offset\) | Branch if less than. |
| **bge** rs1, rs2, offset | if \(rs1 &gt;= rs2\) pc += sext\(offset\) | Branch if greater than or equal. |
| **bltu** rs1, rs2, offset | if \(rs1 &lt; rs2\) pc += sext\(offset\) | Branch if less than, unsigned. |
| **bgeu** rs1, rs2, offset | if \(rs1 &gt;= rs2\) pc += sext\(offset\) | Branch if greater than or equal, unsigned. |
| **lb** rd, offset\(rs1\) | x\[rd\] = sext\(M\[x\[rs1\] + sext\(offset\)\]\[7:0\]\) | Load byte \(8 bits\). |
| **lh** rd, offset\(rs1\) | x\[rd\] = sext\(M\[x\[rs1\] + sext\(offset\)\]\[15:0\]\) | Load halfword \(16 bits\). |
| **lw** rd, offset\(rs1\) | x\[rd\] = sext\(M\[x\[rs1\] + sext\(offset\)\]\[31:0\]\) | Load word \(32 bits\). |
| **lbu** rd, offset\(rs1\) | x\[rd\] = M\[x\[rs1\] + sext\(offset\)\]\[7:0\] | Load byte, unsigned. |
| **lhu** rd, offset\(rs1\) | x\[rd\] = M\[x\[rs1\] + sext\(offset\)\]\[15:0\] | Load halfword, unsigned. |
| **sb** rs2, offset\(rs1\) | M\[x\[rs1\] + sext\(offset\)\] = x\[rs2\]\[7:0\] | Store byte. |
| **sh** rs2, offset\(rs1\) | M\[x\[rs1\] + sext\(offset\)\] = x\[rs2\]\[15:0\] | Store halfword. |
| **sw** rs2, offset\(rs1\) | M\[x\[rs1\] + sext\(offset\)\] = x\[rs2\]\[31:0\] | Store word. |
| **addi** rd, rs1, imm | x\[rd\] = x\[rs1\] + sext\(imm\) | Add immediate. |
| **slti** rd, rs1, imm | x\[rd\] = x\[rs1\] &lt; x\[rs2\] | Set if less than. |
| **sltiu** rd, rs1, imm | x\[rd\] = x\[rs1\] &lt; x\[rs2\] | Set if less than, unsigned. |
| **xori** rd, rs1, imm | x\[rd\] = x\[rs1\] ^ sext\(imm\) | Exclusive OR immediate. |
| **ori** rd, rs1, imm | x\[rd\] = x\[rs1\] \| sext\(imm\) | OR immediate. |
| **andi** rd, rs1, imm | x\[rd\] = x\[rs1\] & sext\(imm\) | AND immediate. |
| **slli** rd, rs1, shamt | x\[rd\] = x\[rs1\] &lt;&lt; shamt | Shift left logical immediate. |
| **srli** rd, rs1, shamt | x\[rd\] = x\[rs1\] &gt;&gt; shamt | Shift right logical immediate. |
| **srai** rd, rs1, shamt | x\[rd\] = x\[rs1\] &gt;&gt; shamt | Shift right arithmetic immediate. |
| **add** rd, rs1, rs2 | x\[rd\] = x\[rs1\] + x\[rs2\] | Add. |
| **sub** rd, rs1, rs2 | x\[rd\] = x\[rs1\] - x\[rs2\] | Subtract. |
| **sll** rs, rs1, rs2 | x\[rd\] = x\[rs1\] &lt;&lt; x\[rs2\] | Shift left logical. |
| **slt** rd, rs1, rs2 | x\[rd\] = x\[rs1\] &lt; x\[rs2\] | Set if less than. |
| **sltu** rd, rs1, rs2 | x\[rd\] = x\[rs1\] &lt; x\[rs2\] | Set if less than, unsigned. |
| **xor** rd, rs1, rs2 | x\[rd\] = x\[rs1\] ^ x\[rs2\] | Exclusive OR. |
| **srl** rd, rs1, rs2 | x\[rd\] = x\[rs1\] &gt;&gt; x\[rs2\] | Shift right logical. |
| **sra** rd, rs1, rs2 | x\[rd\] = x\[rs1\] &gt;&gt; x\[rs2\] | Shift right arithmetic. |
| **or** rd, rs1, rs2 | x\[rd\] = x\[rs1\] \| x\[rs2\] | OR. |
| **and** rd, rs1, rs2 | x\[rd\] = x\[rs1\] & x\[rs2\] | AND. |
| **lwu** rd, offset\(rs1\) | x\[rd\] = M\[x\[rs1\] + sext\(offset\)\]\[31:0\] | Load word, unsigned. |
| **ld** rd, offset\(rs1\) | x\[rd\] = M\[x\[rs1\] + sext\(offset\)\]\[63:0\] | Load doubleword \(64 bits\), unsigned. |
| **sd** rs2, offset\(rs1\) | M\[x\[rs1\] + sext\(offset\)\] = x\[rs2\]\[63:0\] | Store doubleword. |
| **addiw** rd, rs1, imm | x\[rd\] = sext\(\(x\[rs1\] + sext\(imm\)\)\[31:0\]\) | Add word immediate. |
| **slliw** rd, rs1, shamt | x\[rd\] = sext\(\(x\[rs1\] &lt;&lt; shamt\)\[31:0\]\) | Shift left logical word immediate. |
| **srliw** rd, rs1, shamt | x\[rd\] = sext\(\(x\[rs1\] &gt;&gt; shamt\)\[31:0\]\) | Shift right logical word immediate. |
| **sraiw** rd, rs1, shamt | x\[rd\] = sext\(\(x\[rs1\] &gt;&gt; shamt\)\[31:0\]\) | Shift right arithmetic word immediate. |
| **addw** rd, rs1, rs2 | x\[rd\] = sext\(\(x\[rs1\] + x\[rs2\]\)\[31:0\]\) | Add word. |
| **subw** rd, rs1, rs2 | x\[rd\] = sext\(\(x\[rs1\] - x\[rs2\]\)\[31:0\]\) | Subtract word. |
| **sllw** rd, rs1, rs2 | x\[rd\] = sext\(\(x\[rs1\] &lt;&lt; x\[rs2\]\[4:0\]\)\[31:0\]\) | Shift left logical word. |
| **srlw** rd, rs1, rs2 | x\[rd\] = sext\(x\[rs1\]\[31:0\] &lt;&lt; x\[rs2\]\[4:0\]\) | Shift right logical word. |
| **sraw** rd, rs1, rs2 | x\[rd\] = sext\(x\[rs1\]\[31:0\] &lt;&lt; x\[rs2\]\[4:0\]\) | Shift right arithmetic word. |

## Testing

We're going to test instructions we implemented in this step by calculating [a fibonacci number](https://en.wikipedia.org/wiki/Fibonacci_number) and check if the registers are expected values. I prepared a sample binary file available at [d0iasm/rvemu-for-book/step2/](https://github.com/d0iasm/rvemu-for-book/tree/master/step2). Download the [fib.text](https://github.com/d0iasm/rvemu-for-book/blob/master/step2/fib.text) file and execute it in your emulator.

Calculating a fibonacci number is actually not enough to test all RV64I instructions, so it perhaps be better to use [riscv/riscv-tests](https://github.com/riscv/riscv-tests) to make sure if your implementation is correct. However, it's not obvious how to use riscv-tests so I'll skip to use the test in this book for the sake of simplicity. If you interested in using riscv-tests, [the test file in rvemu](https://github.com/d0iasm/rvemu/blob/master/tests/rv64_user.rs) may be helpful.

```text
// fib.c contains the following C code and fib.text is the build result of it:
// int fib(int n);
// int main() {
//   return fib(10); // Calculate the 10th fibonacci number.
// }
// int fib(int n) {
//   if (n == 0 || n == 1)
//     return n;
//   else
//     return (fib(n-1) + fib(n-2));
// }

$ cargo run fib.text
...           
x12=0x0 x13=0x0 x14=0x1 x15=0x37 // x15 should contain 55 (= 10th fibonacci number).
```

### How to Build Test Binary

If you want to execute a bare-metal C program you write, you need to make an ELF binary without any headers because our emulator just starts to execute at the address `0x0` . The [Makefile](https://github.com/d0iasm/rvemu-for-book/blob/master/step2/Makefile) helps you build a test binary.

```bash
$ riscv64-unknown-elf-gcc -S fib.c
$ riscv64-unknown-elf-gcc -Wl,-Ttext=0x0 -nostdlib -o fib fib.s
$ riscv64-unknown-elf-objcopy -O binary fib fib.text
```



