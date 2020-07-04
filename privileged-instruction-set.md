# Privileged Instruction Set \(+ Atomic instructions\)

This is step 4 of the book [_Writing a RISC-V Emulator from Scratch in 10 Steps_](./), whose goal is running [xv6](https://github.com/mit-pdos/xv6-riscv), a small Unix-like OS, in your emulator in the final step.

The source code is available at [d0iasm/rvemu-for-book/step4/](https://github.com/d0iasm/rvemu-for-book/tree/master/step4).

## Goal of This Page

In the end of this page, we can execute the part of supervisor ISA, `mret`, and `sret`. These instructions are used to return from traps in M-mode, S-mode, or U-mode respectively.

We also support the part of "A" standard extension for atomic instructions \(RV64A\), `amoadd.w`, `amoadd.d`, `amoswap.w` and `amoswap.d`, since xv6 uses them.

## Privilege Levels

## Testing

