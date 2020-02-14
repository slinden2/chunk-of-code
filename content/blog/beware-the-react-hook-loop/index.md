---
title: "Beware the React hook loop"
published: true
date: "20190504"
tags: ["hooks", "react", "ref"]
---

This week I was doing some work with React components that involved hooks. I am going to show you an example of a sneaky bug that is not easy to avoid if you don't actively think about it. In this example I will just use some pseudocode to make an example, but the code won't run if you try to follow along by pasting the code in your text editor.

The example code is from an application that allows the user to post blogs to a database. The problem that I had was related to a _ref_. A ref is a reference to a specific instance of a component. By default, React let's you to access only a component, but everything that is inside it, is hidden. Refs allow you to get access to functions or DOM nodes that are within a child component. It this case I needed to access a function within a child component to change its state from _open_ to _closed_.

I instantiated the ref in the `App` component and passed it to the `Togglable` helper component. `Togglable` is a component that has two states: Open or closed. In its closed state, it renders only a button that allows your to open the component that it contains. The contained component in this case is the `BlogForm` component, which is used to create new blog entries to the database.

In my program there were three components directly related to the bug: App.js, Togglable.js and BlogForm.js. Let's start with the App component.

## App.js component

```
import React, { useState, useEffect } from 'react'
import BlogForm from './components/BlogForm'

const App = () => {
  const [blogs, setBlogs] = useState([])
  const [username, setUsername] = useState('')
  const [password, setPassword] = useState('')
  const [user, setUser] = useState(null)
  const [notification, setNotification] = useState({})
  const [timeoutId, setTimeoutId] = useState(0)

  const notify = (message, error) => {
    if (timeoutId) clearTimeout(timeoutId)
    setNotification({ message, error })
    const id = setTimeout(() => {
      setNotification({})
      setTimeoutId(0)
    }, 5000)
    setTimeoutId(id)
  }

  const blogFormRef = React.createRef()

  const blogForm = () => (
    <Togglable buttonLabel='add blog' ref={blogFormRef}>
      <BlogForm
        blogs={blogs}
        setBlogs={setBlogs}
        notify={notify}
        blogFormRef={blogFormRef}
      />
    </Togglable>
  )

  const blogRows = () => {
    const sortedBlogs = blogs.sort((a, b) => b.likes - a.likes)
    return sortedBlogs.map(blog =>
      <Blog
        key={blog.id}
        user={user}
        blog={blog}
        blogs={blogs}
        notify={notify}
        setBlogs={setBlogs}
      />)
  }

return (
  <div>
    <h2>Log in to application</h2>
    <Notification notification={notification} />
    <form onSubmit={handleLogin} className='loginform'>
      <div>
        username
        <input
          type=text
          name=Username
          value={username}
          onChange={({ target }) => setUsername(target.value)}
        />
      </div>
      <div>
        password
        <input
          type=text
          name=Password
          value={password}
          onChange={({ target }) => setPassword(target.value)}
        />
      </div>
      <button type=submit>log in</button>
    </form>
  </div>
)}

export default App
```

The `App` component is the heart of the application. It contains most of the state handling I use throughout the app. Take a look at the highlighted rows 22-33. I create the ref and store it to `blogFormRef`. Then I pass it to the `Togglable` component as a prop. Note that the ref is passed also to the BlogForm component on line 30, but this is different. On line 25 I am binding the ref to the `Togglable` component and on line 30 I am passing the result object of the ref to the `BlogForm` component so that I can get access to its value from within the component.

## Togglable.js component

```
import React, { useState, useImperativeHandle } from 'react'

const Togglable = React.forwardRef((props, ref) => {
  const [visible, setVisible] = useState(false)

  const visibleWhenHidden = { display: visible ? 'none' : '' }
  const visibleWhenShown = { display: visible ? '' : 'none' }

  const toggleVisibility = () => {
    setVisible(!visible)
  }

  useImperativeHandle(ref, () => {
    return {
      toggleVisibility
    }
  })

  return (
    <div>
      <div style={visibleWhenHidden}>
        <button onClick={toggleVisibility}>{props.buttonLabel}</button>
      </div>
      <div style={visibleWhenShown}>
        {props.children}
        <button onClick={toggleVisibility}>cancel</button>
      </div>
    </div>
  )
})

export default Togglable
```

