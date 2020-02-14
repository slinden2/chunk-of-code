---
title: "How to define multiple types for a field in MongoDB/mongoose"
published: true
date: "20190815"
tags: ["mongodb", "mongoose", "refpath"]
---

## Introduction

Sometimes when you are designing a database schema, you may encounter a situation where you would need to define more than one types for a document field.

If you are no familiar with how to populate fields in `mongoose`, please take a look at my previous article [here](/how-to-deep-populate-using-mongodb-and-mongoose/).

Say that we have a schema for a `person` document like this:

```
|-- person
    |-- name
	|-- age
	|-- personClass
	|-- abilities
```

`name`, `age` and `personClass` are simple field with no subdocuments, but the `abilities` field should contain a subdocument with some attributes that define the abilities of the person.

Let's try to make this clearer with an analogy. There is a person. His name is Gael and he is 500 years old. The `personClass` of Gael is _elf_. Elves have some abilities that are intrinsic to their class: _healing_ and _telepathy_.

Then there is another person, Thor. Thor is 50 years old and he is a _dwarf_. The abilities that Thor has are: _crafting_ and _axe-handling_.

Both Gael and Thor are persons that share some common data, in fact, all of the fields in the person document must be the same. Anyway, they have different abilities, so the subdocument linked to the person document can't be the same.

If subdocument is of a different type, how can we populate it when needed? The answer is `refPath`.

## Working with refPath

First, let's get our example schemas ready. We need 3 schemas in total: _person_, _dwarf-ability_, _elf-ability_.

person.js

```javascript
const mongoose = require('mongoose')

const personSchema = mongoose.Schema({
  name: {
    type: String,
    required: true,
  },
  age: {
    type: Number,
    required: true,
  },
  personClass: {
    type: String,
    required: true,
  },
  abilityType: {
    type: String,
    required: true,
	enum: ['ElfAbility', 'DwarfAbility'],
  },
  abilities: {
    type: mongoose.Schema.Types.ObjectId,
	refPath: 'abilityType',
  }

module.exports = mongoose.model('Person', personSchema)
```

elf-ability.js

```javascript
const mongoose = require('mongoose')

const elfAbilitySchema = mongoose.Schema({
  healing: {
    type: Number,
    required: true,
  },
  telepathy: {
    type: Number,
    required: true,
  },

module.exports = mongoose.model('ElfAbility', elfAbilitySchema)
```

dwarf-ability.js

```javascript
const mongoose = require('mongoose')

const dwarfAbilitySchema = mongoose.Schema({
  crafting: {
    type: Number,
    required: true,
  },
  axeHandling: {
    type: Number,
    required: true,
  },

module.exports = mongoose.model('DwarfAbility', dwarfAbilitySchema)
```

Actually everything there is to it is in the person schema. See how we defined a new field `abilityType`. This field will contain all schema types that we want to use in our `abilities` field. `abilityType` will only validate if its value is one of the strings defined in the `enum` array. This way we can be sure that the field contains a name of an actual schema that we are going to be using for population.

The `abilities` contains a reference to an existing document `_id`. That is the document that is going to be used for population. Notice that we use `refPath` instead of `ref` here. Normally we would use write something like `ref: 'DwarfAbility'` if we had to reference only one document type. `refPath: 'abilityType'` means that `mongoose` will check the `abilityType` field to get the schema name to which to reference. We defined `ElfAbility` and `DwarfAbility` as the only valid values for `abilityType`, so now we can safely populate all dwarf and elf documents with their respective abilities.

## Conclusion

If you are able to learn the concepts of this blog post and you have read my blog post on [population](../how-to-deep-populate-using-mongodb-and-mongoose/), you have already a good grasp of how population works in `mongoose`.

This stuff is not easy to learn before you need it in your own project, but once you do, remember these blog post and come back and read them again. Once you have a real application where you need to do something similar, the concepts will get easier to understand and memorize.

P.S. I hope my fantasy analogue was not too bad. 😊
