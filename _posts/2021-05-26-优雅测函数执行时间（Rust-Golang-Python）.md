---
layout: post
lang: zh
title: ' 优雅测函数执行时间（Rust, Golang, Python）'
categories: rust
tags: rust python golang usage
date: 2021-05-26
---

<p class="intro"><span class="dropcap">有</span>时非常需要测量函数的执行时间。在其他语言中很容易实现比如像 Python 与 Golang，而在 Rust 实现即相对比较困难。</p>

因此，本文将演示一些衡量函数执行时间的方法与技巧。

本文从最简单的版本到可重用的版本均有，同时也包括了 Python 和 Golang。

-----
Table of Contents

* TOC
{:toc}

-----

# 基础版本

在任何语言中，你都可以简单地通过计算时间差以测量函数的执行时间。

例如，

```rust
fn main() {
    let start = Instant::now();
    expensive_func();
    let duration = start.elapsed();
    println!("Time elapsed is: {:?}", duration);
}
```

这种实现很容易，但并不优雅且不可重用。

# 可重用版本

## 简短版 : Rust

你可以在这里拿到代码：

[https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=4eee21bf6ed6e14cc1cfde18b30399d6](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=4eee21bf6ed6e14cc1cfde18b30399d6)

```rust
fn main() {
    measure_time(able_to_pass);

    // 带参数的无法工作
    // measure_time(able_to_pass_with_parameter);

    // 但是你可以通过闭包来完成这个工作
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

接下来，我们将使用 Python 和 Golang 的代码来简单演示一下我们到底想要什么样的。

## 在 Python 中

写 Python 既轻松又舒适，尽管它牺牲了可能最重要的性能。

对于衡量时间而言，你可以通过以下代码实现：

```python
# 更多的细节：https://stackoverflow.com/a/803626/8587335
def measure_time(func):
    start_time = time.time()
    _ = func()
    end_time = time.time()
    print("Time: {} seconds".format(end_time - start_time))

# 函数用lambda的特性来传递
measure_time(lambda: expensive_func())

# 你可以同样重用代码来避免写与时间相关的代码
measure_time(lambda: second_expensive_func())
```

## 在 Golang 中

在 Golang 中编写类似 python 这样的代码并不容易。

但是 Golang 可以使用 `defer`。 真是一个好消息！

此版本的代码是我能找到的最简单和实用的代码。你可能会需要用到它。

```go
// 来源: https://dev.to/rubiin/measure-function-execution-time-in-golang-177l
func measureTime(start time.Time, name string) {
	elapsed := time.Since(start)
	log.Printf("Time for %s: %s", name, elapsed)
}

func expensive_func() {
  defer measureTime(time.Now(), "name")

  // 开始你的函数计算
  // ...
}
```

## 在 Rust 中

在 Rust 中编写类似 Python 和 Golang 之类的代码要困难得多，因为将函数作为参数传递确实很困难。

同时，你也很难轻松实现函数参数的动态分发，这是一个令人头疼的问题。

但是后来我突然在[这里](https://stackoverflow.com/a/25182801/8587335)中发现了这一点，这对我很有启发。

您可以在 playground 上找到代码：

[https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=4eee21bf6ed6e14cc1cfde18b30399d6](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=4eee21bf6ed6e14cc1cfde18b30399d6)

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

    // 带参数的无法工作
    // measure_time(able_to_pass_with_parameter);

    // 但是你可以通过closure来完成
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