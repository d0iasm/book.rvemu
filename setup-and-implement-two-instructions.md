# Setup and Implement Two Instructions

This is the step 1 of the book [_Writing a RISC-V Emulator from Scratch in 10 Steps_](./), which goal is running [xv6](https://github.com/mit-pdos/xv6-riscv), a small Unix-like OS, in your emulator in the final step.

The source code is available at [d0iasm/rvemu-for-book/day1/](https://github.com/d0iasm/rvemu-for-book/tree/master/day1).

## Goal of This Page

In the end of this page, we can execute [the sample file](https://github.com/d0iasm/rvemu-for-book/blob/master/day1/add-addi.s) containing `add` and `addi` instructions in our emulator. The `add` instruction adds two 64-bit registers, and the `addi` instruction adds a 64-bit register and 12-bit immediate value.

Sample binary files are also available at [d0iasm/rvemu-for-book/day1/](https://github.com/d0iasm/rvemu-for-book/tree/master/day1). We successfully see the result of addition in the `x31` register when we execute the sample binary file `add-addi.text`.

```bash
// add-addi.text is binary to execute these instructions:
// main:
// .  addi x29, x29, 5
// .  addi x30, x30, 37
// .  add x31, x30, x29 // x31 should contain 42 (0x2a)

$ cargo run add-addi.text
...omitted...
x28=0x0 x29=0x5 x30=0x25 x31=0x2a
```

## Background

[RISC-V](https://riscv.org/) is a new instruction-set architecture \(ISA\) that was originally designed to support computer architecture research and education at the University of California, Berkeley, but now it gradually becomes a standard free and open architecture for industry implementations. RISC-V is also excellent **for students to learn computer architecture** since it's simple enough. We can read [the RISC-V specifications](https://riscv.org/specifications/) for free and we'll implement a part of features in _Unprivileged Specification_ and _Privileged ISA Specification_. The _Unprivileged Specification_ defines instructions, the binaries that the computer processor \(CPU\) can understand.

RISC-V defines 32-bit and 64-bit architecture. The width of registers and the available memory size are different depending on the architecture. The 128-bit architecture also exists but it is currently in a draft state. RISC-V instructions consists of base integer instructions and optional extensions. The base integer instructions must be present in any implementation.

[Rust](https://www.rust-lang.org/) is an open-source systems programming language that focuses on performance and safety. It is popular especially in systems programming like an operating system. We're going to implement our emulator in Rust.

We'll only support 64-bit base integer instructions, which is called `RV64I`, and optional extensions xv6 uses in this book. In simple words, we're going to write **an infinite loop to execute RISC-V binaries one by one**. An instruction is executed at a step in the loop. This book tries to understand the basic RISC-V architecture by making a RISC-V emulator.

## Build RISC-V Toolchain

First, we have to build a RISC-V toolchain for `RV64G`. The default toolchain will use `RV64GC` which contains general-purpose ISAs and compressed ISAs. General-purpose ISAs are the alias of selected standard extensions \(`RV64IMAFD`, `Zicsr`, `Zifencei`\), which includes base integer instructions \(`RV64I`\), integer multiplication and division instructions \(`RV64M`\), atomic instructions \(`RV64A`\), single-precision floating-point instructions \(`RV64F`\), double-precision floating-point instructions \(`RV64D`\), control and status register instructions \(`RVZicsr`\), and instruction-fetch fence instructions \(`RVZifencei`\). This book will explain only instructions that xv6 uses.

Download code from the [riscv/riscv-gnu-toolchain](https://github.com/riscv/riscv-gnu-toolchain) repository and configure it with `rv64g` architecture. After the following commands, we can use `riscv64-unknown-elf-*` commands.

```bash
$ git clone --recursive https://github.com/riscv/riscv-gnu-toolchain
$ cd riscv-gnu-toolchain
$ ./configure --prefix=<path-to-riscv-toolchain> --with-arch=rv64g
$ make && make linux
```

## Create a New Project

We'll use Cargo, the Rust package manager. See [the installation page](https://doc.rust-lang.org/cargo/getting-started/installation.html) in Cargo book to install it. I'll call our project `rvemu-for-book` because I originally implemented [rvemu](https://github.com/d0iasm/rvemu) and I made [the reference code](https://github.com/d0iasm/rvemu-for-book) for this book. We can see "Hello, world!" when we execute an initialized project by the `cargo run` command.

```bash
$ cargo new rvemu-for-book
$ cd rvemu-for-book
$ cargo run
Hello, world!
```

## Create a Basic CPU

CPU is the most important part of a computer to execute instructions. It has registers, a small amount of fast storage CPU can access. The width of registers is 64 bits in 64-bit RISC-V architecture. It also has a program counter to hold the address of the current instruction.

The following struct contains 32 registers, a program counter, and memory. An actual hardware doesn't have a memory inside CPU and a memory connects to CPU via a system bus, but I decided to implement that CPU has a memory in our emulator for the sake of simplicity.

{% code title="src/main.rs" %}
```rust
struct Cpu {
    regs: [u64; 32],
    pc: u64,
    memory: Vec<u8>,
}
```
{% endcode %}

### Fetch-decode-execute Cycle

The main job of the CPU is composed of three main stages: the fetch stage, the decode stage, and the execute stage. The fetch-decode-execute cycle is also known as the instruction cycle. CPU follows the cycle from the computer boots up until it shuts down.

1. Fetch: Fetches the next instruction to be executed from the memory where the program is stored.
2. Decode: Splits an instruction sequence into a form that makes sense to the CPU.
3. Execute: Performs the action required by the instruction.

We'll make `fetch` and `execute` methods in CPU. The decode stage is performed in the execute method for the sake of simplicity. In a real hardware, arithmetic operations such as addition and subtraction are performed by [ALU ](https://en.wikipedia.org/wiki/Arithmetic_logic_unit)\(Arithmetic logic unit\), but in the emulator I decided to implement it in CPU.

```rust
impl Cpu {
    fn fetch(&self) -> u32 {
        // Read 32-bit instruction from a memory.
    }
    
    fn execute(&mut self, inst: u32) {
        // Decode an instruction and execute it.
    }
}
```

### Set Binaries to the Memory

In order to implement the `fetch` method, we need to read a binary file from a command line. We can get command line arguments via the standard `env` module. Let a file name place at the first argument.

The binary is set up to the memory when a new CPU instance is created.

{% code title="src/main.rs" %}
```rust
use std::env;

fn main() -> io::Result<()> {
    let args: Vec<String> = env::args().collect();

    if args.len() != 2 {
        panic!("Usage: rvemu-simple <filename>");
    }
    let mut file = File::open(&args[1])?;
    let mut binary = Vec::new();
    file.read_to_end(&mut binary)?;
    
    let cpu = Cpu::new(binary);
    
    ...
}

impl Cpu {
    fn new(binary: Vec<u8>) -> Self {
        Self {
            regs: [0; 32],
            pc: 0,
            memory: binary,
        }
    }
    
    fn fetch(&self) -> u32 { ... }
    fn execute(&mut self, inst: u32) { ... }
}
```
{% endcode %}

### Fetch Stage

Now, we are ready to fetch an instruction from the memory. 

What we should be careful to fetch an instruction is endianness, which is the term refers to how binary data is stored. There are 2 types of endianness: little-endian and big-endian. A little-endian ordering places the least significant byte \(LSB\) at the lowest address and the most significant byte \(MSB\) places at the highest address in a 32-bit word. While a bit-endian ordering does the opposite.

![Fig 1. Little-endian and big-endian 2 instructions](.gitbook/assets/risc-v_-endianness-2.png)

RISC-V has either little-endian or big-endian memory systems, but our emulator will implement a little-endian system since little-endian systems are currently dominant commercially like x86 systems.

Our memory is the vector of `u8` , so read 4 elements from the memory and shift them in the little-endian order.

{% code title="src/main.rs" %}
```rust
impl Cpu {
    ...  
    fn fetch(&self) -> u32 {
        let index = self.pc as usize;
        return (self.memory[index] as u32)
            | ((self.memory[index + 1] as u32) << 8)
            | ((self.memory[index + 2] as u32) << 16)
            | ((self.memory[index + 3] as u32) << 24);
    }
    ...
}
```
{% endcode %}

### Decode State

The way to interpret the instruction is defined in _Volume I: Unprivileged ISA at riscv.org/specifications_.

### Execute State

In this page, we'll only implement 2 instructions `add` and `addi`.

## Testing







