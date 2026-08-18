---
title: Node.js — Capabilities, Servers & REST APIs
updated: 2026-08-18
published: 2026-08-18
description: A practical introduction to Node.js capabilities, file systems, servers, APIs, and REST API conventions.
image: ""
tags:
  - NodeJS
category: Web-development
draft: false
---
# Node.js — Capabilities, Servers & REST APIs

Node.js allows us to run JavaScript outside the browser. This gives JavaScript access to capabilities that browser-based JavaScript normally does not have, such as working with the file system, creating servers, and interacting directly with the operating system.

---

## Running Scripts Directly (Without the `node` Command)

Normally, we run a Node.js file using:

```bash
node index.js
```

On Linux and macOS, you can also make a JavaScript file executable and run it directly:

```bash
./index.js
```

This is useful when distributing command-line utilities because the user can run the script directly instead of explicitly typing `node`.

For this to work, the file needs:

1. Execute permission
    
2. A **shebang** that tells the operating system which interpreter should run the file
    

---

## Shebang (`#!`)

You may have seen this at the top of Python or JavaScript files:

```js
#!/usr/bin/env node

console.log('hello')
```

The `#!` is called a **shebang**.

It tells the operating system which interpreter should be used to execute the file. In this example:

```text
/usr/bin/env node
```

means "find the `node` executable and use it to run this file."

This approach is mainly used on Unix-like systems such as Linux and macOS.

### Windows

Windows generally doesn't use Unix-style executable permissions and shebangs in the same way.

Instead, Windows commonly relies on:

- File associations
    
- `.cmd` / `.bat` wrapper scripts
    
- Tools such as `npm`, which provide command-line wrappers for Node.js programs
    

For example, a simple Windows batch wrapper could be:

```cmd
@echo off
node "%~dp0myscript.js" %*
```

Here:

- `%~dp0` refers to the directory containing the batch file
    
- `%*` forwards the command-line arguments
    

So the wrapper can run the JavaScript file using Node.js while allowing the user to execute the wrapper directly.

---

## File Permissions in Linux

When you try to run:

```bash
./index.js
```

Linux may give you a permission error if the file isn't executable.

You can inspect permissions using:

```bash
ls -l
```

You might see:

```text
-rw-r--r-- 1 user group 1234 Jan 1 12:34 index.js
```

The first part describes the file type and permissions:

|Part|Meaning|
|---|---|
|`-`|File (`d` would mean directory)|
|`rw-`|Owner can read and write|
|`r--`|Group can read|
|`r--`|Others can read|

There is currently no `x` permission, which means the file isn't executable.

You can give the owner full permissions with:

```bash
chmod 700 index.js
```

The number `7` represents:

```text
read + write + execute
 4   +   2   +   1
```

So:

```text
700
```

means:

```text
Owner  → read + write + execute
Group  → no permissions
Others → no permissions
```

The permissions will then look like:

```text
-rwx------
```

Now the file can be executed directly:

```bash
./index.js
```

---

## File System (`fs` Module)

One of the important capabilities Node.js provides is access to the file system.

Using the built-in `fs` module, Node.js can:

- read files
- create files
- write to files
- update files
- delete files
- work with directories
    

There are both callback-based and promise-based APIs.

### Callback-Based API

For example:

```js
import { readFile } from 'node:fs'

readFile('/etc/passwd', (err, data) => {
    if (err) throw err
    console.log(data)
})
```

The callback receives:

- `err` — an error if something went wrong
    
- `data` — the contents of the file if the operation succeeded
    

### Promise-Based API

Node.js also provides a promise-based version through `fs/promises`.

```js
import { readFile } from 'node:fs/promises'
import { resolve } from 'node:path'

async function readingHtml() {
    const path = resolve('./index.html')
    const file = await readFile(path)

    console.log(file.toString())
}

readingHtml()
```

Run it with:

```bash
node fsdemo.mjs
```

This reads the contents of `index.html` and prints them to the terminal.

