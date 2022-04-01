---
layout: post
lang: en
title: Rust Merge Strings Alternately
categories: rust
tags: rust algorithm usage
date: 2021-05-18
---

<p class="intro"><span class="dropcap">T</span>his posts comes from the real project scenario. Later, I surprisedly find there is one problem in Leetcode. Personally I not a fan of solving Leetcode problems. Let's write a post here since they provide detailed descriptions.</p>

The problem is easy-level one and not complicated. So, just make it become onf of my cheatseets then.



-----
Table of Contents

* TOC
{:toc}
------------------

# Description

Source：[Click Me: Leetcode-1768](https://leetcode.com/problems/merge-strings-alternately/)

Description:

You are given two strings word1 and word2. Merge the strings by adding letters in alternating order, starting with word1. If a string is longer than the other, append the additional letters onto the end of the merged string.

Return the merged string.

Example 1:

```
Input: word1 = "abc", word2 = "pqr"
Output: "apbqcr"
Explanation: The merged string will be merged as so:
word1:  a   b   c
word2:    p   q   r
merged: a p b q c r
```


Example 2:

```
Input: word1 = "ab", word2 = "pqrs"
Output: "apbqrs"
Explanation: Notice that as word2 is longer, "rs" is appended to the end.
word1:  a   b 
word2:    p   q   r   s
merged: a p b q   r   s
```

Example 3:

```
Input: word1 = "abcd", word2 = "pq"
Output: "apbqcd"
Explanation: Notice that as word1 is longer, "cd" is appended to the end.
word1:  a   b   c   d
word2:    p   q 
merged: a p b q c   d
```

# Solution

The algorithm itself is quite simple so I won't repeat this in here. Let's show the code directly:

```rust
fn main() {
    assert_eq!(
        "a1b2c3def",
        merge_alternately("abcdef".to_string(), "123".to_string())
    );
    assert_eq!(
        "a1b2c3456",
        merge_alternately("abc".to_string(), "123456".to_string())
    );
}

pub fn merge_alternately(word1: String, word2: String) -> String {
    let mut ans = String::new();

    let mut i = 0;
    let mut j = 0;

    while i < word1.len() || j < word2.len() {
        if let Some(c) = word1.chars().nth(i) {
            ans.push(c);
        }

        if let Some(c) = word2.chars().nth(i) {
            ans.push(c);
        }

        i += 1;
        j += 1;
    }

    ans
}
```



# Updates

20210520 Updates：

Later, I suddenly find a crate named [itertools](https://docs.rs/itertools/), which is quite convenient for us to release heavy lifting work.

For example, we are able to merge, ordering-style merge(`kmerge`), create an iterator over the “cartesian product” of iterators, as well as interleaving.



No more talking! Let's check the code below:

```rust
use itertools::Itertools;

fn main() {
    assert_eq!(
        "a1b2c3def",
        merge_alternately_itertools("abcdef".to_string(), "123".to_string())
    );
    assert_eq!(
        "a1b2c3456",
        merge_alternately_itertools("abc".to_string(), "123456".to_string())
    );
}

// I know I can simplify code below but I insist this.
pub fn merge_alternately_itertools(word1: String, word2: String) -> String {
    let mut ans = String::new();

    ans = word1.chars().interleave(word2.chars()).collect();

    ans
}
```



<center><blockquote>The End</blockquote></center>