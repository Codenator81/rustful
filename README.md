Rustful
=======

[![Build Status](https://travis-ci.org/Ogeon/rustful.png?branch=master)](https://travis-ci.org/Ogeon/rustful)

A light web micro-framework for Rust. The main purpose of rustful is
to create a simple, modular and non-intrusive foundation for HTTP
applications. It has a mainly stateless structure, which naturally allows it
to run both as one single server and as multiple instances in a cluster.

Some of the features are:

* Generic response handlers. Just implement the Handler trait and you are done.
* Unified log system to make sure everything is working together.
* Some handy macros reduces risk for typos and makes life easier.
* Pluggable request and response filtering.
* Variables and recursive wildcards in routes.

[Online documentation](http://ogeon.github.io/docs/rustful/master/rustful/index.html).

#Getting Started

##Cargo.toml Entries

Add the following lines to your `Cargo.toml` file:

```toml
[dependencies]
rustful = "0.1"
```

There is also a `nightly` feature for additional compatibility with the future.

##Write Your Server
Here is a simple example of what a simple project could look like. Visit
`http://localhost:8080` or `http://localhost:8080/Olivia` (if your name is
Olivia) to try it.

```rust
//Include macros to be able to use `inser_routes!`.
#[macro_use]
extern crate rustful;

use std::error::Error;

use rustful::{Server, Context, Response, TreeRouter, Handler};
use rustful::Method::Get;

fn say_hello(context: Context, response: Response) {
    //Get the value of the path variable `:person`, from below.
    let person = match context.variables.get("person") {
        Some(name) => &name[..],
        None => "stranger"
    };

    //Use the value of the path variable to say hello.
    if let Err(e) = response.into_writer().send(format!("Hello, {}!", person))  {
        //There is not much we can do now
        context.log.note(&format!("could not send hello: {}", e.description()));
    }
}

//Dodge an ICE, related to functions as handlers.
struct HandlerFn(fn(Context, Response));

impl Handler for HandlerFn {
    fn handle_request(&self, context: Context, response: Response) {
        self.0(context, response);
    }
}

fn main() {
    //Build and run the server.
    let server_result = Server {
        //Turn a port number into an IPV4 host address (0.0.0.0:8080 in this case).
        host: 8080.into(),

        //Create a TreeRouter and fill it with handlers.
        handlers: insert_routes!{
            TreeRouter::new() => {
                //Handle requests for root...
                "/" => Get: HandlerFn(say_hello),

                //...and one level below.
                //`:person` is a path variable and it will be accessible in the handler.
                "/:person" => Get: HandlerFn(say_hello)
            }
        },

        //Use default values for everything else.
        ..Server::default()
    }.run();

    match server_result {
        Ok(_server) => {},
        Err(e) => println!("could not start server: {}", e.description())
    }
}
```

##Contributing
Yes, please! You can always post an issue or (even better) fork the project,
implement your idea or fix the bug you have found and send a pull request.
Just remember to test it when you are done. Here is how:

* Run `cargo test` to run unit, documentation and compile tests.
* Run `cargo run --example example_name` to run an example and make sure it behaves as expected.