The promise-based API works particularly well with `async`/`await`, which makes asynchronous file operations easier to read.

> **Interesting:** React can process JSX and transform it into JavaScript. With Node.js and the `fs` module, you can build your own programs that read files, transform their contents, and generate new files.

---

## Mini Template Engine

### What Is Templated HTML?

A template allows us to write HTML containing placeholders for dynamic values.

For example:

```html
<body>
    My name is {{name}} and I am {{age}} years old.
</body>
```

When the server processes this template, it can replace the placeholders with actual values:

```text
{{name}} → John
{{age}}  → 25
```

Template engines are commonly used for server-side rendering.

Popular JavaScript templating engines include:

- **EJS**
    
- **Handlebars**
    
- **Pug**
    

Other languages and frameworks have their own template engines, such as ERB in Ruby and Django Templates in Python.

### Building a Mini Template Engine with Node.js

We can build a very simple version ourselves using the `fs` module:

```js
import { readFile, writeFile } from 'node:fs/promises'
import { resolve } from 'node:path'

async function readingHtml() {
    const path = resolve('./index.html')
    let file = await readFile(path, 'utf8')

    const data = {
        name: 'John',
        age: 25,
        city: 'New York'
    }

    for (const [key, value] of Object.entries(data)) {
        file = file.replace(`{{${key}}}`, value)
    }

    await writeFile(resolve('./new.html'), file)
}

readingHtml()
```

Suppose `index.html` contains:

```html
<body>
    My name is {{name}} and I am {{age}} years old. I live in {{city}}.
</body>
```

The program replaces the placeholders with the values from `data` and creates:

```text
new.html
```

This is a very simplified example of the basic idea behind a template engine.

Real template engines provide many more features, such as:

- loops
    
- conditions
    
- escaping
    
- reusable templates
    
- partials
    
- layouts
    

---

## Client-Side vs Server-Side Rendering

One important difference is **where the HTML is generated**.

| Aspect | Client-Side Rendering | Server-Side Rendering |
|---|---|---|
| Where HTML is generated | Browser | Server |
| JavaScript runs in | Browser | Server and/or browser, depending on the framework |
| Typical example | React SPA | Server-rendered templates |
| Data can be inserted before HTML reaches the browser | ❌ | ✅ |

In **client-side rendering**, the browser receives JavaScript and uses it to build or update the UI.

In **server-side rendering**, the server generates HTML using data and sends the resulting HTML to the browser.

Template engines are commonly used for **server-side rendering**.

---

## Timers in Node.js

Node.js also provides timer APIs similar to those available in browsers.

For example:

```js
setTimeout(() => {
    console.log('hello')
}, 1000)
```

The callback runs after approximately one second.

Node.js also provides other timer functions such as:

```js
setInterval()
setImmediate()
```

These are useful when we need to schedule code to run later or repeatedly.

---

## Network Interactions

Node.js can participate in network communication in two major roles.

### As a Client

Node.js can send requests to other servers.

For example, a Node.js program could request data from a joke API:

```text
Node.js application
       ↓
   HTTP request
       ↓
    Joke API
       ↓
   HTTP response
       ↓
Node.js application
```

### As a Server

Node.js can also create a server that listens for incoming requests:

```text
Browser / Mobile App
        ↓
     Request
        ↓
   Node.js Server
        ↓
    Response
```

This server-side networking capability is one of the major reasons Node.js is popular for backend development.

---

## Scheduled Jobs

Applications often need to perform tasks automatically at specific times.

Examples include:

- sending an email every morning
    
- generating a daily report
    
- cleaning old files
    
- processing scheduled data
    
- sending notifications
    

On Unix-like operating systems, **cron** is a common way to schedule such tasks.

Node.js applications can also use packages such as `cron` when scheduling needs to happen inside the application itself.

Install the package:

```bash
npm install cron
```

Example:

```js
import { CronJob } from 'cron'

const job = new CronJob(
    '0 * * * * *',
    () => {
        console.log('This message runs every minute')
    }
)

job.start()
```

