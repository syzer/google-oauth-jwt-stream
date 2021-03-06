google-oauth-jwt-stream
=======================

[![Build Status](https://travis-ci.org/jed/google-oauth-jwt-stream.svg)](https://travis-ci.org/jed/google-oauth-jwt-stream)

An endless supply of fresh [OAuth access tokens][] for use with [Google APIs][].

There are already several libraries that can generate Google service tokens, but I wanted one that:

- Allows keys to be streamed in, so that you don't have to tie your app to a filesystem path or make it async.
- Allows tokens to be streamed out, so that you don't have to detect authorization failure or refresh tokens in your app.
- Makes the most of stable core runtime libraries.

Example
-------

```javascript
import fs from "fs"
import {Token} from "google-oauth-jwt-stream"

let email = "xxx...xxx@developer.gserviceaccount.com"
let key = fs.createReadStream("./key.pem")

let scopes = ["https://spreadsheets.google.com/feeds"]
let options = {ttl: 10 * 1000, pad: 1000} // silly short for demo
let token = new Token(email, key, scopes, options)

token.createReadStream().on("data", console.log)
// { access_token: "...Dg7w", token_type: 'Bearer', expires_in: 3600 }
// { access_token: "...sixQ", token_type: 'Bearer', expires_in: 3600 }
// { access_token: "...1ftw", token_type: 'Bearer', expires_in: 3600 }
// ...
```

Installation
------------

    npm install google-oauth-jwt-stream

Setup
-----

See the [SETUP](https://github.com/jed/google-oauth-jwt-stream/blob/master/SETUP.md) file.

API
---

### import Token from "google-oauth-jwt-stream"
### let token = Token(email, key, scopes, [options])

Returns a token given the following parameters:

- `email`: The email address assigned to the service from the [Google APIs][] console.
- `key`: The decoded key corresponding to the above service. This can either be as a `String` or `Readable` stream, such as from the filesystem.
- `scopes`: A list of scope URLs pertaining to the accessed services
- `options.ttl`: The time to live for generated tokens, in ms.
- `options.pad`: The amount of time before expiration at which the next token should be fetched.

### let stream = token.createReadStream()

Returns a [readable stream][] of tokens. Note that since this requires a `setTimeout` to keep the stream open, your process will not terminate implicitly.

### token.fetch(callback)

Executes `callback` with `(err, token)`, and caches tokens so that all subsequent calls return the same token. Token refresh is performed automatically.

[readable stream]: https://iojs.org/api/stream.html#stream_class_stream_readable
[OAuth access tokens]: http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html
[Google APIs]: https://console.developers.google.com
[steps 1 through 6]: http://www.nczonline.net/blog/2014/03/04/accessing-google-spreadsheets-from-node-js/
