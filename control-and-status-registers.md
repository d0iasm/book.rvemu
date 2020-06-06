# Control and Status Registers

This is the step 3 of the book [_Writing a RISC-V Emulator from Scratch in 10 Steps_](./), whose goal is running [xv6](https://github.com/mit-pdos/xv6-riscv), a small Unix-like OS, in your emulator in the final step.

The source code is available at [d0iasm/rvemu-for-book/step3/](https://github.com/d0iasm/rvemu-for-book/tree/master/step3).

## Goal of This Page

In the end of this page, we can execute the sample file containing CSR instructions, `csrrw`, `csrrs`, `csrrc`, `csrrwi`, `csrrsi`, and `csrrci`.

## Control and Status Registers \(CSRs\)

Control and status register \(CSR\) is a register that stores various information in CPU.

## CSR Instructions

![Fig 3.1\(Source: RV32/RV64 Zicsr Standard Extension table in Volume I: Unprivileged ISA\)](.gitbook/assets/rvemubook-csr-instructions.png)

## Testing

