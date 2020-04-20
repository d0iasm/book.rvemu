# RV64I ISA

This is the step 2 of the book [_Writing a RISC-V Emulator from Scratch in 10 Steps_](./), whose goal is running [xv6](https://github.com/mit-pdos/xv6-riscv), a small Unix-like OS, in your emulator in the final step.

The source code is available at [d0iasm/rvemu-for-book/step2/](https://github.com/d0iasm/rvemu-for-book/tree/master/step2).

## Goal of This Page

In the end of this page, we can execute the sample file that calculates [a fibonacci number](https://en.wikipedia.org/wiki/Fibonacci_number) in our emulator. We will support RV64 ISAs, the base integer instruction set a 64-bit architecture, to calculate a fibonacci number.

Sample binary files are also available at [d0iasm/rvemu-for-book/step2/](https://github.com/d0iasm/rvemu-for-book/tree/master/step2). We successfully see the result of 10th fibonacci number when we execute the sample binary file `fib.text`.

```text
// fib.c contains the following C code and fib.text is the build result of it:
// int fib(int n);
// int main() {
//   return fib(10);
// }
// int fib(int n) {
//   if (n == 0 || n == 1)
//     return n;
//   else
//     return (fib(n-1) + fib(n-2));
// }

$ cargo run fib.text
...           
x12=0x0 x13=0x0 x14=0x1 x15=0x37 // x15 should contain 55 (= 10th fibonacci number).
```

## RV64I Base Integer Instruction Set

RV64I is a base integer instruction set for the 64-bit architecture, which builds upon the RV32I variant. In this book, we're going to implement 40 instructions \(35 instructions in RV32I and 15 instructions in RV64I\). We've already implemented `add` and `addi` so we'll skip them. Also, we'll skip `fence`, `ecall`, and `ebreak` for now.

![Fig 2.1 RV32I Base Instruction Set \(Source: RV32I Base Instruction Set table in Volume I: Unprivileged ISA\)](.gitbook/assets/rvemubook-rv32i.png)

![Fig 2.2 RV64I Base Instruction Set \(Source: RV64I Base Instruction Set table in Volume I: Unprivileged ISA\)](.gitbook/assets/screen-shot-2020-04-19-at-20.20.35.png)

