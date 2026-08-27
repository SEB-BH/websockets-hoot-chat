<h1>
  <span class="headline">WebSockets with Socket.IO</span>
  <span class="subhead">Instructor Guide</span>
</h1>

## Lesson goal

Students should finish with one small, working global chat and a clear event-flow mental model. The primary outcome is not feature richness. It is the ability to identify which program emitted an event, which program listened, where the data can be inspected, and when React state changes.

## User story

> As a signed-in Hoot user, I want to send and receive live chat messages so I can communicate with other users who currently have Chat open.

## Acceptance criteria

- A signed-in user can navigate to `/chat`.
- The page displays its current connection status.
- Submitting non-empty text sends a `chat message` event to Express.
- Express broadcasts a message object to all currently connected Chat pages.
- Each connected page renders each new message once without refreshing.
- Refreshing clears that tab's visible history because persistence is outside the MVP.

## Suggested pacing

| Section | Suggested time | Main evidence |
| --- | ---: | --- |
| Setup | 15 minutes | Both dependencies installed in the correct projects. |
| Concepts | 20 minutes | Students can trace both event directions. |
| Socket.IO server | 20 minutes | Handshake endpoint responds and REST still works. |
| React connection | 25 minutes | Matching socket IDs and successful cleanup. |
| Controlled form | 15 minutes | Correct object logged without any socket emission. |
| Client-to-server event | 15 minutes | Object appears in Express terminal. |
| Server broadcast | 20 minutes | Same event appears in two browser consoles. |
| React state and rendering | 20 minutes | Same message appears once in two interfaces. |
| Testing and recap | 20 minutes | Students isolate failures using the stage table. |

Total: approximately 2.5 hours. This can be split after the React connection or after the client-to-server event.

## Teaching decisions in this lesson

### Socket.IO instead of the raw WebSocket API

Socket.IO provides approachable `emit` and `on` methods, reconnection, and broadcasting. Explain clearly that it is a WebSocket-oriented real-time library with its own protocol, not a thin alias for the native browser `WebSocket` class.

### One global chat

Rooms require another join event, room membership, route parameters, and decisions about leaving old rooms. A global chat keeps the first event path visible.

### No database

Students already know MongoDB, but adding a model too early makes it difficult to tell whether a failure belongs to transport, validation, persistence, or rendering. The refresh test intentionally demonstrates that real-time delivery and persistence are separate concerns.

### No socket authentication

The React route is protected and passes the current username as a prop. The socket server still trusts client data, which is not production-safe. State this plainly, then reserve token verification for a later lesson or level-up.

### Event code remains in `server.js`

The handler could be extracted into modules. Keeping it in one file for the first build makes `io`, `socket`, `on`, and `emit` visible together. Refactoring can follow once students understand the flow.

### Named listener functions and explicit cleanup

The React effect is slightly longer because every callback has a name and every `on` has a matching `off`. This makes duplicate-event bugs easier to explain and fix.

## Before class

1. Confirm the supplied Hoot back-end runs on port `3000`.
2. Confirm the supplied Hoot front-end runs on port `5173`.
3. Confirm `VITE_BACK_END_SERVER_URL` points to `http://localhost:3000`.
4. Create two test users if you plan to demonstrate distinct usernames.
5. Close unrelated Vite projects so Hoot receives port `5173`.
6. Test the `curl` handshake command on the classroom machines.
7. Keep the Express terminal, browser console, and Chat page visible during the demo.

## Recommended live-coding rhythm

For each behavior:

1. State one expected result.
2. Add only the code required for that result.
3. Run the check.
4. Stop if the evidence does not match.
5. Ask students which boundary the data has crossed.

The most important pause points are:

