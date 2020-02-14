---
title: "Simple dynamic search with Vanilla Javascript (ES6)"
published: true
date: "20190509"
tags: ["ES6", "search", "javascript"]
---

In this blog post I will show you how to do a dynamic search with Javascript. With _dynamic_ I mean that the search results update while the user is typing.

I did this for the company that I work at the moment. I am not a full-time developer, but a Production Manager. Usually the guys in the production don\'t need a computer and they have no idea how to use something like an ERP system, but lately the need for looking up some part numbers has grown so that there have been minor delays in maintenance response time. The tool that they need shouldn't be anything fancy, just something that gets the job done and the work in the factory keeps going. A full-blown ERP would be an overkill for this job and the extra licence would also cost something.

I, being passionate about programming and Javascript, decided to provide them the means to do their work better. A simple Javascript search field with a .csv file holding all the part numbers and description should do the job.

## Let's get started

Firstly, I exported the part numbers from the ERP. You can download the fake file [here](./partnumbers.csv) if you wish to follow along. This is the html markup needed for the tool:

```html
<html lang=en>

<head>
  <title>Iteco Ricerca Codici</title>
  <meta charset=utf-8>
  <meta name=viewport content=width=device-width, initial-scale=1>
  <link rel=stylesheet type=text/css href=index.css>
</head>

<body>
  <div id=root>
    <div class=search>
      <input class=search-field type=text placeholder=Cerca>
    </div>
    <div class=container></div>
  </div>
  <script src=https://d3js.org/d3.v5.min.js></script>
  <script type=text/javascript src=js/index.js></script>
</body>

</html>
```

Take a look at the script tags. I am using an external library called `d3` for loading the `.csv` file. The library can be used for much, much more, but in this case I just use it for the `.csv` file. The second script tag loads the Javascript code that is used for the dynamic search functionality.

This file is basically the same that our technicians are going to use. The only difference is that I removed the company logo from it.

Here is the `.css` file:

```css
#root {
  margin: auto;
  width: 90%;
  border: 2px solid;
  border-radius: 18px;
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif

.search {
  display: flex;
  align-items: center;
  justify-content: center
}

.search-field {
  border-radius: 18px;
  padding: 10px;
  border: 2px solid rgb(209, 209, 209);
  width: 400px;
  height: 10px;
  margin-bottom: 10px;
}

.container {
  display: flex;
  align-items: center;
  justify-content: center;
  justify-items: center;
}

.table > tbody > tr:nth-child(odd) {
  background-color: rgb(224, 224, 224);
}
```

The CSS just give the basic styling for the page so that it is usable.

Then the Javascript file. Note that it has to be in a sub-folder called _js_:

```js
const container = document.querySelector(".container")
const searchField = document.querySelector(".search-field")

const path = "excel_tmp.csv"
let codes = []

const fetchData = async path => {
  return await d3.dsv(";", path)
}

fetchData(path)
  .then(data => (codes = codes.concat(data)))
  .then(createTable)

function createTable(codes = codes) {
  const table = document.createElement("table")
  table.className = "table"
  container.appendChild(table)

  const tbody = document.createElement("tbody")
  table.appendChild(tbody)
  for (const row of codes) {
    const tr = document.createElement("tr")
    for (const key in row) {
      const td = document.createElement("td")
      td.textContent = row[key].slice(0, 80)
      tr.appendChild(td)
    }
    tbody.appendChild(tr)
  }
}

const filterCallback = (code, value) => {
  return (
    code.Codice.toLowerCase().includes(value.toLowerCase()) ||
    code.Descrizione.toLowerCase().includes(value.toLowerCase())
  )
}

const setFilter = ({ target: { value } }) => {
  container.removeChild(container.firstChild)
  const codesToShow = codes.filter(code => filterCallback(code, value))
  createTable(codesToShow)
}

searchField.addEventListener("keyup", setFilter)
```

That is all there is. That is what is needed to load the `.csv` file and dynamically rerender the result table based off of the filter.

