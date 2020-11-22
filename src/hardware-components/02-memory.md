# Memory and System Bus

This is a part of the [Writing a RISC-V Emulator in Rust](../). Our goal is
running [xv6](https://github.com/mit-pdos/xv6-riscv), a small Unix-like OS, in
your emulator in the final step.

The source code used in this page is available at
[d0iasm/rvemu-for-book/02/](https://github.com/d0iasm/rvemu-for-book/tree/master/02).

## Goal of This Page

In this page, we will implement a memory and a system bus. The memory is used to
store and load data. The system bus is a pathway to carry data between the CPU
and the memory.

These components enable to execute load and store instructions which are the
part of base integer instruction set. There are 7 load instructions (`lb`, `lh`,
`lw`, `ld`, `lbu`, `lhu`, and `lwu`) and 3 store instructions (`sb`, `sh`, `sw`,
and `sd`).

## Define Modules

Rust has a powerful module system that can split code into logical units. Each
unit is called a module.

First, we divide `main.rs` implemented in the previous section. The code of the
CPU is splited to a new file `cpu.rs`.

To define a CPU module, we need to `mod` keyword at the beginning of the
`main.rs` file. The `use` keyword allows us to use public items in the CPU
module.

<p class="filename">main.rs</p>

```rust
// This declaration will look for a file named `cpu.rs` or `cpu/mod.rs` and
// will insert its contents inside a module named `cpu` under this scope.
mod cpu;

// Use all public structures, methods, and functions defined in the cpu module.
use crate::cpu::*;
```

<p class="filename">cpu.rs</p>

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

## Memory

The memory we going to implement is a dynamic random-access memory which is
called DRAM. It is used to store/load data while the program is running.

We represent it as a `Memory` struct. It contains a vector of bytes as a member.

The vector of bytes in the `Memory` instance is initialized with the data
containing executable binary when it's created by `Memory::new()`.

<p class="filename">memory.rs</p>

```rust
pub const MEMORY_SIZE: u64 = 1024 * 1024 * 128; // 128MiB

pub struct Memory {
    pub memory: Vec<u8>,
}

impl Memory {
    pub fn new(binary: Vec<u8>) -> Memory {
        let mut memory = vec![0; MEMORY_SIZE as usize];
        memory.splice(..binary.len(), binary.iter().cloned());

        Self { memory }
    }
}
```

### Load and Store Methods

There are `load` and `store` public methods for the `Memory` struct. Arguments in each method are an address and the number of bytes. The number of bytes can be 8, 16, 32, and 64 bytes.

<p class="filename">memory.rs</p>

```rust
impl Memory {
    ...
    pub fn load(&self, addr: u64, size: u64) -> Result<u64, ()> {
        match size {
            8 => Ok(self.load8(addr)),
            16 => Ok(self.load16(addr)),
            32 => Ok(self.load32(addr)),
            64 => Ok(self.load64(addr)),
            _ => Err(()),
        }
    }

    pub fn store(&mut self, addr: u64, size: u64, value: u64) -> Result<(), ()> {
        match size {
            8 => Ok(self.store8(addr, value)),
            16 => Ok(self.store16(addr, value)),
            32 => Ok(self.store32(addr, value)),
            64 => Ok(self.store64(addr, value)),
            _ => Err(()),
        }
    }
    ...
}
```

`load8`, `load16`, `load32`, and `load64` (`store*` as well) are private methods to help us operate the memory with the specific size of bytes. The memory is a little-endian system as described in the previous section so we need to be careful the order of bytes.

The following code is `load32` and `store32`. The byte of a smallest memory address (`index`) is stored at the least signigicant byte at the largest and the byte of a largest memory address (`index + 3`) is stored at the most significant byte of a word.

<p class="filename">memory.rs</p>

```rust
impl Memory {
    ...

    fn load32(&self, addr: u64) -> u64 {
        let index = (addr - MEMORY_BASE) as usize;
        return (self.memory[index] as u64)
            | ((self.memory[index + 1] as u64) << 8)
            | ((self.memory[index + 2] as u64) << 16)
            | ((self.memory[index + 3] as u64) << 24);
    }

    fn store32(&mut self, addr: u64, value: u64) {
        let index = (addr - MEMORY_BASE) as usize;
        self.memory[index] = (value & 0xff) as u8;
        self.memory[index + 1] = ((value >> 8) & 0xff) as u8;
        self.memory[index + 2] = ((value >> 16) & 0xff) as u8;
        self.memory[index + 3] = ((value >> 24) & 0xff) as u8;
    }
    ...
}
```

### Add Memory to Module

Let's add the `Memory` as a module. Once adding one line at `main.rs`, `memory.rs` can be included to the build target by `cargo build` and we can use methods defined at `memory.rs`.

<p class="filename">main.rs</p>

```rust
mod memory;
```

From now on, when a new file is added, we will add it as a module implicitly.

## System Bus

System bus is a component to carry data between the CPU and the memory.

<p class="filename">bus.rs</p>

```rust
pub struct Bus {
    memory: Memory,
}
```


## Update the CPU

### Fetch-decode-execute Cycle

The previous step already mentioned the fetch-decode-execute cycle and we're going to implement it in the `main.rs`. An emulator is ideally an infinite loop and executes the program infinitely unless something wrong happens or a user stops an emulator explicitly. However, we're going to stop an emulator implicitly when the program counter is 0 or over the length of memory, and an error happens during the execution.

<p class="filename">main.rs</p>

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

### Fetch Stage

The fetch stage is basically the same as the previous step, but I prefer to create a new function to read 32-bit data from a memory because there are other instructions to read and write memory.

<p class="filename">cpu.rs</p>

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

### Decode Stage

The decode stage is almost the same as the previous step too and we'll add 2 new fields `funct3` and `funct7`. `funct3` is located from 12 to 14 bits and `funct7` is located from 25 to 31 bits as we can see in Fig 2.1 and 2.2. These fields and opcode select the type of operation.

<p class="filename">cpu.rs</p>

```rust
impl Cpu {
    ... 
    fn execute(&mut self, inst: u32) {
        ...
        let funct3 = (inst & 0x00007000) >> 12;
        let funct7 = (inst & 0xfe000000) >> 25;
        ...
```

In RISC-V, there are many common positions in all formats, but decoding an immediate value is quite different depending on instructions, so we'll decode an immediate value in each operation.

For example, the immediate value in branch instructions is located in the place of `rd` and `funct7`. A branch instruction is a `if` statement in C to change the sequence of instruction execution depending on a condition, which includes `beq`, `bne`, `blt`, `bge`, `bltu`, and `bgeu`.

Decoding is performed by bitwise ANDs and bit shifts. The point to be noted is that an immediate value should be sign-extended. It means we need to fill in the upper bits with 1 when the significant bit is 1. In this implementation, filling in bits with 1 is performed by casting from a signed integer to an unsigned integer.

The way how to decode each instruction is listed in Fig 2.1 and Fig 2.2.

<p class="filename">cpu.rs</p>

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

### Execute Stage

Each operation is performed in each `match` arm. For example, a branch instruction `beq`, which is one of the branch instructions, is executed when `opcode` is 0x63 and `funct3` is 0x0. `beq` sets the `pc` to the current `pc` plus the signed-extended offset if a value in `rs1` equals a value in `rs2`. The current `pc` means the position when CPU fetched an instruction from memory so we need to subtract 4 from `pc` because we added 4 after fetch.

<p class="filename">cpu.rs</p>

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
