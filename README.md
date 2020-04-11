# Writing a RISC-V Emulator from Scratch in 10 Days

## Introduction

This is the book for writing a RISC-V emulator from scratch in Rust. It shows us how to implement an emulator step by step in 10 days. You can run [xv6](https://github.com/mit-pdos/xv6-riscv), a simple Unix-like OS, in your emulator in the final day.

You'll learn the basic computer architecture from making an emulator in Rust:

* Basic RISC-V architecture
* How to write code in Rust language
* Privilege levels
* Exceptions
* Interrupts
* Peripheral devices

The source code is available at [d0iasm/rvemu-simple](https://github.com/d0iasm/rvemu-simple).

| Day | Content |
| :--- | :--- |
| Day1 | [Setup and implement two instructions](setup-and-implement-one-instruction.md) |
| Day2 | RV64I ISAs |
| Day3 | Supervisor ISAs |
| Day4 | A part of CSRs |
| Day5 | Exceptions |
| Day6 | UART \(a universal asynchronous receiver-transmitter\) |
| Day7 | PLIC \(a platform-level interrupt controller\) |
| Day8 | Interrupts |
| Day9 | CLINT \(a core-local interruptor\) |
| Day10 | Virtio |
|  | Congratulations🎉 Now you can run xv6 in your emulator. |

Author is [@d0iasm](https://twitter.com/d0iasm) and please feel free to ask and request anything to me via [Twitter](https://twitter.com/d0iasm) or [GitHub issues](https://github.com/d0iasm/rvemu-simple/issues)!

