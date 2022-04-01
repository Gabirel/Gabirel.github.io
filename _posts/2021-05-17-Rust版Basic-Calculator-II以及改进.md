---
layout: post
lang: zh
title: Rust 版 Basic Calculator II 以及改进
categories: algorithm rust
tags: rust algorithm
date: 2021-05-17
---

<p class="intro"><span class="dropcap">此</span>文源自于<a href="https://leetcode.com/problems/basic-calculator-ii/">Leetcode的基本计算机II</a>。使用Rust语言完成，共计有两个版本。一个是基础版本（可以完成题目的要求），另一个是在此基础之上的改进（可以完成所有的运算符的要求）。</p>

无论是基础版还是改进版，均使用Rust版本。无其他语言版本。

-----
Table of Contents

* TOC
{:toc}

-----

# 题目描述

原始题源：[点我](https://leetcode.com/problems/basic-calculator-ii/)

中文题源：[点我](https://leetcode-cn.com/problems/basic-calculator-ii/)

题目描述：

给你一个字符串表达式 s ，请你实现一个基本计算器来计算并返回它的值。
整数除法仅保留整数部分。

- 示例 1：
```
输入：s = "3+2*2"
输出：7
```

- 示例 2：
```
输入：s = " 3/2 "
输出：1
```

- 示例 3：
```
输入：s = " 3+5 / 2 "
输出：5
```

# 题目解法

## 解法1：基础计算解法

### 思路

- 加法压入栈中
- 减法压入负数的数字
- 乘法和除法直接运算即可，并把结果压入栈中

算法来源：https://leetcode-cn.com/problems/basic-calculator-ii/solution/ji-ben-ji-suan-qi-ii-by-leetcode-solutio-cm28/

### 代码

rust代码(`v1.rs`)如下：

```rust
/// Solution: https://leetcode-cn.com/problems/basic-calculator-ii/solution/ji-ben-ji-suan-qi-ii-by-leetcode-solutio-cm28/
/// PS：v1只能应对`+-*/`，不能处理带括号情况，甚至是其他优先级的情况
pub fn calculate(s: String) -> i32 {
    let n = s.len();
    let mut stack = vec![];
    let mut pre_sign = '+';
    let mut num = 0;

    for (idx, c) in s.chars().enumerate() {
        if c.is_numeric() {
            num = num * 10 + parse_as_int(c)
        }

        if !c.is_numeric() && !c.is_whitespace() || idx == n - 1 {
            match pre_sign {
                '+' => stack.push(num),
                '-' => stack.push(-num),
                '*' => {
                    let prev = stack.pop().unwrap();
                    stack.push(prev * num);
                }
                '/' => {
                    let prev = stack.pop().unwrap();
                    stack.push(prev / num);
                }
                _ => unreachable!(),
            }
            num = 0;
            pre_sign = c;
        }
    }

    stack.iter().sum()
}

fn parse_as_int(c: char) -> i32 {
    (c as u32 - '0' as u32) as i32
}

```


## 解法2：改进型全适应解法

### 改进的地方

- 使用双栈（数字栈和运算符栈）
- 适用于有括号的表达式
- 适用于`+ - * / ^ %`的运算符

### 思路

算法来源：https://leetcode-cn.com/problems/basic-calculator-ii/solution/shi-yong-shuang-zhan-jie-jue-jiu-ji-biao-c65k/

**思路一句话总结：模拟人计算表达式的过程，遇到括号先算括号内部的，遇到高优先级的先计算高优先级**

思路具体如下：
- 一共有两个栈：数字栈和运算符栈
- `(`: 直接加入运算符栈中，等待与之匹配的`)`
- `+ - * / ^ %`: 需要将操作放入运算符栈中。
- 只有「栈内运算符」比「当前运算符」优先级高/同等，才进行运算，（直到没有操作或者遇到左括号），计算结果放到栈中

该算法来源中有个例子，比如当前已读取的运算符为`2 + 1`（完整的运算符为：`2 + 1 * 2`）。
1. 如果后面是`+ 2`或者是`- 1`，满足「栈内运算符」比「当前运算符」优先级相等或者更高，说明可以直接运算`2 + 1`然后把结果放入数字栈中
2. 如果后面是`* 2`或者是`/ 1`，当前的运算符为`*`或者`/`，比栈内运算符`+`要高，所以要先计算`*`或者`/`。即，「栈内运算符」比「当前运算符」低，不能直接运算`2 + 1`，只能先运算后面`1 * 2`。


### 代码

rust代码(`v2.rs`)如下：

```rust
use std::collections::HashMap;

/// Solution: https://leetcode-cn.com/problems/basic-calculator-ii/solution/shi-yong-shuang-zhan-jie-jue-jiu-ji-biao-c65k/
/// PS：v2只能应对`+-*/%^`，甚至可以处理带括号的情况
///
/// 原理：
/// 因为这套「表达式计算」处理逻辑，本质上模拟了人脑的处理逻辑：根据下一位的运算符优先级决定当前运算符是否可以马上计算。
///
/// 改变：未完全按照原solution进行处理，暂时不考虑复杂情况
pub fn calculate(s: String) -> i32 {
    let s = remove_whitespace(&s);
    if s.is_empty() {
        return 0;
    }
    let n = s.len();
    let mut nums_stack: Vec<i32> = vec![];
    let mut ops_stack: Vec<char> = vec![];
    let ops_priority = setup_op_priority();

    let mut i = 0;
    while i < n {
        let c = s.chars().nth(i).unwrap();
        match c {
            '(' => ops_stack.push(c),
            ')' => {
                // 计算到最近一个左括号为止
                while !ops_stack.is_empty() {
                    if ops_stack.last().unwrap() == &'(' {
                        ops_stack.pop();
                        break;
                    }
                    calc(&mut nums_stack, &mut ops_stack);
                }
            }
            '0'..='9' => {
                // 把所有的数字都探测完成，然后压栈保存
                let mut num = 0;
                let mut j = i;
                while j < n && s.chars().nth(j).unwrap().is_numeric() {
                    let j_char = s.chars().nth(j).unwrap();
                    num = num * 10 + parse_as_int(j_char);
                    j += 1;
                }
                i = j - 1;
                nums_stack.push(num);
            }
            _ => {
                // 分支：其他操作符号（无括号的运算符）
                //
                // 有一个新操作要入栈时，先把栈内可以算的都算了
                // 只有满足「栈内运算符」比「当前运算符」优先级高/同等，才进行运算
                while !ops_stack.is_empty() && ops_stack.last().unwrap() != &'(' {
                    let prev_op = ops_stack.last().unwrap();
                    if ops_priority.get(prev_op) >= ops_priority.get(&c) {
                        calc(&mut nums_stack, &mut ops_stack);
                    } else {
                        break;
                    }
                }
                ops_stack.push(c);
            }
        }
        i += 1;
    }

    // 计算剩余的栈中的数据
    while !ops_stack.is_empty() {
        calc(&mut nums_stack, &mut ops_stack);
    }
    // 拿出结果
    nums_stack.pop().unwrap()
}

fn calc(nums_stack: &mut Vec<i32>, ops_stack: &mut Vec<char>) {
    let b = nums_stack.pop().unwrap();
    let a = nums_stack.pop().unwrap();
    let op = ops_stack.pop().unwrap();
    let ans = match op {
        '+' => a + b,
        '-' => a - b,
        '*' => a * b,
        '/' => a / b,
        '%' => a % b,
        '^' => a.pow(b as u32),
        _ => unreachable!(),
    };
    nums_stack.push(ans);
}

fn setup_op_priority() -> HashMap<char, i32> {
    let mut map = HashMap::new();
    map.insert('+', 1);
    map.insert('-', 1);
    map.insert('*', 2);
    map.insert('/', 2);
    map.insert('%', 2);
    map.insert('^', 3);

    map
}

fn parse_as_int(c: char) -> i32 {
    (c as u32 - '0' as u32) as i32
}

fn remove_whitespace(s: &str) -> String {
    s.split_whitespace().collect::<String>()
}

```

<center><blockquote>The End</blockquote></center>

