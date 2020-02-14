---
title: "A custom hook for flash notifications"
published: true
date: "20190628"
tags: ["hooks", "react"]
---

I am working on a web application that needs a notification system (almost every app does need one...). I call this hook `useNotification`, as it is the naming convention of hooks.

## Prerequisities

In this case what I needed from the notification system is a simple way to create 4 types of notifications: `positive`, `negative`, `info` and `warning`. These are the built-in `Message` types in [Semantic UI React](https://react.semantic-ui.com/), which is a CSS frame work that I am using in my project at least in the prototyping phase.

I also need to pass an arbitrary message in the notification and preferably a duration. Some could say that varying the duration of the notification is not good, because it may make the site feel _glitchy_. Anyway, I include the possibility to add duration just in case.

## Code

Let dive into the code:

```javascript
import { useState } from "react"

const useNotification = () => {
  const [notification, setNotification] = useState(null)
  const [timeoutId, setTimeoutId] = useState(null)

  const notify = (type, message, duration) => {
    const delay = duration || 5000
    clearTimeout(timeoutId)
    setNotification({ type, message })
    const id = setTimeout(() => {
      setNotification(null)
      setTimeoutId(null)
    }, delay)
    setTimeoutId(id)
  }

  return [notification, notify]
}
```

So, what is going on here?

The hook uses two `useState` hooks: `notification` and `timeoutId`. `notification` is used for storing the notification in object format. The object shall consist of two properties: `type` and `message`.

The second `useState` hook, `timeoutId` is used for storing the _id_ that is returned by the `setTimeout` function that is built-in with Javascript. We need this id so that we can clear the timeout before setting a new notification. If we didn't store the id in a hook, we would be able to clear the notification, if there was one, when a new notification needs to be shown. This results in a horrible user experience.

For example, say that right now I set a notification for 5 seconds that says _invalid password_. I wait 3 seconds and set another one saying _password correct_. The first notification disappears and then second notification is shown. However, the first timeout has not _expired_. What will happen is that when the **first**, _invalid password_, timeout expires, it will clear also the **second**, _password correct_, timeout. So, we saw _invalid password_ for 3 seconds and _password correct_ for 2 seconds. We wanted to see _invalid password_ for 3 seconds and _password correct_ for 5 seconds.

So in this code block:

```javascript
const notify = (type, message, duration) => {
  const delay = duration || 5000
  clearTimeout(timeoutId)
  setNotification({ type, message })
  const id = setTimeout(() => {
    setNotification(null)
    setTimeoutId(null)
  }, delay)
  setTimeoutId(id)
}
```

First we set the delay to 5000 ms if the user doesn't provide a duration. Then we clear any previous notification there might be by calling `clearTimeout(timeoutId)`. After that we can set a new notification with `setNotification({ type, message })`. Then we store the id of the new timeout to `id` and within the `setTimeout` we clear the notification after `delay`. After the `setTimeout` block, we also store the `id` in the `useState` hook we set up earlier.

At the end of the hook, we return `[notification, notify]` which we can when use in other files. When creating the hook in another file, we could simply call `const [notification, setNotification] = useNotification()` and be good to go.
