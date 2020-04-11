# Setup and implement two instructions

This is Day 1 of [Writing a RISC-V Emulator from Scratch in 10 Days](./), which tries to run [xv6](https://github.com/mit-pdos/xv6-riscv) in your emulator in the final day.

## Goal

In the end of this page, we can execute [the sample file](https://github.com/d0iasm/rvemu-simple/blob/master/day1/add-addi.s) containing `add` and `addi` instructions. The `add` instruction adds two 64-bit registers and the `addi` instruction adds a 64-bit register and 12-bit immediate value. 

```text
// add-addi.s
main:
  addi x29, x0, 5
  addi x30, x0, 37
  add x31, x30, x29
```

```text
$ cargo run add-addi.text
...omitted...
x28=               0x0 x29=               0x5 x30=              0x25 x31=              0x2a
```

## Background

RISC-V is an open standard instruction set architecture \(ISA\).

## Build RISC-V toolchain

We'll only support RV64I instruction sets, so we have to build a RISC-V toolchain for it. The default toolchain will use RV64GC, which contains general-purpose ISAs and compressed ISAs. 

Download code from the [riscv/riscv-gnu-toolchain](https://github.com/riscv/riscv-gnu-toolchain) repository and configure it with `rv64g` architecture.

```text
$ ./configure --prefix=<path-to-riscv-toolchain> --with-arch=rv64g
```

## Create a new project

We'll use Rust and Cargo. To install them, see [the installation page](https://doc.rust-lang.org/cargo/getting-started/installation.html) in Cargo book. I'll call our project `rvemu-simple` because I originally implemented [rvemu](https://github.com/d0iasm/rvemu) and I made a simple version of it for this book.

```text
$ cargo new rvemu-simple
$ cargo run
Hello, world!
```

## Read a file name from a command line

First, you need to read a binary file from a command line.