Let's split it up so that I can explain what is going on.

```js
const container = document.querySelector(".container")
const searchField = document.querySelector(".search-field")

const path = "excel_tmp.csv"
let codes = []
```

Here we get the DOM elements that we need for the application. `container` is the `div` where we will be putting the result table.

`searchField` is pretty self-explanatory and `codes` is the variable that holds the array of art numbers retrieved from the `.csv` file.

```js
const path = "partnumbers.csv"

const fetchData = async path => {
  return await d3.dsv(";", path)
}

fetchData(path)
  .then(data => (codes = codes.concat(data)))
  .then(createTable)
```

Then we create a new function called `fetchData`. This is an asynchronous function so `async/await` syntax is needed. We return the .csv file by calling `await d3.dsv(';', path` where semicolon is the delimiter and path points to the file that we are fetching.

At this point the `codes` variable contains an array of objects. Every object has two properties: _code_ and _description_. These come from the headers of the source file.

Then we pass the date into `codes` and then we call `createTable`. `codes` is passed to the function automatically as an argument.

```js
function createTable(codes = codes) {
  const table = document.createElement("table")
  table.className = "table"
  container.appendChild(table)

  const tbody = document.createElement("tbody")
  table.appendChild(tbody)
  for (const row of codes) {
    const tr = document.createElement("tr")
    for (const key in row) {
      const td = document.createElement("td")
      td.textContent = row[key].slice(0, 80)
      tr.appendChild(td)
    }
    tbody.appendChild(tr)
  }
}
```

This is the function that we use for the table creation. Note that I decided to use _function declaration_ syntax instead of _function expression_ here. This is because function created with the _function declaration_ syntax are _hoisted_. This means that they can be called before their creation as we did in the `fetchData` chain. This way we are able to follow the program from up to bottom. This may not be the best way to structure the program always, but I think in this case it looks nice.

First we create the table element and add the class `table` to it. Then we append the table to the `container` element.

After that we create the `tbody` element and attach that to the table element. Then we iterate over the `codes` array and create a `tr` element for every row.

```js
for (const key in row) {
  const td = document.createElement("td")
  td.textContent = row[key].slice(0, 80)
  tr.appendChild(td)
}
tbody.appendChild(tr)
```

Here we iterate over all the rows and create the `td` elements. Then we add the content from the object by calling `td.textContent = row[key].slice(0, 80)`. Remember that the keys are _code_ and _description_ from the source file. `slice()` method is called in order to limit the lengths of the descriptions to 80 characters. Then the row is appended to the row element.

We've reached the last part of the code:

```js
const filterCallback = (code, value) => {
  return (
    code.Code.toLowerCase().includes(value.toLowerCase()) ||
    code.Description.toLowerCase().includes(value.toLowerCase())
  )
}

const setFilter = ({ target: { value } }) => {
  container.removeChild(container.firstChild)
  const codesToShow = codes.filter(code => filterCallback(code, value))
  createTable(codesToShow)
}

searchField.addEventListener("keyup", setFilter)
```

Check the last row first. We define the `searchField` elements to listen to `keyup` events and use `setFilter` as its callback function. Once again, the event will be passed to the call back automatically, so we will be able to access the value of the field after every key press.

In the `setFilter` function, we destructure the event so that we can access the value property directly. We define `codesToShow` variable to which we save the filtered array of part numbers. The actual filter is defined in `filterCallback` function. It compares both the values of part numbers and description, so that the user is able to perform both kinds of searches.

## Future development

This small application is used by the factory technicians. They need to access the part numbers so that they can communicate to the warehouse specifically which components they need for a certain maintenance task. It works fine but the technician will still need to manually write down the codes with the quantity before communicating their needs to the warehouse.

The next step would be adding a shopping cart, so that they could just click a part number and add it to their part list where they could change the quantity. It shouldn't be to hard of a task, but we'll see if I can make time for it in the future.
