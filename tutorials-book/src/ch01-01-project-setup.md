# Project Setup

In this section, we will set up our project, including the folder structure and crate dependencies.

We will be using a **MySQL** database throughout the tutorials, but all functionalities of SeaORM are **agnostic to the database implementation**, as mentioned before.

## Adding `sea-orm` as a Dependency

```sh
$ cargo init bakery-backend
```

```diff
# Cargo.toml

...

[dependencies]
+ sea-orm = { version = "^0.9.0", features = [ "sqlx-mysql", "runtime-async-std-native-tls", "macros" ] }

...

```

The DB driver feature `sqlx-mysql` is used because we are using MySQL.

The second feature, `runtime-async-std-native-tls` is an async runtime arbitrarily chosen for this project. More information can be found on the [docs](https://www.sea-ql.org/SeaORM/docs/install-and-config/database-and-async-runtime/#async_runtime).

Finally, the `macros` feature is an optional feature that allows the use of some `Derive` macros.

## Connecting to the Database server

We add `futures` as a dependency so that we can make use of asynchronous programming with `async`/`await`.

```diff
# Cargo.toml

...

[dependencies]
+ futures = "0.3.21"
sea-orm = { version = "^0.9.0", features = [ "sqlx-mysql", "runtime-async-std-native-tls", "macros" ] }

...

```

Connect to the database server:

```rust, no_run
// main.rs

use futures::executor::block_on;
use sea_orm::{Database, DbErr};

// Change this according to your database implementation,
// or supply it as an environment variable.
// the whole database URL string follows the following format:
// "protocol://username:password@host:port/database"
// We put the database name (that last bit) in a separate variable simply for convenience.
const DATABASE_URL: &str = "mysql://root:password@localhost:3306";
const DB_NAME: &str = "bakeries_db";

async fn run() -> Result<(), DbErr> {
    let db = Database::connect(DATABASE_URL).await?;

    Ok(())
}

fn main() {
    if let Err(err) = block_on(run()) {
        panic!("{}", err);
    }
}
```

If everything is correctly set up, the program should terminate gracefully.

If it panicks, it could be that the database URL is wrong.

If it hangs, it could be that the database is not up and running.

## Creating a Database

For MySQL and PostgreSQL, we can create a specific database instance. Let's call it `bakeries_db`.

```rust, no_run
...
// main.rs
- use sea_orm::{Database, DbErr};
+ use sea_orm::{ConnectionTrait, Database, DbBackend, DbErr, Statement};

...

async fn run() -> Result<(), DbErr> {
    let db = Database::connect(DATABASE_URL).await?;

+   let db = &match db.get_database_backend() {
+       DbBackend::MySql => {
+           db.execute(Statement::from_string(
+               db.get_database_backend(),
+               format!("CREATE DATABASE IF NOT EXISTS `{}`;", DB_NAME),
+           ))
+           .await?;
+
+           let url = format!("{}/{}", DATABASE_URL, DB_NAME);
+           Database::connect(&url).await?
+       }
+       DbBackend::Postgres => {
+           db.execute(Statement::from_string(
+               db.get_database_backend(),
+               format!("DROP DATABASE IF EXISTS \"{}\";", DB_NAME),
+           ))
+           .await?;
+           db.execute(Statement::from_string(
+               db.get_database_backend(),
+               format!("CREATE DATABASE \"{}\";", DB_NAME),
+           ))
+           .await?;
+
+           let url = format!("{}/{}", DATABASE_URL, DB_NAME);
+           Database::connect(&url).await?
+       }
+       DbBackend::Sqlite => db,
+   };

    Ok(())
}

...
```

This snippet shows that SeaORM is database-agnostic. You may only handle the case for your chosen database if you are sure only one type of database will be used.
