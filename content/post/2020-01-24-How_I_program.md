---
date: 2021-01-24T23:34:20Z
title: How I program
tags:
    - Rust
    - Programming
---

Getting better at Programming is, as any skill, a gradual process. You start as a beginner, and then
grow from there. Sadly, I did __not__ write down my thoughts or feelings during that time. Or, at
least not in any succinct form. As of this year, I've now programmed for at least 10 years. I had
started with C and frankly had no idea what I was doing (wanting to program 'a game', probably an
MMO like all 16 year olds). So, I turned my mind to the web, exploring HTML, CSS and Javascript.  I
did not get any of it, but I managed to scrape by; copy-pasting various snippets from sites like
[SelfHTML](https://www.selfhtml.org/) or [StackOverflow](https://stackoverflow.com/). I am happy to
say that nowadays I write most of my code myself, and only have to look online for examples or
documentation. As my language is Rust (and has been since 2015), I will focus in this blog post on
how I create a program.

## Thinking about the Problems

Before I write any kind of code, I first need to think about what I actually want to achieve.
Frankly, I am not sure how one could do it in any other way. As an example, let's say we want to open
a file given as a command line parameter and we will sum up all the lines as integers. What kind of
problems could arise? And how could they be handled? 'Power user'-centric software requires different
UX than say one accessible for all.

So, I search for all the different parts of the program I need:

- Command line handling
- File IO
- Program limits

For each I either use something I've already learned about in the past (like `clap` or rather
`structopt` for command line handling), or I have to use online searches (ddg/stackoverflow/etc…) to
find out what to expect. What happens if the file is bigger than the available RAM? How could I do
this in a streaming fashion? Are there any cool algorithms I could use to make it faster?

After this is done I have a good idea of the overall structure and possibly already an architecture
in mind. So I start laying down the barebones of the program.


## Abstractions

Not too long ago I had a heated discussion with a friend of mine who was a mathematician. I had
claimed that "an abstraction is awesome, because it allows you to do more" to which he answered
rather confused "No, if you abstract you can only do less." We then threw around some examples but
had to settle it as a disagreement of what we were discussing. Today I realize what our two point of
views are: On one side, you have the realists who looking at an object think "I can do
5 things with this," and when seen through an abstract see that now only a single thing can be done.
On the other side are the abstractionists (I made that word up), who when looking at an
abstraction think "I can use this with so many objects." and when seeing a single object see it
reduced to it's actual utility.  My friend was a 'realist' in his arguments, and I was an
'abstractionist' which is why we could only disagree. But we were both right. When looking at an
object through a lense you put the focus on a subset of its capabilities, thus reducing your
options! But when looking at abstractions, you realize (heh) that you only need one object that
fits the bill to achieve your goal. In my opinion, being able to switch between these two is the
key to unlocking how to program effectively. Some advice may frame it as 'use the right amount of
abstraction' but I believe that it misses the aspect that you need both.

## The actual programming

Having now said that, how does this translate to my daily programming? In practice, it's quite simple.
You simple write it down, __as if that is already your working code__. For example:

```rust

fn main() {
    let file_path = get_file_path();
    let file = std::io::File::open();
    let mut sum = 0;
    for line in file.lines() {
        sum += line.parse();
    }

    println!("{}", sum);
}
```

Now, this program doesn't compile and is not even complete, but it gives us a _start_ and an _end_! You see,
we have `sum` variable that _somehow_ needs to be incremented until the end of the `main` function so that
the correct result gets printed out. If we keep the earlier idea of abstraction in mind, we don't
think about 'file access' or 'parsing integers' but we are simply thinking about the abstract process of
summing up numbers.

The next step is to find out _how_ do we get those numbers? I usually try to stay away from anything
'specific' until I know that it fits the bill. For now, let's focus on the main part:

```rust
for line in file.lines() {
    sum += line.parse();
}
```

These three lines already assume quite a bunch:

- We have lines
- They are numbers
- They result fits into a single integer

And of course any kind of errors that could happen when doing IO. Can we clean this up? Yeah! Let's punt
it away from here and ignore any errors for now.

```rust
let sum = get_lines(file)
    .map(|line| line.parse())
    .sum();
```

Now, this might not seem like much, but already we have simplified the problem by quite a lot. There is no
more 'loop' but only an iterator. Whatever `parse` returns, it has to be "summable" and `get_lines` has
to return an iterator with 'lines' that can be parsed (or the compiler will not be able to compile this later).

Now, I feel like we have gone as far as we could to simplify this section, so let's work through this from the
beginning and adjust the code as needed. Starting with `get_lines`:

```rust
fn get_lines(input: File) -> impl Iterator<Item=&str> {
    todo!()
}
```

Not bad for a first try! This is the barebones code to satisfy our previous program snippet. Now,
how do we turn a `File` into an `Iterator` of lines? Let's check the
[docs](https://doc.rust-lang.org/std/index.html?search=Lines) for the most straightforward option.
Now, I know that the correct choice here would be to use a `BufReader`, but how would one know that
is the correct choice?  First, you could just check them out from the top. The first item tells you
"This struct is generally created by calling `lines` on a `BufRead`." It is an `Iterator` which
returns `String`! Hm, a `str` and `String` is not important here, so: Success! But what about the
other search results? Are there better ones? The second item says "This struct is created with the
`lines` method on `str`." And it returns an `Iterator` of `&str`. Hmm.. we don't have a
`str` and instead have a `File`. That is not a problem though, since we could call `read_to_string`
on our `File` and use the resulting `String`! And I would say that is a perfectly valid strategy. Right
now, it's all about finding a path between the what you have (the input) and what you need (the
output). You can always optimize later.  Since we already know of a good choice, we will simply use
`BufReader` here. How do we turn a `File` into a `BufReader`? Time to read some more documentation!

