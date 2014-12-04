# Logger [![GoDoc](https://godoc.org/github.com/unrolled/webapp/logger?status.svg)](http://godoc.org/github.com/unrolled/webapp/logger)

Logger is an HTTP middleware for Go that logs web requests to an io.Writer (the default being `os.Stdout`. It's a standard net/http [Handler](http://golang.org/pkg/net/http/#Handler), and can be used with many frameworks or directly with Go's net/http package.

## Usage

~~~ go
// main.go
package main

import (
    "log"
    "net/http"

    "github.com/unrolled/webapp/logger"  // or "gopkg.in/unrolled/webapp/logger.v1"
)

var myHandler = http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("hello world"))
})

func main() {
    loggerWithConfigMiddleware := logger.New(logger.Options{
        Prefix: "MySampleWebApp",
        RemoteAddressHeaders: []string{"X-Forwarded-Proto"},
        OutputFlags: log.LstdFlags,
    })

    // loggerWithDefaults := logger.New()

    app := loggerMiddleware.Handler(myHandler)
    http.ListenAndServe("0.0.0.0:3000", app)
}
~~~
A simple GET request to "/info/" will output:
~~~ bash
  [MySampleWebApp] 2014/11/21 14:11:21 (12.34.56.78) "GET /info/ HTTP/1.1" 200 11 12.54µs
~~~
The format is as follows: `[prefix] date time (remote_addres) "METHOD url protocol" status size time`.
*/

Be sure to use the Logger middleware as the very first handler in the chain. This will ensure that your subsequent handlers (like [recovery](https://github.com/unrolled/webapp/recovery)) will always be logged.

### Available Options
Logger comes with a variety of configuration options (Note: these are not the default option values. See the defaults below.):

~~~ go
// ...
l := logger.New(logger.Options{        
    Prefix: "myApp", // Prefix is the outputted keyword in front of the log message. Logger automatically wraps the prefix in square brackets (ie. [myApp] ) unless the `DisableAutoBrackets` is set to true. A blank value will not have brackets added. Default is blank (with no brackets).
    DisableAutoBrackets: false, // DisableAutoBrackets if set to true, will not automatically add the square brackets and space to the prefix. Default is false.
    RemoteAddressHeaders: []string{"X-Forwarded-Proto"}, // RemoteAddressHeaders is slice of header keys that Logger will look at to determine the remote address. Useful when using a proxy like Nginx: `[]string{"X-Forwarded-Proto"}`. Default is blank slice, and thus will use `reqeust.RemoteAddr`.
    Out: os.Stdout, // Out is the destination to which the logged data will be written too. Default is `os.Stdout`.
    OutputFlags log.Ldate|log.Ltime, // OutputFlags defines the logging properties. See http://golang.org/pkg/log/#pkg-constants. To disable all flags, set this to `-1`. Defaults to log.LstdFlags (2009/01/23 01:23:23).
})
// ...
~~~

### Default Options
These are the preset options for Logger:

~~~ go
l := logger.New()

// Is the same as the default configuration options:

l := logger.New(logger.Options{        
    Prefix: "",
    DisableAutoBrackets: false,
    RemoteAddressHeaders: []string{},
    Out: os.Stdout,
    OutputFlags log.LstdFlags,
})
~~~

### Capturing the proper remote address
If your app is behind a load balancer or proxy, the default `Request.RemoteAddr` will likely be wrong. To ensure you're logging the correct IP address, you can set the `RemoteAddressHeaders` option to a list of header names you'd like to use. Logger will iterate over the slice and use the first header value it finds. If it finds none, it will default to the `Request.RemoteAddr`.

~~~ go
package main

import (
    "log"
    "net/http"

    "github.com/unrolled/webapp/logger"  // or "gopkg.in/unrolled/webapp/logger.v1"
)

var myHandler = http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("hello world"))
})

func main() {
    loggerWithConfigMiddleware := logger.New(logger.Options{
        RemoteAddressHeaders: []string{"X-Real-IP", "X-Forwarded-Proto"},
    })

    app := loggerMiddleware.Handler(myHandler)
    http.ListenAndServe("0.0.0.0:3000", app)
}
~~~