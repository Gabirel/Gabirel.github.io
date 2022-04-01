---
layout: post
lang: en
title: Code Refactoring of Bob - Exercism in Rust
categories: rust algorithm
tags: algorithm rust exercism
date: 2019-10-14
---

<p class="intro"><span class="dropcap">T</span>his comes from the side exercise of exercism in rust. Mentor `gaetanww` helped me a lot on code refactoring and logical improvements advice with great patience and careful instruction. He even changed my attitude towards work and researching. I have nothing else to say to express my thanks to my mentor.</p>

So I decide to record the whole process of code improvements and advice from `gaetanww`.

I am pretty sure that you could gain lessons from this post as well.



-----
Table of Contents

* TOC
{:toc}

-----

# Instructions

Bob is a lackadaisical teenager. In conversation, his responses are very limited.

Bob answers 'Sure.' if you ask him a question, such as "How are you?".

He answers 'Whoa, chill out!' if you YELL AT HIM (in all capitals).

He answers 'Calm down, I know what I'm doing!' if you yell a question at him.

He says 'Fine. Be that way!' if you address him without actually saying anything.

He answers 'Whatever.' to anything else.

Bob's conversational partner is a purist when it comes to written communication and always follows normal rules regarding sentence punctuation in English.

# Test suite

```rust
use bob;

fn process_response_case(phrase: &str, expected_response: &str) {
    assert_eq!(bob::reply(phrase), expected_response);
}

#[test]
fn test_stating_something() {
    process_response_case("Tom-ay-to, tom-aaaah-to.", "Whatever.");
}

#[test]
#[ignore]
fn test_shouting() {
    process_response_case("WATCH OUT!", "Whoa, chill out!");
}

#[test]
#[ignore]
fn test_shouting_gibberish() {
    process_response_case("FCECDFCAAB", "Whoa, chill out!");
}

#[test]
#[ignore]
fn test_asking_a_question() {
    process_response_case("Does this cryogenic chamber make me look fat?", "Sure.");
}

#[test]
#[ignore]
fn test_asking_a_numeric_question() {
    process_response_case("You are, what, like 15?", "Sure.");
}

#[test]
#[ignore]
fn test_asking_gibberish() {
    process_response_case("fffbbcbeab?", "Sure.");
}

#[test]
#[ignore]
fn test_talking_forcefully() {
    process_response_case("Let's go make out behind the gym!", "Whatever.");
}

#[test]
#[ignore]
fn test_using_acronyms_in_regular_speech() {
    process_response_case("It's OK if you don't want to go to the DMV.", "Whatever.");
}

#[test]
#[ignore]
fn test_forceful_question() {
    process_response_case(
        "WHAT THE HELL WERE YOU THINKING?",
        "Calm down, I know what I'm doing!",
    );
}

#[test]
#[ignore]
fn test_shouting_numbers() {
    process_response_case("1, 2, 3 GO!", "Whoa, chill out!");
}

#[test]
#[ignore]
fn test_no_letters() {
    process_response_case("1, 2, 3", "Whatever.");
}

#[test]
#[ignore]
fn test_question_with_no_letters() {
    process_response_case("4?", "Sure.");
}

#[test]
#[ignore]
fn test_shouting_with_special_characters() {
    process_response_case(
        "ZOMG THE %^*@#$(*^ ZOMBIES ARE COMING!!11!!1!",
        "Whoa, chill out!",
    );
}

#[test]
#[ignore]
fn test_shouting_with_no_exclamation_mark() {
    process_response_case("I HATE THE DMV", "Whoa, chill out!");
}

#[test]
#[ignore]
fn test_statement_containing_question_mark() {
    process_response_case("Ending with ? means a question.", "Whatever.");
}

#[test]
#[ignore]
fn test_nonletters_with_question() {
    process_response_case(":) ?", "Sure.");
}

#[test]
#[ignore]
fn test_prattling_on() {
    process_response_case("Wait! Hang on. Are you going to be OK?", "Sure.");
}

#[test]
#[ignore]
fn test_silence() {
    process_response_case("", "Fine. Be that way!");
}

#[test]
#[ignore]
fn test_prolonged_silence() {
    process_response_case("          ", "Fine. Be that way!");
}

#[test]
#[ignore]
fn test_alternate_silence() {
    process_response_case("\t\t\t\t\t\t\t\t\t\t", "Fine. Be that way!");
}

#[test]
#[ignore]
fn test_multiple_line_question() {
    process_response_case(
        "\nDoes this cryogenic chamber make me look fat?\nNo.",
        "Whatever.",
    );
}

#[test]
#[ignore]
fn test_starting_with_whitespace() {
    process_response_case("         hmmmmmmm...", "Whatever.");
}

#[test]
#[ignore]
fn test_ending_with_whitespace() {
    process_response_case("Okay if like my  spacebar  quite a bit?   ", "Sure.");
}

#[test]
#[ignore]
fn test_other_whitespace() {
    process_response_case("\n\r \t", "Fine. Be that way!");
}

#[test]
#[ignore]
fn test_nonquestion_ending_with_whitespace() {
    process_response_case(
        "This is a statement ending with whitespace      ",
        "Whatever.",
    );
}
```



