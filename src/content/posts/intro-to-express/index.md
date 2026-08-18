---
title: Intro to Express
updated: 2026-08-18
published: 2026-08-18
description: Learn Express.js by building a small Twitter-style API, covering routes, middleware, request data, logging, and request-body parsing.
image: ""
tags:
  - NodeJS
  - Express
category: Web-development
draft: false
---
# Intro to Express

Express is a **Node.js web framework** that makes it easier to build web servers and REST APIs.

In this tutorial, we'll learn Express by building a small **Twitter-style API**. We'll start with a basic server, then gradually add routes, middleware, logging, and different ways of receiving data from requests.

## What We'll Cover

- Setting up an Express project
- Creating a server and defining routes
- Testing APIs with Postman
- Automatically restarting the server
- Understanding and chaining middleware
- Using `app.use()` and `morgan`
- Reading query parameters, URL parameters, and request bodies
- Parsing JSON and URL-encoded data
- Handling unknown routes

---

## Project Setup

Create the project:

```bash
mkdir TwitterApp
cd TwitterApp
npm init -y
npm install express
```

Create the following structure:

```text
TwitterApp/
├── src/
│   └── index.js
├── package.json
└── package-lock.json
```

Enable ES modules by adding `"type": "module"` to `package.json`:

```json
{
    "type": "module"
}
```

### Basic Server

```javascript
import express from 'express';

const app = express();

app.listen(3000, () => {
    console.log('Server is running on port 3000');
});
```

Run the server:

```bash
node src/index.js
```

At this point, the server is running, but it doesn't have any routes.

---

## Testing with Postman

Open Postman and send a `GET` request to:

```text
http://localhost:3000
```

You'll receive:

```text
404 Not Found
Cannot GET /
```

This happens because we created the server but haven't defined what it should do when a request arrives at `/`.

---

## Defining Routes

A route tells Express how to respond to a particular **HTTP method and URL**.

```javascript
app.get('/ping', (req, res) => {
    return res.json({ message: 'ping' });
});
```

Here:

- `app.get()` handles a `GET` request.
- `/ping` is the route.
- `req` contains information about the request.
- `res` is used to send a response.

Send:

```text
GET http://localhost:3000/ping
```

The response is:

```json
{
    "message": "ping"
}
```

The server returns `200 OK`.

If you send:

```text
POST http://localhost:3000/ping
```

you'll get:

```text
Cannot POST /ping
```

because we only defined a `GET` route.

We can define a `POST` route separately:

```javascript
app.post('/hello', (req, res) => {
    return res.json({ message: 'hello' });
});
```

---

## Automatically Restarting the Server

Express doesn't automatically restart the server when you change your code.

Modern Node.js provides a built-in `--watch` option:

```bash
node --watch src/index.js
```

Another popular option is **nodemon**:

```bash
npm install nodemon
```

Because it is installed locally, you can run it with:

```bash
npx nodemon src/index.js
```

You can also create an npm script in `package.json`:

```json
{
    "scripts": {
        "start": "nodemon src/index.js"
    }
}
```

Then run:

```bash
npm start
```

---

## Middleware

**Middleware** is a function that runs during the request-response cycle.

It has access to:

```text
req  → request
res  → response
next → passes control forward
```

A middleware can:

- Inspect a request
- Modify a request
- Modify a response
- Perform authentication or validation
- Log requests
- End the request
- Pass control to the next middleware

A simple middleware:

```javascript
function mid1(req, res, next) {
    console.log('middleware1');
    next();
}
```

The important part is `next()`.

Calling `next()` tells Express:

> "I'm done. Continue processing this request."

### Using Middleware on a Route

Without middleware:

```javascript
app.get('/hello', (req, res) => {
    return res.json({ message: 'hello' });
});
```

With middleware:

```javascript
app.get('/hello', mid1, (req, res) => {
    return res.json({ message: 'hello' });
});
```

The request flows like:

```text
Client
  ↓
mid1
  ↓
Route Handler
  ↓
Response
```

If `mid1` doesn't call `next()` or send a response, the request will remain pending.

### Multiple Middleware Functions

Middleware can be chained:

```javascript
function mid1(req, res, next) {
    console.log('middleware1');
    next();
}

function mid2(req, res, next) {
    console.log('middleware2');
    next();
}

function mid3(req, res, next) {
    console.log('middleware3');
    next();
}

app.get('/hello', mid1, mid2, mid3, (req, res) => {
    return res.json({ message: 'hello' });
});
```

The execution order is:

```text
Client
  ↓
mid1
  ↓
mid2
  ↓
mid3
  ↓
Route Handler
  ↓
Response
```

You can also pass middleware as an array:

```javascript
app.get('/hello', [mid1, mid2, mid3], (req, res) => {
    return res.json({ message: 'hello' });
});
```

Each middleware receives the same `req` and `res` objects. If one middleware modifies `req`, later middleware and the route handler can access that modification.

### Global Middleware with `app.use()`

If middleware should run for every request, use `app.use()`:

