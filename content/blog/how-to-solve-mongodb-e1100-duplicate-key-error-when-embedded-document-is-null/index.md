---
title: "How to solve MongoDB E11000 duplicate key error when embedded document is null"
published: true
date: "20191115"
tags: ["mongodb", "mongoose"]
---

This post is going to be a quick one.

## Problem

I encountered a hairy problem today with my `mongoose` - `MongoDB` setup. I was fetching new data from an API into my database and I was greeted with the following error message:

```
E11000 duplicate key error collection: db_name.collection index: field_name.embedded_field_name key: { : null }
```

My schema is built more less like this:

```js
**GAME SCHEMA**
const gameSchema = mongoose.Schema({
  gameId: {
    type: Number,
    required: true,
    unique: true,
  },
  link: {
    type: String,
    required: true,
    unique: true,
  },
  video: videoSchema,
}

**VIDEO SCHEMA**
const videoSchema = mongoose.Schema({
  videoId: {
    type: Number,
    required: true,
    unique: true,
  },
  link: {
    type: String,
    required: true,
    unique: true,
  },
```

So the error message was saying that I was trying to insert in the DB a document with a duplicate videoId value of `null`. This surprised me, because when I designed the schema, I was thinking that the embedded document (`video` field) was **optional**.

This is not the case. I had already one game document in the DB without `video` field so that was already taking up `null` id for `videoId` field. I couldn't add more documents without `video` field, because their `videoId` would be `null` as well.

## Solution

I needed a way to still mark the `videoId` field as unique, but I needed a way to allow multiple `null` values as well. The solution to this is `sparse` property.

```js
**VIDEO SCHEMA**
const videoSchema = mongoose.Schema({
  videoId: {
    type: Number,
    required: true,
    unique: true,
	sparse: true,
  },
  ...
}
```

That small property saved the day. In order for it to work though, the index of the `game` collection needed to be dropped. The index is created when the first document is inserted in the collection. To recreate it I just dropped the old index reran my fetch script to recreate it.

You can drop MongoDB indexes by running `db.collection.dropIndexes()`.
