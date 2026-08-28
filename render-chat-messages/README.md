<h1>
  <span class="headline">WebSockets with Socket.IO</span>
  <span class="subhead">Render Chat Messages</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to add incoming socket data to React state and render a live message list.

## Add message-list state

Open `src/pages/Chat.jsx`.

Add a third state variable:

```javascript
const [isConnected, setIsConnected] = useState(socket.connected)
const [formData, setFormData] = useState('')
const [messages, setMessages] = useState([])
```

`messages` begins as an empty array because no chat events have arrived in this browser yet.

## Update state when an event arrives

Find `handleChatMessage` inside the effect and update it:

```javascript
const handleChatMessage = (newMessage) => {
  console.log('Chat event received from server:', newMessage)

  setMessages((previousMessages) => {
    return [...previousMessages, newMessage]
  })
}
```

The important state update is:

```javascript
setMessages((previousMessages) => {
  return [...previousMessages, newMessage]
})
```

Read it in steps:

1. React provides the most recent array as `previousMessages`.
2. `[...previousMessages]` copies every existing message into a new array.
3. `newMessage` is added to the end.
4. `setMessages()` stores the new array.
5. React re-renders the component.

We use the callback form of `setMessages()` because socket events can arrive at any time. It ensures that each update starts with the most recent state.

It also lets our effect keep an empty dependency array. The effect does not need to reconnect every time `messages` changes.

## Render an empty state

Below the connection status and above the form, add:

```javascript
<section>
  <h2>Messages</h2>

  {messages.length === 0 && (
    <p>No messages yet. Start the conversation!</p>
  )}
</section>
```

## Render the messages

Inside the same `<section>`, below the empty-state condition, map over the array:

```javascript
{messages.map((message) => (
  <article key={message.id}>
    <strong>{message.username}</strong>
    <p>{message.text}</p>
  </article>
))}
```

The server created `message.id`, so React can use it as a stable key for this temporary list.

## Complete Chat page

Your completed `src/pages/Chat.jsx` should now look like this:

```javascript
// src/pages/Chat.jsx

import socket from '../socket'
import { useState, useEffect } from 'react'

const Chat = (props) => {
    const [isConnected, setIsConnected] = useState(socket.connected)
    const [formData, setFormData] = useState('')
    const [messages, setMessages] = useState([])

    useEffect(() => {
        const handleConnect = () => {
            console.log('Connected to chat: ', socket.id)
            setIsConnected(true)
        }   

        const handleDisconnect = () => {
            console.log('Disconnected from chat')
            setIsConnected(false)
        }

        const handleChatMessage = (newMessage) => {
            console.log('Chat event received from server: ', newMessage)
            setMessages((previousMessages) => {
                return [...previousMessages, newMessage]
            })
        }

        socket.on('connect', handleConnect)
        socket.on('disconnect', handleDisconnect)
        socket.on('chat message', handleChatMessage)

        socket.connect()

        return () => {
            console.log('Leaving chat and closing socket')
            socket.off('connect', handleConnect)
            socket.off('disconnect', handleDisconnect)
            socket.off('chat message', handleChatMessage)
            socket.disconnect()
        }
    }, [])

    const handleChange = (event) => {
        setFormData(event.target.value)
    }

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
        socket.emit('chat message', messageData)

        setFormData('')
    }

    return (
        <main>
            <h1>Hoot Chat</h1>
            <p>
                Status: { isConnected ? 'Connected' : 'Disconnected'}
            </p>

            <section>
                <h2>Messages</h2>

                {messages.length === 0 && (
                    <p>No messages yet. Start the conversation!</p>
                )}
                {messages.map(message => (
                    <article key={message.id}>
                        <strong>{message.username}</strong>
                        <p>{message.text}</p>
                    </article>
                ))}
            </section>

            <form onSubmit={handleSubmit}>
                Message:
                <input type="text" name='message' value={formData} onChange={handleChange} />
                <button type='submit' disabled={!isConnected}>SEND</button>
            </form>
        </main>
    )
}

export default Chat
```

## Stop and check in one tab

Refresh your React app.  Submit a message.

Confirm that:

- The input clears.
- The empty-state paragraph disappears.
- The username and message appear.
- The page does not refresh.
- The browser and Express logs still show the complete event path.

## Stop and check in two tabs

Open Chat in two browser tabs and send a message from each.

Both messages should appear in both tabs without a refresh.

If the sender does not see its own message, check that the server uses `io.emit()` rather than `socket.broadcast.emit()`.

If each message appears twice, check that:

- `handleChatMessage` is registered only once inside the effect.
- Cleanup calls `socket.off('chat message', handleChatMessage)`.
- The effect dependency array is `[]`.
- You performed a full page reload after editing `src/socket.js`.

## Check what happens after a refresh

Refresh one browser tab.

Its message list returns to the empty state. The other tab still displays the messages already held in its React state.

This result proves that:

- Real-time delivery works.
- Message persistence has not been implemented.


### Congrats 🎉 You've successfully implemented websockets!