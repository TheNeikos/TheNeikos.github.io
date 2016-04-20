---
layout: post
date: 2016-04-19 09:37:03
title: On our way to a Minecraft Clone
tags: rust minecraft
---

It's been somewhat of a goal of mine to create a semi-functional 'Minecraft'
clone. I usually get stuck somewhere, and today I want to document what I do and
how far I get, simply for anteriority purposes. *(Why Minecraft? It is one of
the rare games that were actually **fun** to play, and still are today)*

## Starting out

We will be using Piston, more specifically I have looked for crates that might
fit and found theses:

- [Piston Window](https://github.com/PistonDevelopers/piston_window), I will be
  using my local branch until [this
  PR](https://github.com/PistonDevelopers/piston_window/pull/130) gets merged as
  I prefer that style.
- [Camera Controllers](https://github.com/PistonDevelopers/camera_controllers)
- [Image](https://github.com/PistonDevelopers/image) since we will have textures
  at some point

- Perhaps [CGMath](https://github.com/bjz/cgmath) at some point?


Alright, let's get started:

```sh
$ cargo new --bin rustcraft
```

We will call it rustcraft for now, we can always change it later on.

Okay, let's `cd` into the directory and add some dependencies:

```sh
$ cargo add piston_window
$ cargo add camera_controllers
$ cargo add image
```

Let's also directly download/build everything so that later on it will be
quicker. (And we won't need internet until we update either Rust or the
dependencies)

```sh
$ cargo build
```

Now let's get started, our first objective is to get the camera setup and to
draw a simple cube!


I will be using (neo)vim as an editor, but of course you can use whatever you
want.

```sh
$ vim src/main.rs
```

There is still the default Hello World, we can leave it for now.

However, we will have to tell the Rust Compiler to include `piston_window` and
`camera_controllers`. Put it right at the top of `main.rs`.

```rust
extern crate piston_window;
extern crate camera_controllers;
```

Now let's replace the default *Hello World* with one made for **Piston Window**.

```rust
use piston_window::*;

fn main() {
    let mut window : PistonWindow = WindowSettings::new("Hello Piston!",
                                                        [640, 480])
        .exit_on_esc(true).build().unwrap();
    let mut events : WindowEvents = window.events();

    while let Some(e) = events.next(&mut window) {
        window.draw_2d(&e, |_c, g| {
            clear([0.3, 0.85, 0.2, 1.0], g); // A lovely green
        });
    }
}
```

If you now run `cargo run` you should have a little window with a nice green in
it. Success! Let's get to drawing a cube.


## Getting a cube

So a cube is nothing more than 6 squares appropriately positioned. Keeping that
in mind we can construct an Array of *Vertices* (sg. Vertex) in which to hold
positioning information. However, we don't have a `Vertex` *struct*ure yet.
Let's change that.

PistonWindow uses [gfx](https://github.com/gfx-rs/gfx) to handle the whole
drawing stuff. So we need to add that:

Exit your editor and run:

```sh
$ cargo add gfx --vers=0.10
```

We chose `0.10` because it is what PistonWindow depends on.

Let's also `cargo build` right away. You should notice that it did not build it
again; due to PistonWindow we already had it built.

Time to add it to our `main.rs`:

```sh
$ vim src/main.rs
```

*I want to note here that for the rest of this part I am using the
[piston-examples](https://github.com/PistonDevelopers/piston-examples/blob/master/src/cube.rs)
repo as a reference. After all, no need to rewrite code that has already been
written.*

Same thing as earlier, but this time we also would like to have the macros they
define! So we put `#[macro_use]` infront of the `extern`; like this:

```rust
extern crate camera_controllers; // From earlier
#[macro_use] extern crate gfx;
```

Now we have access to two macros we will need:

- [`gfx_vertex_struct!`](http://docs.piston.rs/piston_window/gfx/macro.gfx_vertex_struct!.html)
  which will allow us define our Vertex.
- [`gfx_pipeline!`](http://docs.piston.rs/piston_window/gfx/macro.gfx_pipeline!.html)
  which will allow us to create our render pipeline.

Let's put them to use:

```rust
gfx_vertex_struct!( Vertex {
    a_pos: [i8; 4] = "a_pos",
    a_tex_coord: [i8; 2] = "a_tex_coord",
});

impl Vertex {
    fn new(pos: [i8; 3], tc: [i8; 2]) -> Vertex {
        Vertex {
            a_pos: [pos[0], pos[1], pos[2], 1],
            a_tex_coord: tc,
        }
    }
}

gfx_pipeline!( pipe {
    vbuf: gfx::VertexBuffer<Vertex> = (),
    u_model_view_proj: gfx::Global<[[f32; 4]; 4]> = "u_model_view_proj",
    t_color: gfx::TextureSampler<[f32; 4]> = "t_color",
    out_color: gfx::RenderTarget<gfx::format::Srgba8> = "o_color",
    out_depth: gfx::DepthTarget<gfx::format::DepthStencil> =
        gfx::preset::depth::LESS_EQUAL_WRITE,
});


fn main() { // From earlier
```

Check out [this blog post](https://gfx-rs.github.io/2016/01/22/pso.html) to get
an understanding of the Pipeline macro. There is also documentation in the code.


Now that we have a way to represent the cube, let's do just that and use the
Vector from `piston-example`. It should be fairly straightforward to understand.

```rust
    let vertex_data = vec![
        //top (0, 0, 1)
        Vertex::new([-1, -1,  1], [0, 0]),
        Vertex::new([ 1, -1,  1], [1, 0]),
        Vertex::new([ 1,  1,  1], [1, 1]),
        Vertex::new([-1,  1,  1], [0, 1]),
        //bottom (0, 0, -1)
        Vertex::new([ 1,  1, -1], [0, 0]),
        Vertex::new([-1,  1, -1], [1, 0]),
        Vertex::new([-1, -1, -1], [1, 1]),
        Vertex::new([ 1, -1, -1], [0, 1]),
        //right (1, 0, 0)
        Vertex::new([ 1, -1, -1], [0, 0]),
        Vertex::new([ 1,  1, -1], [1, 0]),
        Vertex::new([ 1,  1,  1], [1, 1]),
        Vertex::new([ 1, -1,  1], [0, 1]),
        //left (-1, 0, 0)
        Vertex::new([-1,  1,  1], [0, 0]),
        Vertex::new([-1, -1,  1], [1, 0]),
        Vertex::new([-1, -1, -1], [1, 1]),
        Vertex::new([-1,  1, -1], [0, 1]),
        //front (0, 1, 0)
        Vertex::new([-1,  1, -1], [0, 0]),
        Vertex::new([ 1,  1, -1], [1, 0]),
        Vertex::new([ 1,  1,  1], [1, 1]),
        Vertex::new([-1,  1,  1], [0, 1]),
        //back (0, -1, 0)
        Vertex::new([ 1, -1,  1], [0, 0]),
        Vertex::new([-1, -1,  1], [1, 0]),
        Vertex::new([-1, -1, -1], [1, 1]),
        Vertex::new([ 1, -1, -1], [0, 1]),
    ];

    let index_data: &[u8] = &[
         0,  1,  2,  2,  3,  0, // top
         4,  6,  5,  6,  4,  7, // bottom
         8,  9, 10, 10, 11,  8, // right
        12, 14, 13, 14, 12, 16, // left
        16, 18, 17, 18, 16, 19, // front
        20, 21, 22, 22, 23, 20, // back
    ];

    let (vbuf, slice) = window.factory.create_vertex_buffer_indexed(&vertex_data,
        index_data);
```

In the last line we then use our window (which has the OpenGl context) and
create an indexed vertex buffer.

Let's try compiling it!

Oops, looks like we forgot to add the trait for it, let's add those.

```rust
use gfx::traits::*;
```

Put that line just below our earlier `use` and it should compile with just two
`unused variable` warnings. Let's go use them!


## Texturing a Cube

Now that we have an idea where the different points of our cube should look
like, it's time to tell the game what they look like.

I've decided to go with a simple image of stone for now.

![stone.png](/public/rustcraft/stone.png)

Okay, let's see what we need to draw this:

