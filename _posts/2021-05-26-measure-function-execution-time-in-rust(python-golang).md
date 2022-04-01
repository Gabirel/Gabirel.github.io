---
layout: post
lang: en
title: Measure function execution time elegantly in Rust(Python, Golang)
categories: rust python golang
tags: rust python golang usage
date: 2021-05-26
---

<p class="intro"><span class="dropcap">M</span>easuring function time is quite necessary sometimes. It is easy to implement this in other languages but not in Rust. Therefore, this post will demonstrate some tricks to measure the function execution time.</p>

From the easiest one to the reusable version. Other implementation will also be included, such as Python and Golang.

-----
Table of Contents

* TOC
{:toc}

-----

# Basic version

In any language, you can complete this task simply by calculating the time differences to measure the exection time.

For example,

```rust
fn main() {
    let start = Instant::now();
    expensive_func();
    let duration = start.elapsed();
    println!("Time elapsed is: {:?}", duration);
}
```

This implementation is easy but not elegant. Not reusable as well.

# Reusable version

## TL;DR: In Rust

Use this:

https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=4eee21bf6ed6e14cc1cfde18b30399d6

```rust
fn main() {
    measure_time(able_to_pass);

    // does not work with parameters
    // measure_time(able_to_pass_with_parameter);

    // but you can pass it using powerful closure
    measure_time(|| {
        able_to_pass_with_parameter("test".to_string());
    });

    measure_time(|| {
        sleep(Duration::new(1, 0));
        println!("works!");
    });
}

fn able_to_pass() {
    println!("works! in function");
}

fn able_to_pass_with_parameter(x: String) {
    println!("works! in function with {}", x);
}

// https://stackoverflow.com/a/25182801/8587335
fn measure_time<F: FnOnce()>(func: F) {
    let start = Instant::now();
    func();
    let duration = start.elapsed();
    println!("Time elapsed is: {:?}", duration);
}
```


We will take a few examples from Python and Golang to demonstrate what we want exactly.

## In Python

Writing python is easy and comfortable although it sacrifices the most important performance.

You can achieve this purpose by the following code:

```python
# more detail: https://stackoverflow.com/a/803626/8587335
def measure_time(func):
    start_time = time.time()
    _ = func()
    end_time = time.time()
    print("Time: {} seconds".format(end_time - start_time))

# function is passwd as a parameter using lambda feature
measure_time(lambda: expensive_func())

# you can reuse the code again without writing time-related codes
measure_time(lambda: second_expensive_func())
```

## In Golang

It is not easy to write similar code like python in Golang. 

But Golang has `defer` to work with. Great news!

This version of code is the most simplest and practical I could find. You may want to use this as well.

```go
// Comes from: https://dev.to/rubiin/measure-function-execution-time-in-golang-177l
func measureTime(start time.Time, name string) {
	elapsed := time.Since(start)
	log.Printf("Time for %s: %s", name, elapsed)
}

func expensive_func() {
  defer measureTime(time.Now(), "name")

  // do your expensive calculation
  // ...
}
```


## In Rust

It is much harder to write similar code like Python and Golang in Rust since passing function as a argument is really hard.

At the same time, you can't easily achieve dynamic dispatch for function argument.

But then I suddenly find this in [here](https://stackoverflow.com/a/25182801/8587335), which enlightens me greatly.

You can find codes in playground: 

https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=4eee21bf6ed6e14cc1cfde18b30399d6

```rust
use std::thread::sleep;
use std::time::{Duration, Instant};

fn main() {
    measure_time(able_to_pass);

    let mut i = 0;
    measure_time(|| {
        i = able_to_pass_with_return();
    });
    
    // result: 1
    println!("i = {}", i);

    // does not work
    // measure_time(able_to_pass_with_parameter);

    // but you can pass it using closure
    measure_time(|| {
        able_to_pass_with_parameter("test".to_string());
    });

    measure_time(|| {
        println!("works!");
        sleep(Duration::new(1, 0));
    });
}

fn able_to_pass() {
    println!("works! in function");
}

fn able_to_pass_with_return() -> i32 {
    println!("works! in function (return)");
    return 1;
}

fn able_to_pass_with_parameter(x: String) {
    println!("works! in function with {}", x);
}

// https://stackoverflow.com/a/25182801/8587335
fn measure_time<F: FnOnce()>(func: F) {
    let start = Instant::now();
    func();
    let duration = start.elapsed();
    println!("Time elapsedpsed in is: {:?}", duration);
}
```

<center><blockquote>The End</blockquote></center>

