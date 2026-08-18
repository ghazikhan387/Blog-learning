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

## Introduction

**Node.js** allows JavaScript to run outside the browser. This gives JavaScript access to capabilities such as the file system, networking, servers, and the operating system.

This markdown covers:

- Running Node.js scripts
- File system operations
- Template engines and rendering
- Timers and scheduled jobs
- Network communication
- Creating servers with Node.js and Express
- APIs and REST API conventions

---

## Running Node.js Scripts

Normally, a Node.js file is run with:

```bash
node index.js
```

On Linux and macOS, a script can also be executed directly:

```bash
./index.js
```

For this, the file needs execute permission and a **shebang**:

```javascript
#!/usr/bin/env node

console.log('hello');
```

The shebang tells the operating system to use Node.js to execute the file.

On Windows, executable scripts are commonly handled through file associations, `.cmd`/`.bat` wrappers, or tools such as `npm`.

---

## File Permissions

On Linux, file permissions can be checked with:

```bash
ls -l
```

For example:

```text
-rw-r--r-- index.js
```

The permissions are:

| Permission | Value |
| ---------- | ----: |
| Read       | `4`   |
| Write      | `2`   |
| Execute    | `1`   |

For example:

```bash
chmod 700 index.js
```

means:

```text
Owner  → read + write + execute
Group  → no permissions
Others → no permissions
```

The file can then be executed with:

```bash
./index.js
```

---

## File System (`fs`)

Node.js provides the built-in `fs` module for working with files and directories.

It can:

- Read files
- Create files
- Write files
- Update files
- Delete files
- Work with directories

### Callback API

```javascript
import { readFile } from 'node:fs';

readFile('./index.html', (err, data) => {
    if (err) throw err;
    console.log(data);
});
```

### Promise API

Node.js also provides `fs/promises`:

```javascript
import { readFile } from 'node:fs/promises';

async function readHtml() {
    const file = await readFile('./index.html', 'utf8');
    console.log(file);
}

readHtml();
```

The promise-based API works well with `async`/`await`.

---

## Template Engines

A template contains placeholders for dynamic data:

```html
<body>
    My name is {{name}} and I am {{age}} years old.
</body>
```

A server can replace them with actual values:

```text
{{name}} → John
{{age}}  → 25
```

Popular JavaScript template engines include:

- **EJS**
- **Handlebars**
- **Pug**

A very simple template system could be created using `fs`:

```javascript
import { readFile, writeFile } from 'node:fs/promises';

const file = await readFile('./index.html', 'utf8');

const data = {
    name: 'John',
    age: 25
};

let output = file;

for (const [key, value] of Object.entries(data)) {
    output = output.replace(`{{${key}}}`, value);
}

await writeFile('./new.html', output);
```

Real template engines provide additional features such as loops, conditions, escaping, layouts, and reusable templates.

---

## Client-Side vs Server-Side Rendering

One important difference is **where the HTML is generated**.

| Aspect                                    | Client-Side Rendering | Server-Side Rendering     |
| ----------------------------------------- | --------------------- | ------------------------- |
| HTML generated in                         | Browser               | Server                    |
| JavaScript runs in                        | Browser               | Server and/or browser     |
| Example                                   | React SPA             | Server-rendered templates |
| Data inserted before HTML reaches browser | ❌                    | ✅                        |

In **client-side rendering**, the browser receives JavaScript and uses it to build or update the UI.

In **server-side rendering**, the server generates HTML using data and sends the resulting HTML to the browser.

Template engines are commonly used for server-side rendering.

---

## Timers

Node.js provides timer APIs such as:

```javascript
setTimeout(() => {
    console.log('hello');
}, 1000);
```

Other timer functions include:

```javascript
setInterval();
setImmediate();
```

They allow code to run later or repeatedly.

---

## Network Communication

Node.js can act as both a **client** and a **server**.

### As a Client

A Node.js application can send requests to another server:

```text
Node.js application
       ↓
   HTTP request
       ↓
    API server
       ↓
   HTTP response
       ↓
Node.js application
```

### As a Server

Node.js can listen for incoming requests:

```text
Browser / Mobile App
        ↓
      Request
        ↓
   Node.js Server
        ↓
     Response
```

---

## Scheduled Jobs

Applications often need to run tasks automatically, such as:

- Generating reports
- Cleaning files
- Processing data
- Sending notifications

On Unix-like systems, **cron** is commonly used for scheduling.

Node.js applications can also use packages such as `cron`:

```bash
npm install cron
```

Example:

```javascript
import { CronJob } from 'cron';

const job = new CronJob('* * * * *', () => {
    console.log('Running every minute');
});

job.start();
```

---

## Creating a Server with Node.js

Before creating a server, understand two basic networking concepts.

### IP Address

An **IP address** identifies a device on a network.

