---
layout: post
date: 2016-10-08 12:38:18
title: Using Rust for Webdev as a Hobby Programmer
---

I've been a huge fan of rust ever since I started using it (which was right
before 1.0 as I didn't like updating my code all the time to the latest syntax).
At first I created a few silly applications, some games, tried my hand at
libraries; but in the end I was still drawn back into Web Development.
You have to know that I grew up with PHP and then Ruby for Web Development, so I
went from CakePHP to Ruby on Rails, which was amazing! RoR had so many features
out of the box that I used, it just worked at first.
But then came the pitfalls of frameworks: indirection over indirection.
This is what causes the hours upon hours of debugging on why when removing a
part from A it breaks B.

Enter [Rust][2]. I have personally found few issues that I really disagree with
that are not simply design choices (based on taste rather than engineering) or
were my fault all along. A few others have talked about their experiences, so if
you want a more nuanced approach you could check those out, the [/r/rust][6]
subreddit is also an excellent place to look for. (Some I've read: [here][3],
[here][4], or [here][5]). While they do adress a few shortcomings it is overall
possible to work with it. (And most problems seem not intrinsic to the language,
but due to immaturity or ignorance of existing features/solutions).

A few months back I found the page [arewewebyet.org][1] and wanted to see how
fun Rust would be to use. I will outline the different things I've learned and
encountered here. If you think I could have gone into more detail or might have
missed out on something please do tell me! (I can be reached on Reddit as
TheNeikos or by mail at "neikos at neikos.email")


## Rust and the libraries

Due to the libraries and plugins I planned to use I had to switch to the nightly
branch of Rust. This can be a deal break to some, however I had no troubles so
far since the only thing unstable I use are `plugin`, `custom_derive`,
`custom_attribute` and `question_mark`. It might be that some of the crates use
more experimental features, however they are well-tested from what I saw.

So, after having run `cargo init` I needed a few things that I was promised that
are covered:

- Web Framework
- ORM
- Crypto
- Logging
- Templating

What I wanted to write is a website to host an art community on. So I would also
need an image handling library.

My choices so far are:

- [`iron`][iron] -- I like the middleware approach, I'm slowly getting to it's pitfalls
  (regarding code duplication and resource loading) but I have ideas to solve
  those and thus still recommend using it
- [`diesel`][diesel] -- A good looking (API-wise) library, while slightly verbose at
  times, it works really well.
- [`bcrypt`][bcrypt] -- I only need it for password hashing, so this is a weakly held
  choice in term of library. It works and it does what it says on the tin.
- [`log`][log] + [`log4rs`][log4rs] -- I had some log4j at University and didn't like it (might
  be the Java though... brr) but log4rs seems to fit better into my paradigms
  and it works.
- [`maud`][maud] -- My favorite find in terms of libraries. It allows you to write a
  Rust Macro that spits out HTML! It's a first taste of what plugins will allow
  you to do and I love it.

A lot of these choices are subjective at the end due to the fact that there are
many different choices and that none have differentiated themselves yet. So feel
free to use something else! I can vouch for the aformentioned Libraries though.

## Middleware and Iron

Iron uses Handlers to respond to a given request, the way it is built allows for
quite a flexible array of patterns to be used. For example the one I use is a
mix of `Router` and `Chain`. `Router` allows me to specify (in text) a mapping
from paths to another Handler. `Chain` then allows me to run through a _chain_
of `BeforeMiddleware` to `AroundMiddleware` or `AfterMiddleware` with my handler
in the middle. A few examples:

_Note that this code is just to get an idea of what is possible, it might not
compile due to missing implementations that would be too long/verbose to add!_

**Hello World in Iron**
```rust
use iron::{Iron, Request, Response, IronResult};

fn handler(_: &mut Request) -> IronResult<Response> {
    Ok(Response::with("Hello World"))
}

fn main() {
    let server = Iron::new(handler);
    match server.http("0.0.0.0:3000") {
        Ok(()) => { /*Listening blocks the thread */ }
        Err(e) => {
            println!("Could not create Iron Server: {}", e);
        }
    }
}
```

