<h1>
  <span class="prefix">MERN Stack</span>
  <span class="headline">WebSockets with Socket.IO</span>
</h1>

## About

In this module, students will add a small real-time chat page to the completed Hoot application. The chat will use **Socket.IO**, a library that gives us an event-based way to create real-time communication between an Express server and a React client.

## Learning objectives

By the end of this module, students will be able to:

- Describe the main difference between an HTTP request and a persistent socket connection.
- Attach a Socket.IO server to an Express application's HTTP server.
- Connect a React client to a Socket.IO server.
- Use matching `emit` and `on` events to move data in both directions.
- Store incoming messages in React state and render them.
- Use logs, connection IDs, two browser tabs, and WebSocket frames to debug a real-time feature.

**This assumes you have completed the Hoot Frontend and Hoot Backend already**

- Hoot Frontend Lecture Code (solution branch): [Frontend](https://github.com/SEB-14-Bahrain/hoot-frontend/tree/solution)
- Hoot Backend Lecture Code: [Backend](https://github.com/SEB-14-Bahrain/express-api-hoot-back-end/tree/main)
  
#### Solution code

- Hoot Frontend with Websockets complete Solution Code (websockets branch): [Frontend with websockets](https://github.com/SEB-14-Bahrain/hoot-frontend/tree/websockets)
- Hoot Backend with Websockets complete Solution Code (websockets branch): [Backend with websockets](https://github.com/SEB-14-Bahrain/express-api-hoot-back-end/tree/websockets)

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

<!-- | [Test and Debug](./test-and-debug/README.md) | Testing the complete flow with multiple clients and browser tools. | -->

## Level Up content

| Lesson | Skills |
| --- | --- |
| [Level Ups](./level-ups/README.md) | Planning message history, rooms, socket authentication, and other improvements. |


## References

📖 [Reference Materials](./references/README.md)