The cron expression determines when the job runs.

> **Challenge:** Create a Node.js program that sends a motivational quote by email every day at 8 AM. You could use `nodemailer` to send the email and experiment with creating a styled HTML email.

---

# Creating a Server with Node.js

Before creating a server, it helps to understand a few basic networking concepts.

### IP Address

An **IP address** identifies a device on a network.

You can think of it like the address of a house.

### Port Number

A **port** identifies a particular network service or application running on a machine.

You can think of it like an apartment number inside the house.

For example:

```text
127.0.0.1:3000
```

contains:

```text
127.0.0.1 → IP address
3000      → port
```

Together, they identify where a network service can be reached.

There are 65,536 TCP/UDP port numbers in the range `0–65535`. Some ports are conventionally associated with particular services, such as MongoDB's default port:

```text
27017
```

When the client and server are running on the same machine, we can usually use:

```text
localhost
```

or:

```text
127.0.0.1
```

### Does the Server Need to Know the Client's Address?

The server doesn't need to know the client's address **before** receiving a request.

The client initiates the connection, and the server listens for incoming connections.

Once the connection is established, the server can obtain information about the client from the network connection.

---

## Raw HTTP Server Using Node's `http` Module

Node.js provides a built-in `http` module that allows us to create an HTTP server without installing a framework.

```js
import http from 'node:http'

const server = http.createServer((req, res) => {
    console.log(req.url)

    if (req.url === '/home') {
        res.end('Welcome to home page')
    } else {
        res.end('Hello world')
    }
})

server.listen(3000, () => {
    console.log('Server started on port 3000')
})
```

Run it with:

```bash
node server.js
```

Then open:

```text
http://localhost:3000
```

in your browser.

If you visit:

```text
http://localhost:3000/home
```

the server responds with:

```text
Welcome to home page
```

This works, but handling larger applications directly with the low-level `http` module can become repetitive.

That's where frameworks such as Express.js become useful.

---

# Creating a Server with Express.js

**Express.js** is a lightweight and flexible framework for building web applications and APIs with Node.js.

Install it with:

```bash
npm install express
```

### CommonJS

```js
const express = require('express')
const app = express()

app.listen(3000, () => {
    console.log('Server started on port 3000')
})
```

### ES Modules

```js
import express from 'express'

const app = express()

app.listen(3000, () => {
    console.log('Server started on port 3000')
})
```

Express gives us a cleaner way to define routes, handle requests, work with middleware, and build APIs.

For example:

```js
app.get('/users', (req, res) => {
    res.json([
        { id: 1, name: 'John' },
        { id: 2, name: 'Jane' }
    ])
})
```

Now a client can send a `GET` request to:

```text
/users
```

and receive JSON data.

---

# APIs

## What Is an API?

**API** stands for **Application Programming Interface**.

An API is a defined way for one piece of software to communicate with another.

Think about a restaurant.

You don't walk into the kitchen and tell the chef how to prepare your food. Instead:

```text
You
 ↓
Menu
 ↓
Order
 ↓
Restaurant
 ↓
Food
```

The menu defines what you can order and what information you need to provide.

An API works in a similar way.

```text
Client
   ↓
API request
   ↓
Server
   ↓
API response
   ↓
Client
```

The client doesn't need to know how the server internally processes the request.

It only needs to know the API's contract:

- which URL to call
    
- which HTTP method to use
    
- what data to send
    
- what format to use
    
- what response to expect
    

### Why Are APIs Useful?

A backend can contain business logic that shouldn't be duplicated across every client.

For example, a company might have:

```text
             Backend API
             /    |    \
            /     |     \
       Website  Mobile  Admin App
```

The website, mobile application, and other clients can all communicate with the same backend API.

This allows the business logic to remain centralized on the server.

---

# REST APIs

**REST** stands for **Representational State Transfer**.

REST is an architectural style that provides conventions for designing web APIs.

It isn't a programming language or a library.

You don't install REST.

