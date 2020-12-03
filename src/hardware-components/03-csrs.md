# Control and Status Registers

This is a part of [_Writing a RISC-V Emulator in Rust_](../). Our goal is
running [xv6](https://github.com/mit-pdos/xv6-riscv), a small Unix-like OS, in
your emulator eventually.

The source code used in this page is available at
[d0iasm/rvemu-for-book/03/](https://github.com/d0iasm/rvemu-for-book/tree/master/03).

## The Goal of This Page

In this page, we will implement read-and-modify control and status registers
(CSRs), which are defined at the Zicsr extension. CSRs are registers that store
additional information of the result of instructions.

We will add Zicsr instructions, `csrrw`, `csrrs`, `csrrc`, `csrrwi`, `csrrsi`,
and `csrrci`.

## Control and Status Registers (CSRs)

Control and status register (CSR) is a register that stores various information
in CPU. RISC-V defines a separate address space of 4096 CSRs associated with
each hardware thread so we can have at most 4096 CSRs. RISC-V only allocates a
part of address space so we can add custom CSRs if we want. Also, not all CSRs
are required on all implementations. In this book, I'll describe only CSRs used
in [xv6-riscv](https://github.com/mit-pdos/xv6-riscv).

Fig 3.2-3.4 list the machine-level and supervisor CSRs that are currently
allocated CSR addresses. The next page will talk about what machine-level and
supervisor-level are.

We will support a part of the allocated CSRs. The book describes them in the
following sections.

![Fig 3.2 Machine-level CSRs 1 (Source: Table 2.5: Currently allocated RISC-V
machine-level CSR addresses. in Volume II: Privileged
Architecture)](../img/1-3-2.png)

<p class="caption">Fig 3.2 Machine-level CSRs 1 (Source: Table 2.5: Currently
allocated RISC-V machine-level CSR addresses. in Volume II: Privileged
Architecture)</p>

![Fig 3.3 Machine-level CSRs 2 (Source: Table 2.6: Currently allocated RISC-V
machine-level CSR addresses. in Volume II: Privileged
Architecture)](../img/1-3-3.png)

<p class="caption">Fig 3.3 Machine-level CSRs 2 (Source: Table 2.6: Currently
allocated RISC-V machine-level CSR addresses. in Volume II: Privileged
Architecture)</p>

![Fig 3.4 Supervisor-level CSRs (Source: Table 2.3: Currently allocated RISC-V
supervisor-level CSR addresses. in Volume II: Privileged
Architecture)](../img/1-3-4.png)

<p class="caption">Fig 3.4 Supervisor-level CSRs (Source: Table 2.3: Currently
allocated RISC-V supervisor-level CSR addresses. in Volume II: Privileged
Architecture)</p>

### Status Registers (mstatus/sstatus)

The status registers, `mstatus` as a machine-level CSR and `sstatus` as
a supervisor-level CSR, keep track of and control the CPU's current operating
status.

<p class="filename">cpu.rs</p>

```rust
/// Machine status register.
pub const MSTATUS: usize = 0x300;
/// Supervisor status register.
pub const SSTATUS: usize = 0x100;
```

### Trap-vector Base-address Registers (mtvec/stvecc)

The trap-vector base address registers, `mtvec` as a machine-level CSR and
`stvec` as a supervisor-level CSR, trap vector configuration.

<p class="filename">cpu.rs</p>

```rust
/// Machine trap-handler base address.
pub const MTVEC: usize = 0x305;
/// Supervisor trap handler base address.
pub const STVEC: usize = 0x105;
```

### Machine Trap Delegation Registers (medeleg/mideleg)

The trap delegation registers, `medeleg` for machine-level exception delegation
and `mideleg` for machine-level interrupt delegation, indicate the certain
exceptions and interrupts should be directly by a lower privileged level.

```rust
/// Machine exception delefation register.
pub const MEDELEG: usize = 0x302;
/// Machine interrupt delefation register.
pub const MIDELEG: usize = 0x303;
```

## CSRs in CPU

First, we're going to add `csrs` field to `Cpu` structure. We now have 4 fields
including `regs`, `pc`, and `bus` in CPU.

<p class="filename">cpu.rs</p>

```rust
pub struct Cpu {
    pub regs: [u64; 32],
    pub pc: u64,
    /// Control and status registers. RISC-V ISA sets aside a 12-bit encoding
    /// space (csr[11:0]) for up to 4096 CSRs.
    pub csrs: [u64; 4096],
    pub bus: Bus,
}
```

## Zicsr Standard Extension

Fig 3.1 is the list for instructions to read-modify-write CSRs. RISC-V calls
the 6 instructions **Zicsr standard extension**.

A CSR specifier is encoded in the 12-bit `csr` field of the instruction placed
at 31–20 bits. There are 12 bits for specifying which CSR is selected so that we
have 4096 CSRs (=2\*\*12). The `uimm` field is unsigned immediate value, a 5-bit
zero-extended.

![Fig 3.1 RV64Zicsr Instruction Set (Source: RV32/RV64 Zicsr Standard Extension
table. in Volume I: Unprivileged ISA)](../img/1-3-1.png)

<p class="caption">Fig 3.1 RV64Zicsr Instruction Set (Source: RV32/RV64 Zicsr
Standard Extension table in Volume I: Unprivileged ISA)</p>

