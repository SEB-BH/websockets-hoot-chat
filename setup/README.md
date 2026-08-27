<h1>
  <span class="headline">WebSockets with Socket.IO</span>
  <span class="subhead">Setup</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to prepare the completed Hoot applications for Socket.IO development and verify both new dependencies.

## Start from the completed Hoot application

This lesson adds a feature to the completed Hoot application. You should already have two separate project folders:

```plaintext
express-api-hoot-back-end
react-hoot-front-end
```

The back-end should already include:

- An Express server running on port `3000`.
- MongoDB and the Hoot models.
- JWT authentication.
- The Hoot and comment routes.

The front-end should already include:

- A Vite React application.
- Sign-up, sign-in, and sign-out functionality.
- Hoot CRUD functionality.
- A `user` state variable in `App.jsx`.
- Protected routes for signed-in users.
- A `.env` file containing:

```plaintext
VITE_BACK_END_SERVER_URL=http://localhost:3000
```

> The code in this lesson passes `user` from `App.jsx` to the Chat page as a prop. It does not add `useContext`.

## Baseline check

Before adding a new technology, make sure the existing application still works.

Open the back-end folder in one terminal and start the Express server:

```bash
cd express-api-hoot-back-end
npm run dev
```

Open the front-end folder in a second terminal and start Vite:

```bash
cd react-hoot-front-end
npm run dev
```

In the browser:

1. Open the Vite URL.
2. Sign in.
3. Open the Hoot list.
4. Open one Hoot details page.

### Stop and check

Do not continue until all four checks pass:

- The Express terminal shows that the server is ready.
- The Express terminal shows that MongoDB is connected.
- The browser can sign in.
- The browser can retrieve Hoot data from the back-end.

If something is broken now, it is part of the existing application rather than the Socket.IO feature. Fix the baseline before introducing another moving part.

## Install the server package

Stop the Express server with `Control + C`.

Make sure the terminal is inside `express-api-hoot-back-end`, then install the Socket.IO server package:

```bash
npm install socket.io@4
```

We use `@4` to keep the server and client on the same major version.

### Stop and check

Run:

```bash
npm list socket.io
```

You should see a version beginning with `4` beneath the back-end project name.

Also open the back-end `package.json`. Confirm that `socket.io` appears inside `dependencies`.

## Install the client package

Stop the Vite server with `Control + C`.

In the other terminal, make sure you are inside `react-hoot-front-end`, then install the Socket.IO browser client:

```bash
npm install socket.io-client@4
```

The package names are intentionally different:

| Application | Package |
| --- | --- |
| Express back-end | `socket.io` |
| React front-end | `socket.io-client` |

### Stop and check

Run:

```bash
npm list socket.io-client
```

You should see a version beginning with `4` beneath the front-end project name.

Also open the front-end `package.json`. Confirm that `socket.io-client` appears inside `dependencies`.

## Restart both applications

Start the Express server again:

```bash
npm run dev
```

Start Vite again in the front-end terminal:

```bash
npm run dev
```

The Hoot application should still behave exactly as it did during the baseline check. We have installed two packages, but we have not used them yet.

## Checkpoint

Before moving on, explain why we installed two packages instead of one:

- The Express application needs code that accepts and manages socket connections.
- The React application needs code that opens a connection from the browser.
