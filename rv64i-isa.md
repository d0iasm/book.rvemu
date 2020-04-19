# RV64I ISA

This is the step 2 of the book [_Writing a RISC-V Emulator from Scratch in 10 Steps_](./), whose goal is running [xv6](https://github.com/mit-pdos/xv6-riscv), a small Unix-like OS, in your emulator in the final step.

The source code is available at [d0iasm/rvemu-for-book/step2/](https://github.com/d0iasm/rvemu-for-book/tree/master/step2).

## Goal of This Page

In the end of this page, we can execute the sample file that calculates [a fibonacci number](https://en.wikipedia.org/wiki/Fibonacci_number) in our emulator. We will support RV64 ISAs, the base integer instruction set a 64-bit architecture, to calculate a fibonacci number.

Sample binary files are also available at [d0iasm/rvemu-for-book/step2/](https://github.com/d0iasm/rvemu-for-book/tree/master/step2). We successfully see the result of 10th fibonacci number when we execute the sample binary file `fib.text`.

```text
$ cargo run fib.text
...

...
```

