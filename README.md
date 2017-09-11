# Talkback

Talkback is a standalone HTTP proxy that can record and playback requests.
Very useful for integration tests environments or mocking HTTP servers.
Heavily inspired by [flickr/yakbak](https://github.com/flickr/yakbak).

## Installation

```
npm install talkback
```

## Usage

Talkback is pretty easy to setup.
Define which host it will be proxying, which port it should listen to and where to find and save tapes.

When a request arrives to talkback, it will try to match it against an existing tape and return the tape's response.
If no tape matches the request, it will forward it to the origin host save the tape to disk for future uses and return the response.

```javascript
const talkback = require("talkback");

const opts = {
  host: "https://api.myapp.com/foo",
  port: 5544,
  path: "./my-tapes"
};
const server = talkback(opts);
server.start(() => console.log("Talkback Started");
server.close();
```

### talback(opts)
Returns an unstarted talkback server instance.

Options:
* `host`: (String) Where to proxy unknown request
* `port`: (Number) Talkback port. Default 8080
* `path`: (String) Path where to load and save tapes. Default `./tapes/`
* `ignoreHeaders`: (Array(String)) List of headers to ignore when matching tapes. Useful when having dynamic headers like cookies or correlation ids. Default `[]`
* `record`: (Boolean) Wether talkback should proxy and record unknown requests or fail fast and return 404. Default `true`

### start([callback])
Starts the HTTP server and if provided calls `callback` after the server has succesfully started.

### stop()
Stops the HTTP server.

## Tapes
Tapes are simple JSON files that are loaded from the `path` directory at startup.

#### Format
All tapes have the following 3 properties:
* **meta**: Stores metadata about the tape.
* **req**: Request object. Used to match incoming requests against the tape.
* **res**: Response object. The HTTP response that will be returned in case the tape matches a request.

Both `req` and `res` objects can be freely edited. Since tapes are only loaded on startup, any changes to a tape requires a server restart to be applied.

#### File Name
New tapes will be created under the `path` directory with the name `unnamed-n.json`, where `n` is the tape number.
Tapes can be renamed at will, for example to give some meaning to the scenario the tape represents.

#### Request and Response body
If the content type of the request or response is considered _human readable_ and _uncompressed_, the body will be saved in plain text.
Otherwise, the body will be saved as a Base64 string, allowing to save binary content.

## No recording
Talkback proxying and recording can be disabled throguh the `record` option.
When recording is disabled and an unknown requests arrives, talkback will just log an error message, and return a 404 response without proxying the request to `host`.

It is recommended to disable recording when using talkback for test running. This way, there are no side-effects and broken tests fail faster.

# Licence
MIT