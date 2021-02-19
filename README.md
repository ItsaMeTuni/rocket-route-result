# Rocket RouteResult

## Overview

This crate provides a `RouteResult` type to be used with the [Rocket](https://rocket.rs/) framework.
It is basically a `Result` with more variants that can be returned from your routes. You just
have to return one of the variants (based on the desired status code) and it
will handle sending the correct HTTP status code, payload serialization
and error logging. 

Examples:
```rust
use rocket_route_result::RouteResult;

struct Payload {
    number: i32
}

#[get("/some/resource")]
fn get_some_resource() -> RouteResult<Payload> {
    let payload = Payload {
        number: 42
    };

    // Just return our payload
    RouteResult::Ok(payload)
}

#[post("/restricted/action")]
fn get_restricted_resource() -> RouteResult<()> {
    if check_credentials() {
        // do stuff
    } else {
        // User unauthorized
        RouteResult::Unauthorized
    }   
}
```


It implements `Try`, so you can use the `?` operator inside your routes and,
if a `Result` is an `Err`, a `RouteResult::InternalError` will be returned,
a 500 response will be sent to the client and the error will be logged.

```rust
#[get("/broken/resource")]
fn get_broken_resource() -> RouteResult<Foo> {
    
    // db.get_foo() returns an Err.
    // Because of the ?, this line will return
    // a RouteResult::InternalError with whatever
    // error get_foo returns. The error will also
    // be logged, but it will NOT get sent to the user.
    let payload = db.get_foo()?;

    // This will never execute because get_foo is broken
    RouteResult::Ok(payload)
}
```

## Installation

This package is not on cargo because I'm lazy, you can use it like
```
rocket_route_result = { git = "https://github.com/ItsaMeTuni/rocket-route-result" }
```

If you want to use it with the [okapi](https://github.com/GREsau/okapi)
crate for use with Swagger, just enable the `okapi-0_4` feature.
```
rocket_route_result = { git = "https://github.com/ItsaMeTuni/rocket-route-result", features = ["okapi"] }
```

## The `RouteResult`

Here's the declaration of `RouteResult`, so you can see all of its
variants.
```rust
#[derive(Debug)]
pub enum RouteResult<T>
    where T: Serialize
{
    /// 200 Ok
    /// Sends {0} in the response body.
    Ok(T),

    /// 201 Created
    /// Sends {0} in the response body, sets
    /// the response Location header as {1}.
    Created(T, String),

    /// 404 Not Found
    NotFound,

    /// 400 Bad requests, serializes payload (if present)
    /// and sends as response body. This payload is
    /// used to supply details to the client.
    BadRequest(Option<Box<dyn Serializable>>),

    /// 401 Unauthorized
    Forbidden,

    /// 500 Internal Server Error
    /// Logs the payload, does NOT send it to the
    /// client.
    InternalError(Box<dyn Error>),
}
```