# Rocket RouteResult

This crate provides a `RouteResult` type to be used with the [Rocket](https://rocket.rs/) framework.
It is basically a `Result` with a variant for a few of the most common
HTTP status codes.

`RouteResult` implements `Responder` and it will return a proper response
based on its value (return a 404 for `RouteResult::NotFound` or a 200 with
json-serialized payload for `RouteResult::Ok`).

It implements `Try`, so you can use the `?` operator inside your routes and,
if a `Result` is an `Err`, a `RouteResult::InternalError` will be returned,
a 500 response will be sent to the client and the error will be logged.

This package is not on cargo because I'm lazy, you can use it like
```
rocket_route_result = { git = "https://github.com/ItsaMeTuni/rocket-route-result" }
```

If you want to use it with the [okapi](https://github.com/GREsau/okapi)
crate for use with Swagger, just enable the `okapi-0_4` feature.
```
rocket_route_result = { git = "https://github.com/ItsaMeTuni/rocket-route-result", features = ["okapi"] }
```

To use it just return `RouteResult` from your Rocket routes. E.g.
```rust
use rocket_route_result::RouteResult;
#[get("/some/resource")]
fn get_some_resource() -> RouteResult<String> {
    RouteResult::Ok("It works!".to_owned())
}
```

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