# go-http-response-recorder

[Caddy](https://github.com/caddyserver/caddy) implements an extremely useful struct called `ResponseRecorder`, which allows for processing of HTTP middleware calls without actually sending them to the user.

I want to be able to use this as part of my software stack without bringing all of Caddy's dependencies into my stack.

## Usage

See the helpful text in `NewResponseRecorder` for usage. To import:

```go
package main

import (
    recorder "git.cmcode.dev/cmcode/go-http-response-recorder"
)

```

And then, to use:

```go
rec := recorder.NewResponseRecorder(w, buf, shouldBuffer)
err := next.ServeHTTP(rec, req)
if err != nil {
    return err
}
if !rec.Buffered() {
    return nil
}
// process the buffered response here
```
