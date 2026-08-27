<h1>
  <span class="headline">WebSockets with Socket.IO</span>
  <span class="subhead">Level Ups</span>
</h1>

The MVP should remain small until its connection, event flow, listener cleanup, and two-client broadcast all work. Add the following improvements one at a time and repeat the complete test after each change.

## Level Up: add minimal chat styling

Add class names to the Chat JSX:

```javascript
<main className='chat-page'>
  <header className='chat-header'>
    <h1>Hoot Chat</h1>
    <p className={isConnected ? 'chat-status connected' : 'chat-status'}>
      Status: {isConnected ? 'Connected' : 'Disconnected'}
    </p>
  </header>

  <section className='chat-messages' aria-live='polite'>
    <h2>Messages</h2>

    {messages.length === 0 && (
      <p>No messages yet. Start the conversation!</p>
    )}

    {messages.map((message) => (
      <article className='chat-message' key={message.id}>
        <strong>{message.username}</strong>
        <p>{message.text}</p>
      </article>
    ))}
  </section>

  <form className='chat-form' onSubmit={handleSubmit}>
    {/* Keep the existing label, input, and button here */}
  </form>
</main>
```

Add a few rules to the application's global stylesheet:

```css
.chat-page {
  width: min(700px, 100%);
  margin: 0 auto;
  padding: 2rem 1rem;
}

.chat-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
}

.chat-status {
  color: var(--color-error);
}

.chat-status.connected {
  color: var(--color-success);
}

.chat-messages {
  min-height: 300px;
  margin: 1rem 0;
  padding: 1rem;
  overflow-y: auto;
  border: 1px solid var(--color-border);
  border-radius: 0.5rem;
  background: var(--color-surface);
}

.chat-message {
  margin-bottom: 0.75rem;
  padding-bottom: 0.75rem;
  border-bottom: 1px solid var(--color-border);
}

.chat-message p {
  margin: 0.25rem 0 0;
}

.chat-form {
  display: flex;
  align-items: end;
  gap: 0.5rem;
}

.chat-form input {
  flex: 1;
}
```

Adjust the variable names only if your Hoot stylesheet uses different variables.

## Level Up: add a character limit

Add a front-end limit to the input:

```javascript
<input
  id='message-input'
  name='message'
  type='text'
  value={messageText}
  onChange={handleChange}
  maxLength={280}
  autoComplete='off'
/>
```

Then enforce the rule again on the server. Client validation improves the interface, but a client can be modified or bypassed.

```javascript
socket.on('chat message', (messageData) => {
  if (!messageData || typeof messageData.text !== 'string') {
    return
  }

  const text = messageData.text.trim()

  if (!text || text.length > 280) {
    return
  }

  const newMessage = {
    id: `${socket.id}-${Date.now()}`,
    username: messageData.username,
    text: text,
  }

  io.emit('chat message', newMessage)
})
```

Check blank, one-character, 280-character, and 281-character inputs.

## Level Up: add a timestamp

Create the timestamp on the server so every browser receives the same value:

```javascript
const newMessage = {
  id: `${socket.id}-${Date.now()}`,
  username: messageData.username,
  text: messageData.text,
  createdAt: new Date().toISOString(),
}
```

Format it in the browser:

```javascript
<small>
  {new Date(message.createdAt).toLocaleTimeString()}
</small>
```

## Level Up: save message history

Real-time delivery and persistence solve different problems:

- Socket.IO delivers new messages immediately.
- MongoDB stores messages so they can be retrieved later.

A common next architecture is:

1. Create a `Message` model.
2. Add a protected `GET /messages` route for recent history.
3. Fetch history when Chat mounts.
4. In the socket handler, validate and save the incoming message.
5. Broadcast the saved document after MongoDB creates its `_id` and timestamp.
6. Merge new socket messages into the history already in React state.

Do not broadcast before a required database save succeeds. Otherwise, users may see a message that was never stored.

## Level Up: create one room per hoot

The MVP is one global room. To attach chat to a Hoot details page, use the Hoot ID as a room name.

The client can request to join:

```javascript
socket.emit('join hoot room', hootId)
```

The server can place that socket in the room:

```javascript
socket.on('join hoot room', (hootId) => {
  socket.join(hootId)
})
```

The server can then send only to that room:

```javascript
io.to(hootId).emit('chat message', newMessage)
```

This level-up also requires decisions about leaving old rooms, validating the Hoot ID, and storing which Hoot owns each message.

## Level Up: authenticate the socket

The MVP protects the React route, but the server trusts the username sent by the browser. That means it is not secure enough for production. A modified client could claim another username.

A secure next step is to:

1. Send the JWT in the Socket.IO client's `auth` data.
2. Add Socket.IO middleware on the server.
3. Verify the token during the connection handshake.
4. Store the verified user on `socket.data`.
5. Build the message username or author ID from `socket.data`, not from client-supplied message data.

This should be taught after students understand the basic event flow because Socket.IO middleware is separate from the familiar Express route middleware.

## Other possible improvements

- Show a typing indicator.
- Display a connected-user count.
- Automatically scroll to the newest message.
- Add an error event for invalid messages.
- Acknowledge successful delivery.
- Add moderation and rate limiting.
- Paginate older messages.
- Add connection-state recovery.

Each feature adds a new event or state transition. Add one, name it clearly, and verify its sender and receiver before building the interface around it.
