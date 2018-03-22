---
title: "Rust References, and how to think about them"
date: 2017-11-26T12:46:53+01:00
draft: true
---

I got the idea for this blog post after a friend of mine asked me how he could
return a `&str`. And given that Rust makes sure that you cannot just return
things willy-nilly I figured that all they were lacking was some mind models to
use them correctly.

## Getting Started

This blog post was written during the **1.18** release. However, these ideas are
pretty fundamental, so I do not see them changing anytime soon.

I will post the gists

## Data flow

Usually when programming we do not manipulate data, instead we write algorithms
that, given inputs, return a specific output. Now, this distinction is
important, as it allows you to stop thinking about specific values and allows
you think about input *ranges*. This is in my opinion an important piece of
thinking.

---

**Example:**


Imagine you are writing an application that takes in contacts, filters them by
age and then returns them.

Which one of these are more likely to be correct?

{{< gist theneikos 15a6edb4aa9fb90e562db11f93fe1e00 "type_example_simple.rs" >}}

{{< gist theneikos 15a6edb4aa9fb90e562db11f93fe1e00 "type_example_complete.rs" >}}

---

**Output:**

Simple:

```
neikos ~/t/blog (master)> cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/blog`
Please enter your email: asd

Please enter your age: 123

Hello asd, you are 123 years old.
```

Complete:

```
neikos ~/t/blog (master)> cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.48 secs
     Running `target/debug/blog`
Please enter your email: hello

Please enter your age: 15

Error: Email address contain an @ sign
```

---



- Explain how data flow works in a program and what Rust does to help
- Explain a few examples of this -> Result, Option, Vector

## Borrowing

- Explain how borrowing fits into this
- Explain difference between non-mut and mut and what it assures

## Modeling Dataflow with Rust

- Explain how types and newtypes help with modelling data flow

