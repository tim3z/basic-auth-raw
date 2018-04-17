# Raw Basic Authentication

A [Rocket](https://github.com/SergioBenitez/Rocket) library to provide a base for
Basic Authentication over which a concrete authentication mechanism can be built.

This library exports `BasicAuthRaw` which you could directly use on the request handler.
#### Example

```rust
#[get("/secure-path")
fn secret(basic: BasicAuthRaw) -> String {
    format!("Your username is {}", basic.username);
}
```

Or you could build Request Guards on top of it (Recommended).
#### Example

```rust
struct Admin(User);

impl<'a, 'r> FromRequest<'a, 'r> for Admin {
    type Error = ();

    fn from_request(request: &Request) -> Outcome<Self, Self::Error> {
        let basic = BasicAuthRaw::from_request(request)?;
        let user = User::from_db(basic.username, basic.password);
        if user.is_admin {
            Outcome::Success(user);
        } else {
            Outcome::Failure((Status::Unauthorized, ()));
        }
    }
}

#[get("/secure-path")
fn secret(admin: Admin) -> String {
    format!("Your username is {}", admin.user.username);
}
```