| Pause | Ask students | Evidence |
| --- | --- | --- |
| After `server.listen()` | Did ordinary HTTP survive the server change? | Hoot list still loads. |
| After `socket.connect()` | Is this one connection visible on both sides? | Matching socket IDs. |
| After form submit handler | Is the object correct before networking? | Browser log only. |
| After client `emit` | Has data crossed into Express? | Express terminal log. |
| After server `io.emit` | Has data crossed back to each client? | Two browser logs. |
| After `setMessages` | Has transport data become UI state? | Two visible lists. |
| After refresh | Was anything persisted? | Refreshed list is empty. |

## Board model

Keep this mapping visible while coding:

| Sender | Sends | Receiver | Listens |
| --- | --- | --- | --- |
| React | `socket.emit('chat message', messageData)` | Express | `socket.on('chat message', handler)` |
| Express | `io.emit('chat message', newMessage)` | React clients | `socket.on('chat message', handler)` |

Also keep the server distinction visible:

- `io` means the server and its connected clients.
- `socket` means one client connection.

## React Strict Mode note

In development, Strict Mode may run an effect setup, cleanup, and setup again. Students may see a quick extra connection and disconnection.

The important distinction is:

- A brief extra lifecycle log during development can be normal.
- Rendering every message twice is not the desired result.

For duplicate messages:

1. Confirm each `socket.on()` has a matching `socket.off()` using the same named function.
2. Confirm the effect dependency array is empty.
3. Perform a full browser reload after changing `src/socket.js` because hot module replacement can leave an earlier module-level connection alive.

## Common misconceptions

### "The message should display as soon as React emits it"

In this design, React does not add its own outgoing message directly to state. It waits for the server broadcast. This gives the sender the same server-created object that every other client receives and avoids adding the same message twice.

### "`io.emit()` sends to Express"

On the server, `io.emit()` sends outward to connected clients. The server receives client data through `socket.on()`.

### "The socket ID is the user ID"

The socket ID represents one temporary connection. Refreshing produces a new one, and two tabs for one user have separate IDs.

### "Socket.IO saves message history"

It delivers events. Persistence must be deliberately added with a model and retrieval flow.

### "The Express CORS middleware already covers everything"

The Socket.IO server has its own CORS option for its handshake. Keep both configurations in this two-origin development setup.

### "Postman is the best first test"

The most useful beginner test is the actual Socket.IO browser client, because Socket.IO adds a protocol above raw WebSocket frames. Browser tabs, logs, and DevTools show the same client used by the finished feature.

## Suggested formative questions

1. Why did we replace `app.listen()` with `server.listen()`?
2. Why do the event strings need to match but callback parameter names do not?
3. Why is the server's incoming listener attached to `socket`?
4. Why does the server broadcast with `io`?
5. Why does the React effect return a cleanup function?
6. Why use the callback form of `setMessages()`?
7. Why does one refresh remove only that tab's history?
8. Why should a production server not trust `messageData.username`?

## Answer guide

1. Socket.IO needs the same explicit HTTP server used by Express.
2. Event names route events; parameter names are local variables.
3. The event came from one particular client connection.
4. The message must reach every currently connected client, including the sender.
5. To remove listeners and close the page-specific connection when Chat unmounts.
6. Socket events may arrive over time, so each update should use the latest state.
7. Each tab owns separate in-memory React state, and nothing was saved remotely.
8. Client data can be modified, so identity should come from a verified token.

## Code review checklist

- Only `server.listen()` remains.
- `http.createServer(app)` receives the Express app.
- Socket.IO receives the HTTP server.
- Socket.IO CORS matches the Vite origin.
- The client URL comes from `VITE_BACK_END_SERVER_URL`.
- `autoConnect` is spelled correctly.
- The Chat route passes `user` as a prop.
- The form rejects whitespace-only text.
- Event names match exactly.
- The server uses `io.emit()` for the global broadcast.
- The React listener uses a functional state update.
- Every listener has matching cleanup.
- Two clients each render one copy.

## Natural stopping point

End the core lesson after students run the persistence-boundary test. If time remains, a timestamp or message-length rule is a better first level-up than rooms or authentication because it adds little new architecture.
