# Interrupts

This is step 8 of the book [_Writing a RISC-V Emulator from Scratch in 10 Steps_](./), whose goal is running [xv6](https://github.com/mit-pdos/xv6-riscv), a small Unix-like OS, in your emulator in the final step.

The source code is available at [d0iasm/rvemu-for-book/step08/](https://github.com/d0iasm/rvemu-for-book/tree/master/step08).

## Goal of This Page

In the end of this page, we support interrupts, external asynchronous events that may cause a hardware thread to experience an unexpected transfer of control.

