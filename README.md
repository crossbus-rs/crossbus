<div align="center">
  <h1>CrossBus</h1>
  <p>
    <strong>A Platform-Less, Runtime-Less Actor Computing Model</strong>
  </p>

  <p>

[![API Document](https://img.shields.io/docsrs/crossbus/latest)](https://docs.rs/crossbus)
[![crates.io](https://img.shields.io/crates/v/crossbus.svg)](https://crates.io/crates/crossbus)

  </p>
</div>

## Overview
[CrossBus](https://github.com/hominee/crossbus) is implementation of
[Actor Computing Model](https://en.wikipedia.org/wiki/Actor_model), 
with the concept that 

- **Runtime-less**

  crossbus neither provide runtime for app execution 
  nor access the system interface abstraction. 

  runtime-located features like `concurrency` or `I/O`
  are up to implementor. 

- **Bare-Metal Compatible**

  no system libraries, no libc, and just a few upstream libraries
  enbale you run rust code on bare metal.

  the [examples](https://github.com/hominee/crossbus/tree/master/examples/no-std)
  folder contains some demos to use crossbus. 

- **Platform-less by Runtime-less** 

  take the advantage of runtime-less, crossbus is able to run 
  across many platforms. 

- **Future-oriented Routine and Events**

  the futures way to execute task is retained even 
  without runtime with rust. crossbus define a set of types 
  and traits that follow style during execution.

- **Real-time Execution Control**

  crossbus provides a handle with which more routines and events
  can be fulfilled for each spawned future.

**Currently crossbus is in its alpha version, all APIs and archtecture 
is not stable yet.**

## Documentation

- [API Document](https://docs.rs/crossbus)

- [Manual Book Coming Soon]()

## Feature Flags 
To reduce code redundancy and speed up compilation, crossbus use feature flag to mark the necessary modules/functions, Currently here are some supported Features:

- `core`/`no-std`: the default feature, bare-metal compatible 
- `std`: rust std library dependent
- `derive`: enable `#[derive(Message)]` and `#[crossbus::main(...)]`
- `log`: enable logging out states and infos during runtime
- `time`: enable Actor to known time when blocking/delaying some duration
- `rt`: enable use some common convenient runtime 
- `tokio`: convenience to use bare tokio-based runtime 
- `async-std`: convenience to use async-std-based runtime  
- `wasm32`: convenience to use wasm-bindgen-futures-based runtime 
- `unstable`: marker for unstable feature
- `force-poll`: it is `unstable`, `std`/`time`-dependent to periodically wakeup future polling

## How to Use 
First of all, add `crossbus` to `Cargo.toml` of project
```toml 
[dependencies]
crossbus = "0.0.1-a"
```
#### Types and Imports

define Some types and methods according to your business logic.

let's say a summer to add some number up, 
Okay, then the message should be numbers
the summer should know the result after adding.
```rust
use crossbus::prelude::*;

struct Num(uisze);
impl Message for Num {}

struct CrossBus {
    sum: isize
}
```

#### Actor Implementation

the actor should be created before using it

we tell crossbus how to create it

```rust 
impl Actor for CrossBus {
    type Message = Num;
...

    fn create(ctx: &mut Context<Self>) -> Self {
        Self { sum: 0, }
    }
...

}

```

#### Message Action & Reaction

Okay, How the Summer respond when the message comes?
we should tell crossbus.

```rust 
...
impl Actor for CrossBus {
    type Message = Num;
...

    fn action(&mut self, msg: Self::Message, ctx: &mut Context<Self>) {
        self.sum += msg.0;
    }
...

}

```
So far so good, but how to obtain the final result 

it can be available when the actor process all messages 
and stopped, right? 

```rust 
...
impl Actor for CrossBus {
    type Message = Num;
...

    fn stopped(&mut self, _: &mut Context<Self>) {
        println!("final sum: {}", self.sum);
    }
...

}

```

Done. Congratulation! You just knew the basic routine to use crossbus.
the Full code see below.

## Example
Here presents a simple demo to sum numbers.

For more examples, you can go to the [examples](https://github.com/hominee/crossbus/examples) folder
and clone the repo and run them locally.

```rust 
use crossbus::prelude::*;

struct Num(uisze);
impl Message for Num {}

struct CrossBus {
    sum: isize
}
impl Actor for CrossBus {
    type Message = Num;

    fn create(ctx: &mut Context<Self>) -> Self {
        Self { sum: 0, }
    }

    fn action(&mut self, msg: Self::Message, ctx: &mut Context<Self>) {
        self.sum += msg.0;
    }

    fn stopped(&mut self, _: &mut Context<Self>) {
        assert_eq!(self.sum, 7);
        println!("final sum: {}", self.sum);
    }

}

#[main(runtime = tokio)]
async fn main() {
    let (addr, _) = CrossBus::start();
    let sender = addr.sender();
    sender.send(Num(1)).unwrap();
    sender.send(Num(2)).unwrap();
    sender.send(Num(4)).unwrap();
}
```

## Contributing

Feel Free to Contribute! All issues / bugs-report / feature-request / etc
are welcome!

## References 

- [Actix](https://actix.rs)
- [Yew](https://yew.rs)
- [Actor Model](https://en.wikipedia.org/wiki/Actor_model)

