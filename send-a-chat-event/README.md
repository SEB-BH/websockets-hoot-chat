<h1>
  <span class="headline">WebSockets with Socket.IO</span>
  <span class="subhead">Send a Chat Event</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to emit a named event from React and receive its data in the Express terminal.

## Focus on one direction

We will now test only this half of the complete chat flow:

```plaintext
React form -> Socket.IO event -> Express server
```

The message will not appear on the page yet, only in our Express server (backend).

## Emit from React

Open `src/pages/Chat.jsx` in the front-end.

Inside `handleSubmit`, add one line after the `messageData` object is created:

```jsx
const handleSubmit = (event) => {
  event.preventDefault()

  if (!formData.trim()) {
      return
  }

  const messageData = {
      username: props.user.username,
      text: formData.trim(),
  }

  console.log('Chat form submitted:', messageData)
  // add this line 
  socket.emit('chat message', messageData)

  setFormData('')
}
```

This line sends an event through the open connection:

```javascript
socket.emit('chat message', messageData)
```

It has two main arguments:

| Argument | Value | Purpose |
| --- | --- | --- |
| Event name | `'chat message'` | Tells the receiver which listener should run. |
| Event data | `messageData` | Contains the username and text we want to send. |

Emitting does not automatically call an Express route. Socket.IO events are separate from routes such as `POST /hoots`.

## Listen on the server

Open `server.js` in the back-end.

Find the existing connection handler:

```javascript
io.on('connection', (socket) => {
  console.log('Socket connected:', socket.id)

  socket.on('disconnect', () => {
    console.log('Socket disconnected:', socket.id)
  })
})
```

Inside it, add a listener for the exact same event name:

```javascript
io.on('connection', (socket) => {
  console.log('Socket connected:', socket.id)

  // add this listener
  socket.on('chat message', (messageData) => {
    console.log('Chat event received:', messageData)
  })

  socket.on('disconnect', () => {
    console.log('Socket disconnected:', socket.id)
  })
})
```

The listener belongs inside the `connection` callback because it listens to events from that particular `socket`.

Let's connect the two lines:

```javascript
// React sends
socket.emit('chat message', messageData)

// Express receives
socket.on('chat message', (messageData) => {
  // use messageData
})
```

The parameter name `messageData` does not technically need to match. It is a local variable on each side. The event name `'chat message'` **must** match.

## Stop and check the client-to-server direction

Make sure both applications are running.

1. Open Hoot Chat.
2. Confirm the page says `Status: Connected`.
3. Submit `Can the server hear me?`.

### Browser console

You should see:

```plaintext
Chat event emitted: {username: 'your-name', text: 'Can the server hear me?'}
```

### Express terminal

You should see:

```plaintext
Chat event received: { username: 'your-name', text: 'Can the server hear me?' }
```

## A useful debugging rule

When an event listener does not run, compare the names character by character:

```javascript
'chat message'
```

These are different event names and will not match:

```javascript
'chat-message'
'chatMessage'
'Chat message'
'chat messages'
```

Socket.IO does not report a spelling error for a custom event. It simply has no matching listener to call.