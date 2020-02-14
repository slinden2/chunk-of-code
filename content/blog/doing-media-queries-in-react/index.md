---
title: "Doing media queries in React"
published: true
date: "20190822"
tags: ["css", "mediaquery", "react-media", "react-responsive"]
---

## Introduction

Doing media queries in CSS is as mundane as it gets. We use media queries to make responsive web designs that are functional and look good on small and big devices.

Sometimes doing media queries at CSS level can get a bit tricky and one may think that would be so much easier just not to render a component at all when certain conditions are reached and this is where we can resort to an external library. Two popular once are `react-media` and `react-responsive`.

## react-media

[react-media](https://www.npmjs.com/package/react-media) is a library developed by the same team that has developed `react-router`. It currently amounts some 115 000 weekly downloads on [npm](https://www.npmjs.com/).

Unfortunately it currently doesn't support hooks and relies on render props.

This example code is from the react-media docs:

```jsx
import React from "react"
import Media from "react-media"

class App extends React.Component {
  render() {
    return (
      <div>
        <Media query={{ maxWidth: 599 }}>
          {matches =>
            matches ? (
              <p>The document is less than 600px wide.</p>
            ) : (
              <p>The document is at least 600px wide.</p>
            )
          }
        </Media>
      </div>
    )
  }
}
```

We import the Media component from `react-media` and use it in the `render` method of the `App` component. We can write the media query in camelCased object format or in standard media query format: `"(max-width: 599px)"`.

## react-responsive

[react-responsive](https://www.npmjs.com/package/react-responsive) is a library developed by the same team that has developed `react-router`. It currently amounts some 140 000 weekly downloads on [npm](https://www.npmjs.com/).

This is a trimmed example from the `react-responsive` documentation.

```jsx
import MediaQuery from 'react-responsive';

const Example = () => (
  <div>
    <div>Device Test!</div>
    <MediaQuery minDeviceWidth={1224}>
      <div>You are a desktop or laptop</div>
    <MediaQuery minDeviceWidth={1824}>
      <div>You also have a huge screen</div>
    </MediaQuery>
  </div>
);
```

We can see that the syntax is much cleaner that the render prop syntax of `react-media`. I took a look at the open issues that it seems that they have also the hook support ready, but it is still in beta.

When they get it ready, the API will look close to something like this:

```jsx
import React from "react"
import MediaQuery, { useMediaQuery } from "react-responsive"

const App = () => {
  const isDesktop = useMediaQuery({ minDeviceWidth: 1000 })
  return (
    <div>
      <h1>{`is desktop: ${isDesktop}`}</h1>
      {isDesktop && <div>Rendered only on desktop</div>}
    </div>
  )
}
```

With the hook support the syntax going get even more concise and easier to read.

## Conclusion

Even if I love the more concise syntax of `react-responsive`, I would go with `react-media`. [This](https://github.com/ReactTraining/react-media/issues/70#issuecomment-347774260) comment explains the difference well between the two libraries. `react-media` is more efficient as it doesn't need to uselessly create components that are not used for in case of no match in the media query. Creating useless desktop components on a small low-end mobile device can really affect negatively on the performance of your site.