# Solution

## Version 1

```rust
pub fn reply(message: &str) -> &str {
    if is_all_capitalized(message) && have_letters(message) {
        if message.chars().nth(message.len() - 1) == Some('?') {
            return "Calm down, I know what I'm doing!";
        }
        return "Whoa, chill out!";
    }

    if have_letters(message) {
        if message.trim().chars().nth(message.trim().len() - 1) == Some('?') {
            return "Sure.";
        }
        return "Whatever.";
    }
    if have_nothing(message) {
        return "Fine. Be that way!";
    }
    if message.chars().nth(message.len() - 1) == Some('?') {
        return "Sure.";
    }
    return "Whatever.";
}

fn is_all_capitalized(message: &str) -> bool {
    for i in 0..message.len() {
        let f = message.chars().nth(i);
        if f >= Some('a') && f <= Some('z') {
            return false;
        }
    }
    true
}

fn have_letters(message: &str) -> bool {
    for i in 0..message.len() {
        let f = message.chars().nth(i);
        if f >= Some('a') && f <= Some('z') || f >= Some('A') && f <= Some('Z') {
            return true;
        }
    }
    false
}

fn have_nothing(message: &str) -> bool {
    if !have_letters(message) {
        for i in 0..message.len() {
            let f = message.chars().nth(i);
            if f != Some(' ') && f != Some('\n') && f != Some('\t') && f != Some('\r') {
                return false;
            }
        }
    }
    true
}
```



## Comments on Version 1



* Check out using `match` instead of `if`. For some things, like destructuring, it's required. It's an idiom specific to Rust, so I always default to it unless the logic is comparing many different things and a match, in that case, makes it messy.

* Have a look at [`.ends_with()`](https://doc.rust-lang.org/std/string/struct.String.html#method.ends_with) to check if it's a question, it expresses your intent better.

* at line 48 you wrote:

  ```rust
  if f != Some(' ') && f != Some('\n') && f != Some('\t') && f != Some('\r')
  ```

  The method `.trim()` gets rid of all these characters for you :)

* We don't necessarily need to create separate functions here. We can put this all in one by setting these calls to variables instead (check iterator methods, like `.any()` etc.)

* Your function `is_all_capitalized()` does not handle non-ASCII characters.



## Version 2

```rust
pub fn reply(message: &str) -> &str {
    if message.trim().ends_with("?") {
        match is_all_capitalized(message.trim()) {
            true => if have_letters(message.trim()) {
                return "Calm down, I know what I'm doing!";
            } else {
                return "Sure.";
            },
            false => return "Sure.",
        }
    }
    match message.trim().len() <= 0 {
        true => return "Fine. Be that way!",
        _ => {}
    }
    match is_all_capitalized(message.trim()) {
        true => {
            if have_letters(message.trim()) {
                return "Whoa, chill out!";
            }
        }
        _ => {}
    }
    "Whatever."
}

fn is_all_capitalized(message: &str) -> bool {
    for ch in message.chars() {
        match ch {
            'a'..='z' => return false,
            _ => {}
        }
    }
    true
}

fn have_letters(message: &str) -> bool {
    for i in 0..message.len() {
        let f = message.chars().nth(i);
        if f >= Some('a') && f <= Some('z') || f >= Some('A') && f <= Some('Z') {
            return true;
        }
    }
    false
}
```