**Basic Router**
```rust
extern crate iron;
extern crate router;

use iron::{Iron, Request, Response, IronResult};
use router::Router;

mod handlers {
    use iron::{Request, Response, IronResult};
    use router::Params;

    pub fn world(_: &mut Request) -> IronResult<Response> {
        Ok(Response::with("Hello World"))
    }

    pub fn answer(_: &mut Request) -> IronResult<Response> {
        use params::{Params, Value};
        let map = req.get_ref::<Params>().unwrap();

        let name;
        if let Some(&Value::String(ref nam)) = map.find("name") {
            name = nam;
        } else {
            name = "Anonymous";
        }

        Ok(Response::with(format!("Hello {}", name)))
    }
}

fn main() {
    let mut router = Router::new();
    router.get("/", handlers::world, "index");
    router.get("/hello/:name", handlers::answer, "answer");

    let server = Iron::new(handler);
    match server.http("0.0.0.0:3000") {
        Ok(()) => { /*Listening blocks the thread */ }
        Err(e) => {
            println!("Could not create Iron Server: {}", e);
        }
    }
}
```

**More Advanced Routing with Middleware and Mount**
```rust
extern crate iron;
extern crate router;
extern crate mount;

use iron::{Iron, Request, Response, IronResult, Chain};
use router::Router;
use mount::Mount;

mod controllers;

fn main() {
    let user_router = router! {
        user_index: get "/" => controllers::user::index,
        user_view: get "/:id" => controllers::user::view,
        user_create: post "/" => controllers::user::create,
        user_update: post "/:id" => {
            let mut chain = Chain::new(controllers::user::update);
            chain.link_before(Authorizer::new(
                IsOwnerOf::<User>::new()
            ));
            chain
        },
    };

    let mut mount = Mount::new();
    mount.mount("/user", user_router);

    let server = Iron::new(mount);
    match server.http("0.0.0.0:3000") {
        Ok(()) => { /*Listening blocks the thread */ }
        Err(e) => {
            println!("Could not create Iron Server: {}", e);
        }
    }
}
```

For an example of how I am currently using this you can check out my
[routing][7] as well as my [authorization implementation][8].

All in all I find that other than the verbosity of using `router::Param` it is
quite enjoyable and flexible to use.

## The ORM, Diesel (with r2d2)

Diesel is a quite ergonomic compiler plugin that allows one to specify rust
structs for given database tables. I've only used it in conjunction with
Postgres so I cannot say anything about other backends.

### The Basic Premise

There are a few steps to get Diesel up and running to be able to use it nicely:

- Create a module that calls `infer_schema!(<DATABASE_URL>)` to generate rust
  code for your database
- Create a Struct to be used as a representation of a given table (examples
  below)
- Use `rd2d` and `r2d2_diesel` to create a connection pool to your Database
- Start using it

It is actually really simple to use once one gets beyond the setup phase (which
at the time I was starting out was not clear at all).

The way I use it is to create a `lazy_static` connection pool one can clone:

```rust
use std::env;

use diesel::pg::PgConnection;
use r2d2_diesel::ConnectionManager;
use r2d2;

lazy_static! {
    static ref CONNECTION: r2d2::Pool<ConnectionManager<PgConnection>> = {
        let database_url = env::var("DATABASE_URL")
            .expect("DATABASE_URL must be set");
        let config = r2d2::Config::default();
        let manager = ConnectionManager::<PgConnection>::new(database_url);
        r2d2::Pool::new(config, manager).expect("Failed to create pool")
    };
}

pub fn connection() -> r2d2::Pool<ConnectionManager<PgConnection>> {
    CONNECTION.clone()
}
```

then in another module I infer the schema:

```rust
infer_schema!(dotenv!("DATABASE_URL"));
```

Then, I define the actual model:

```rust
#[derive(Queryable, Identifiable, Debug)]
pub struct User {
    pub id: i64,
    pub email: String,
    pub password_hash: String,
    pub name: String,
    pub created_at: diesel::data_types::PgTimestamp,
    pub updated_at: diesel::data_types::PgTimestamp,
    profile_image: Option<i64>
}
```

