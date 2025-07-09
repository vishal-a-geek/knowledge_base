#go #go-embed #go-directives

### Need for embed
If we just do `template.Parse(filepath.Join("template", "faq.gohtml"))` , the program reads as it starts up, and they are not part of the code or binary that is build and it's something that is read externally. We can verify that by running the build from different directories:
```sh
# if the build was created while from within /lenslocked dir,
lenslocked> go build -o app .

# and run app here itself, it runs successfully
lenslocked> ./app

# but if we run from diff directory
lenslocked> cd ..
projects> ls
lenslocked
projects> ./lenslocked/app
panic: open templates/faq.gohtml: no such file or directory
# this gives error because the filepath.Join() starts from the directory where the binary is being run
```

**Embedding** allows us to take other files that aren't necessarily go source and easily **embed them into our binary** so that we can run the binary from anywhere and it doesn't have to read from the local file system to get those things it can actually read from the binary itself.

### Embedding in go using `go:embed` directive
```go
// templates/fs.go
package templates

import "embed"

//go:embed *
var FS embed.FS
```
The patterns are interpreted relative to the package directory containing the source file. 
For example, here are three ways to embed a file named hello.txt and then print its contents at run time.
Embedding one file into a string:
```go
import _ "embed"

//go:embed hello.txt
var s string
print(s)
```
Embedding one file into a slice of bytes:
```go
import _ "embed"

//go:embed hello.txt
var b []byte
print(string(b))
```
Embedded one or more files into a file system:
```go
import "embed"

//go:embed hello.txt
var f embed.FS
data, _ := f.ReadFile("hello.txt")
print(string(data))
```