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

**If the database changes later, the browser does not automatically know. React must make another request to receive newer data.**

### What changes in a real-time feature?

A chat message should appear in other browsers immediately. We do not want every browser to repeatedly ask:

> Are there any new messages now? How about now? How about now?

Instead, the browser opens a connection and keeps it available. Once the connection exists, either side can send an event when something happens.

<!-- | HTTP request and response | Socket connection |
| --- | --- |
| Usually begins with a client request. | Begins by opening a connection. |
| The server sends one response to that request. | Client and server can send many events. |
| The request and response cycle ends. | The connection remains available until it disconnects. |
| Routes use method and path combinations. | Communication uses named events. |
| Postman is often enough for testing. | Two browsers, logs, and WebSocket frames are more useful. | -->

## What is Socket.IO?

Socket.IO gives us a simple event-based API for real-time communication. It also provides conveniences such as reconnection and broadcasting, which we will go over later.

Socket.IO is not the same thing as the browser's raw `WebSocket` API. A plain WebSocket client cannot communicate directly with a Socket.IO server because Socket.IO adds its own protocol. That is why we installed matching Socket.IO packages on both sides.

For this lesson, Socket.IO lets us focus on three ideas:

- Connect a browser.
- Emit a named event.
- Listen for a named event.

## The important objects (we will focus primarily on these during the lesson)

On the server, we will use both `io` and `socket`.

| Name | Meaning on the server |
| --- | --- |
| `io` | The Socket.IO server and its collection of connected clients. |
| `socket` | One particular browser connection. |


If Aisha opens Hoot Chat in two tabs, she has:
- 2 browser tabs
- 2 sockets
- only 1 user

Now imagine Sara sends a message in Hoot Chat.  Sara's browser tab uses it's `socket` to send the message to the server (using `emit`).  The server listens for this message.  Then the server uses `io` to send the message to everyone's `socket`.  The browser listens for that "announcement" from the server.

So sessentially:
- a `socket` is one open connection in the browser
- `io` is on the server managing all the connections
- the Hoot Chat is the feature we will put in our app that uses those connections

## Events must form matching pairs

An emitted event needs a listener with the same event name.

### Some example code (we're not implementing yet, just looking)

Sara sends a message - React will emit:

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

Sara and Aisha both see the message in Hoot Chat because React is listening for an "announcemnt":

```javascript
socket.on('chat message', (newMessage) => {
  console.log(newMessage)
})
```

The words `chat message` are not built into Socket.IO. We chose that event name. **If one side uses `chat-message` (with a dash) and the other uses `chat message` (a space instead of a dash), the listener will never run.**

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

This behavior demonstrates the difference between **real-time delivery** and **persistence**.