```javascript
function commonmid(req, res, next) {
    console.log('common middleware');
    next();
}

app.use(commonmid);
```

Now the middleware runs before the matching routes.

This is useful for functionality such as:

- Logging
- Authentication
- Request parsing
- Validation
- CORS

---

## Logging with Morgan

**Morgan** is middleware for logging HTTP requests.

Install it:

```bash
npm install morgan
```

Import and use it:

```javascript
import morgan from 'morgan';

app.use(morgan('combined'));
```

A request might produce a log similar to:

```text
::1 - - [22/Jun/2026:11:03:51 +0000] "GET /ping HTTP/1.1" 200 18 "-" "PostmanRuntime/7.54.0"
```

The log contains information such as:

- Client address
- Request method
- URL
- HTTP version
- Status code
- Response size
- User agent

Browsers can also send `GET` requests by entering a URL directly, but tools such as Postman are more useful when testing different HTTP methods and request bodies.

> Middleware is a fundamental part of Express. Other frameworks provide similar concepts under names such as **filters** or **interceptors**.

---

## Accessing Data from Requests

A client can send data to an Express server in several common ways:

| Type             | Example               | Express      |
| ---------------- | --------------------- | ------------ |
| Query parameters | `/users?name=ghazi`   | `req.query`  |
| URL parameters   | `/users/123`          | `req.params` |
| Request body     | `{ "name": "ghazi" }` | `req.body`   |

### Query Parameters

Query parameters appear after `?` in the URL.

```text
http://localhost:3000/ping?name=ghazi
```

Access them with `req.query`:

```javascript
app.get('/ping', (req, res) => {
    console.log(req.query);
    return res.json({ message: 'hello' });
});
```

For:

```text
/ping?name=ghazi
```

`req.query` contains:

```javascript
{
    name: 'ghazi'
}
```

Query parameters are commonly used for filtering, searching, sorting, and pagination.

### URL Parameters

URL parameters are dynamic values inside the route path.

```javascript
app.get('/tweets/:tweet_id', (req, res) => {
    console.log(req.params);
    return res.json({ message: 'tweet details' });
});
```

Here:

```text
/tweets
```

is fixed, while:

```text
/:tweet_id
```

is dynamic.

For:

```text
http://localhost:3000/tweets/123
```

Express gives:

```javascript
req.params;
// { tweet_id: '123' }
```

URL parameters are useful when identifying a specific resource.

### Request Body

The request body is commonly used with `POST`, `PUT`, and `PATCH` requests.

In Postman:

**Body → raw → JSON**

```json
{
    "name": "ghazi",
    "company": "vaydick"
}
```

Then:

```javascript
app.post('/hello', (req, res) => {
    console.log(req.body);
    return res.json({ message: 'world' });
});
```

Without body-parsing middleware, `req.body` may be `undefined`.

### Parsing Request Bodies

Unlike URL and query parameters, the request body can contain different formats.

For JSON requests:

```javascript
app.use(express.json());
```

This middleware reads JSON request bodies and converts them into JavaScript objects.

For URL-encoded data:

```javascript
app.use(express.urlencoded({ extended: true }));
```

This parses data sent using the `application/x-www-form-urlencoded` format.

A typical Express setup might therefore include:

```javascript
import express from 'express';

const app = express();

app.use(express.json());
app.use(express.urlencoded({ extended: true }));
```

The important idea is:

```text
Raw request body
       ↓
Body-parsing middleware
       ↓
JavaScript object
       ↓
req.body
```

---

## Request Data at a Glance

```text
GET /tweets?author=ghazi
             ↓
         req.query

GET /tweets/123
             ↓
         req.params

POST /tweets
{
    "text": "Hello"
}
             ↓
         req.body
```

Each mechanism serves a different purpose:

- **Query parameters** → filtering, searching, sorting, pagination
- **URL parameters** → identifying a specific resource
- **Request body** → sending data to create or update a resource

---

## Handling Unknown Routes

If no defined route matches a request, Express can use a final catch-all handler.

For example:

```javascript
app.use((req, res) => {
    res.status(404).json({
        message: 'Route not found'
    });
});
```

This middleware should be placed **after your other routes**, so it only runs when none of them handled the request.

---

## Summary

Express provides a simple layer on top of Node.js for building web servers and APIs.

The core request flow is:

```text
Client
  ↓
Express Server
  ↓
Middleware
  ↓
Route
  ↓
Controller / Handler
  ↓
Response
```

The most important concepts introduced here are:

- **Routes** define how the server responds to requests.
- **Middleware** runs during request processing and can pass control using `next()`.
- **`app.use()`** registers middleware globally or for a path.
- **Morgan** provides request logging.
- **`req.query`** reads query parameters.
- **`req.params`** reads URL parameters.
- **`req.body`** reads parsed request-body data.
- **`express.json()`** parses JSON request bodies.
- **`express.urlencoded()`** parses URL-encoded request bodies.

These concepts form the foundation for building more complete Express applications and REST APIs.