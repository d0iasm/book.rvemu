# Writing a RISC-V Emulator in Rust

**NOTE: This project is actively ongoing. Pages are not perfect yet and it possible to change dramatically.**

## Introduction

This is the book for writing a 64-bit RISC-V emulator from scratch in Rust. You can run [xv6](https://github.com/mit-pdos/xv6-riscv), a simple Unix-like OS, in your emulator once you finish the book.

You'll learn the basic computer architecture such as ISA, previleged architecture, exceptions, interrupts, peripheral devices, and virtual memory system from making an emulator.

The source code used in this book is available at [d0iasm/rvemu-for-book](https://github.com/d0iasm/rvemu-for-book).

## Chapter 1

[Chapter 1](hardware-components/index.md) shows all hardward components we need to implement for running `xv6`.

1. [CPU with Two Instructions](hardware-components/01-cpu.md)
2. [Memory and System Bus](hardware-components/02-memory.md)
4. [Control and Status Registers](hardware-components/03-csrs.md)
3. [Privileged Architecture](hardware-components/04-privileged-architecture.md)
5. Exceptions
6. PLIC (a platform-level interrupt controller) and CLINT (a core-local interrupter)
7. UART (a universal asynchronous receiver-transmitter)
8. Interrupts
9. Virtio
10. Virtual Memory System

## Chapter 2

[Chapter 2](instruction-set/index.md) shows all ISAs we need to implement for running `xv6`.

- [RV64I Base Integer Instruction Set](instruction-set/01-rv64i.md)
- "M" Standard Extension for Integer Multiplication and Division
- "A" Standard Extension for AtomicInstructions

## Outcome

Once you read this book and implement the emulator, you will be able to run xv6 in your emulator!

![Demo for running xv6 on the emulator](img/2020-08-16-rvemu-for-book-xv6.png)

## Contact

The author is [@d0iasm](https://twitter.com/d0iasm) and please feel free to ask and request anything to me via [Twitter](https://twitter.com/d0iasm) or [GitHub issues](https://github.com/d0iasm/rvemu-for-book/issues)!

