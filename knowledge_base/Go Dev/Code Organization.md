#go #code-organization #mvc

## Flat Structure
All code in single package
```bash
myapp/
	gallery_store.go
	gallery_handler.go
	gallery_templates.go
	user_store.go
	user_handler.go
	user_templates.go
	router.go
	main.go
```

## Separation of concerns - Model-View-Controller (MVC)
- `models` => data, logic, rules; usually database
- `views` => rendering things; usually html
- `controllers` => connects it all. 
	- accepts user input, 
	- passes that to models to do stuff, 
	- then passes data to views to render things; 
	- usually handlers
```bash
myapp/
	controllers/
		user_handler.go
		gallery_handler.go
		...
	view/
		user_templates.go
		gallery_templates.go
		...
	models/
		user_store.go
		gallery_store.go
		...
	router.go
	main.go
```

## Dependency Based Structure
Structured based on dependencies, but with a common set of interfaces and types
```bash
myapp/
	user.go # defines a User type which is not locked into any specific data store
	user_store.go # defines a UserStore interface
	
	psql/
		user_store.go # implements UserStore interface with PostgreSQL specific implementation
```

## Domain Driven Design
#ddd
## Onion Architecture
## ...  

