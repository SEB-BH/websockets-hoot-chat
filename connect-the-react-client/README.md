<h1>
  <span class="headline">WebSockets with Socket.IO</span>
  <span class="subhead">Connect the React Client</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to open a Socket.IO connection from a React page, display its status, and remove its event listeners during cleanup.

## Create a shared socket object

Move to the `react-hoot-front-end` project.

Create a file directly inside `src`:

```bash
touch src/socket.js
```

Add the following code:

```javascript
// src/socket.js

import { io } from 'socket.io-client'

const BACK_END_URL = import.meta.env.VITE_BACK_END_SERVER_URL

const socket = io(BACK_END_URL, {
  autoConnect: false,
})

export default socket
```

Let's examine each part.

### Import `io`

```javascript
import { io } from 'socket.io-client'
```

On the front-end, `io()` creates a client connection. This `io` function is different from the server-side `io` object we created with `new Server()`.

### Reuse the existing URL

```javascript
const BACK_END_URL = import.meta.env.VITE_BACK_END_SERVER_URL
```

The Hoot services already use this environment variable for `fetch()` requests. Reusing it means the REST API and Socket.IO client both connect to the same back-end.

### Turn off automatic connection

```javascript
const socket = io(BACK_END_URL, {
  autoConnect: false,
})
```

Without this option, importing `socket.js` immediately opens a connection. Setting `autoConnect` to `false` lets the Chat page explicitly decide when to connect.

This will make our lifecycle easier to see:

- Opening Chat connects.
- Leaving Chat disconnects.

### Export one shared object

```javascript
export default socket
```

Other front-end files can now import the same socket object. We do not create a new connection on every render.

### Stop and check

Save the file and check both terminals.

Expected result: nothing new happens. Creating and exporting the object does not connect because `autoConnect` is `false`.

If the Express terminal already logs a new socket connection, check the spelling and capitalization of `autoConnect`.

## Scaffold the Chat page

Create a new page component:

```bash
touch src/pages/Chat.jsx
```

Add this first version:

```javascript
// src/pages/Chat.jsx

import { useEffect, useState } from 'react'
import socket from '../socket'

const Chat = () => {
  const [isConnected, setIsConnected] = useState(socket.connected)

  useEffect(() => {
    const handleConnect = () => {
      console.log('Connected to chat:', socket.id)
      setIsConnected(true)
    }

    const handleDisconnect = () => {
      console.log('Disconnected from chat')
      setIsConnected(false)
    }

    socket.on('connect', handleConnect)
    socket.on('disconnect', handleDisconnect)

    socket.connect()

    return () => {
      console.log('Leaving chat and closing socket')
      socket.off('connect', handleConnect)
      socket.off('disconnect', handleDisconnect)
      socket.disconnect()
    }
  }, [])

  return (
    <main>
      <h1>Hoot Chat</h1>
      <p>
        Status: {isConnected ? 'Connected' : 'Disconnected'}
      </p>
    </main>
  )
}

export default Chat
```

## Understand the initial state

```javascript
const [isConnected, setIsConnected] = useState(socket.connected)
```

The Socket.IO client has a boolean `connected` property. Because we used `autoConnect: false`, its initial value should be `false`.

We copy that value into React state so a connection change can re-render the page.

## Understand the effect

The empty dependency array means the effect is tied to this page being mounted:

```javascript
useEffect(() => {
  // Set up the connection

  return () => {
    // Clean up the connection
  }
}, [])
```

Inside the setup phase, we:

1. Define named handler functions.
2. register those handlers with `socket.on()`.
3. Call `socket.connect()`.

Inside cleanup, we:

1. Log that the page is intentionally closing its connection.
2. Remove each exact handler with `socket.off()`.
3. Disconnect this browser from the chat server.

Named functions make the cleanup explicit. Socket.IO needs the same event name and the same function reference when removing a listener.

## Why cleanup matters

React components can unmount and mount again. Vite can also update components while we edit.

If we repeatedly add listeners without removing them, one incoming event may run several callbacks. A single chat message can then appear two or three times. Cleanup prevents these old listeners from accumulating.

## Add the protected route

Open `src/App.jsx` and import the page:

```javascript
// src/App.jsx

import Chat from './pages/Chat'
```

Inside the group of routes available to signed-in users, add:

```javascript
<Route path='/chat' element={<Chat user={user} />} />
```

We are already passing `user`, even though the current Chat component does not use it yet. The form will use `user.username` in the next lesson.

The protected route section should contain the new route alongside the existing Hoot routes:

```javascript
{user ? (
  <>
    <Route path='/hoots' element={<HootList hoots={hoots} />} />
    <Route path='/hoots/:hootId' element={<HootDetails />} />
    <Route path='/chat' element={<Chat user={user} />} />
  </>
) : (
  <>
    <Route path='/sign-up' element={<SignUpForm />} />
    <Route path='/sign-in' element={<SignInForm />} />
  </>
)}
```

Keep any other routes already present in your completed application.

## Add a navigation link

Open the navigation component. Inside the list of links shown to a signed-in user, add:

```javascript
<li>
  <Link to='/chat'>CHAT</Link>
</li>
```

Do not add Chat to the guest link list. This lesson's interface is available only after signing in.

## Stop and check the complete connection

Make sure both applications are running, then:

1. Sign in to Hoot.
2. Open the browser console.
3. Click the **CHAT** link.

Check all three locations:

### On the page

You should see:

```plaintext
Status: Connected
```

### In the browser console

You should see a message containing a socket ID:

```plaintext
Connected to chat: abc123...
```

### In the Express terminal

You should see the same socket ID:

```plaintext
Socket connected: abc123...
```

The matching IDs prove that both logs describe the same browser connection.

## Check the cleanup

Navigate away from Chat to the Hoot list.

The browser console should show:

```plaintext
Leaving chat and closing socket
```

The Express terminal should show:

```plaintext
Socket disconnected: abc123...
```

The cleanup removes the `disconnect` listener before calling `socket.disconnect()`, so the browser uses our explicit `Leaving chat` log for an intentional page change. The `handleDisconnect` function remains useful for an unexpected interruption, such as the server stopping while Chat is still open.

Return to Chat. A new connection will open, usually with a new socket ID.

> In React development mode, Strict Mode may perform a quick setup, cleanup, and second setup to help find lifecycle bugs. You may briefly see an extra connection and disconnection. The final page status should be connected. If messages later appear more than once, perform a full page reload and verify that every `socket.on()` has a matching `socket.off()`.
