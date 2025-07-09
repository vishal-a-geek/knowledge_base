#go #code-organization 

[Source](https://youtu.be/oL6JBUk6tj0?si=9NLlq6xZKisrIXEd)
[Github Repo](https://github.com/katzien/go-structure-examples)
### A Beer Reviewing Service: Requirement Specs
```
A Beer Reviewing Service

- Users can add a beer.
- Users can add a review for a beer.
- Users can list all beers.
- Users can list all reviews for a given beer.
- Option to store data either in memory or in a JSON file.
- Ability to add some sample data.
- For simplicity skip deleting, updating and some error handling and tests.
```

### 1. Flat Structure
#flat-structure

```bash
myapp/
	data.go # for sample data
	handlers.go # HTTP handlers
	main.go # starting point of app/server
	model.go # define Beer and Review models
	storage.go # define storage functionality
	storage_mem.go # in-mem storage implementation
	storage_json.go # json-file storage implementation
```
- Advantages
	- Easy to navigate, reason about
	- Great starting point if don't know where to start
	- Works well for smaller apps and libraries, 10-20 business operations
	- Goes well with idiomatic go - keep it nice and simple, not over-complicating if not needed
	- Removes any chances of circular dependencies, b/c everything is in the `package main` 
- Disadvantages
	- Everything is in global state. No option to black-box or separate out
	- Everything can be accessed and modified by everything
	- Looking at the app structure or file names, can't tell what the app actually does

Break the flat structure into few smaller packages
### 2. Group by function (Layered architecture)
#layered-architecture #circular-dependencies

```bash
# layers
myapp/
	presentation / user interface
	business logic
	external dependencies / infrastructure (db)

# structure
myapp/
	data.go
	main.go
	handlers/
		beers.go
		reviews.go
	models/
		beer.go
		review.go
		storage.go
	storage/
		json.go
		memory.go
```
- Adv
	- Easy to decide what goes where, at least until a certain
	- Discourages to use global state, encourages to use things in the relevant packages
- Dis-adv
	- If a variable is shared between layers, where should it be defined or should it be duplicated across layers?
	- Where to initialise everything? 
		- does `main` initialise `storage` and then passes to each `model`
		- or leave it up to each `model` to initialise its own `storage`
	- One file or Separate files?
		- Do we have just one file `models/beer.go` for everything related to `Beer` 
		- or separate `models/single_beer.go` model file for `SingleBeer` and a separate `models/multiple_beer.go` file for `MultipleBeer`
	- If we have just one definition of `type Beer struct`, then does everything (every functionality) shares that one definition?
		- Example: Adding a new `Beer` to our system VS Fetching all `Beers` in our system
		- Thus, the code below doesn't really guide us much as a programmer for which fields should be used, we would have to reason about it or work it out ourselves.
```go
package models

type Beer struct {
	BeerId string `column:"beer_id"`
	Name string
	Age int
	// ...
}

func (b *Beer) Add() {
	// but here, we would feel that we shouldn't need to specify the BeerId and the system should handle the BeerId generation for us
	// but the Beer struct has BeerId field
	// so as a user, we don't know if we need to specify it or not, or should we have the BeerId field in the struct or not?
}

func (b *Beer) GetAll() {
	// here, we want all beers in our system with their BeerId. Thus we need the BeerId to be present in the struct.
}
```
- Major disadvantage: Won't compile due to **circular dependencies**
	- `storage` package uses `models` package for the definition of `Beer` 
	- and the `models` package uses `storage` package to persist into or query the data store

### 3. Group by module (Modular architecture)
#modular-architecture 

```bash
myapp/
	main.go
	beers/
		beer.go # model
		handler.go # handler
	reviews/
		review.go # model
		handler.go # handler
	storage/
		data.go
		json.go
		memory.go
		storage.go
```
- Here things are grouped logically - probably the only advantage of this app structure
- Disadvantage
	- Still hard to decide if beer related reviews should go into `package beers` or `package reviews`
	- file naming got worse, because we've got `beers/beer.go`

### 4. Group by context (Domain Driven Development)
#ddd

- Quite a natural way to think about our design
- Absolutely not about where our files go, but it can guide our code structure and can help us find the right one really way
- Makes us think about the domain we're dealing with
- Makes us think a bout all of the business logic in our app, before even writing a single line of code
- We define 
	- our Bounded context(s)
		- Limit/boundary around our model(s)
		- E.g.: User in an app
			- In **sale's context**, user might have properties like Lead time, cost of acquisition
			- In **customer support's context**, user might have completely different properties attached like response time, # tickets submitted
		- Bounded context helps us decide what has to stay consistent within a boundary and what can change independently
			- E.g.: if we want to have a diff property to user from sale's POV like # days took to onboard user, we can change that without affecting **customer support's context**
	- the models within each context
	- and ubiquitous language
	- define and categorise building blocks of system: 
		- Entity
		- Value Object
		- Domain Event
		- Aggregate
		- Service
		- Repository
		- Factory
```sh
# DDD

Context: beer tasting, beer ingestion
# Beer tasting context: 
	# For a beer connoisseur who wants to give reviews for a beer
	# For a beer connoisseur, to drink and review the beer, the beer has a specific meaning which is defined by what the beer connoisseur cares about, like
		# - Alchocol context
		# - Flavour notes
	# But the beer connoisseur doesn't really care about things like the barcode, 
		# but however if we had another different context in our app/system like a beer manufacturer, the barcode might very much be an important part of the beer

Language: beer, review, storage, ... 
# This might seem obvious but helps stay consistent with namings not only while developing but also while communicating with fellow team and stakeholders

Entities: Beer, Review
# Can be an abstract concepts which then can have concrete instances
# Eg: Customer, Order, Blog Post, Beer are entities which can be instantiated

Value Objects: Brewery, Author
# Similar to Entity but it represents a value which on it's own isn't really an entity but a part of it
# E.g Barcode Value object is a part of Beer entity
# ~ properties to entities

Aggregates: BeerReview
# Combines entities

Services: 
	Adding: BeerAddingService, ReviewAddingService, 
	Listing: BeerListingService, ReviewListingService
# Stateless operations that cannot go beyond particular entity's natural responsibility

Events: 
	Acknowledgments: Beer added, Review added
	Errors: Beer already exists, Beer not found
# They capture a memory of something interesting that happened in our system that can affect the state of our system/application

Repository:
	Beer Repository
	Review Repository
# Provides a facade over a backing store like a database
# sits in between our domain logic and actual store or database
```
- Now the package names communicate what they do
```sh
# DDD
# removes circular dependencies
myapp/
	main.go # http server 
	
	# services (use storage)
	adding/
		endpoint.go
		service.go
	listing/
		endpoint.go
		service.go
	reviewing/
		endpoint.go
		service.go
	
	# models (entities & repositories) (don't care about storage directly)
	beers/
		beer.go
		sample_beers.go
	reviews/
		review.go
		sample_reviews.go
	
	
	storage/
		json.go
		memory.go
		type.go
		
```
- Dis-advantages
	- So far, we only had one entry point to our app i.e. `main.go` and if we want to do anything else other than serving HTTP traffic, we would have to change the `main.go` itself to say include adding of sample data
	- We cannot run our system separately other than via HTTP
	- The input to our internal system (packages and domain) has only been via HTTP in the `main.go`
	- So in order to have anything else like adding sample data, we have to add it in the `main.go` alone, no other option because we only have one `func main`.
	- What if we want to add a completely different entry point to the application, like a command-line version for the ingestion of beers and reviews, rather than accepting HTTP requests to add stuff. It would be tricky to add something like this to this existing structure as it currently only has one `func main`
	- The **Hexagonal Architecture** can help in such cases

### 5. Hexagonal Architecture
#hexagonal-architecture
![[Hexagonal-Architecture 1.png]]![[Hexagonal-Architecture 2.png]]
- We gradually start distinguishing between 
	- the parts of the system which form a core domain (business logic)
	- and all the external dependencies are just the implementation details like (diff sides of hexagon)
		- external apis, 
		- databases, 
		- mail clients, 
		- cloud, 
		- ... anything that our app interacts with
- This hexagonal architecture gives the ability to change one part of the app without having to much else
	- like changing the type of interface to talk to our system/app, like from HTTP to CLI, shouldn't really mean to re-write the entire thing. For e.g.
		- If we want to swap out databases from mysql to no-sql database like mongo db, that shouldn't require to touch any of the business logic because the core business logic doesn't care where the data gets stored or fetched from
- The Hex model recognises such needs.
- For the hex model, the inputs to and outputs from the system are treated on the same level. It doesn't care whether something is an input or an output. It's just an external interface.
- Key rule here is that the dependencies are only allowed to point inwards.
	- The outer layers can reach out inward or the domain as much as they like, 
		- they can use the definitions of entities and so on defined in the domain
	- but the domain absolutely cannot reference out to any of the outer layers.
- Thus, there's heavy use of interfaces and inversion of control
- Interfaces at between each layer
- The **Core Domain** defines the business logic in fairly abstract terms without bothering about the **implementation details**
- The rest of the application will implement those interfaces to satisfy the domain's needs
- Then the internal layers would define their own interfaces, like Application Layer would define interface for storage because it needs storage to perform these functions
- The outer layers would have to satisfy and implement that interface with a particular storage type
- As long as the interface required by the inner layer is satisfied by something, the outer layer can use that thing and swap them in and out with different such things easily
#### Folder structure
#cmd #cmd-pkg

```sh
# Hexagonal Architecture

myapp/
	cmd/
		beer-server/
			main.go
		sample-data/
			main.go
	pkg/
		# services
		adding/
			# each service has it's own Beer struct, thus we can get rid of properties or add properties based on the service/usecase
			beer.go # doesn't have BeerId field
			service.go
			sample_beer.go
		listing/
			beer.go # has BeerId field
			service.go
		reviewing/
			review.go
			sample_review.go
			service.go
			
		# input
		http/
			rest/
				handler.go
			rpc/
			soap/
			...
		
		
		storage/
			json/
				beer.go
				repository.go
				review.go
			memory/
				beer.go
				repository.go
				review.go
```
- The CMD-PKG trend helps to have multiple entry points to the same go program
- The main package is now split into two
	- beer-server: The HTTP server
	- sample-data: adding sample data
- Thus, now they are completely independent and can run one without running the other
- Thus, if our project has any other non-go file, like Dockerfile or documentation or anything else
- It's just like a nicer and cleaner way to put all the go packages under `/pkg` directory
- If we want to add any other client like command line (CLI) version of the application/system
	- check in another subdirectory under `/cmd` and create a new binary which could start the app in a different way
- Looking at the structure, we could get pretty good idea of what the app does just by looking at the names
	- We could see 
		- beers, reviews
		- verbs like adding, listing and reviewing
	- This structure actually gives us quite a good idea of what app does including in `/cmd` folder, it says
		- it has server binary to serve beers : `beer-server` 
		- and allows adding sample data: `sample-data`
- Easy to extend and add bits to the library, without having to change much else where
	- If we want to create an `rpc` version of the app or system or service, just check in a directory under the `http` package and use the same services already been defined like `adding`, `listing`, `reviewing`
- **DDD** thinking can be applied to a flat structure too. DDD doesn't really enforce a particular structure
- If we group the group go programs by context or the domain, rather than the functionality or the implementation detail, it works more in our favour