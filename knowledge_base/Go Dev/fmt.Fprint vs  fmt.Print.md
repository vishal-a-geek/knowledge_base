#go #fmt #fmt-Fprint #fmt-Println

- both are roughly the same
```
fmt.Println("Starting the server on :3000 ...")
fmt.Fprintln(os.Stdout, "Starting the server on :3000 ...") // we mention here where to write to (os.Stdout)
```
- in fact:
```
package fmt

// Println formats using the default formats for its operands and writes to standard output.
// Spaces are always added between operands and a newline is appended.
// It returns the number of bytes written and any write error encountered.
func Println(a ...any) (n int, err error) {
	return Fprintln(os.Stdout, a...)
}
```
