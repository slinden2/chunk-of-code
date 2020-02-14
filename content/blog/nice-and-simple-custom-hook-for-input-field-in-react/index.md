---
title: "Nice and simple custom hook for input field in React"
published: true
date: "20190521"
tags: ["hooks", "react"]
---

This week I am just going to share with you a nice and simple custom hook for controlling an input field in React. Let's start with the the code:

```
import { useState } from 'react'

export const useField = (type, name) => {
  const [value, setValue] = useState('')

  const onChange = (event) => {
    setValue(event.target.value)
  }

  const reset = () => {
    setValue('')
  }

  return [{
    type,
    name,
    value,
    onChange,
    'data-cy': name
  }, reset]
}
```

We use the `useState` hook to control the state of the input field as we would do normally. Every time when the value of the input field changes, the `change` event will fire and we run the `onChange()` callback function to store the value of the field to `value`. We have also a `reset()`function so that the field can be set to an empty string, for example, after a form has been submitted.

The hook returns a list with two items in it. The first one is the object that contains all the attributes needed for creating an input field. You may wonder what is up with the `data-cy` attribute. It is just a handle that I use with `cypress` library that is used to end-to-end testing. If you don't need it, feel free to delete it. It doesn't change the functionality of the hook. The second item in the list is the `reset()` function. Reset is not a valid input attribute, so it has to be kept separated form the other attributes.

## How to use the hook?

```
const App = () => {
  const [username, resetUsername] = useField('text', 'username')

  return (
    <div>
      <label>Username:</label>
      <input {...username} />
    </div>
  )
}
```

First we use destructuring so that we get `username` and `resetUsername` variables out from the hook. Then we have the `username` variable available for use in our component. We can nicely spread the attributes to the input field and it makes the code much cleaner. We could also call `resetUsername()` to go back to the initial state.

You probably already see that when you have many input fields, the custom hook becomes quickly very useful. You do not need to repeat all the state handling logic for every field.

Feel free to use the hook and make your code cleaner and more readable! :)
