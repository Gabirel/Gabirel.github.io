---
layout: post
lang: en
title: Atbash Cipher Code Refactoring Record
categories: rust algorithm
tags: rust exercism algorithm
date: 2019-12-28
---

<p class="intro"><span class="dropcap">T</span>This comes from the main exercise of exercism in rust. I asked some friends from a small Rust community and I got these elegant codes. These codes are so beautiful and tricky. So I decide to write it down and hope it may be useful for you as well.</p>

You may learn how to "chunk" vector in rust as a string without using a loop while joining characters.



-----
Table of Contents

* TOC
{:toc}

-----

# Instructions

Create an implementation of the atbash cipher, an ancient encryption system created in the Middle East.

The Atbash cipher is a simple substitution cipher that relies on transposing all the letters in the alphabet such that the resulting alphabet is backwards. The first letter is replaced with the last letter, the second with the second-last, and so on.

An Atbash cipher for the Latin alphabet would be as follows:

```shell
Plain:  abcdefghijklmnopqrstuvwxyz
Cipher: zyxwvutsrqponmlkjihgfedcba
```
It is a very weak cipher because it only has one possible key, and it is a simple monoalphabetic substitution cipher. However, this may not have been an issue in the cipher's time.

Ciphertext is written out in groups of fixed length, the traditional group size being 5 letters, and punctuation is excluded. This is to make it harder to guess things based on word boundaries.

Examples
- Encoding `test` gives `gvhg`
- Decoding `gvhg` gives `test`
- Decoding `gsvjf rxpyi ldmul cqfnk hlevi gsvoz abwlt` gives `thequickbrownfoxjumpsoverthelazydog`


# Test suite

```rust
use atbash_cipher as cipher;

#[test]
fn test_encode_yes() {
    assert_eq!(cipher::encode("yes"), "bvh");
}

#[test]
#[ignore]
fn test_encode_no() {
    assert_eq!(cipher::encode("no"), "ml");
}

#[test]
#[ignore]
fn test_encode_omg() {
    assert_eq!(cipher::encode("OMG"), "lnt");
}

#[test]
#[ignore]
fn test_encode_spaces() {
    assert_eq!(cipher::encode("O M G"), "lnt");
}

#[test]
#[ignore]
fn test_encode_mindblowingly() {
    assert_eq!(cipher::encode("mindblowingly"), "nrmwy oldrm tob");
}

#[test]
#[ignore]
fn test_encode_numbers() {
    assert_eq!(
        cipher::encode("Testing,1 2 3, testing."),
        "gvhgr mt123 gvhgr mt"
    );
}

#[test]
#[ignore]
fn test_encode_deep_thought() {
    assert_eq!(cipher::encode("Truth is fiction."), "gifgs rhurx grlm");
}

#[test]
#[ignore]
fn test_encode_all_the_letters() {
    assert_eq!(
        cipher::encode("The quick brown fox jumps over the lazy dog."),
        "gsvjf rxpyi ldmul cqfnk hlevi gsvoz abwlt"
    );
}

#[test]
#[ignore]
fn test_decode_exercism() {
    assert_eq!(cipher::decode("vcvix rhn"), "exercism");
}

#[test]
#[ignore]
fn test_decode_a_sentence() {
    assert_eq!(
        cipher::decode("zmlyh gzxov rhlug vmzhg vkkrm thglm v"),
        "anobstacleisoftenasteppingstone"
    );
}

#[test]
#[ignore]
fn test_decode_numbers() {
    assert_eq!(cipher::decode("gvhgr mt123 gvhgr mt"), "testing123testing");
}

#[test]
#[ignore]
fn test_decode_all_the_letters() {
    assert_eq!(
        cipher::decode("gsvjf rxpyi ldmul cqfnk hlevi gsvoz abwlt"),
        "thequickbrownfoxjumpsoverthelazydog"
    );
}

#[test]
#[ignore]
fn test_decode_with_too_many_spaces() {
    assert_eq!(cipher::decode("vc vix    r hn"), "exercism");
}

#[test]
#[ignore]
fn test_decode_with_no_spaces() {
    assert_eq!(
        cipher::decode("zmlyhgzxovrhlugvmzhgvkkrmthglmv"),
        "anobstacleisoftenasteppingstone",
    );
}
```



# Solution

## Version 1

```rust
const PLAIN: &str = "abcdefghijklmnopqrstuvwxyz";
const CIPHER: &str = "zyxwvutsrqponmlkjihgfedcba";

pub fn atbash(text: &str) -> Vec<char> {
    text.
        to_lowercase().
        chars().
        filter(|&x| x.is_ascii_alphabetic() || x.is_ascii_digit()).
        map(|f| {
            match PLAIN.find(f) {
                Some(c) => PLAIN.chars().rev().nth(c),
                None => Some(f),
            }.unwrap()
        }).collect::<Vec<char>>()
}

/// "Encipher" with the Atbash cipher.
pub fn encode(plain: &str) -> String {
    let tmp = atbash(plain);
    let mut result = String::new();
    for i in 0..tmp.len() {
        if i.rem_euclid(5) == 0 && i != 0 {
            result.push(' ');
        }
        result.push(tmp[i]);
    }
    result
}

/// "Decipher" with the Atbash cipher.
pub fn decode(cipher: &str) -> String {
    atbash(cipher).into_iter().collect()
}
```



