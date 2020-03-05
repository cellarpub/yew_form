[![License:MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT) [![License:Apache](https://img.shields.io/badge/License-Apache-yellow.svg)](https://opensource.org/licenses/Apache-2.0)
# Yew Form
Bringing MVC to Yew! A set of mildly opinionated Yew component to map and validate a model to a HTML form.

[Live demo](http://chronogears.com/yew-form/)

Supports:
- 2-way Binding to struct (with support for nested structs)
- Validation ([using Keats Validator](https://github.com/Keats/validator))
- Only `String` and `bool` fields are supported presently. More to come

## Usage
Cargo.toml:
```toml
[dependencies]
validator = "0.10"
validator_derive = "0.10"
yew = "0.12"
yew_form = "0.1"
yew_form_derive = "0.1"
```
main.rs:
```rust
#[macro_use]
extern crate validator_derive;
extern crate yew_form;
#[macro_use]
extern crate yew_form_derive;
```

## Example
Consider the following model:
```rust
#[derive(Model, Validate, PartialEq, Clone)]
struct Address {
    #[validate(length(min = 1))]
    street: String,
    #[validate(length(min = 1))]
    city: String,
    #[validate(regex = "PROVINCE_RE")]
    province: String,
    postal_code: String,
    country: String,
}

#[derive(Model, Validate, PartialEq, Clone)]
struct Registration {
    #[validate(length(min = 1))]
    first_name: String,
    #[validate(length(min = 1, message="Enter 2 digit province code"))]
    last_name: String,
    #[validate]
    address: Address,
}
```

The struct can be bound to a Yew form using the following definition:

```rust
struct App {
    form: Form<Registration>,
    ...
}
```

For now, the `Form` needs to be instantiated as follows:
```rust
fn create(_props: Self::Properties, link: ComponentLink<Self>) -> Self {
    // Create model initial state
    let model = Registration {
        first_name: String::from("J-F"),
        last_name: String::from("Bilodeau"),
        address: Address {
            street: String::new(),
            city: String::from("Ottawa"),
            province: String::from("ONT"),
            postal_code: String::from("K2P 0A4"),
            country: String::new(),
        },
    };

    Self {
        form: Form::new(model),
        ...
    }
    ...
```

Fields can then be added to the form as follows:
```html
<Field<Registration> form=&self.form field_name="first_name" oninput=self.link.callback(|_: InputData| AppMessage::Update) />
...
<Field<Registration> form=&self.form field_name="address.street" oninput=self.link.callback(|_: InputData| AppMessage::Update) />
...
<CheckBox<Registration> field_name="accept_terms" form=&self.form />
```
The `Field` component takes care of two way binding between `struct Registration` and the HTML `<input>`

Validation is done automatically when the user edits the form or programmatically.

```rust
if self.form.validate() {
    ...
}
```

Todo/Wish List:
- [ ] Add documentation (In progress)
- [ ] ~~Remove clone requirement from model~~
- [X] Add `derive` for model to remove need for `vec!` of fields
- [X] Make `oninput` optional
- [ ] Make Yew update the view when `Field` is updated
- [ ] Need to add additional HTML attribute to `Field`
- [ ] Remove hard-coded Bootstrap styles
- [ ] Add support for additional types such as `i32`
- [ ] Support `Vec<T>`

## Change Log

### 0.1.3
**BREAKING CHANGES**
- Added `#[derive(Model)]`
- No need to manually pass a vector of fields to `Form::new()`

### 0.1.2
- Added CheckBox

### 0.1.1
- Make `Field::oninput` optional