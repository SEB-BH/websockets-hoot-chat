<h1>
  <span class="headline">WebSockets with Socket.IO</span>
  <span class="subhead">Build the Chat Interface</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to build a controlled chat form and verify its data before sending a socket event.

## Keep communication and form handling separate

Before sending any data to Express, we will make sure the React form works by itself.

This follows the same sequence used for other controlled forms:

1. Create state.
2. Create a form in the return.
3. Update the state when the input changes with `handleChange`.
4. Handle submission with `handleSubmit`.
5. Connect the form to an external system with a service function.

## Add message state

In `src/pages/Chat.jsx`, add another state variable below `isConnected`:

```javascript
const [isConnected, setIsConnected] = useState(socket.connected)
const [formData, setFormData] = useState('')
```

The state is a string because this form has only one input.

## Add a change handler

Below the `useEffect`, add:

```javascript
const handleChange = (event) => {
  setFormData(event.target.value)
}
```

This function copies the input's current value into React state.


Add a simple chat form:

```javascript
return (
  <main>
    <h1>Hoot Chat</h1>
    <p>
      Status: {isConnected ? 'Connected' : 'Disconnected'}
    </p>

    <form>
        Message:
        <input type="text" value={formData} onChange={handleChange} />
        <button type='submit'>SEND</button>
    </form>
  </main>
)
```

### Stop and check

Temporarily add this log directly above the `return`:

```javascript
console.log('formData state:', formData)
```

Type `Hello Hoot!` into the input.

- The characters should appear in the input.
- The browser console should show the state growing from an empty string to `Hello Hoot!`.

Once this works, remove the temporary `console.log()` so it does not run on every render.

At this point, submitting the form refreshes the page. That is expected because we have not created a submit handler yet.

## Add a submit handler

Below `handleChange`, add:

```javascript
const handleSubmit = (event) => {
  event.preventDefault()

  if (!formData) {
    return
  }

  const messageData = {
    username: props.user.username,
    text: trimmedMessage,
  }

  console.log('Chat form submitted:', messageData)

  setFormData('')
}
```
<!-- 
Let's break this function into its separate jobs.

### Prevent the refresh

```javascript
event.preventDefault()
```

The browser's normal form submission would refresh the page. We prevent that so JavaScript can handle the message.

### Remove extra whitespace

```javascript
const trimmedMessage = formData.trim()
```

`trim()` removes spaces from the beginning and end. A message containing only spaces becomes an empty string. -->

### Reject an empty message

```javascript
if (!formData) {
  return
}
```

If the string is empty, the function stops before creating or sending anything.

### Build a small data object

```javascript
const messageData = {
  username: props.user.username,
  text: formData,
}
```

The `user` was passed from `App.jsx`. For now, the message contains only the two values the chat needs. **👈 BE SURE YOU'RE ACCEPTING PROPS IN YOUR COMPONENT**

### Clear the controlled input

```javascript
setFormData('')
```

Because the input's `value` comes from state, resetting state also clears the field on the page.

## Connect the handler to the form

Update the opening form tag:

```javascript
<form onSubmit={handleSubmit}>
```

Also disable the button when this browser is not connected:

```javascript
<button type='submit' disabled={!isConnected}>
  SEND
</button>
```

You can stop your Express server with `ctrl + c` to check if your button is disabling when not connected.

## Complete Chat page at this checkpoint

Your component should now look like this:

```javascript
// src/pages/Chat.jsx
import socket from '../socket'
import { useState, useEffect } from 'react'

const Chat = (props) => {
    const [isConnected, setIsConnected] = useState(socket.connected)
    const [formData, setFormData] = useState('')

    useEffect(() => {
        const handleConnect = () => {
            console.log('Connected to chat: ', socket.id)
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

    const handleChange = (event) => {
        setFormData(event.target.value)
    }

    const handleSubmit = (event) => {
        event.preventDefault()

        if (!formData) {
            return
        }

        const messageData = {
            username: props.user.username,
            text: formData,
        }

        console.log('Chat form submitted:', messageData)

        setFormData('')
    }

    return (
        <main>
            <h1>Hoot Chat</h1>
            <p>
                Status: { isConnected ? 'Connected' : 'Disconnected'}
            </p>

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

## Stop and check submission

Submit `Hello Hoot!` and confirm:

- The page does not refresh.
- The input clears.
- The browser console displays an object similar to:

  ```plaintext
  Chat form submitted: {username: 'nabila', text: 'Hello Hoot!'}
  ```

## Handle submitting only spaces

Inside of our `handleSubmit`, let's tack on `.trim()` to each instance of `formData` - this will remove white space and prevent the user from submitting an empty chat message.  

Updated section of `handleSubmit`:

```jsx
 if (!formData.trim()) {
    return
 }

 const messageData = {
     username: props.user.username,
     text: formData.trim(),
}
```