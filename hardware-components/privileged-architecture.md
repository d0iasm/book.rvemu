# Privileged Architecture

This is step 4 of the book [_Writing a RISC-V Emulator from Scratch in 10 Steps_](../), whose goal is running [xv6](https://github.com/mit-pdos/xv6-riscv), a small Unix-like OS, in your emulator in the final step.

The source code is available at [d0iasm/rvemu-for-book/step04/](https://github.com/d0iasm/rvemu-for-book/tree/master/step04).

## Goal of This Page

In the end of this page, we can execute the part of supervisor ISA, `mret` and `sret`. These instructions are used to return from traps in M-mode, S-mode, or U-mode respectively. In addition, we'll add `sfence.vma` but we don't do anything for now.

## Privilege Levels

## Testing