## Comments on Version2

- One of the main goals of exercism is to teach 'fluency' in a programming language. One of the aspects of fluency in Rust is finding information easily in the standard library and third-party libraries, there is often a way to do things more shortly, correctly and efficiently.

  For example, you can replace some of your functions by methods on characters (`.is_alphabetic()`, `.is_uppercase`) or iterators and strings (`.is_empty()` instead of `string.len() == 0`). May I suggest you look up these functions and see if it would help you? It might also bring some edge-cases that you haven't considered (Is `Ü` uppercase and/or alphabetic? ).

  - My answer: `Ü` is uppercase and is not alphabetic.

## Version 3

```rust
pub fn reply(message: &str) -> &str {
    if message.trim().ends_with("?") {
        match is_all_capitalized(message.trim()) {
            true => if have_letters(message.trim()) {
                return "Calm down, I know what I'm doing!";
            } else {
                return "Sure.";
            },
            false => return "Sure.",
        }
    }
    match message.trim().is_empty() {
        true => return "Fine. Be that way!",
        _ => {}
    }
    match is_all_capitalized(message.trim()) {
        true => {
            if have_letters(message.trim()) {
                return "Whoa, chill out!";
            }
        }
        _ => {}
    }
    "Whatever."
}

fn is_all_capitalized(message: &str) -> bool {
    for ch in message.chars() {
        match ch.is_uppercase() {
            false => {
                if ch.is_ascii_alphabetic() {
                    return false;
                }
            }
            _ => {
                println!("{}: {}", ch, ch.is_uppercase());
            }
        }
    }
    true
}
fn have_letters(message: &str) -> bool {
    for ch in message.chars() {
        match ch.is_ascii_alphabetic() {
            true => return true,
            _ => {}
        }
    }
    false
}
```



## Comments on Version 3

- You recompute `message.trim()` several times (at line 2, 12, 16 and 18), it could be better to store it in a variable like:

  ```rust
  let trimmed_message = message.trim();
  or even
  let message = message.trim();
  ```

  This is probably going to get optimized by the compiler, but it would at least make your program a bit shorter.

- In the functions `is_all_capitalized` and `have_letters` you use an iterator (`message.chars()`) match on the elements to verify some properties on them. You can rewrite those two functions with only iterators methods, would you like to try it?

  It would look a bit like that:

  ```rust
  let is_all_capitalized = message.chars().method1(...);
  ```

  If you're stuck, have a look at the [any()](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.any) and [all()](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.all) methods.



## Version 4

```rust
pub fn reply(message: &str) -> &str {
    let message = message.trim();
    if message.ends_with("?") {
        match is_all_capitalized(message) {
            true => if have_letters(message) {
                return "Calm down, I know what I'm doing!";
            } else {
                return "Sure.";
            },
            false => return "Sure.",
        }
    }
    match message.is_empty() {
        true => return "Fine. Be that way!",
        _ => {}
    }
    match is_all_capitalized(message) {
        true => {
            if have_letters(message) {
                return "Whoa, chill out!";
            }
        }
        _ => {}
    }
    "Whatever."
}

fn is_all_capitalized(message: &str) -> bool {
    !message.chars().any(|x| x.is_ascii_alphabetic() && x.is_lowercase())
}

fn have_letters(message: &str) -> bool {
    message.chars().any(|x| x.is_ascii_alphabetic())
}
```



## Comments on Version4

- You can use `match` statements on tuples. For example:

  ```rust
  match (condition1, condition2) {
      (_, true) => ...,
      (true, false) => ..., 
  }
  ```

  Can you use that to make your code more straightforward?



## Version 5

```rust
pub fn reply(message: &str) -> &str {
    let message = message.trim();
    if message.ends_with("?") {
        match (is_all_capitalized(message), have_letters(message)) {
            (true, true) => return "Calm down, I know what I'm doing!",
            (true, false) => return "Sure.",
            (false, _) => return "Sure.",
        }
    }
    match message.is_empty() {
        true => return "Fine. Be that way!",
        _ => {}
    }
    match (is_all_capitalized(message), have_letters(message)) {
        (true, true) => return "Whoa, chill out!",
        (_, _) => {}
    }
    "Whatever."
}

fn is_all_capitalized(message: &str) -> bool {
    !message.chars().any(|x| x.is_ascii_alphabetic() && x.is_lowercase())
}

fn have_letters(message: &str) -> bool {
    message.chars().any(|x| x.is_ascii_alphabetic())
}
```



