# Hasura Hello World

This quickstart will take you over Hasura's instant backend APIs (BaaS) and how to deploy your custom code too.

## Clone & deploy

Any project on hasura.io/hub can be cloned and deployed. In fact, this hello-world is a hasura project itself.

**Step 1:** Install the hasura CLI: [installation instructions](https://docs.hasura.io/0.15/manual/install-hasura-cli.html)

**Step 2:** Create a hasura project on your machine

```
$ # 1) Run the quickstart command
$ hasura quickstart hasura/hello-world
```

**Step 3:** Deploy the project to your free cluster!

```
$ # 2) Git add, commit & push to deploy to your cluster
$ cd hello-world
$ git add . && git commit -m 'First commit'
$ git push hasura master
```

**Note**: Your free cluster got automatically created when you ran the `quickstart` command.

### What got 'deployed'?

This hello-world project contains a sample data schema and some sample data (files in `migrations`). When you ran the `git push` these tables got created.

In the next few steps you'll be browsing the instant Hasura APIs and exploring adding custom microservice too.

### Using the API console

The hasura CLI gives you a web UI to manage your data modelling, manage your app users and explore the Hasura APIs.
The API explorer gives you a collection of all the Hasura APIs and lets you test them easily.

Access the **api-console** via the following command:

```
$ hasura api-console
```

This will open up Console UI on the browser. You can access it at [http://localhost:9695](http://localhost:9695)

## GraphQL / Data APIs

The Hasura Data API provides ready-to-use GraphQL APIs and also a HTTP/JSON API backed by a PostgreSQL database.

These APIs are designed to be used by any client capable of making HTTP requests, especially
and are carefully optimized for performance.

The Data API provides the following features:
* GraphQL APIs with authorisation.
* CRUD APIs on PostgreSQL tables with a MongoDB-esque JSON query syntax.
* Rich query syntax that supports complex queries using relationships.
* Role based access control to handle permissions at a row and column level.

 The url to be used to make these queries is always of the type: `https://data.cluster-name.hasura-app.io/v1/query` (in this case `https://data.awesome45.hasura-app.io`)

As mentioned earlier, this quickstart app comes with two pre-created tables `author` and `article`.

#### author

column | type
--- | ---
id | integer NOT NULL *primary key*
name | text NOT NULL

#### article

column | type
--- | ---
id | serial NOT NULL *primary key*
title | text NOT NULL
content | text NOT NULL
rating | numeric NOT NULL
author_id | integer NOT NULL *foreign key*


Alternatively, you can also view the schema for these tables on the api console by heading over to the tab named `Data`.

Let's look at a sample query to explore the Data APIs:

### GraphQL Operations
GraphQL Operations are supported by Hasura Data APIs.

* Select all entries in the article table, ordered by desc rating:
```graphql

query fetch_article {
  article (order_by: ["-rating"]) {
    id
    title
    content
    rating
    author_id
  }
}

```

For more examples of GraphQL queries, check out the [docs](https://docs.hasura.io/0.15/manual/data/graphql.html)

## Auth APIs

Every app almost always requires some form of authentication. This is useful to identify a user and provide some sort of personalised experience to the user. Hasura provides various types of authentication (username/password, mobile/otp, email/password, Google, Facebook etc).

### Signup
Lets look at an example for simple username/password signup.

* Signup using username and password:
POST `auth.cluster-name.hasura-app.io/v1/signup` HTTP/1.1
Content-Type: application/json
```json
{
  "provider" : "username",
  "data" : {
     "username": "johnsmith",
     "password": "somepass123"
  }
}
```

Typical response of a signup request looks like:
```json
{
  "auth_token": "b4b345f980ai4acua671ac7r1c37f285f8f62e29f5090306",
  "username": "johnsmith",
  "hasura_id": 2,
  "hasura_roles": [
      "user"
  ]
}
```

Once a user is registerd (or signed-up) on Hasura, it attaches a Hasura Identity or (hasura_id) to every user. A Hasura identity is an integer. You can use this value in your application to tie your application’s user to this identity.

To learn more, check out our [docs](https://docs.hasura.io/0.15/manual/users/index.html)

### Add instant authentication via Hasura’s web UI kit

Every project comes with an Authentication kit, you can restrict the access to your app to specific user roles.
It comes with a UI for Signup and Login pages out of the box, which takes care of user registration and signing in.

![Auth UI](https://docs.hasura.io/0.15/_images/uikit-dark.png)

Follow the [Authorization docs](https://docs.hasura.io/0.15/manual/users/uikit.html) to add Authentication kit to your app.

## File APIs

Sometimes, you would want to upload some files to the cloud. This can range from a profile pic for your user or images for things listed on your app. You can securely add, remove, manage, update files such as pictures, videos, documents using the Hasura filestore.

This is done via simple POST, GET and DELETE requests on a single endpoint.

Just like the Data service, the File API supports Role based access control to the files, along with custom authorization hooks. (Check out our [ documentation ](https://docs.hasura.io/0.15/manual/filestore/index.html) for more!)

### Uploading files

Uploading a file requires you to generate a file_id and make a post request with the content of the file in the request body and the correct mime type as the content-type header.

```http
POST https://filestore.cluster-name.hasura-app.io/v1/file/05c40f1e-cdaf-4e29-8976-38c899 HTTP/1.1
Content-Type: image/png
Authorization: Bearer <token>

<content-of-file-as-body>
```

This is a very simple to use system, and lets you directly add an Upload button on your frontend, without spending time setting up the backend.

### Downloading files
Downloading a file requires the unique file id that was used to upload it. This can be stored in the database and retrieved for download.

To download a particular file, what is required is a simple GET query.
```http
GET https://filestore.cluster-name.hasura-app.io/v1/file/05c40f1e-cdaf-4e29-8976-38c899 HTTP/1.1
Authorization: Bearer <token>
```

## Notify APIs

Check out the [ Learning center ](http://localhost:9695/learning-center) tab on the API Console for short tutorials on all the APIs!

## Add your own custom microservice

### Docker microservice

```
$ hasura microservice create <service-name> -i <docker-image> -p <port>
```

### git push microservice

```bash
$ hasura microservice create <service-name>
```

Once you have added a new service, you need to add a route to access the service.

Let's say you want to add a microservice already present in another Hub project, you can make use of `hasura clone`. You can quickly add a Node.js microservice by running the following:

```bash
$ hasura microservice clone api --from hasura/hello-nodejs-express
```

This will clone the api microservice from hello-nodejs-express project available on Hub.

### Add route for the service created.

```bash
$ hasura conf generate-route <service-name> >> conf/routes.yaml
```

It will generate the route configuration for the service and append it to `conf/routes.yaml`.

### Add a remote for the service [Only for git push based services]

```bash
$ hasura conf generate-remote <service-name> >> conf/ci.yaml
```

This will append the remotes configuration to the conf/ci.yaml file under the {{ cluster.name }} key.

### Apply your changes

```
$ git add .
$ git commit -m "Added a new service"
$ git push hasura master
```
