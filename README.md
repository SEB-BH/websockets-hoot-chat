<h1>
  <span class="prefix">MERN Stack</span>
  <span class="headline">WebSockets with Socket.IO</span>
</h1>

## About

In this module, students will add a small real-time chat page to the completed Hoot application. The chat will use **Socket.IO**, a library that gives us an event-based way to create real-time communication between an Express server and a React client.

The feature is intentionally small: every signed-in user who opens the Chat page joins one global chat. Messages are shared immediately with every connected browser, but they are not saved in MongoDB. This keeps the first WebSocket lesson focused on connections, events, and React state.

Although the final code is short, the lesson builds it in small steps. Students will check the Express terminal, browser console, page interface, and browser network tools as each part is connected.

## Learning objectives

By the end of this module, students will be able to:

- Describe the main difference between an HTTP request and a persistent socket connection.
- Attach a Socket.IO server to an Express application's HTTP server.
- Connect a React client to a Socket.IO server.
- Use matching `emit` and `on` events to move data in both directions.
- Store incoming messages in React state and render them.
- Use logs, connection IDs, two browser tabs, and WebSocket frames to debug a real-time feature.

## Content

| Lesson | Skills |
| --- | --- |
| [Setup](./setup/README.md) | Preparing the completed Hoot front-end and back-end applications. |
| [From HTTP to WebSockets](./websocket-concepts/README.md) | Building a mental model for persistent, event-based communication. |
| [Create the Socket.IO Server](./create-the-socket-server/README.md) | Attaching Socket.IO to the Express HTTP server. |
| [Connect the React Client](./connect-the-react-client/README.md) | Opening and cleaning up a Socket.IO connection in React. |
| [Build the Chat Interface](./build-the-chat-interface/README.md) | Creating and checking a controlled chat form. |
| [Send a Chat Event](./send-a-chat-event/README.md) | Emitting data from the React client and receiving it on the server. |
| [Broadcast a Chat Event](./broadcast-a-chat-event/README.md) | Sending one server event to every connected browser. |
| [Render Chat Messages](./render-chat-messages/README.md) | Updating React state when socket events arrive. |
| [Test and Debug](./test-and-debug/README.md) | Testing the complete flow with multiple clients and browser tools. |

## Level Up content

| Lesson | Skills |
| --- | --- |
| [Level Ups](./level-ups/README.md) | Planning message history, rooms, socket authentication, and other improvements. |

## What students will build

The completed feature has one protected route:

```plaintext
/chat
```

On that page, a signed-in user can:

- See whether the browser is connected to the chat server.
- Enter a message in a controlled React form.
- Send the username and message text to the Express server.
- Receive the server's broadcast.
- See messages sent from other open browsers without refreshing the page.

## Deliberate limitations

This is a learning build, not a production chat system.

- Messages are not stored in MongoDB.
- Refreshing the page clears the visible message list.
- A user who opens Chat later does not receive older messages.
- There is one global chat rather than a room for each hoot.
- The React route is protected, but the socket connection itself is not yet authenticated.

These limitations are useful. They let students see exactly what a socket does before adding databases, rooms, middleware, or delivery guarantees.

## References

📖 [Reference Materials](./references/README.md)

## Internal

### Prerequisites

- Completed Express API - Hoot Back-End
- Completed React - Hoot Front-End
- React state and controlled forms
- React `useEffect`
- Express server setup
- Basic HTTP request and response concepts

### Suggested delivery time

Approximately 2 to 2.5 hours, including the two-browser test and discussion checks.

### Resources

✏️ [Instructor Guide](./internal-resources/instructor-guide.md)

🏗️ [Release Notes](./internal-resources/release-notes.md)
