---
title: 'How to "deep populate" using MongoDB and Mongoose?'
published: true
date: "20190808"
tags: ["mongodb", "mongoose", "populate"]
---

## One-layer population

If you found this blog post, you most likely already know what populating a field in a MongoDB document means, you are just wondering how to populate a field in a _populated_ document.

In any case, here is a small example of regular population:

_person.js - schema for persons_

```js
const mongoose = require('mongoose')

const personSchema = mongoose.Schema({
  name: {
    type: String,
    required: true,
  },
  address: {
    mongoose.Schema.Types.ObjectId,
    ref: 'Address',
  }

module.exports = mongoose.model('Person', personSchema)
```

_address.js - schema for addresses_

```js
const mongoose = require('mongoose')

const addressSchema = mongoose.Schema({
  street: {
    type: String,
    required: true,
  },
  zipCode: {
    type: String,
	required: true,
  },
  city: {
    type: String,
	required: true,
  },
  country: {
    type: String,
	required: true,
  },

module.exports = mongoose.model('Address', addressSchema)
```

So we have two schemas, one for person and one for address. The person schema refers to the address schema. That means that the field `address` in `personSchema` contains the `_id` of an address document. With that `_id`, we can _join_ the two documents together in SQL terms.

This is how it is done:

```js
const person = await Person.findOne({
  name: "John",
}).populate("address")
```

We don't have to populate all of the fields of an address document. We can select just the ones we need.

This is how it is done:

```js
const person = await Person.findOne({
  name: "John",
}).populate("address", { street: 1, zipCode: 1 })
```

That would return a `person` document with only `street` and `zipCode` in the address field. I should mention that the `_id` field of the `address` document will always be there. You don't have to explicitly select it.

## Two-layer population

What if you needed to populate a field that is in a document that you need to populate? I know, that phrase is a bit hard to grasp. I try again. Take a look at the `address` schema. What if `country` field was a reference to a `country` document:

```js
const addressSchema = mongoose.Schema({
  street: {
    type: String,
    required: true,
  },
  zipCode: {
    type: String,
	required: true,
  },
  city: {
    type: String,
	required: true,
  },
  country: {
    mongoose.Schema.Types.ObjectId,
    ref: 'Country',
  },

module.exports = mongoose.model('Address', addressSchema)
```

Let's create a schema for the country as well:

```js
const mongoose = require('mongoose')

const countrySchema = mongoose.Schema({
  name: {
    type: String,
    required: true,
  },
  population: {
    type: Number,
    required: true,
  },

module.exports = mongoose.model('Country', personSchema)
```

In the first example we had to populate `address` field with an `address` document. If we populate only once, and we need also details about the `country`, we have to populate more than one layer deep.

If you have encountered this problem, the first thing you try is probably chaining two `populate` methods like this:

```js
const person = await Person.findOne({
  name: "John",
})
  .populate("address")
  .populate("country")
```

This doesn't work and once you think about it you notice that it shouldn't work either. The second `populate` call is trying to populate a field called `country` in the `person` schema, not in the `address` schema.

In order to populate the second layer, the syntax changes. First let's take a look at an example for one-layer population with the other syntax:

```js
const person = await Person.findOne({ name: "John" }).populate([
  {
    path: "address",
    model: "Address",
    select: "street zipCode",
  },
])
```

- `path` refers to the field to be populated. If there were more fields to populate, we could put the here in the same string separated by spaces.
- `model` refers to the model/schema to be used for population.
- `select` let's us select the fields we want to populate.

That is the equivalent of:

```js
const person = await Person.findOne({
  name: "John",
}).populate("address", { street: 1, zipCode: 1 })
```

Now, let's populate also the `country` field:

```js
const person = await Person.findOne({ name: "John" }).populate([
        {
          path: 'address',
          model: 'Address',
          select: 'street zipCode'
		  populate: {
		    path: 'country',
			model: 'Country',
		  }
        },
      ])
```

We added a new property `populate` to the population options. The property as its value has a new object with `path` and `model` properties. If there was yet another field to populate, we could use `populate` property even nested there. So we are not limited to one-layer or two-layer population.

## There's more

You may have noticed that the `populate` method consumes an array as an argument. It is not an array just for laughs. It is an array, because we may need to populate more fields in the same document using different models.

You could do something like this:

```javascript
const person = await Person.findOne({ name: "John" }).populate([
  {
    path: "address",
    model: "Address",
    select: "street zipCode",
  },
  {
    path: "friends",
    model: "Person",
    select: "age",
  },
])
```

That `populate` call would populate also a hypothetical `friends` field with the `_id` and `age` fields of the persons that are listed as friends of John.

## Conclusion

I think that this blog post covers pretty well the cases of field population using `mongoose`. I didn't include populating aggregates, because I wanted to keep this blog post as concise as possible, so that it would be easier to reference to when needed.
