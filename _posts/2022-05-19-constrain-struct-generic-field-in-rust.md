---
layout: post
lang: en
title: Constrain Struct Generic Field in Rust
categories: rust generic
tags: rust generic trait
date: 2022-05-19
---

<p class="intro"><span class="dropcap">I</span>t is a quite common need to constrain struct generic field to certain types in Rust. We usually have two different ways to implement this. Trait or dispatching each of them by ourselves.</p>

 In this post, I am going to show you how to implement this. Therefore, this post should be meant for the "beginners". This post is gonna show you from the basic to the final working version.

It is recommended to read the most fundamental basics from [this article][ref].
First part of codes are based on it.

-----
Table of Contents

* TOC
{:toc}

-----

## Demands

We need a struct which has some fields with generics,
but we need some restrictions as well.

Let's call it by `Container`, which contains something.
But we don't know what type it is, so generics are necessary.

First, in the form of code, we define struct like this:

```rust
struct Container<T> {
    field: T,
}
```

Second, we would like to constrain types to certain types,
like integer, float or custom structs.

Let's name a custom struct, call `Color`.

Therefore, for exmaple, `T` can only be `i32`, `f64` and `Color`.
**Nothing else.**

Rust compiler now knows the specific forms of `Container`,
which will [expand codes into detailed ones](https://doc.rust-lang.org/book/ch10-01-syntax.html).

## Two ways to constrain

### Dispatching

#### Original

From [this article][ref], you get these codes:

```rust
struct Color {
    red: u8,
    green: u8,
    blue: u8,
}

struct Container<T> {
    field: T,
}

impl Container<Color> {
    fn foo(&self) {
        println!("I exist");
    }
}

fn main() {
    let inst1 = Container {
        field: Color { red: 10, green: 20, blue: 30 },
    };
    let inst2 = Container { field: 10 };
    let inst3 = Container { field: 12.2 };

    inst1.foo(); // This will work
    inst2.foo(); // Won't work - method doesn't exist
    inst3.foo(); // Won't work - method doesn't exist
}
```

If you know some very basic knowledge about generics,
you should know what it is doing.

Anyway, short explanation: Only `Color` container works
because it has its own implementation for `Color` container only.
Integer type or float type won't work due to empty of implementation.

So, it gets what we want. Some constraints on generic types.

#### Modification

Let's modify codes to make it more elaborate for future purpose.

Print some details to make it look like workable and provable.

```rust
#[derive(Debug)]
struct Color {
    red: u8,
    green: u8,
    blue: u8,
}

struct Container<T> {
    field: T,
}

impl Container<Color> {
    fn print(&self) {
        println!("{:?}", self.field);
    }
}

impl Container<i32> {
    fn print(&self) {
        println!("{:?}", self.field);
    }
}

impl Container<f64> {
    fn print(&self) {
        println!("{:?}", self.field);
    }
}

pub fn fun() {
    let inst1 = Container {
        field: Color {
            red: 10,
            green: 20,
            blue: 30,
        },
    };
    let inst2 = Container { field: 10 };
    let inst3 = Container { field: 12.2 };

    inst1.print(); // This will work
    inst2.print(); // This will work
    inst3.print(); // This will work
}
```

As you can see, we need to dispatch
three types manually in order to make this work.
Three times duplication as well.

But it will work anyway. Its output:

```shell
Color { red: 10, green: 20, blue: 30 }
10
12.2
```

**Try to comment out implementation for `f64`. See what it happens.**

```shell
error[E0599]: no method named `print` found for struct `Container<{float}>` in the current scope
  --> src/fun_1_basic.rs:43:11
   |
8  | struct Container<T> {
   | ------------------- method `print` not found for this
...
43 |     inst3.print(); // This will work
   |           ^^^^^ method not found in `Container<{float}>`
   |
   = note: the method was found for
           - `Container<Color>`
           - `Container<i32>`

For more information about this error, try `rustc --explain E0599`.
```

Rust compiler complains about `Container<{float}>` because we do not implement them.
Therefore, we are only allow to use `Color` and `i32` type.

#### One More Step: Remove Deuplication

The problem is that we still needs to duplicate exact codes for 3 times.
We can remove that duplication by just implementing for all types.
(Yes, we do not consider restrictions for now. Deal with it later.)

Check codes:

```rust
use std::fmt::Debug;

#[derive(Debug)]
struct Color {
    red: u8,
    green: u8,
    blue: u8,
}

struct Container<T> {
    field: T,
}

// `Debug` to set bounds for generics
impl<T: Debug> Container<T> {
    fn print(&self) {
        println!("{:?}", self.field);
    }
}

pub fn fun() {
    let inst1 = Container {
        field: Color {
            red: 10,
            green: 20,
            blue: 30,
        },
    };
    let inst2 = Container { field: 10 };
    let inst3 = Container { field: 12.2 };

    inst1.print(); // This will work
    inst2.print(); // This will work
    inst3.print(); // This will work
}
```

We just implement for all types by using `impl<T: Debug> Container<T> { }`.
Therefore, rust compiler will know expand codes automatically by using `impl<T>`.

### Trait & Bounds

However, in previous section we cannot set constraints on generic type
just for convenience.

There is a solution for this. **Traits and bounds.**

[The Book](https://doc.rust-lang.org/book/ch10-02-traits.html) says
traits can be used to define shared behaviors in an abstract way.
The examples from [The Book](https://doc.rust-lang.org/book) can
demonstrate how it works precisely.
(`NewsArticle` and `Tweet` can share the same behavior,
which is `summary` from `Summary` trait.)

However, it can also be used for dispatching,
which is already a common sense for veterans. In the following secions,
you may learn it from some demos.

1. **Make constraints**

   ```rust
   struct Container<T: ContainerTrait> {
       field: T,
   }
   ```

   All types must follow constraints, which is called `ContainerTrait`.

2. **Define Trait/Boundary**

   ```rust
   // you can alow use empty trait only
   // `foo` is used to show some details
   trait ContainerTrait {
       fn foo(&self);
   }
   ```

3. **Define what boundary it is**

   Explanation: Traits act like the void, which is nothing there.
Rust compiler would never know when it is the void, right?
So, we need to add some actual stuff in that "void" trait for the compiler.

   Let's add then.

   ```rust
   impl ContainerTrait for Color {
       fn foo(&self) {
           println!("Color -> {:?}", self);
           println!("\tColor.red -> {}", self.red);
           println!("\tColor.green -> {}", self.green);
           println!("\tColor.blue -> {}", self.blue);
       }
   }
   
   impl ContainerTrait for i32 {
       fn foo(&self) {
           println!("i32 -> {}", self);
       }
   }
   
   impl ContainerTrait for f64 {
       fn foo(&self) {
           println!("f64 -> {}", self);
       }
   }
   ```

   So, other types does not satisfy the bound `ContainerTrait`, such as `u32`.
These are basic constraints for that trait.

4. **Full picture**

   That's it. Let's see the big picture and check whether it works or not.

   ```rust
   use std::fmt::Debug;
       
   #[derive(Debug)]
   struct Color {
       red: u8,
       green: u8,
       blue: u8,
   }
       
   trait ContainerTrait {
       fn foo(&self);
   }
       
   impl ContainerTrait for Color {
       fn foo(&self) {
           println!("Color -> {:?}", self);
           println!("\tColor.red -> {}", self.red);
           println!("\tColor.green -> {}", self.green);
           println!("\tColor.blue -> {}", self.blue);
       }
   }
       
   impl ContainerTrait for i32 {
       fn foo(&self) {
           println!("i32 -> {}", self);
       }
   }
       
   impl ContainerTrait for f64 {
       fn foo(&self) {
           println!("f64 -> {}", self);
       }
   }
       
   struct Container<T: ContainerTrait> {
       field: T,
   }
       
   pub fn fun() {
       let inst1 = Container {
           field: Color {
               red: 10,
               green: 20,
               blue: 30,
           },
       };
       let inst2 = Container { field: 10 };
       let inst3 = Container { field: 12.2 };
       
       inst1.field.foo(); // This will work
       inst2.field.foo(); // This will work
       inst3.field.foo(); // This will work
   }
   ```

   It still works:

   ```shell
   Color { red: 10, green: 20, blue: 30 }
   10
   12.2
   ```

   If we comment out the implementation for `f64`, check the compiler:

   ```shell
   error[E0277]: the trait bound `{float}: ContainerTrait` is not satisfied
     --> src/fun_3_trait.rs:48:17
      |
   48 |     let inst3 = Container { field: 12.2 };
      |                 ^^^^^^^^^ the trait `ContainerTrait` is not implemented for `{float}`
      |
      = help: the following implementations were found:
                <Color as ContainerTrait>
                <i32 as ContainerTrait>
   note: required by a bound in `Container`
     --> src/fun_3_trait.rs:35:21
      |
   35 | struct Container<T: ContainerTrait> {
      |                     ^^^^^^^^^^^^^^ required by this bound in `Container`
   
   error[E0689]: can't call method `foo` on ambiguous numeric type `{float}`
     --> src/fun_3_trait.rs:52:17
      |
   52 |     inst3.field.foo(); // Won't work - "the trait `ContainerTrait` is not implemented for `{float}`"
      |                 ^^^
   
   Some errors have detailed explanations: E0277, E0689.
   For more information about an error, try `rustc --explain E0277`.
   ```

Rust compiler says no implementation for `{float}`. That's it. Constraints!

#### One More: Struct Implementation with Trait

One more step: define implementation for that struct for usable demo.

```rust
use std::fmt::Debug;

#[derive(Debug)]
struct Color {
    red: u8,
    green: u8,
    blue: u8,
}

trait ContainerTrait {
    fn foo(&self);
}

impl ContainerTrait for Color {
    fn foo(&self) {
        println!("Color -> {:?}", self);
        println!("\tColor.red -> {}", self.red);
        println!("\tColor.green -> {}", self.green);
        println!("\tColor.blue -> {}", self.blue);
    }
}

impl ContainerTrait for i32 {
    fn foo(&self) {
        println!("i32 -> {}", self);
    }
}

impl ContainerTrait for f64 {
    fn foo(&self) {
        println!("f64 -> {}", self);
    }
}

struct Container<T: ContainerTrait> {
    field: T,
}

// New here
impl<T: ContainerTrait> Container<T> {
    fn print(&self) {
        self.field.foo();
    }
}

pub fn fun() {
    let inst1 = Container {
        field: Color {
            red: 10,
            green: 20,
            blue: 30,
        },
    };
    let inst2 = Container { field: 10 };
    let inst3 = Container { field: 12.2 };

    inst1.print(); // This will work
    inst2.print(); // This will work
    inst3.print(); // This will work
}
```

Now, the real-life example should be complete.

If you find anything wrong, feel free to [contact me](https://godeep.pro/contact/).

<center><blockquote>The End</blockquote></center>

[ref]: https://dev.to/talzvon/rust-generic-types-in-method-definitions-4iah