`BufReader` has two methods to create one, and both take a `T: Read` bound. Meaning, whatever
you give to it, it has to implement the `Read` trait. (This is quite similar to the idea of a function getting a `File`
and returning an `Iterator<&str>` except applied to objects) Luckily, `File` _does_ implement `Read`
so we can simply pass it into the constructor.

Now, there is one issue. Every time we try to read the file an error could happen. Which is why
`std::io::Lines` returns an iterator of `Result`s. So, let's switch from `Iterator<Item=&str>` to
`Iterator<Item=Result<String>>` and hope that we can deal with it down the line.

```rust
fn get_lines(input: File) -> impl Iterator<Item=Result<String>> {
    let buf_read = std::io::BufRead::new(input);

    buf_read.lines()
}
```
And we are done for that part.


Going back to our iterator:
```rust
let sum = get_lines(file)
    .map(|line| line.parse())
    .sum();
```

We now know that `get_lines` returns an iterator of `Result`s, so what are our options here?

- `unwrap` everything and let the program crash
- handle the error gracefully

Since now we can take an informed decision, let's opt to crash the program. IO Errors are really out of
scope of the program and the error message should help the user diagnose the issue.

Instead of `unwrap` I'd use `expect` here, just to customize the panic message.

```rust
let sum = get_lines(file)
    .map(|line| line.expect("to read the file").parse())
    .sum();
```

So, let's do a mid-way check! How do we get from `String` to `usize`? Luckily such a `parse`
function really does exist! You can find it here:
[parse](https://doc.rust-lang.org/std/primitive.str.html#method.parse). `usize` implements `FromStr`, so all's well.
But wait, `parse` returns another `Result`! What if the line is not a parseable number? Well, we can't really
fix that, so we can also expect the user to fix it. Another `expect` it is:


```rust
let sum = get_lines(file)
    .map(|line| line.expect("to read the file").parse().expect("to be a number line"))
    .sum();
```

And that is our main loop done! Let's go check out the how to parse arguments. I know that `structopt` is a good choice,
so I add it to my `Cargo.toml` and check out its [documentation](https://docs.rs/structopt/0.3.21/structopt/). Well,
that is fairly simple to use:

```rust
#[derive(Debug, structopt::StructOpt)]
#[structopt(name = "file summarizer", about = "Sums up all the lines of a file")]
struct Opt {
    #[structopt(name = "FILE", parse(from_os_str))]
    file_name: PathBuf,
}
```

Great! Everything is there. Now, let's plug all the individual pieces together:

```rust
#[derive(Debug, structopt::StructOpt)]
#[structopt(name = "file summarizer", about = "Sums up all the lines of a file")]
struct Opt {
    #[structopt(name = "FILE", parse(from_os_str))]
    file_name: std::path::PathBuf,
}

fn get_lines(input: std::fs::File) -> impl Iterator<Item = std::io::Result<String>> {
    use std::io::BufRead;
    let buf_read = std::io::BufReader::new(input);

    buf_read.lines()
}

fn main() {
    use structopt::StructOpt;
    let opt = Opt::from_args();
    let file = std::fs::File::open(&opt.file_name).expect("to open the file");
    let sum: usize = get_lines(file)
        .map(|line| {
            line.expect("to read the file")
                .parse::<usize>()
                .expect("to be a number")
        })
        .sum();
    println!("{}", sum);
}
```

I ran it through an extra `cargo fmt` and plugged some type holes `rustc` was complaining about.
And it works!

```
[01:05:20] neikos@katzenpalast /home/neikos/projects/rust/blog_example
> cat test
123
34
540
908284089
324
111
1
1
1
1
1
[01:05:23] neikos@katzenpalast /home/neikos/projects/rust/blog_example
> cargo run -- test
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `target/debug/blog_example test`
908285226
[01:05:25] neikos@katzenpalast /home/neikos/projects/rust/blog_example
> cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `target/debug/blog_example`
error: The following required arguments were not provided:
    <FILE>

USAGE:
    blog_example <FILE>

For more information try --help
[01:05:33] neikos@katzenpalast /home/neikos/projects/rust/blog_example
> cargo run -- --help
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `target/debug/blog_example --help`
file summarizer 0.1.0
Sums up all the lines of a file

USAGE:
    blog_example <FILE>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

ARGS:
    <FILE>

```

So, this is the gist of I program all my current projects. I start with a high level overview and
then keep on iterating while switching between 'realist' (aka actual types that do the work) and
'abstractionist' (aka abstract API points). The walkthrough of how I program should also serve as an overview
of what I consider the thresholds of when to change from which viewpoint to the other. In my mind it feels a bit
like playing Tennis where you keep switching sides and punt the 'ball' to the other.

Cheers
