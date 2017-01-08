---
layout: post
date: 2017-01-08 18:06:03
title: Using Stainless with Rocket
---

> In this Blog post I will see how Rocket, Stainless and Serde work together to
> create easy and usable testing.

*This blog posts assumes Rust nightly from at least version 1.15. Others haven't
been tested, so YMMV.*

[Rocket](rocket) is a new Rust web framework that has been released to the
public late last year. It aims to provide a simple way to write and maintain
fast and safe web applications.

I have been waiting for something like this for quite some time, from the way it
looks it tries to be the [Rails](rails) of the Rust world.

One part of that is being able to easily test your applications. After all,
without tests it is almost impossible to be confident that the output conforms
to what is expected, especially in an evolving codebase.

I've been recently wanting to try out [stainless](stainless) as well, so I
thought why not combine both and see how they play together!

## Setting up

I will be using a simple rocket application that will just send back a JSON
formatted string at the `/status` endpoint.

Create the rust project:

```sh
cargo new --bin rocket_stainless
     Created binary (application) `rocket_stainless` project
cd rocket_stainless/
```

Now, let's add rocket and stainless as dependencies:

```sh
cargo add rocket
cargo add rocket_contrib
cargo add rocket_codegen
cargo add serde_derive
cargo add -D stainless
```
> `cargo add` is part of the useful [`cargo-edit`][ce] package of utilities

We also need to enable the testing feature when testing. Edit the `Cargo.toml`
and add to the `dev-dependencies` table the following line:

```
rocket = { version = "0.1.4", features = ["testing"] }
```
> Be sure to match up the version with whatever rocket version you are using!

Next step is setting everything up in `src/main.rs`:

```rust
#![feature(plugin)]
#![plugin(rocket_codegen)]
#![cfg_attr(test, plugin(stainless))]
extern crate rocket;
extern crate rocket_contrib;
#[macro_use] extern crate serde_derive;

mod routes {
    use rocket_contrib::JSON;

    #[derive(Serialize)]
    pub struct Status {
        status: String,
        hello: String,
    }

    #[get("/status")]
    pub fn status() -> JSON<Status> {
        JSON(Status {
            status: String::from("Ok"),
            hello: String::from("world"),
        })
    }
}

fn main() {
    rocket::ignite().mount("/", routes![routes::status]).launch();
}
```

The code should be fairly self-explanatory. In the first part we setup the
executable to include the `rocket_codegen` plugin and also include the stainless
plugin if we are compiling for tests.  Then we have a small module (to keep
things clean) where we declare a struct to serialize in a Rocket endpoint.  In
the main function we then launch rocket and and the server is up! (in orbit)

Okay, we are all set up. You can now run it with `cargo run` and it should open
a local server:

```sh
neikos ~/p/r/b/rocket_stainless (master) [130]> cargo run
    Finished debug [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/rocket_stainless`
🔧  Configured for development.
    => listening: localhost:8000
    => logging: Normal
🛰  Mounting '/':
    => GET /status
🚀  Rocket has launched from http://localhost:8000...
GET /status:
    => Matched: GET /status
    => Outcome: Success
    => Response succeeded.

// On another Shell

neikos ~> curl localhost:8000/status
{"status":"Ok","hello":"world"}
```

Now it's time to get to the...

## Testing

We now have a working Rocket application that returns a piece of JSON on
`/status`. Let's write up some tests for it.

We add this to the `src/main.rs`:

```rust

#[cfg(test)]
#[allow(unused_variables)]
mod tests {
    use rocket;
    use rocket::testing::MockRequest;
    use rocket::http::{Status, Method};

    use routes;

    describe! route_tests {
        before_each {
            let rocket = rocket::ignite().mount("/", routes![routes::status]);
        }

        describe! status {
            before_each {
                let mut req = MockRequest::new(Method::Get, "/status");
                let mut res = req.dispatch_with(&rocket);
                let body_str = res.body().and_then(|b| b.into_string()).unwrap();
            }

            it "responds with status OK 200" {
                assert_eq!(res.status(), Status::Ok);
            }

            it "responds with OK" {
                assert!(body_str.contains("Ok"));
            }

            it "responds with hello world" {
                assert!(body_str.contains("hello"));
                assert!(body_str.contains("world"));
            }
        }
    }
}
```

This is a very simple test suite that just checks the above status endpoint.
It should give one a simple overview of how these awesome crates work together
seamlessly! There are still some errors here and there due to needing rust
nightly, but all those I've encountered so far were during compilation and not
in runtime, and usually I was doing something wrong.

We can now run the tests and check if our endpoint behaves correctly:

```sh
 cargo test
    Finished debug [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/rocket_stainless-d93b1173e0670f3f

running 3 tests
test tests::route_tests::status::responds_with_OK ... ok
test tests::route_tests::status::responds_with_hello_world ... ok
test tests::route_tests::status::responds_with_status_OK_200 ... ok

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured
```

Looks like it does, yay!

This concludes this post; we have now a working test suite that is easy to
extend without need to write a lot of boilerplate and are able to focus on
the important bits of the tests.

I would love to hear back what kind of testing strategies you use! So do send me
a message on [reddit](reddit) or shoot (launch?) me an email.


[rocket]: https://rocket.rs
[rails]: http://rubyonrails.org/
[stainless]: https://github.com/reem/stainless
[ce]: https://github.com/killercup/cargo-edit
[reddit]: https://www.reddit.com/message/compose?to=theneikos