```text
127.0.0.1
```

### Port

A **port** identifies a particular network service on a device.

```text
127.0.0.1:3000
```

Here:

```text
127.0.0.1 → IP address
3000      → port
```

`localhost` usually refers to the current machine.

---

## HTTP Server with Node.js

Node.js provides the built-in `http` module:

```javascript
import http from 'node:http';

const server = http.createServer((req, res) => {
    if (req.url === '/home') {
        res.end('Welcome home');
    } else {
        res.end('Hello world');
    }
});

server.listen(3000, () => {
    console.log('Server started on port 3000');
});
```

Run it with:

```bash
node server.js
```

Then visit:

```text
http://localhost:3000
```

The low-level `http` module works well for understanding how servers operate, but larger applications often use a framework such as **Express.js**.

---

## Express.js

**Express.js** is a lightweight framework for building web applications and APIs with Node.js.

Install it with:

```bash
npm install express
```

### ES Modules

```javascript
import express from 'express';

const app = express();

app.listen(3000, () => {
    console.log('Server started on port 3000');
});
```

Express makes it easier to define routes, handle requests, use middleware, and build APIs.

Example:

```javascript
app.get('/users', (req, res) => {
    res.json([
        { id: 1, name: 'John' },
        { id: 2, name: 'Jane' }
    ]);
});
```

A `GET /users` request now returns JSON data.

---

## APIs

### What Is an API?

**API** stands for **Application Programming Interface**.

An API defines how different pieces of software communicate.

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

The client does not need to know how the server internally processes the request. It only needs to know the API's contract, such as:

- URL
- HTTP method
- Data to send
- Data format
- Expected response

APIs allow multiple applications to communicate with the same backend:

```text
             Backend API
             /    |    \
            /     |     \
       Website  Mobile  Admin App
```

---

## REST APIs

**REST** stands for **Representational State Transfer**.

REST is an architectural style for designing web APIs. It is not a programming language or library.

Other API styles include:

- RPC
- GraphQL

REST is commonly used with HTTP.

---

### REST Conventions

#### 1. Use JSON

REST APIs commonly use **JSON** to represent data.

JavaScript object:

```javascript
const user = {
    name: 'Ghazi',
    age: 21
};
```

Convert it to JSON:

```javascript
JSON.stringify(user);
```

Convert JSON back to an object:

```javascript
JSON.parse('{"name":"Ghazi","age":21}');
```

JSON is language-independent, allowing applications written in different languages to communicate.

#### 2. Build APIs Around Resources

A **resource** is an entity managed by the API.

For example:

```text
/users
/products
/orders
```

A specific resource can be identified with an ID:

```text
/users/123
```

Nested resources can represent relationships:

```text
/users/123/orders
```

This could represent orders belonging to user `123`.

#### 3. Use HTTP Methods

HTTP methods describe the requested operation.

| Method   | Purpose                     |
| -------- | --------------------------- |
| `GET`    | Retrieve data               |
| `POST`   | Create a resource           |
| `PUT`    | Replace a resource          |
| `PATCH`  | Partially update a resource |
| `DELETE` | Delete a resource           |

Example:

```text
GET    /users
POST   /users
GET    /users/123
PATCH  /users/123
DELETE /users/123
```

The **URL identifies the resource**, while the **HTTP method describes the operation**.

#### 4. Use HTTP Status Codes

Status codes communicate the result of a request.

| Status | Meaning               |
| -----: | --------------------- |
| `200`  | OK                    |
| `201`  | Created               |
| `400`  | Bad Request           |
| `401`  | Unauthorized          |
| `403`  | Forbidden             |
| `404`  | Not Found             |
| `500`  | Internal Server Error |

For example:

```text
POST /users
→ 201 Created
```

or:

```text
GET /users/999
→ 404 Not Found
```

#### 5. Use the Request Body for Data

Data for creating or updating resources is commonly sent in the request body.

```text
POST /users
```

```json
{
    "name": "Ghazi",
    "age": 21,
    "city": "Delhi"
}
```

Sensitive information should be transmitted securely, typically using HTTPS.

#### 6. Use Query Parameters for Filtering

Query parameters are useful for filtering, searching, and sorting.

```text
/products?category=phones
```

Multiple parameters can be used:

```text
/products?min_price=5000&max_price=10000&category=electronics
```

---

## Putting It All Together

Node.js allows JavaScript to work beyond the browser:

```text
Run scripts
    ↓
Work with files
    ↓
Schedule tasks
    ↓
Make network requests
    ↓
Create HTTP servers
    ↓
Build APIs
```

Express makes server and API development easier.

REST provides conventions for organizing those APIs:

```text
Resource
   +
HTTP Method
   +
Request / Response
   +
Status Code
   +
JSON
```

Together, these concepts form an important foundation for **Node.js backend development**.