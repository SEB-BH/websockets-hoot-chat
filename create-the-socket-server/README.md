<h1>
  <span class="headline">WebSockets with Socket.IO</span>
  <span class="subhead">Create the Socket.IO Server</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to attach a Socket.IO server to the same HTTP server used by an Express application.

## Review the current Express startup code

Open `server.js` in `express-api-hoot-back-end`.

Near the bottom, the application currently starts with code like this:

```javascript
app.listen(PORT, () => {
  console.log(`The express app is ready on port ${PORT}! 😀`)
})
```

`app.listen()` is convenient because it creates an HTTP server for the Express app and immediately starts it.

For Socket.IO, we need a reference to that HTTP server. Both Express routes and socket connections must use the same server and the same port. We will therefore create the HTTP server ourselves.

## Import Node's HTTP module

Near the other imports at the top of `server.js`, add:

```javascript
const http = require('http')
```

`http` is built into Node. We do not install it with npm.

## Import the Socket.IO server

Add another import:

```javascript
const { Server } = require('socket.io')
```

`Server` is the class we will use to create our Socket.IO server.

The top of your file should now look like this:

```javascript
const dotenv = require('dotenv').config()
const express = require('express')
const mongoose = require('mongoose')
const cors = require('cors')
const morgan = require('morgan')
const http = require('http')
const { Server } = require('socket.io')

const app = express()
```

If you get an error saying `Cannot find module 'socket.io'`, check that you installed `socket.io` in the **back-end** folder (check the setup section for instructions).

## Create an HTTP server

Immediately after creating the Express `app`, add:

```javascript
const app = express()
const server = http.createServer(app)
```

1. `app` is the Express application.
2. We pass `app` to `http.createServer()`.
3. Node creates an HTTP server that uses Express to handle regular HTTP requests.
4. We store that server in a variable named `server`.

Express has not been replaced. We have simply made the underlying HTTP server visible so Socket.IO can share it.

## Attach Socket.IO to the HTTP server

Below the new `server` variable, create the Socket.IO server:

```javascript
const io = new Server(server, {
  cors: {
    origin: 'http://localhost:5173',
  },
})
```

The first argument is the HTTP `server` Socket.IO will share with Express.

The `cors` option allows the Vite application at `http://localhost:5173` to open a connection during development. **MAKE SURE REACT IS RUNNING ON 5173 - IF NOT, CHANGE THE PORT NUMBER HERE**

Your existing Express middleware already includes this line:

```javascript
app.use(cors())
```

Keep it. The two CORS configurations serve related but separate parts of the application:

| Configuration | Used by |
| --- | --- |
| `app.use(cors())` | Regular Express HTTP routes such as `/hoots` and `/auth/sign-in`. |
| `new Server(server, { cors: ... })` | The Socket.IO handshake and connection. |

> If Vite starts on a different port, the Socket.IO `origin` must match the exact URL shown by Vite. For this lesson, stop other Vite apps so Hoot can use port `5173`.

## Listen with the HTTP server

At the bottom of `server.js`, replace `app.listen()` with `server.listen()`:

```javascript
server.listen(PORT, () => {
  console.log(`The express app is ready on port ${PORT}! 😀`)
})
```

Do not keep both calls. Only one server should listen on port `3000`.


### Stop and check: regular HTTP still works

Restart the back-end and front-end applications.

In the browser:

1. Sign in to Hoot.
2. Open the Hoot list.
3. Open a Hoot details page.

The existing REST API should still work. This proves the new HTTP `server` is correctly passing ordinary requests to the Express `app`.

If you get an error that says `EADDRINUSE`, another process is already using port `3000`, or both `app.listen()` and `server.listen()` are still present.

## Check the Socket.IO endpoint

Socket.IO creates its own endpoint on the server. In the browser, vist:

```bash
http://localhost:3000/socket.io/?EIO=4&transport=polling
```

You should see a JSON response that begins with a `0` followed by an object containing values such as `sid`, `upgrades`, `pingInterval`, and `pingTimeout`:

```plaintext
0{"sid":"f6FQyc2eAN6uPZdtAAAA","upgrades":["websocket"],"pingInterval":25000,"pingTimeout":20000,"maxPayload":1000000}
```

The values may not exactly match the example. The important result is that you receive a Socket.IO handshake instead of a `404` response.

This does not yet test our React client. It only proves that Socket.IO is attached and listening in ou rbackend.

## Listen for connections

After the Express routes and before `server.listen()`, add:

```javascript
io.on('connection', (socket) => {
  console.log('Socket connected: ', socket.id)

  socket.on('disconnect',() => {
    console.log('Socket disconnected: ', socket.id)
  })
})
```

Let's read this code carefully:

- `io.on('connection', ...)` listens for a browser connection.
- Socket.IO passes the new connection into the callback as `socket`.
- `socket.id` is a temporary ID that helps us distinguish open browser connections while debugging.
- Inside that connection, we listen for its future `disconnect` event.

The socket ID is not a user ID. It changes after a refresh or reconnection, so we will use it only for debugging and for a temporary React list key.

Your new server setup now has this overall shape:

```javascript
const dotenv = require('dotenv').config()
const express = require('express')
const mongoose = require('mongoose')
const cors = require('cors')
const morgan = require('morgan')
const http = require('http')
const { Server } = require('socket.io')

const app = express()
const server = http.createServer(app)

const io = new Server(server, {
  cors: {
    origin: 'http://localhost:5174',
  },
})

const PORT = process.env.PORT ? process.env.PORT : "3000"

// Keep the existing database connection here
// Keep the existing Express middleware here
// Keep the existing Express routes here

// add new code here
io.on('connection', (socket) => {
  console.log('Socket connected: ', socket.id)

  socket.on('disconnect',() => {
    console.log('Socket disconnected: ', socket.id)
  })
})

// this code didn't change
server.listen(PORT, () => {
  console.log(`The express app is ready on port ${PORT}! 😀`)
})
```

### Stop and predict

After restarting the server, you will probably **not** see `Socket connected` yet. Why?

The server is ready to accept connections, but our React application has not opened one. We will build that client next.
