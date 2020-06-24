# Control and Status Registers

This is the step 3 of the book [_Writing a RISC-V Emulator from Scratch in 10 Steps_](./), whose goal is running [xv6](https://github.com/mit-pdos/xv6-riscv), a small Unix-like OS, in your emulator in the final step.

The source code is available at [d0iasm/rvemu-for-book/step3/](https://github.com/d0iasm/rvemu-for-book/tree/master/step3).

## Goal of This Page

In the end of this page, we can execute the sample file containing CSR instructions, `csrrw`, `csrrs`, `csrrc`, `csrrwi`, `csrrsi`, and `csrrci`.

## Control and Status Registers \(CSRs\)

Control and status register \(CSR\) is a register that stores various information in CPU. RISC-V defines a separate address space of 4096 CSRs associated with each hardware thread.

## CSR Instructions

![Fig 3.4 RV64Zicsr Instruction Set \(Source: RV32/RV64 Zicsr Standard Extension table in Volume I: Unprivileged ISA\)](.gitbook/assets/rvemubook-csr-instructions.png)

## CSR List

Fig 3.1-3.3 list the CSRs that are currently allocated CSR addresses.

![Fig 3.1 Machine-level CSRs 1 \(Source: Table 2.4: Currently allocated RISC-V machine-level CSR addresses. in Volume II: Privileged Architecture](.gitbook/assets/rvemubook-machine-csr-list.png)

![Fig 3.2 Machine-level CSRs 2 \(Source: Table 2.5: Currently allocated RISC-V machine-level CSR addresses. in Volume II: Privileged Architecture](.gitbook/assets/rvemu-machine-csr-list-2.png)

![Fig 3.3 Supervisor-level CSRs \(Source: Table 2.3: Currently allocated RISC-V supervisor-level CSR addresses. in Volume II: Privileged Architecture\)](.gitbook/assets/rvemubook-supervisor-csr-list.png)

## Testing

