# Getting Started

Let’s create and run our first actix application. We’ll create a new Cargo project
that depends on actix and then run the application.

In previous section we already installed required rust version. Now let's create new cargo projects.

## Ping actor

Let’s write our first actix application! Start by creating a new binary-based
Cargo project and changing into the new directory:

```bash
cargo new actor-ping --bin
cd actor-ping
```

Now, add actix as a dependency of your project by ensuring your Cargo.toml
contains the following:

```toml
[dependencies]
actix = "0.7"
```

Let's create an actor that will accept a `Ping` message and respond with the number of pings processed.

An actor is a type that implements the `Actor` trait:

```rust
# extern crate actix;
use actix::prelude::*;

struct MyActor {
    count: usize,
}

impl Actor for MyActor {
    type Context = Context<Self>;
}

# fn main() {}
```

Each actor has an execution context, for `MyActor` we are going to use `Context<A>`. More information
on actor contexts is available in the next section.

Now we need to define the `Message` that the actor needs to accept. The message can be any type
that implements the `Message` trait.

```rust
# extern crate actix;
use actix::prelude::*;

struct Ping(usize);

impl Message for Ping {
    type Result = usize;
}

# fn main() {}
```

The main purpose of the `Message` trait is to define a result type. The `Ping` message defines
`usize`, which indicates that any actor that can accept a `Ping` message needs to
return `usize` value.

And finally, we need to declare that our actor `MyActor` can accept `Ping` and handle it.
To do this, the actor needs to implement the `Handler<Ping>` trait.

```rust
# extern crate actix;
# use actix::prelude::*;
#
# struct MyActor {
#    count: usize,
# }
# impl Actor for MyActor {
#     type Context = Context<Self>;
# }
#
# struct Ping(usize);
#
# impl Message for Ping {
#    type Result = usize;
# }

impl Handler<Ping> for MyActor {
    type Result = usize;

    fn handle(&mut self, msg: Ping, ctx: &mut Context<Self>) -> Self::Result {
        self.count += msg.0;

        self.count
    }
}

# fn main() {}
```

That's it. Now we just need to start our actor and send a message to it.
The start procedure depends on the actor's context implementation. In our case can we use
`Context<A>` which is tokio/future based. We can start it with `Actor::start()`
or `Actor::create()`. The first is used when the actor instance can be created immediately.
The second method is used in case we need access to the context object before we can create
the actor instance. In case of the `MyActor` actor we can use `start()`.

All communication with actors goes through an address. You can `do_send` a message
without waiting for a response, or `send` to an actor with a specific message.
Both `start()` and `create()` return an address object.

In the following example we are going to create a `MyActor` actor and send one message.

```rust
# extern crate actix;
# extern crate futures;
# use futures::Future;
# use actix::prelude::*;
# struct MyActor {
#    count: usize,
# }
# impl Actor for MyActor {
#     type Context = Context<Self>;
# }
#
# struct Ping(usize);
#
# impl Message for Ping {
#    type Result = usize;
# }
# impl Handler<Ping> for MyActor {
#     type Result = usize;
#
#     fn handle(&mut self, msg: Ping, ctx: &mut Context<Self>) -> Self::Result {
#         self.count += msg.0;
#         self.count
#     }
# }
#
fn main() {
    let system = System::new("test");

    // start new actor
    let addr = MyActor{count: 10}.start();

    // send message and get future for result
    let res = addr.send(Ping(10));

    Arbiter::spawn(
        res.map(|res| {
            # System::current().stop();
            println!("RESULT: {}", res == 20);
        })
        .map_err(|_| ()));

    system.run();
}
```

The Ping example is available in the [examples directory](https://github.com/actix/actix/tree/master/examples/).
