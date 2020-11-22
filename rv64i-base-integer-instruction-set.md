# RV64I Base Integer Instruction Set

## Goal of This Page

In the end of this page, we can execute [the sample file](https://github.com/d0iasm/rvemu-for-book/blob/master/step02/fib.c) that calculates [a Fibonacci number](https://en.wikipedia.org/wiki/Fibonacci_number) in our emulator. We will support RV64 ISAs, the base integer instruction set a 64-bit architecture, to calculate a Fibonacci number.

Sample binary files are also available at [d0iasm/rvemu-for-book/step02/](https://github.com/d0iasm/rvemu-for-book/tree/master/step02). We successfully see the result of the 10th Fibonacci number when we execute the sample binary file `fib.bin`.

```text
// fib.c contains the following C code and fib.bin is the build result of it:
// int fib(int n);
// int main() {
//   return fib(10); // Calculate the 10th fibonacci number.
// }
// int fib(int n) {
//   if (n == 0 || n == 1)
//     return n;
//   else
//     return (fib(n-1) + fib(n-2));
// }

$ cargo run fib.bin
...           
x12=0x0 x13=0x0 x14=0x1 x15=0x37 // x15 should contain 55 (= 10th fibonacci number).
```

## RV64I: Base Integer Instruction Set

RV64I is a base integer instruction set for the 64-bit architecture, which builds upon the RV32I variant. RV64I shares most of the instructions with RV32I but the width of registers is different and there are a few additional instructions only in RV64I.

In this step, we're going to implement 47 instructions \(35 instructions from RV32I and 12 instructions from RV64I\). We've already implemented `add` and `addi` so we'll skip them. Also, we'll skip implementing `fence`, `ecall`, and `ebreak` for now. I'll cover `ecall` and `ebreak` in the following step and won't explain `fence`. The `fence` instruction is a type of barrier instruction to apply an ordering constraint on memory operations issued before and after it. We don't need it since our emulator is a single core system and doesn't reorder memory operations \(out-of-order execution\).

Fig 2.1 and Fig 2.2 are the lists for RV32I and RV64I, respectively. We're going to implement all instructions in the figures.

![Fig 2.1 RV32I Base Instruction Set \(Source: RV32I Base Instruction Set table in Volume I: Unprivileged ISA\)](.gitbook/assets/rvemubook-rv32i.png)

![Fig 2.2 RV64I Base Instruction Set \(Source: RV64I Base Instruction Set table in Volume I: Unprivileged ISA\)](.gitbook/assets/screen-shot-2020-04-19-at-20.20.35.png)

## 

