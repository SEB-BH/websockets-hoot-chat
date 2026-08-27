<h1>
  <span class="headline">WebSockets with Socket.IO</span>
  <span class="subhead">Test and Debug</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to verify every stage of a Socket.IO chat flow and isolate common connection, event, and React lifecycle problems.

## Why socket testing feels different

An HTTP feature often has one request and one response. We can inspect the route in Postman and receive a clear status code.

Our chat has more moving parts:

1. The browser connects.
2. The browser emits an event.
3. The server receives the event.
4. The server broadcasts another event.
5. One or more browsers receive it.
6. Each React app updates its own state.

A reliable test checks each stage instead of treating chat as one black box.

## Test 1: connection and cleanup

1. Start both applications.
2. Sign in and open Chat.
3. Compare the socket ID in the browser console and Express terminal.
4. Navigate away from Chat.
5. Confirm the browser logs its intentional cleanup and Express logs the disconnection.
6. Return to Chat and confirm a new connection.

Passing result: the page status, browser log, and server log agree about when the connection is open.

## Test 2: one complete message path

Submit a unique message such as:

```plaintext
owl-check-27
```

Look for the same text in this order:

| Stage | Location | Expected evidence |
| --- | --- | --- |
| Form submits | Browser console | `Chat event emitted` |
| Server receives | Express terminal | `Chat event received` |
| Server sends | Express terminal | `Chat event broadcast` |
| Browser receives | Browser console | `Chat event received from server` |
| React renders | Chat page | Visible username and message text |

If one row passes and the next row fails, the problem is located between those two stages.

## Test 3: broadcast to two clients

1. Open Chat in two tabs.
2. Confirm two different socket IDs in the Express terminal.
3. Submit `from tab one` in the first tab.
4. Confirm that it appears once in each tab.
5. Submit `from tab two` in the second tab.
6. Confirm that it appears once in each tab.

For a stronger test, use a normal window and a private/incognito window with two different Hoot accounts.

Passing result: every emitted message appears once for every connected client, including the sender.

## Test 4: inspect the network connection

The consoles show our application logs. Browser DevTools can show the connection itself.

In Chrome or another Chromium browser:

1. Open DevTools.
2. Select the **Network** panel.
3. Reload the Chat page while Network is recording.
4. Filter for `socket.io`.
5. Find the request using a WebSocket transport or showing status `101`.
6. Open it and select the **Messages** tab.
7. Submit a chat message.

You should see frames that contain the event name `chat message` and the object being sent. Socket.IO adds protocol characters around the event. Focus on finding the event name and data rather than memorizing the prefix.

Socket.IO may begin with HTTP polling and then upgrade to WebSocket. Therefore, you may also see several `polling` requests before the WebSocket entry.

### Stop and check

Find both directions in the Messages panel:

- A frame carrying the browser's `messageData` to the server.
- A frame carrying the server's `newMessage` back to the browser.

## Test 5: temporary server interruption

Keep Chat open and stop the Express server with `Control + C`.

Expected results:

- The page changes to `Status: Disconnected`.
- The send button becomes disabled.
- The browser console logs the disconnection.

Restart the Express server:

```bash
npm run dev
```

Socket.IO normally attempts to reconnect. The page should return to `Status: Connected` and the button should become available.

Messages already stored in the page's React state remain visible because stopping Express does not unmount the React page. However, a message emitted while disconnected is not handled by our lesson as guaranteed chat delivery.

## Test 6: prove the persistence boundary

1. Send two messages.
2. Confirm both are visible.
3. Refresh that tab.

Passing result: the tab reconnects, but its list is empty.

That is not a failed socket test. It is the expected result of storing messages only in React state.

## Test 7: regression check

After changing the server startup code, verify the original application still works:

- Sign out and sign back in.
- Open the Hoot list.
- Create or open a Hoot.
- Add a comment.

Passing result: regular HTTP routes and the Socket.IO connection both work on port `3000`.

## Troubleshooting guide

| Symptom | Likely cause | Check |
| --- | --- | --- |
| `Cannot find module 'socket.io'` | Server package installed in the wrong folder. | Run `npm list socket.io` in the back-end. |
| Vite cannot resolve `socket.io-client` | Client package installed in the wrong folder. | Run `npm list socket.io-client` in the front-end. |
| `EADDRINUSE` on port `3000` | Two servers are listening or an old process is running. | Keep only `server.listen()` and stop the old process. |
| Hoot REST routes stopped working | The Express `app` was not passed to the HTTP server. | Confirm `http.createServer(app)`. |
| Chat always says disconnected | URL, port, server, or CORS mismatch. | Check the Vite URL, environment variable, and Socket.IO `origin`. |
| Server logs a connection but no message | Event names do not match or `emit()` is missing. | Compare `'chat message'` on both sides. |
| Sender logs the event but other tabs do not | Server is not broadcasting. | Confirm the server uses `io.emit()`. |
| Sender does not see its own message | Server excludes the sender. | Replace `socket.broadcast.emit()` with `io.emit()`. |
| Every message appears more than once | Event listeners accumulated. | Pair every `socket.on()` with `socket.off()` and reload the page. |
| Messages disappear after refresh | No database persistence exists. | This is expected in the MVP. |
| A new tab cannot see old messages | The server sends only new events. | This is expected until message history is added. |

## Final explanation check

Be prepared to explain the completed flow without looking at the code:

1. What opens the connection?
2. What closes it?
3. Which line sends form data to Express?
4. Which line receives that data on Express?
5. Which line broadcasts to every browser?
6. Which line receives the broadcast in React?
7. Which line causes the visible list to re-render?
8. Why does refreshing remove the messages?

### Answer check

1. `socket.connect()` inside the Chat effect.
2. `socket.disconnect()` inside the effect cleanup.
3. The client's `socket.emit('chat message', messageData)`.
4. The server's `socket.on('chat message', ...)`.
5. The server's `io.emit('chat message', newMessage)`.
6. The client's `socket.on('chat message', handleChatMessage)`.
7. `setMessages()` creates new state and React re-renders.
8. The messages exist only in that page's React state and were never saved.

## Completion criteria

The feature is complete when:

- Chat is accessible to a signed-in user.
- The page accurately displays connected or disconnected status.
- Empty messages are rejected.
- One browser can send a message to Express.
- Express broadcasts the message to all connected browsers.
- Two clients display each new message once without refreshing.
- Listener cleanup prevents duplicate events.
- Existing Hoot HTTP features still work.
- Students can explain why message history disappears.
