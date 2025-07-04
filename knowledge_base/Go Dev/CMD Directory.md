#go #cmd

- Abbreviated for `command`
- Typically pattern found:
	- nested folders within `cmd` and each folder has it's own `main` package
```
/cmd/
/cmd/folder-1/
/cmd/folder-2/
/cmd/folder-3/
/main.go
```
- General idea about this:
	- it's not that there's one program which has multiple main packages
	- what's actually happening is
		- every folder inside the `cmd` directory is a separate program that can be built (separate binary which can be run)
		- each folder has it's own `main` package that then starts the whole program
	- So, we're separating all those little programs that  all might share the rest of the source code in the project  