# Setup and implement two instructions

This is the step 1 of the book [_Writing a RISC-V Emulator from Scratch in 10 Steps_](./), which tries to run [xv6](https://github.com/mit-pdos/xv6-riscv) in your emulator in the final day.

The source code is available at [d0iasm/rvemu-for-book/day1/](https://github.com/d0iasm/rvemu-for-book/tree/master/day1).

## Goal of This Page

In the end of this page, we can execute [the sample file](https://github.com/d0iasm/rvemu-for-book/blob/master/day1/add-addi.s) containing `add` and `addi` instructions. The `add` instruction adds two 64-bit registers, and the `addi` instruction adds a 64-bit register and 12-bit immediate value.

Sample binary files are also available at [d0iasm/rvemu-for-book/day1/](https://github.com/d0iasm/rvemu-for-book/tree/master/day1).

```bash
$ cargo run add-addi.text
...omitted...
x28= 0x0  x29= 0x5  x30= 0x25  x31= 0x2a
```

## Background

[RISC-V](https://riscv.org/) is a new instruction-set architecture \(ISA\) that was originally designed to support computer architecture research and education at the University of California, Berkeley, but now it gradually becomes a standard free and open architecture for industry implementations. RISC-V is also excellent **for students to learn computer architecture** since it's simple enough. We can see [the RISC-V specifications](https://riscv.org/specifications/) for free and we'll implement a part of features in _Unprivileged Specification_ and _Privileged ISA Specification_. The _Unprivileged Specification_ defines CPU instructions, 

[Rust](https://www.rust-lang.org/) is an open-source systems programming language that focuses on performance and safety. It is popular especially in systems programming. We're going to implement our emulator in Rust.

We'll implement a RISC-V emulator in Rust in this book. In simple words, we're going to write **an infinite loop to execute binaries of RISC-V step by step**. An instruction is executed at an one step in the loop. This book tries to understand the basic RISC-V architecture by making a RISC-V emulator.

## Build RISC-V toolchain

We'll only support RV64I instruction sets, so we have to build a RISC-V toolchain for it. The default toolchain will use RV64GC which contains general-purpose ISAs and compressed ISAs. 

Download code from the [riscv/riscv-gnu-toolchain](https://github.com/riscv/riscv-gnu-toolchain) repository and configure it with `rv64g` architecture. After the following commands, we can use `riscv64-unknown-elf-` commands.

```bash
$ git clone --recursive https://github.com/riscv/riscv-gnu-toolchain
$ cd riscv-gnu-toolchain
$ ./configure --prefix=<path-to-riscv-toolchain> --with-arch=rv64g
$ make
$ make linux
```

## Create a new project

We'll use Cargo, the Rust package manager. See [the installation page](https://doc.rust-lang.org/cargo/getting-started/installation.html) in Cargo book to install it. I'll call our project `rvemu-simple` because I originally implemented [rvemu](https://github.com/d0iasm/rvemu) and I made [a simple version of it](https://github.com/d0iasm/rvemu-simple) for this book. We can see "Hello, world!" when we execute an initialized project.

```bash
$ cargo new rvemu-simple
$ cargo run
Hello, world!
```

## Read a file name from a command line

First, we need to read the name of a binary file from a command line. We can get command line arguments via the standard `env` module. Let a file name place at the first argument.

{% code title="src/main.rs" %}
```rust
use std::env;

fn main() -> io::Result<()> {
    let args: Vec<String> = env::args().collect();

    if args.len() != 2 {
        panic!("Usage: rvemu-simple <filename>");
    }
    let mut file = File::open(&args[1])?;
    let mut buffer = Vec::new();
    file.read_to_end(&mut buffer)?;
}
```
{% endcode %}

## Create a basic CPU

A computer composed of 

{% code title="src/main.rs" %}
```rust
struct Cpu {
    xregs: [u64; 32],
    pc: u64,
    memory: Vec<u8>,
}
```
{% endcode %}





