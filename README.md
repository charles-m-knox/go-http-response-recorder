# go-http-response-recorder

**The primary use case for this**: Determining the content length of a payload that is being sent to the user so that we can determine if it needs to be compressed.

More specifically, an example of this is when deciding to use gzip compression or not. It is sometimes the case that a payload is too small for compression to help - in fact, it can *increase* the size of the payload after compression in some cases.

[Caddy](https://github.com/caddyserver/caddy) implements an extremely useful interface called `ResponseRecorder`, which allows for processing of HTTP middleware calls without actually sending them to the user.

I want to be able to use this as part of my software stack without bringing all of Caddy's dependencies into my stack.

It ships with a `shouldBuffer` qualifier that allows the recorder to decide whether it should write to the response writer's underlying writable byte buffer or not.

Processing the http handler's buffer without writing it is important, because the easiest alternative (that doesn't work) is chaining together multiple middleware handlers - first a handler that *counts how many bytes* are written, and then the next handler that *actually does e.g. gzip compression*. The problem with this is that you end up with a buffer that contains both the original payload but *also* the gzipped payload. Pretty clunky.

Caddy's implementation is flexible, battle-tested, and only relies on the standard library. I am releasing it here under the same Apache 2.0 license with which it was published (only modification is the package name and one or two mentions of "caddy" in the code) so that it can be used by my own projects (and anyone else).

## Usage

For an extremely detailed breakdown of how do use this package, please see the helpful text in `NewResponseRecorder`.

To import:

```go
package main

import (
    recorder "git.cmcode.dev/cmcode/go-http-response-recorder"
)

```

And then, as a practical example that *always* buffers, as used in [go-gz-middleware](https://git.cmcode.dev/cmcode/go-gz-middleware).

```go
rec := recorder.NewResponseRecorder(w, b, func(status int, header http.Header) bool { return true })
next.ServeHTTP(rec, r)

s := rec.Size()

if s < minLen {
    // No need to gzip, just return the original response
    next.ServeHTTP(w, r)
    return
}
```
