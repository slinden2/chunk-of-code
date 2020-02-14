---
title: "Using styled components with React"
published: true
date: "20190802"
tags: ["css", "css-in-js", "styled-components"]
---

Recently I have started learning how to use styled components CSS-in-JS library for styling my React application. I am still quite a beginner in this stuff, but I decided to write a small blog post about it in order to get better comprehension of the library.

## What are styled components?

Styled components are components that enhance regular React components with some CSS styling, as the name states. For example, a simple React components could be like this:

```javascript
import React from "react"

const App = () => {
  return <div>This is a test component.</div>
}
```

If we wanted to modify the background color with styled components, we could do it like this:

```javascript
import React from "react"
import styled from "styled-components"

const ColoredDiv = styled.div`
  background-color: red;
`

const App = () => {
  return <ColoredDiv>This is a test component.</ColoredDiv>
}
```

That is the idea of styled components in a nutshell. They enable your to use CSS directly in the components within tagged template string literals. CSS-in-JS helps you to create _dynamic_ CSS via `props` and you do not need to do a separate CSS class for everything, as you would with regular CSS.

## Why to use styled components?

Here are the first five positive things that make me think that using styled components is a good idea.

1. You can scope CSS on a component basis and stop worrying about overriding existing class names. This is particularly useful when you are working on bigger applications.
2. Faster development. When you have the CSS on the top of the component, the workflow becomes more natural as you don't need to switch between files when editing CSS and jsx of the component.
3. You can create self-contained, reusable components, library style. When your code base gets bigger and bigger, you can probably start reusing the components that you create in various projects. With styled components, you can have the logic and the CSS in the same file, so importing the components becomes trivial.
4. You don't have to worry about vendor prefixes. Styled components will take care of them for you.
5. Obviously, the full power of JS is available for you. You don't need to create a new class for everything. The state of a styled component can be change via `props`.

## Why not to use styled components?

Many think that using styled components or any other CSS-in-JS library is a **really bad idea**. I have read a lot of negative blog posts and comments from various sources. Many people seem to get outright angry about the idea of using CSS in Javascript, but I beg to differ.

There is one good argument against styled components, though. It is just a library. The libraries come and go, but CSS will always be there. Before delving into a library, you should have a good grasp of basic CSS. It is a skill that you will always need even if the libraries on top of it change.

## Conclusion

Styled components is definitely something that you should consider to take a look at. If you don't know CSS, learn it well first and only after start to take a look at styled components and maybe other CSS-in-JS libraries, so that you see other implementations as well.
