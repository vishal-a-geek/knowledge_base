#go #cmd #cmd-pkg

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

### CMD-PKG trend
[[GopherCon - How Do You Structure Your Go Apps#5. Hexagonal Architecture#Folder structure]]
```sh
myapp/
	cmd/
		beer-server/
			main.go
		sample-data/
			main.go
	pkg/
		# services
		adding/
		listing/
		reviewing/
			
		# input
		http/
			rest/
				handler.go
			rpc/
			...
		
		storage/
			json/
			memory/
```
- The cmd-pkg has been a trend in the go community that has emerged overtime, and has been a kind of standard now
- GOAL: Separate binaries from the actual go code
- If we have at least one binary or more, we can put them under CMD package