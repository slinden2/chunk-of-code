---
title: "How to run backend and frontend in the same terminal window?"
published: true
date: "20190618"
tags: ["concurrently", "workflow"]
---

I have recently stumbled upon a cool library that allows you to run frontend and backend in the same terminal window. This way, you need just one command to start your development environment.

I am talking about [concurrently](https://www.npmjs.com/package/concurrently). Everyone of us knows that one of the hardest things in software development is **starting** to work. `concurrently` eliminates some of that friction in an easy way.

## How to put it all together?

First, create a new folder in your project directory. In this example I will just use `C:\Project\` because it is easier to type.

Navigate to the project folder and create the frontend like this:

```
npx create-react-app frontend
```

Create a new folder called `backend` in the project directory. At this point you should have `backend` and `frontend` directories in your project directory.

Navigate to `backend` and run:

```
npm init
```

Fill in the date that the init program needs and install `express`:

```
npm install express --save
```

Let's install also `nodemon`:

```
npm install nodemon --save-dev
```

Then create a new file `server.js` in the `backend` directory. Open the file in your text editor and paste in the following code:

```js
const express = require("express")
const app = express()
app.listen(3000, () => {
  console.log("Server running on port 3000")
})
```

At this point we have both, our frontend and backend, ready to go.
The next thing we need to do is to modify the `package.json` file in the `backend` folder.

You can paste in my version:

```json
{
  "name": "project",
  "version": "0.0.1",
  "description": "",
  "main": "server.js",
  "scripts": {
    "watch": "nodemon server.js",
    "client": "npm start --prefix ../frontend",
    "dev": "concurrently \"npm run watch\" \"npm run client\""
  },
  "author": "author",
  "license": "MIT",
  "devDependencies": {
    "concurrently": "^4.1.0",
    "nodemon": "^1.19.1"
  },
  "dependencies": {
    "express": "^4.17.0"
  }
}
```

Take a look at the `script` tag. I have created three commands. `watch` for the backend, `client` for the frontend, and `dev` for the development environment, that runs both of the commands at the same time, concurrently. :)

Note that in the `client` script a prefix is needed, because the root folder of the frontend is not the same than that of the backend.

This neat little trick has really improved my quality of life as a developer. If you already didn't know this I hope that you find this post useful. Cheers!
