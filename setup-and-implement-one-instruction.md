# Setup and implement one instruction

This is Day 1 of Writing a RISC-V Emulator from Scratch in 10 Days, which tried to run [xv6](https://github.com/mit-pdos/xv6-riscv) in your emulator. Xv6 is a simple Unix-like OS for education.

In this page, we're going to set up environment for Rust and implement basic CPU to execute one instruction in RISC-V.

## Background

RISC-V is an open standard instruction set architecture \(ISA\) 

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

