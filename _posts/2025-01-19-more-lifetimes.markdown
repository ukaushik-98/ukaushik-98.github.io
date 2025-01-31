---
layout: post
title: "More Lifetimes"
date: 2025-01-19 09:00:00 -0800
categories: rust
---

### Owned Types

### What does a generic T actually mean?

### The implications of 'static
T: 'static is some T that can be safely held indefinitely long, including up until the end of the program. T: 'static includes all &'static T however it also includes all owned types, like String, Vec, etc. The owner of some data is guaranteed that data will never get invalidated as long as the owner holds onto it, therefore the owner can safely hold onto the data indefinitely long, including up until the end of the program. T: 'static should be read as "T can live at least as long as a 'static lifetime" not "T has a 'static lifetime". 

### T: 'a vs &'a T

### Eliding Lifetimes
