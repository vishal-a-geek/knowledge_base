#go #code-organization

### User Model and Service
```go
// models/user.go
package models

import (
	"database/sql"
)

type User struct {
	ID int
	Email string
	PasswordHash string
}

type UserService struct {
	DB *sql.DB
}
```
- Create user
```go
// models/create.go
package models

import (
	"fmt"
	"strings"
	"golang.org/x/crypto/bcrypt"
)

type CreateUserRequest struct {
	Email string
	Password string
}

type CreateUserResponse struct {
	User *User
}

func (us *UserService) Create(r CreateUserRequest) (*CreateUserResponse, error) {
	email := strings.ToLower(r.Email)
	passh, err := bcrypt.GenerateFromPassword([]byte(r.Password), bcrypt.DefaultCost)
	
	if err != nil {
		return nil, fmt.Errorf("create: %w", err)
	}
	
	hashedP := string(passh)
	row := us.DB.QueryRow(`
	INSERT INTO users (email, password_hash)
	VALUES ($1, $2)
	RETURNING id;
	`, email, hashedP)
	
	user := User{
		Email: email,
		PasswordHash: hashedP,
	}
	
	err = row.Scan(&user.ID)
	if err != nil {
		return nil, fmt.Errorf("create user: %w", err)
	}
	
	return &CreateUserResponse{
		User: &user,
	}, nil
}
```

- Using `UserService`
```go
package main

import (
	"database/sql"
	"fmt"
	_ "github.com/jackc/pgx/v4/stdlib"
	models "github.com/vishal2098govind/lenslocked/models/user"
)

  

type PostgresConfig struct {
	Host string
	Port string
	User string
	Password string
	Database string
	SSLMode string
}

func (cfg PostgresConfig) String() string {
	return fmt.Sprintf(
		"host=%s port=%s user=%s password=%s dbname=%s sslmode=%s",
		cfg.Host, cfg.Port, cfg.User, cfg.Password, cfg.Database, cfg.SSLMode,
	)
}

func main() {
	
	cfg := PostgresConfig{
		Host: "localhost",
		Port: "8000",
		User: "baloo",
		Password: "junglebook",
		Database: "lenslocked",
		SSLMode: "disable",
	}
	
	db, err := sql.Open("pgx", cfg.String())
	if err != nil {
		panic(err)
	}
	
	defer db.Close()
	
	us := models.UserService{
		DB: db,
	}
	
	res, err := us.Create(models.CreateUserRequest{
		Email: "vishal.govind@gmail.com",
		Password: "vishal@2098",
	})
	
	if err != nil {
		panic(err)
	}
	
	fmt.Println(res)
}
```
- Problem with this pattern: this gives a false impression that `UserService` can be used with any database connection, when in reality, the `models.UserService.Create` method is very specific to `postgres`
```
```