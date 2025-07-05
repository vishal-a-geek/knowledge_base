#go #URL-Path #URL-RawPath #EscapedPath #http-URL-RawPath

```go (net/url)
package url

...

type URL struct {
	Scheme string
	Opaque string // encoded opaque data
	User *Userinfo // username and password information
	Host string // host or host:port (see Hostname and Port methods)
	Path string // path (relative paths may omit leading slash)
	RawPath string // encoded path hint (see EscapedPath method)
	OmitHost bool // do not emit empty host (authority)
	ForceQuery bool // append a query ('?') even if RawQuery is empty
	RawQuery string // encoded query values, without '?'
	Fragment string // fragment for references, without '#'
	RawFragment string // encoded fragment hint (see EscapedFragment method)
}
```

```go
package main

import (
	"fmt"
	"log"
	"net/url"
)

func main() {
	u, err := url.Parse("http://example.com/x/y%2Fz")
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("Path:", u.Path)
	fmt.Println("RawPath:", u.RawPath)
	fmt.Println("EscapedPath:", u.EscapedPath())
}
// Output:

// Path: /x/y/z
// RawPath: /x/y%2Fz
// EscapedPath: /x/y%2Fz
```

- `http.URL.RawPath` vs `http.URL.Path`
- RawPath is not always set
	- only set on cases where the actual distinction is to be made
	- i.e. if the actual url was encoded while sending the request
- Path will always have the decoded value of the URL
```go

func pathHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Path: %v\n", r.URL.Path)
	fmt.Fprintf(w, "Raw path: %v\n", r.URL.RawPath)
}

// url: http://localhost:3000/afef?aefe=aefne
// Path: afef (? or anything after it, is not considered part of the path)
// Raw path:  (empty)

// url: http://localhost:3000/afef%3Faefe%3Daefne
// Path: /afef?aefe=aefne (decoded path)
// Raw path: /afef%3Faefe%3Daefne (encoded path)
// the only reason here RawPath was not empty is that we need to know if there is a ? which was encoded or not. if RawPath is present, it means the request url was encoded
```