---
title: How to Set up FaunaDB for local development
published: false
description: FaunaDB is a serverless database with a GraphQL interface. Learn how to setup a local instance on your machine for development and testing.
tags: #python, #serverless, #database, #graphql
---

If you're feeling impatient and want to skip to the end, all the code is available at this [repo](https://github.com/cfranklin11/faunadb-local).

---

## What is FaunaDB, and why should I try it?

[FaunaDB](https://docs.fauna.com/fauna/current/introduction) is a serverless database that is an ideal choice for serverless applications, because it has the same benefits as the latter: auto-scaling, pay-for-what-you-use billing, not requiring server configuration or maintenance. FaunaDB accomplishes this by executing database operations via calls to its HTTP API rather than maintaining a connection to a database server. There are other options available for serverless database services. DynamoDB from AWS is one of the most commonly referenced when looking up information about serverless databases, but I find a basic key-value data store to be too limiting when trying to model an application's business domain, as it makes modelling relationships among entities very difficult. Relational databases are ideal for this, and AWS has another serverless option, Aurora Serverless, which combines the advantages of serverless architecture with the expressiveness of relational databases and SQL queries. Although I haven't used it myself, Aurora Serverless seems to be a good option. It comes with a large caveat, however, in that Aurora Serverless requires all applications and services that access its databases to be in the same Amazon Virtual Private Cloud. This means committing yourself even further to vendor lock-in, as any move away from using AWS, even partially, would mean having to completely change your database infrastructure. Also, my knowledge of ops stuff is pretty basic, and setting up an Amazon VPC is just the sort of extra complication that I wanted to avoid by going serverless in the first place. FaunaDB isn't strictly a relational database, but it still offers some of the same advantages by allowing for the inclusion of all the usual entity relationships (e.g. one-to-one, one-to-many). Also, FaunaDB databases can be called from any application hosted on any cloud service, giving you more flexibility in how and where you deploy applications, thus reducing vendor lock-in.

FaunaDB supports two different ways of querying its databases: FaunaDB Query Language (FQL) and GraphQL (GQL). For now, the GQL interface is somewhat limited, making FQL a better option if you need the sort of functionality that you can get from SQL. If you're not sure, or if you anticipate only needing basic queries, I recommend starting with the GQL interface, because it's much simpler to work with (if you already know GQL, then there's not much more to learn), and it offers the benefit of having a schema file as the source of truth about the structure of your database. If you start with GQL and find out later that you need more-complex queries, you can mix in FQL functions by defining custom resolvers, so you shouldn't ever need to completely migrate to FQL. Since I prefer the GQL interface (and it's all I've worked with so far), that's what I'll use for this tutorial.

If you want more information on how FaunaDB works in general, feel free to check out their [Getting Started](https://docs.fauna.com/fauna/current/start/) guide, but it's not necessary for following the rest of this tutorial.

## Requirements

To start, make sure you have the following installed on your machine:

- [Docker](https://docs.docker.com/get-docker/), so we can run an instance of FaunaDB in a container.
- [`fauna-shell`](https://docs.fauna.com/fauna/current/integrations/shell/) to be able to interact with local FaunaDB databases.
- An API testing tool (e.g. [Postman](https://www.postman.com/downloads/), [Insomnia](https://insomnia.rest/download/)) to easily send GraphQL queries (or you can use `curl` if you _really_ want to). For the examples below, I will use Insomnia, because its GraphQL support is a bit better than Postman's (for example, it can fetch a GraphQL schema from an API endpoint).
- [Python](https://www.python.org/downloads/) if you want to set up integration testing.

## Setting up local FaunaDB

### Running FaunaDB in a Docker container

FaunaDB is kind enough to provide us with an [official Docker image](https://hub.docker.com/r/fauna/faunadb), which greatly simplifies running it on our local environment. Run the following commands in your terminal to get an instance of FaunaDB running:

```bash
docker pull fauna/faunadb
docker run --name faunadb -p 8443:8443 -p 8084:8084 fauna/faunadb
```

We expose port `8443` for standard database interactions and `8084` for calls to the GraphQL API. As with other database images, you can use other Docker container options for data persistence and such. See the [FaunaDB documentation](https://docs.fauna.com/fauna/current/integrations/dev#run) for alternatives.

### Create a database in the FaunaDB instance

Now that we have FaunaDB up and running, the easiest way to interact with it is to use `fauna-shell`. In a new tab or window, run the following to create a database and an API key for it.

```bash
fauna add-endpoint http://localhost:8443/ --alias localhost --key secret
fauna create-database development_db --endpoint=localhost
fauna create-key development_db --endpoint=localhost
```

You can change the alias for the endpoint (currently `localhost`) and the name of the database (currently `development_db`) to anything you want. If one doesn't already exist, create a `.env` file in the root of the project, copy the API key printed by `fauna create-key`, and assign its value to `FAUNADB_KEY` in `.env` like below:

```bash
FAUNADB_KEY=<copied API key>
```

## Using GraphQL with FaunaDB

### Create a GraphQL schema

A full introduction to GraphQL is outside the scope of this tutorial, so if you want more information, you can check out the [official introduction](https://graphql.org/learn/). Also, if you want to get a better view of the GraphQL queries that FaunaDB creates by default for entities, check out their [getting started](https://docs.fauna.com/fauna/current/start/graphql) page.

To keep things simple, we'll create a basic schema file for a blogging platform, where we have users who write posts. Such a schema for FaunaDB looks like this:

```graphql
type User {
  username: String! @unique
  password: String!
  posts: [Post!] @relation
}

type Post {
  title: String!
  text: String!
  author: User!
}

type Query {
  allUsers: [User!]
  allPosts: [Post!]
}
```

This schema creates collections (FaunaDB's version of data tables) of users and posts, with a couple general-purpose query fields. It doesn't matter where you save this file, but it's best to name it following GraphQL conventions to take advantage of tools such as linters (I use `schema.gql`, but there are a few other acceptable alternatives). Most of this is standard GraphQL schema syntax, but notice the FaunaDB directives that start with `@`. These give extra information to FaunaDB about the structure of collections and documents.

- `@unique` indicates that the value for this attribute must be unique within the collection. FaunaDB will return an error response if we try to create a `user` with a duplicate `username`.
- `@relation` indicates a bi-directional relationship between two collections. In this case, a `user` has many `posts`, and a `post` has one `user`, which we call its `author`. Notice that we only need to use the `relation` directive on the "many" side of the relationship. FaunaDB has more information on how to model [different relationships](https://docs.fauna.com/fauna/current/api/graphql/relations) among collections.

These are the directives that I use most, but there are others for more-advanced use cases. You can see all the available directives [here](https://docs.fauna.com/fauna/current/api/graphql/directives/).

Also, by default, FaunaDB creates a few basic queries and mutations for each collection that you create (e.g. `createUser`, `updateUser`, `findUserById`), but doesn't add any queries for multiple records, so we add `allUsers` and `allPosts` queries for convenience.

### Import the GraphQL schema to the FaunaDB database

From now on, we're going to interact with the FaunaDB database via HTTP calls to its GraphQL API, which should be running on `http://localhost:8084`. The simplest way to do this is with an API testing tool like Insomnia, but for an application, you'll obviously want to automate these calls with an HTTP library like `requests` for Python or `axios` for Node.

Our first call will be to import our GraphQL schema. Open Insomnia, and create a request that will `POST` to `http://localhost:8084/import`. Remember that secret key that we saved? Use it in the `Authorization` header with the format `Bearer <FaunaDB secret key>`. Finally, let's add our schema file to the request (in the case of Insomnia, select "Binary File" from the body options, then "Choose File" to upload the schema).

### Create and query records in the database

Since the FaunaDB instance has a standard GraphQL API layer accessed at `http://localhost:8084/graphql`, you can use GraphQL tooling to export the schema and documentation, and load it in an interactive tool like GraphiQL or GraphQL Playground. The easiest solution, however, is to use Insomnia's GraphQL documentation feature by selecting "GraphQL Query" for the request body, then clicking "schema" and selecting "Refresh Schema". This will give you access to auto-generated documentation for your GraphQL API without needing any extra setup.

To start, we will want to create some data that we will then be able to query. The following mutations will create two users with two posts each. We return the IDs in the response for future reference, or you can use the `allUsers` query to get them again later.

```graphql
mutation createBob {
  createUser(data: {
    username: "burgerbob",
    password: "password1234",
    posts: {
      create: [
        {
          title: "Burgers Are Great!",
          text: "Burgers have meat, and bread, and..."
        },
        {
          title: "Top 10 Burgers",
          text: "1. Cheeseburger, 2. Hamburger..."
        },
      ]
    }
  }) {
    _id
    posts {
      data { _id }
    }
  }
}

mutation createLinda {
  createUser(data: {
    username: "momsense",
    password: "password1234",
    posts: {
      create: [
        {
          title: "Why Baked Ziti Is Overrated",
          text: "Baked ziti is really not that great..."
        },
        {
          title: "Top 10 Wines to Pair with Burgers",
          text: "1. Pinot Noir, 2. Merlot..."
        },
      ]
    }
  }) {
    _id
    posts {
      data { _id }
    }
  }
}
```

Now that we have some records in the database, we can query them with the built-in `find<Collection>ByID` query or fetch all of a collection's records with one of the `all<Collection>` queries that we included in the schema. If you're using Insomnia, explore the schema documentation and try out different queries.


## Bonus: Using FaunaDB while testing a Python application (with Pytest)

A common challenge when testing application code is handling database transactions in integration tests. Thankfully, we have frameworks like Rails and Django that, with a little configuration, can handle database setup and teardown for us. Unfortunately, such frameworks' database connectors generally have limited support for NoSQL databases like FaunaDB, and even then, it's only for the most-popular options like MongoDB. So, how can we use our local FaunaDB instance for integration tests without corrupting our development database?

When setting up integration tests, I wanted to start by making sure that I didn't accidentally change data in my development database. Since FaunaDB identifies the specific database that you're calling with the API token that you use, the easiest way to avoid accidental calls is to make sure that the token for your development database isn't available in the code. Assuming that you're using environment variables for your tokens and secrets, the easiest way to hide the dev API token is to make it blank in `os.environ` before every test. Pytest allows us to automatically run code before tests with `conftest.py` files. What's more, these files can be nested to run different pre-test code in different test modules. So, we can define a `conftest.py` file at the root of our `tests` directory with the following code to prevent unwanted FaunaDB calls:

```python
# src/tests/conftest.py
import os

os.environ["FAUNADB_KEY"] = ""
```

I tried doing the same thing using a Pytest fixture or `unittest`'s `patch`, which can work as well, but require you to use them before every test or at least before every test suite, and such details are easy to forget when writing tests for the next feature. The code above, however, is guaranteed to run before any tests, making it a foolproof way of keeping our development data safe.

Now that we've prevented unwanted calls to our dev database, how do we set up and call our test database for integration tests? Why, with another `conftest.py` of course! I have `tests` separated into `unit` and `integration` submodules. Inside `integration` we can include the following `conftest.py`:

```python
# src/tests/integration/conftest.py

import os
from unittest.mock import patch

import pytest

# Names of module & FaunaDB client class depend
# on your application code
from src.app.faunadb import FaunadbClient


# Use "session" scope and autouse to run once before all tests.
# This is to make sure that the "localhost" endpoint exists.
@pytest.fixture(scope="session", autouse=True)
def _setup_faunadb():
    os.system(
        "npx fauna add-endpoint http://faunadb:8443/ --alias localhost --key secret"
    )


# Scope "function" means that this only applies to the test function
# that uses it
@pytest.fixture(scope="function")
def faunadb_client():
    # We create and delete the database for each test,
    # because it's reasonably quick, and simpler than manually deleting
    # all data.
    os.system("npx fauna create-database test --endpoint localhost")

    # Creating an API key produces output in the terminal
    # that includes the following line: secret: <API token>
    create_key_output = (
      os.popen("npx fauna create-key test --endpoint=localhost").read()
    )
    faunadb_key = (
        re.search("secret: (.+)", create_key_output).group(1).strip()
    )

    client = FaunadbClient(faunadb_key=faunadb_key)
    client.import_schema()

    # For any test that uses this fixture, we patch the environment
    # variable for the FaunaDB API key and return the client. This way,
    # all FaunaDB calls will use the test DB, and the test function
    # will have a valid client to make DB calls for test setup
    # or assertions.
    with patch.dict(
      "os.environ", {**os.environ, 'FAUNADB_KEY': faunadb_key}
    ):
        yield client

    os.system("npx fauna delete-database test --endpoint localhost")
```

You might need to modify the code above depending on the specifics of your app and tests (for example, I patch my app's `settings` module rather than `os.environ` directly), but I've been using a similar file for a few weeks, and it's been working well. Now, if you wanted to test user creation with actual database transactions, you could run something like the following:

```python
# src/tests/integration/test_user.py

def test_user_creation(faunadb_client):
  username = "burgerbob"
  created_user = faunadb_client.create_user(
    username=username,
    password="password1234"
  )
  assert created_user['username'] == username

  all_users = faunadb_client.all_users()
  assert len(all_users) == 1
```

If you want to see the full test and FaunaDB client code, I have an example in the [repo](https://github.com/cfranklin11/faunadb-local) for this tutorial.

## Conclusion

There you have it: a full FaunaDB setup for local development and testing (as long as you're using Python). Thanks to its GraphQL interface, service-agnostic architecture, and its ability to model complex data relationships, I think FaunaDB is a solid choice for data persistence for any serverless application. The documentation for setting up a local instance of FaunaDB, however, is a bit sparse. So, I had to figure it out by piecing together disparate posts and code samples, but, as you can see, it's not too complicated once you know the necessary commands and configuration. So, give FaunaDB a try for your next serverless project.