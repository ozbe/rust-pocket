# rust-pocket <a href="https://travis-ci.org/kstep/rust-pocket"><img src="https://img.shields.io/travis/kstep/rust-pocket.png?style=flat-square" /></a> <a href="https://crates.io/crates/pocket"><img src="https://img.shields.io/crates/d/pocket.png?style=flat-square" /></a> <a href="https://crates.io/crates/pocket"><img src="https://img.shields.io/crates/v/pocket.png?style=flat-square" /></a>

[Pocket API](http://getpocket.com/developer/docs/overview) bindings
(http://getpocket.com)

API is very easy, actually. The most complex code is for authorization.
You will need a consumer key and an access token in order to use the
API.

A consumer_key can be obtained by creating an app at the
[My Applications](http://getpocket.com/developer/apps/) page. An access
token is obtained by walking through
[OAuth authentication workflow](http://getpocket.com/developer/docs/authentication).

The OAuth workflow is implemented with a pair of methods in this
implementation:

```rust
extern crate pocket;

use pocket::Pocket;

fn authenticate() {
  let auth_requester = Pocket::auth("YOUR-CONSUMER-KEY-HERE");
  let authorizer = auth_requester.request("rustapi:finishauth").unwrap();
  println!("Follow auth URL to provide access and press enter when finished: {}", authorizer.url());
  let _ = io::stdin().read_line(&mut String::new());
  
  let user = authorizer.authorize().unwrap;
}
```

So you
1. Initiate auth with ` Pocket::auth()`
2. Generate OAuth access request URL with `auth_requester.request()`,
3. Let the user follow the URL and confirm app access
4. Call `authorizer.authorize()` and either get an error, or the
   username and access token of user just authorized.

You can then convert that user into an `Pocket` instance if you choose

```rust
let pocket = user.pocket();
```

I recommend storing the access token after you get it, so you don't have
to repeat this workflow again next time. The access token can be
obtained via `user.access_token` field. Store it somewhere and use it to
construct a `Pocket` instance:

```rust
let pocket = Pocket::new("YOUR-CONSUMER-KEY-HERE", "YOUR-STORED-ACCESS-TOKEN");
```

A `Pocket` instance allows you to add, modify and retrieve items to and
from your pocket.

To add an item, use the `Pocket::add()`, `Pocket::push()`, or
`Pocket::send()` method:

```rust
// Quick add by URL only
let added_item = pocket.push("http://example.com").unwrap();

// Add with all meta-info provided (title, tags, tweet id)
let added_item = pocket.add("https://example.com", Some("Example title"), Some("example-tag"), Some("example_tweet_id")).unwrap();

// Add with one or more actions
use hyper::client::IntoUrl;

let added_item = pocket.send(&PocketSendRequest { 
    actions: &[
        &PocketSendAction::Add {
            item_id: None,
            ref_id: None,
            tags: Some("example-tag".to_string()),
            time: None,
            title: Some("Example title".to_string()), 
            url: Some("https://example.com".into_url().unwrap()), 
        }
    ]
};
```

To query your pocket, use `Pocket::filter()` and `Pocket::get()`
methods:

```rust
let items = {
    let mut f = pocket.filter();
    f.complete(); // complete data
    f.archived(); // archived items only
    f.videos();   // videos only
    f.offset(10); // items 10-20
    f.count(10);
    f.sort_by_title(); // sorted by title
    // There are other methods, see `PocketGetRequest` struct for details

    pocket.get(&f); // get items
};
```

To modify one or multiple items or tags at a time, use `Pocket::send()`

```rust
let item_id = 1583845180185;
let results = pocket.send(&PocketSendRequest {
    actions: &[
        &PocketSendAction::Archive { item_id, time: None },
        &PocketSendAction::TagsAdd { item_id, tags: "one,two".to_string(), time: None },
        &PocketSendAction::TagRename { old_tag: "one".to_string(), new_tag: "1".to_string(), time: None },
    ]
})
```

## License

Licensed under either of

* Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or
  http://www.apache.org/licenses/LICENSE-2.0)
* MIT license ([LICENSE-MIT](LICENSE-MIT) or
  http://opensource.org/licenses/MIT)

at your option.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally
submitted for inclusion in the work by you, as defined in the Apache-2.0
license, shall be dual licensed as above, without any additional terms
or conditions.