In the `Togglable` the creation of the component differs slightly. The function component is wrapped to `forwardRef` function. `forwardRef` allows for a function component to receive and pass forward refs.

The thing that I need to access from the `BlogForm` component is the `toggleVisibility` funtion. I use `useImperativeHandle` to make it accessible from the ref by calling `blogFormRef.current.toggleVisibility()`.

## BlogForm.js component

```
import React, { useState } from 'react'
import PropTypes from 'prop-types'
import blogService from '../services/blogs'

const BlogForm = ({ blogs, setBlogs, notify, blogFormRef }) => {
  const [title, setTitle] = useState('')
  const [author, setAuthor] = useState('')
  const [url, setUrl] = useState('')

  const handleBlogCreation = async event => {
    event.preventDefault()

    let blogObject = {}
    for (const input of event.target.querySelectorAll('input')) {
      blogObject[input.name] = input.value
    }

    try {
      const blog = await blogService.create(blogObject)
      setTitle('')
      setAuthor('')
      setUrl('')
      setBlogs(blogs.concat(newBlog))
      notify(`a new blog ${newBlog.title} successfully added`)
      blogFormRef.current.toggleVisibility()
    } catch (exception) {
      notify(`${exception.response.data.error}`, true)
    }
  }

  return (
    <div>
      <h1>create new</h1>
      <form onSubmit={event => handleBlogCreation(event)}>
        <div>
          title:
          <input
            type=text
            name=title
            value={title}
            onChange={({ target }) => setTitle(target.value)}
          />
        </div>
        <div>
          author:
          <input
            type=text
            name=author
            value={author}
            onChange={({ target }) => setAuthor(target.value)}
          />
        </div>
        <div>
          url:
          <input
            type=text
            name=url
            value={url}
            onChange={({ target }) => setUrl(target.value)}
          />
        </div>
        <button type=submit>create</button>
      </form>
    </div>
  )
}

export default BlogForm
```

Finally the `BlogForm` components. This is where the bug is nested. Particularly take a look at the `handleBlogCreation` function. You can see that the in the `try` block the first thing I do is I save the blog to the database. Then I empty the input fields that were used for the blog creation. Then I add the blogs also to the list that is rendered to the page by calling `setBlogs` and lastly I `notify` about the successful operation and hide the field by calling `blogFormRef.current.toggleVisibility()`.

But the blog creation for remains open. What gives?

I spent a good amount of time debugging this issue. I console.logged the `blogFormRef` in the `App` component and the `BlogForm` component. Everything seemed to be ok. `blogFormRef` contained the function at the beginning of the `handleBlogCreation` function and even in the `try` blog, naturally, but somehow just before the `blogFormRef.current.toggleVisibility()` function call `blogFormRef` became `null`.

I started to move the `blogFormRef.current.toggleVisibility()` function call around and got it working. I figured out that `notify()` and `setBlogs()` were causing the problem. Why did the change the value of `blogFormRef`? They shouldn't be related in any way! But once you have solved the bug it seems super obvious. Why didn't I see it right away. `notify()` and `setBlogs()` update the state of the `App` component whereas `setTitle()`, `setAuthor()` and `setUrl()` operate in the `BlogForm` component. Updating the state in the `BlogForm` doesn't interfere with `blogFormRef`, but updating `App` does. When I updated the blogs or sent a notification, all the code in the `App` component was rerun, and as a result, `const blogFormRef = React.createRef()` was called again making it `null`.

I hope that next time when I stumble on a similar problem, I will have easier time solving it. This surely was a useful lesson for me. 😊
