# Introduction

A simple TODO app that demostrates using SeaORM, SQLite and Postgres to build a simple TODO application. TCP connections are used instead of web frameworks for simplicity due to the APIs being available in the standard library, which is mirrored by async-std async library. 

Let's get started.

#### Symbols Used

To show added or removed code from files, we will use comments or 

`+` to show added code

`-` to show removed code

`...` is used to show only part of the existing code instead of rewriting already existing code in the examples.

`$ ` shows an operation is done on the console/shell 

`postgres=#` shows a postgres prompt

This will make it easier to visualize changes to a file

First, install PostgreSQL and SQLite and ensure PostgreSQL server is running.

### Initializing the project directory

A cargo workspace make development easier and share the building environment. The `HTTP` TODO client will be called `client` and the `HTTP server` will be called `server`.

#### Initialize the `client` and `server`

Create the workspace directory `todo-app`

```sh
$ mkdir todo-app
```

Then switch to the workspace directory

```sh
$ cd todo-app
```

Create the `client` and `server` projects

```sh
$ cargo new client
```

```sh
$ cargo new server
```

Create a `Cargo.toml` in the root of the workspace directory to register the two projects

`File: todo-app/Cargo.toml`

```toml
[workspace]
members = [
	"client",
	"server",
]
```

Next up is building the TCP server.