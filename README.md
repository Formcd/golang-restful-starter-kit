# Go RESTful Application Starter Kit

This starter kit is designed to get you up and running with a project structure optimal for developing
RESTful services in Go. The kit promotes the best practices that follow the [SOLID principles](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design))
and encourage writing clear and idiomatic Go code. 

The kit provides the following features right out of the box 

* RESTful endpoints in the widely accepted format
* Standard CRUD operations of a database table
* Application configuration via environment variable and configuration file
* Structured logging with contextual information
* Panic handling and proper error response generation
* Automatic DB transaction handling
* Data validation
* Full test coverage
 
The kit uses the following Go packages which can be easily replaced with your own favorite ones
since their usages are mostly localized and abstracted. 

* Routing framework: [ozzo-routing](https://github.com/go-ozzo/ozzo-routing)
* Database: [ozzo-dbx](https://github.com/go-ozzo/ozzo-dbx)
* Data validation: [ozzo-validation](https://github.com/go-ozzo/ozzo-validation)
* Logging: [logrus](https://github.com/Sirupsen/logrus)
* Configuration: [viper](https://github.com/spf13/viper)
* Dependency management: [glide](https://github.com/Masterminds/glide)
* Testing: [testify](https://github.com/stretchr/testify)


## Getting Started

If this is your first time encountering Go, please follow [the instructions](https://golang.org/doc/install) to
install Go on your computer. The kit requires Go 1.5 or above.

After installing Go, run the following commands to download and install this starter kit:

```shell
# install the starter kit
go get github.com/qiangxue/golang-restful-starter-kit

# install glide (a vendoring and dependency management tool), if you don't have it yet
go get -u github.com/Masterminds/glide

# fetch the dependent packages
cd $GOPATH/qiangxue/golang-restful-starter-kit
glide up -u -s
```

Next, create a PostgreSQL database named `music` and execute the SQL statements given in the file `data/db.sql`.
The starter kit uses the following default database connection information:
* server address: 127.0.0.1 (local machine)
* server port: 5432
* database name: music
* username: music
* password: music

If your connection is different from the above, you should define an environment variable named
`MUSIC_DSN` whose value is in the following format:

```
postgres://<username>:<password>@<server-address>:<server-port>/<db-name>
```

Now you can build and run the application by running the following command under the
`$GOPATH/qiangxue/golang-restful-starter-kit` directory:

```shell
go run server.go
```

or simply the following if you have the `make` tool:

```shell
make
```

The application runs as an HTTP server at port 8080. It provides the following RESTful endpoints:

* `GET /ping`: a ping service mainly provided for health check purpose
* `GET /v1/artists`: returns a paginated list of the artists
* `GET /v1/artists/:id`: returns the detailed information of an artist
* `POST /v1/artists`: creates a new artist
* `PUT /v1/artists/:id`: updates an existing artist
* `DELETE /v1/artists/:id`: deletes an artist

For example, if you access the URL `http://localhost:8080/v1/artists` in a browser or via `cURL`:

```shell
curl -X GET "http://localhost:8080/v1/artists"
```

you should be able to see a list of the artists returned in the JSON format.

## Next Steps

In this section, we will describe the steps you may take to make use of this starter kit in a real project.
You may jump to the [Project Structure](#project-structure) section if you mainly want to learn about 
the project structure and the recommended practices.

### Renaming the Project

To use the starter kit as a starting point of a real project whose package name is something like
`github.com/abc/xyz`, take the following steps:
 
* move the directory `$GOPATH/github.com/qiangxue/golang-restful-starter-kit` to `$GOPATH/github.com/abc/xyz`
* do a global replacement of the string `github.com/qiangxue/golang-restful-starter-kit` in all of
  project files with the string `github.com/abc/xyz`

### Implementing CRUD of Another Table
 
To implement the CRUD APIs of another database table (assuming it is named as `album`), 
you will need to develop the following files which are similar to the `artist.go` file in each folder:

* `models/album.go`: contains the data structure representing a row in the new table.
* `services/album.go`: contains the business logic that implements the CRUD operations.
* `daos/album.go`: contains the DAO (Data Access Object) layer that interacts with the database table.
* `apis/album.go`: contains the API layer that wires up the HTTP routes with the corresponding service APIs.

Then, wire them up by modifying the `serveResources()` function in the `server.go` file.

### Implementing a non-CRUD API

* If the API uses a request/response structure that is different from a database model,
  define the request/response model(s) in the `models` package.
* In the `services` package create a service type that should contain the main service logic for the API.
  If the service logic is very complex or there are multiple related APIs, you may create
  a package under `services` to host them.
* If the API needs to interact with the database or other persistent storage, create
  a DAO type in the `daos` package. Otherwise, the DAO type can be skipped.
* In the `apis` package, define the HTTP route and the corresponding API handler.
* Finally, modify the `serveResources()` function in the `server.go` file to wire up the new API.

## Project Structure

This starter kit divides the whole project into four main packages:

* `models`: contains the data structures used for communication between different layers.
* `services`: contains the main business logic of the application.
* `daos`: contains the DAO (Data Access Object) layer that interacts with persistent storage.
* `apis`: contains the API layer that wires up the HTTP routes with the corresponding service APIs.

[Dependency inversion principle](https://en.wikipedia.org/wiki/Dependency_inversion_principle)
is followed to make these packages independent of each other and thus easier to test and maintain.

The rest of the packages in the kit are used globally:
 
* `app`: contains routing middlewares and application-level configurations
* `errors`: contains error representation and handling
* `util`: contains utility code

The main entry of the application is in the `server.go` file. It does the following work:

* load external configuration
* establish database connection
* instantiate components and inject dependencies
* start the HTTP server