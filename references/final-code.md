<h1>
  <span class="headline">WebSockets with Socket.IO</span>
  <span class="subhead">Final Code Reference</span>
</h1>

Use this page to compare completed code after working through the microlessons. Preserve all existing Hoot code that is not shown here.

## Back-end dependency

From `express-api-hoot-back-end`:

```bash
npm install socket.io@4
```

## Back-end server additions

At the top of `server.js`:

```javascript
const http = require('http')
const express = require('express')
const { Server } = require('socket.io')

const app = express()
const server = http.createServer(app)

const io = new Server(server, {
  cors: {
    origin: 'http://localhost:5173',
  },
})
```

Keep the existing Hoot imports, database connection, middleware, and routes.

After the Express routes and before the server starts:

```javascript
io.on('connection', (socket) => {
  console.log('Socket connected:', socket.id)

  socket.on('chat message', (messageData) => {
    console.log('Chat event received:', messageData)

    const newMessage = {
      id: `${socket.id}-${Date.now()}`,
      username: messageData.username,
      text: messageData.text,
    }

    console.log('Chat event broadcast:', newMessage)

    io.emit('chat message', newMessage)
  })

  socket.on('disconnect', () => {
    console.log('Socket disconnected:', socket.id)
  })
})
```

Replace the old `app.listen()` with:

```javascript
server.listen(3000, () => {
  console.log('The Express and Socket.IO server is ready!')
})
```

## Front-end dependency

From `react-hoot-front-end`:

```bash
npm install socket.io-client@4
```

## Front-end socket object

```javascript
// src/socket.js

import { io } from 'socket.io-client'

const BACK_END_URL = import.meta.env.VITE_BACK_END_SERVER_URL

const socket = io(BACK_END_URL, {
  autoConnect: false,
})

export default socket
```

## Front-end Chat page

```javascript
// src/pages/Chat.jsx

import { useEffect, useState } from 'react'
import socket from '../socket'

const Chat = (props) => {
  const [isConnected, setIsConnected] = useState(socket.connected)
  const [messageText, setMessageText] = useState('')
  const [messages, setMessages] = useState([])

  useEffect(() => {
    const handleConnect = () => {
      console.log('Connected to chat:', socket.id)
      setIsConnected(true)
    }

    const handleDisconnect = () => {
      console.log('Disconnected from chat')
      setIsConnected(false)
    }

    const handleChatMessage = (newMessage) => {
      console.log('Chat event received from server:', newMessage)

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
    setMessageText(event.target.value)
  }

  const handleSubmit = (event) => {
    event.preventDefault()

    const trimmedMessage = messageText.trim()

    if (!trimmedMessage) {
      return
    }

    const messageData = {
      username: props.user.username,
      text: trimmedMessage,
    }

    console.log('Chat event emitted:', messageData)
    socket.emit('chat message', messageData)

    setMessageText('')
  }

  return (
    <main>
      <h1>Hoot Chat</h1>
      <p>
        Status: {isConnected ? 'Connected' : 'Disconnected'}
      </p>

      <section aria-live='polite'>
        <h2>Messages</h2>

        {messages.length === 0 && (
          <p>No messages yet. Start the conversation!</p>
        )}

        {messages.map((message) => (
          <article key={message.id}>
            <strong>{message.username}</strong>
            <p>{message.text}</p>
          </article>
        ))}
      </section>

      <form onSubmit={handleSubmit}>
        <label htmlFor='message-input'>Message:</label>
        <input
          id='message-input'
          name='message'
          type='text'
          value={messageText}
          onChange={handleChange}
          autoComplete='off'
        />
        <button type='submit' disabled={!isConnected}>
          SEND
        </button>
      </form>
    </main>
  )
}

export default Chat
```

## App route

Import the page in `src/App.jsx`:

```javascript
import Chat from './pages/Chat'
```

Add the route inside the signed-in route group:

```javascript
<Route path='/chat' element={<Chat user={user} />} />
```

## Navigation link

Add this inside the signed-in navigation list:

```javascript
<li>
  <Link to='/chat'>CHAT</Link>
</li>
```