It is important that the fields are in the same order as you have defined them
in your Database. (This is an implementation problem that might be fixed in the
Future so I've read) If you do not it will not compile.

Then, you can start using it:

```rust
pub fn find(uid: i64) -> Result<Option<User>, error::Error> {
    use diesel::prelude::*;
    use models::schema::users::dsl::*; // This import allows you to use the
                                       // table names as a dsl
                                       // It has the same name as your DB table

    users.limit(1).filter(id.eq(uid))
         .get_result::<models::user::User>(&*database::connection().get().unwrap())
         .optional()
}
```
_Note: The `error::Error` type here is a custom one_

What I find nice is that you need to create a second struct to be able to
insert:

```rust
#[insertable_into(users)]
pub struct NewUser<'a> {
    pub email: &'a str,
    pub password_hash: String,
    pub name: &'a str,
}
```

And then:

```rust
pub fn create_from(nu: NewUser) -> Result<i64, error::FurratoriaError> {
    use diesel;
    use diesel::prelude::*;
    use models::schema::users::dsl::*;
    let user_id = try!(diesel::insert(&nu)
                                .into(users)
                                .returning(id)
                                .get_result(&*database::connection().get().unwrap())
                                );
    Ok(user_id)
}
```

For a complete overview of how one could handle users you can check out how I
[do it][9].

## Using Maud for Templates

In the end I wanted to output HTML, since `format!`ing everything seemed like a
bad idea. In the end I went with Maud since I liked the idea of working with a
plugin.

It is also quite intuitive to work with:

```rust
    let name = "Noone";
    let output = html! {
        div.hello "World!"
        p {
            strong { "Hello " (name) }
        }
    };
```

It returns a `RenderOnce<String>` which means it is easy to use and reason with.
The thing I found really fancy is that you can nest those calls! Allowing to me
to do something like that:

```rust
    html! {
        div.row (PreEscaped(Column::new(html! {
            h1 "Hello there!"

            p {
                "Heya there"
            }
        })))
    };
```

Which then in turn allows me transform or wrap the code!

Another neat thing is to build forms:

```rust
    html! {
       (PreEscaped(Form::new(FormMethod::Post, &format!("/submissions/{}", sub.id))
          .with_encoding("multipart/form-data")
          .with_fields(&[
               &Input::new("Image", "sub_image")
                    .with_type("file")
                    .with_errors(errors.as_ref().map(|x| &x.image)),
               &Input::new("Title", "sub_name")
                    .with_value(&sub.title[..])
                    .with_errors(errors.as_ref().map(|x| &x.title)),
               &Textarea::new("Description", "sub_desc")
                    .with_value(&sub.description)
                    .with_errors(None),
               &Select::new("Visibility", "sub_visibility")
                    .add_option("Public","0")
                    .add_option("Private", "2")
                    .with_selected(sub.get_visibility().as_str()),
               &Input::new("", "")
                    .with_value("Update")
                    .with_type("submit")
                    .with_class("btn btn-primary")
          ])))
    }
```

Although `Form` and `Column` are my own types I [built][10].

# Conclusion

All in all I think that it is quite possible to write Web Applications nicely.
While one has to use Rust nightly with my choice of libraries once could use
diesel with syntex on the stable branch and substitute Maud with something like
Horrorshow.

[1]:http://arewewebyet.org
[2]:https://rust-lang.org
[3]:https://www.jimmycuadra.com/posts/the-highs-and-lows-of-rust/
[4]:http://beyermatthias.de/blog/2016/01/31/the-annoyance-of-programming-language-tutorials-or-why-learning-rust-feels-amazing/
[5]:https://medium.com/@robertgrosse/my-experience-rewriting-enjarify-in-rust-723089b406ad
[6]:https://rust.reddit.com
[7]:https://github.com/TheNeikos/furratoria.space/blob/master/src/main.rs#L55
[8]:https://github.com/TheNeikos/furratoria.space/blob/master/src/middleware/authorization/is_owner.rs#L23
[9]:https://github.com/TheNeikos/furratoria.space/blob/master/src/models/user.rs
[10]:https://github.com/TheNeikos/furratoria.space/tree/master/src/views/components
[iron]:http://ironframework.io/
[diesel]:http://diesel.rs/
[bcrypt]:https://github.com/Keats/rust-bcrypt
[log]:https://github.com/rust-lang-nursery/log
[log4rs]:https://github.com/sfackler/log4rs
[maud]:https://github.com/lfairy/maud