## Comments on Version 5

- We don't tend to use lots of return statements, so if you have a lot of them you might be able to iprove your code. The reason why wer don't use them so much is that it makes the control flow of your program more difficult to read.
For example, in that exercise, you can get rid of all the return statements and have only one big match. Try and find how to do it :)

- One way you can move forward with this exercise is to store boolean results in variable: for example:

  ```rust
  let is_empty = message.is_empty();
  let is_question = message.ends_with("?");
  let is_all_capitalized = etc...
  ```

  Hopefully this will clear things up for the `match` statement.



## Version 6

```rust
pub fn reply(message: &str) -> &str {
    let message = message.trim();
    let is_empty = message.is_empty();
    let is_question = message.ends_with("?");
    let is_all_capitalized = is_all_capitalized(message);
    let is_having_letters = have_letters(message);

    match is_question {
        true => {
            match (is_all_capitalized, is_having_letters) {
                (true, true) => return "Calm down, I know what I'm doing!",
                (true, false) => return "Sure.",
                (false, _) => return "Sure.",
            }
        }
        false => {
            match is_empty {
                true => return "Fine. Be that way!",
                false => match (is_all_capitalized, is_having_letters) {
                    (true, true) => return "Whoa, chill out!",
                    (_, _) => {}
                }
            }
        }
    }

    "Whatever."
}

fn is_all_capitalized(message: &str) -> bool {
    message.chars().all(|x| x.is_ascii_alphabetic() && x.is_uppercase() || !x.is_ascii_alphabetic())
}

fn have_letters(message: &str) -> bool {
    message.chars().any(|x| x.is_ascii_alphabetic())
}
```



## Comments on Version 6

- Rust  has other paradigms that are fun and interesting to explore. One of them is that Rust is an [expression-oriented language](https://doc.rust-lang.org/1.8.0/book/glossary.html#expression-oriented-language) where all statements return a value, and that's why you don't necessarily need a `return` to return a value from a function. You'll see that if you avoid using it (when it makes sense to, returning early is a valid use case for using `return`) it will compose very well with other bits of rust code.

------

As for your code, if you want to get rid of te return statements, try:

```rust
match(is_question, is_question, is_having_letters, etc.) {...}
```

- I think you can simplify the problem to have two or three variables in the match statements.

  Consider matching with:

  ```rust
  let is_yelled = is_all_capitalized && is_having_letters;
  match (is_empty, is_question, is_yelled) {...}
  ```

  Or even:

  ```rust
  if message.is_empty() {
          return "Fine. Be that way!";
  }
  match (is_question, is_yelled) {...}
  ```

- Your code can be refactored. Please see:

```rust
if message.is_empty() {
    return "Fine. Be that way!";
}

match (is_question, is_yelled) {
    (true, true) => "Calm down, I know what I'm doing!",
    (true, false) => "Sure.",
    (false, true) => "Whoa, chill out!",
    (false, false) => "Whatever."
    }
```



## Version 7

```rust
pub fn reply(message: &str) -> &str {
    let message = message.trim();
    let is_question = message.ends_with("?");
    let is_yelled = is_all_capitalized(message) && have_letters(message);

    if message.is_empty() {
        return "Fine. Be that way!";
    }
    match (is_question, is_yelled) {
        (true, true) => "Calm down, I know what I'm doing!",
        (true, false) => "Sure.",
        (false, true) => "Whoa, chill out!",
        (false, false) => "Whatever.",
    }
}

fn is_all_capitalized(message: &str) -> bool {
    message.chars().all(|x| x.is_ascii_alphabetic() && x.is_uppercase() || !x.is_ascii_alphabetic())
}

fn have_letters(message: &str) -> bool {
    message.chars().any(|x| x.is_ascii_alphabetic())
}
```



## Comments on Version 7

- Well done!

<center><blockquote>The End</blockquote></center>

