---
layout: post
title: "Exploring Lifetimes in Rust with StrSplit"
date: 2025-01-12 22:33:44 -0800
categories: rust
---

Recently, while I was working on my CityDB personal project, I ran into a few cases where I'd like to improve performance by not cloning anywhere. As such, I started diving deeper into the realm of lifetimes and variance and decided to chart out my learnings here.

[Jon Gjengset][jon-github] did an indepth [stream](https://www.youtube.com/watch?v=rAl-9HwD858) about lifetimes and did a deep dive on StrSplit, a C Library that takes a string and a delimeter and splits the passed in string at the first occurrence. StrSplit also mutates the input in the process, which also turns out to be especially important.

The port of the code that I wrote up while going through his stream can be found in my [rust-playground repo][StrSplit]. The rest of this post will go over the issues that were discussed and how variance played a big role at guranteeing safety.

## What is a lifetime?

### Defining Lifetimes

### Generic Lifetimes

```rust
struct StrSplit<'a> {
    remainder: &'a str,
    delimeter: &'a str
}

impl<'a> StrSplit<'a> {
    ...
}
```

### <'\_> Lifetime

## Variance

I'll be making a future post on variance going over yet another port of a C library but briefly variance consists of the following types:

- Covariance
- CounterVariance
- Invariance

The [Rustonomican](https://doc.rust-lang.org/nomicon/subtyping.html) does a fantastic job going over the definitions and I highly recommend reviewing that.

A brief summary of this can be found here:
![Variance table screenshot](/assets/variance-table.png)

## Multiple Lifetimes!

```rust
struct StrSplit<'a, 'b> {
    remainder: &'a str,
    delimeter: &'b str
}

impl<'a, 'b> StrSplit<'a, 'b> {
    ...
}
```

[jon-github]: https://github.com/jonhoo
[StrSplit]: https://github.com/ukaushik-98/rust-playground/blob/master/src/strSplit/mod.rs