Instead, you design your API according to REST principles and conventions.

Other API styles also exist, including:

- RPC
    
- GraphQL
    

REST is especially common for HTTP-based web APIs.

---

## REST Conventions

### 1. Use JSON for Data

REST APIs commonly use **JSON** to represent data.

For example, a JavaScript object:

```js
const user = {
    name: 'Ghazi',
    age: 21,
    city: 'Delhi'
}
```

can be converted into a JSON string using:

```js
JSON.stringify(user)
```

Result:

```json
{"name":"Ghazi","age":21,"city":"Delhi"}
```

When receiving JSON, we can convert it back into a JavaScript object:

```js
JSON.parse('{"name":"Ghazi","age":21,"city":"Delhi"}')
```

JSON is language-independent, so clients and servers written in different programming languages can communicate using the same data format.

---

### 2. Build APIs Around Resources

A **resource** is an entity that the API manages.

For example, a movie-booking application might have:

```text
users
movies
theaters
bookings
```

An e-commerce application might have:

```text
users
products
orders
payments
```

The URL identifies the resource.

For example:

```text
/users
/products
/orders
```

A particular resource can be identified using an ID:

```text
/users/123
```

This means:

```text
user with ID 123
```

Resources are commonly represented using plural nouns.

For example:

```text
/users
```

rather than:

```text
/user
```

Resources can also be nested when there is a meaningful relationship:

```text
/users/123/orders
```

This could represent:

```text
orders belonging to user 123
```

---

### 3. Use HTTP Methods

HTTP methods tell the server what kind of operation the client wants to perform.

The commonly used methods are:

|Method|Typical purpose|
|---|---|
|`GET`|Retrieve data|
|`POST`|Create a resource|
|`PUT`|Replace/update a resource|
|`PATCH`|Partially update a resource|
|`DELETE`|Delete a resource|

For example:

```text
GET    /users
POST   /users
GET    /users/123
PATCH  /users/123
DELETE /users/123
```

The URL identifies the resource, while the HTTP method describes the operation.

---

### 4. Use HTTP Status Codes

HTTP responses include status codes that communicate the result of a request.

Some common examples are:

|Status|Meaning|
|---|---|
|`200`|OK|
|`201`|Created|
|`400`|Bad Request|
|`401`|Unauthorized|
|`403`|Forbidden|
|`404`|Not Found|
|`500`|Internal Server Error|

For example, if a client successfully creates a user, the server might respond with:

```text
201 Created
```

If the requested user doesn't exist:

```text
404 Not Found
```

Status codes allow the client to understand the result without having to inspect the entire response body.

---

### 5. Use the Request Body for Data

When creating or updating resources, larger pieces of data are commonly sent in the **request body**.

For example:

```http
POST /users
```

with a JSON body:

```json
{
    "name": "Ghazi",
    "age": 21,
    "city": "Delhi"
}
```

The server can then use this data to create the new user.

Sensitive data such as passwords should also be sent using appropriate secure mechanisms, such as HTTPS, rather than putting it in a URL.

---

### 6. Use Query Parameters for Filtering and Searching

Query parameters are useful when we want to filter, sort, or search for resources.

For example:

```text
/products?min_price=5000&max_price=10000&category=electronics
```

Here:

```text
min_price=5000
max_price=10000
category=electronics
```

are query parameters.

The API can use them to determine which products should be returned.

Another example:

```text
/products?category=phones
```

could request only products in the `phones` category.

---

# Putting It All Together

Node.js gives JavaScript access to capabilities beyond the browser.

We can use it to:

```text
Read and write files
        ↓
Run scripts
        ↓
Schedule tasks
        ↓
Make network requests
        ↓
Create HTTP servers
        ↓
Build APIs
```

Express makes building those servers and APIs easier.

REST then gives us a set of conventions for designing those APIs in a predictable way:

```text
Resource
   +
HTTP Method
   +
Request/Response
   +
Status Code
   +
JSON
```

Together, these concepts form a large part of the foundation of Node.js backend development.