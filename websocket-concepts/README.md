<h1>
  <span class="headline">WebSockets with Socket.IO</span>
  <span class="subhead">From HTTP to WebSockets</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to compare an HTTP request with a persistent socket connection and describe an event's path through our chat application.

## Start with the communication pattern we already know

Most of Hoot currently uses HTTP.

When the React app requests a list of hoots, the flow looks like this:

1. React sends a `GET /hoots` request.
2. Express receives the request.
3. Express sends one response.
4. That request and response cycle ends.

If the database changes later, the browser does not automatically know. React must make another request to receive newer data.

This pattern is a good fit for CRUD operations. A user performs an action, the client sends a request, and the server returns a response.

## What changes in a real-time feature?

A chat message should appear in other browsers immediately. We do not want every browser to repeatedly ask:

> Are there any new messages now? How about now? How about now?

Instead, the browser opens a connection and keeps it available. Once the connection exists, either side can send an event when something happens.

| HTTP request and response | Socket connection |
| --- | --- |
| Usually begins with a client request. | Begins by opening a connection. |
| The server sends one response to that request. | Client and server can send many events. |
| The request and response cycle ends. | The connection remains available until it disconnects. |
| Routes use method and path combinations. | Communication uses named events. |
| Postman is often enough for testing. | Two browsers, logs, and WebSocket frames are more useful. |

## What is Socket.IO?

Socket.IO gives us a simple event-based API for real-time communication. It normally attempts to use a WebSocket transport and can fall back to HTTP long-polling when needed. It also provides conveniences such as reconnection and broadcasting.

Socket.IO is not the same thing as the browser's raw `WebSocket` API. A plain WebSocket client cannot communicate directly with a Socket.IO server because Socket.IO adds its own protocol. That is why we installed matching Socket.IO packages on both sides.

For this first lesson, Socket.IO lets us focus on three ideas:

- Connect a browser.
- Emit a named event.
- Listen for a named event.

## The important objects

On the server, we will use both `io` and `socket`.

| Name | Meaning on the server |
| --- | --- |
| `io` | The Socket.IO server and its collection of connected clients. |
| `socket` | One particular browser connection. |

This difference matters when the server sends data:

```javascript
socket.emit('chat message', message)
```

The code above sends to one socket.

```javascript
io.emit('chat message', message)
```

The code above sends to every connected socket, including the browser that originally sent the message. Our global chat will use `io.emit()`.

On the React side, `socket` represents that browser's connection to the server.

## Events must form matching pairs

An emitted event needs a listener with the same event name.

### React to Express

React will emit:

```javascript
socket.emit('chat message', messageData)
```

Express will listen:

```javascript
socket.on('chat message', (messageData) => {
  console.log(messageData)
})
```

### Express to every React client

Express will emit:

```javascript
io.emit('chat message', newMessage)
```

React will listen:

```javascript
socket.on('chat message', (newMessage) => {
  console.log(newMessage)
})
```

The words `chat message` are not built into Socket.IO. We chose that event name. If one side uses `chat-message` and the other uses `chat message`, the listener will never run.

## Trace one message before writing code

Our completed message flow will be:

1. A user submits the React form.
2. React creates a small `messageData` object.
3. React emits a `chat message` event to Express.
4. Express listens for that event and receives `messageData`.
5. Express creates a `newMessage` object with an ID.
6. Express emits a `chat message` event to every connected browser.
7. Each browser listens for that event.
8. Each browser adds `newMessage` to its own React state.
9. React re-renders the visible message list.

Notice that the message travels in two directions. We will test each direction separately before displaying anything.

## What a socket does not do

A socket moves data between connected programs. It does not automatically save that data.

In this lesson:

- Express does not save messages in an array or in MongoDB.
- React stores only the messages received while that Chat page is mounted.
- A newly connected browser receives only future messages.
- Refreshing the page creates fresh React state.

This behavior will become an important final check. It demonstrates the difference between **real-time delivery** and **persistence**.

## Knowledge check

Answer these questions before continuing:

1. Why can the Express server send a chat event without waiting for a new HTTP request?
2. What is the difference between `io` and `socket` on the server?
3. What must match between `emit()` and `on()`?
4. Will Socket.IO automatically save our messages in MongoDB?

### Answer check

1. The socket connection remains open and available for events.
2. `io` represents the Socket.IO server and all connections; `socket` represents one connection.
3. The event name must match exactly.
4. No. We must write separate persistence code if we want stored history.