`L11` uses `.rev()` feature to pin-point the index of `CIPHER` text. This is a little bit tricky.

In this version, I didn't find any solutions to wrap pushing string into a `.map()`, which makes it look like C/C++-style and ugly since we need a loop to append `space` for me. This can be changed!



## Version 2

```rust
const PLAIN: &str = "abcdefghijklmnopqrstuvwxyz";
const CIPHER: &str = "zyxwvutsrqponmlkjihgfedcba";

pub fn atbash(text: &str) -> Vec<char> {
    text
        .to_lowercase()
        .chars()
        .filter(|&x| x.is_ascii_alphabetic() || x.is_ascii_digit())
        .map(|f| {
            match PLAIN.find(f) {
                Some(c) => PLAIN.chars().rev().nth(c),
                None => Some(f),
            }.unwrap()
        }).collect::<Vec<char>>()
}

/// "Encipher" with the Atbash cipher.
pub fn encode(plain: &str) -> String {
    let tmp = atbash(plain);
    tmp
        .chunks(5)
        .map(|x| x.iter().collect::<String>())
        .collect::<Vec<String>>()
        .join(" ")
}

/// "Decipher" with the Atbash cipher.
pub fn decode(cipher: &str) -> String {
    atbash(cipher).into_iter().collect()
}
```



In version 2, I've made changes about wrapping encoded string. It happens at `L19~24`.

Explanation:

1. `L21` takes value for chunking vector as several slices(vector)
2. `L22` maps the chunked vector into `String`
3. `L23` collects all data generated in step 2 together and outputs as `Vec<String>`
4. `L24` use `.join()` method for joining item in `Vec<String>` with space and outputs the final result.

So, basically, you don't need a loop any more to add space after 5 characters. Removing loops in Rust is a kind of beautiful way of processing data.

## Version 3

```rust
const PLAIN: &str = "abcdefghijklmnopqrstuvwxyz";
const CIPHER: &str = "zyxwvutsrqponmlkjihgfedcba";

pub fn atbash(text: &str) -> Vec<char> {
    text
        .to_lowercase()
        .chars()
        .filter(|&x| x.is_ascii_alphabetic() || x.is_ascii_digit())
        .map(|f| {
            match PLAIN.find(f) {
                Some(c) => PLAIN.chars().rev().nth(c),
                None => Some(f),
            }.unwrap()
        }).collect::<Vec<char>>()
}

/// "Encipher" with the Atbash cipher.
pub fn encode(plain: &str) -> String {
    let tmp = atbash(plain);
    tmp
        .iter()
        .enumerate()
        .flat_map(|(i, c)| {
            if i.rem_euclid(5) == 0 && i != 0 {
                Some(' ')
            } else {
                None
            }.into_iter().chain(std::iter::once(*c))
        })
        .collect::<String>()
}

/// "Decipher" with the Atbash cipher.
pub fn decode(cipher: &str) -> String {
    atbash(cipher).into_iter().collect()
}
```



In version 3, in order to challenge memory usage, I've mainly removed one layer of `.collect()` since `.collect()` will consume heap memory usage which affects performance. So, I avoid using `.chunk()` and allocating memory on the heap again in `.map()` and replace it with the `iterator`. Why I choose `iterator` is that their lazily-evaluated nature, don't allocate on the heap until explicitly told so (with `.collect()` for instance).

Also, `.enumerate()` yields pairs so we could use `.flat_map()` to remove the layer of iterator and collect them as `String` only once.

Saving memory allocation on the heap. Goal Achieved.



## Version 4

```rust
const PLAIN: &str = "abcdefghijklmnopqrstuvwxyz";
const CIPHER: &str = "zyxwvutsrqponmlkjihgfedcba";

/// return Iterator type: https://gist.github.com/conundrumer/4e57c14705055bb2deac1b9fde84f83b
pub fn atbash<'a>(text: &'a str) -> impl Iterator<Item=char> + 'a {
    text
        .chars()
        .flat_map(|x| x.to_lowercase())
        .filter(|&x| x.is_ascii_alphabetic() || x.is_ascii_digit())
        .map(|f| {
            match PLAIN.find(f) {
                Some(c) => PLAIN.chars().rev().nth(c),
                None => Some(f),
            }.unwrap()
        })
}

/// "Encipher" with the Atbash cipher.
pub fn encode(plain: &str) -> String {
    let tmp = atbash(plain);
    tmp
        .enumerate()
        .flat_map(|(i, c)| {
            if i.rem_euclid(5) == 0 && i != 0 {
                Some(' ')
            } else {
                None
            }.into_iter().chain(std::iter::once(c))
        })
        .collect::<String>()
}

/// "Decipher" with the Atbash cipher.
pub fn decode(cipher: &str) -> String {
    atbash(cipher).collect()
}
```



In version 4, `atbash()` no longer returns `Vec<char>`, which avoids the memory allocation on the heap again.  But returning `Iterator` is a little bit tricky. You can find more examples in [here](https://gist.github.com/conundrumer/4e57c14705055bb2deac1b9fde84f83b).



Another change is that I move backward of `.to_lowercase()` and use `.flat_map()` instead since filtering all upprcase outside the function `atbash()` is not graceful and does not conform to the engineering.

So, basically in this version,  `.collect()`  operation is only used once. Memory allocation on the heap only happens once as well.

<center><blockquote>The End</blockquote></center>