# Writing a RISC-V Emulator in Rust

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![GitHub Actions status](https://github.com/d0iasm/book.rvemu/workflows/book.rvemu/badge.svg)](https://github.com/d0iasm/book.rvemu/actions)

[https://book.rvemu.app/](https://book.rvemu.app/)

This is the book for writing a 64-bit RISC-V emulator from scratch in Rust. The book shows you how to implement an emulator step by step. You can run [xv6](https://github.com/mit-pdos/xv6-riscv), a simple Unix-like OS, in your emulator.

## Tool

This book is written by [mdBook](https://github.com/rust-lang/mdBook).

Converts markdown files under `src/` directory to html files under `book` directory.
```
$ mdbook build
```

Watchs markdown files to rebuild on every change.
```
$ mdbook watch
```

Serves the book at `http://localhost:3000`.
```
$ mdbook serve
```

## Deploy

GitHub Actions with [peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages) deploys the page to GitHub pages.
