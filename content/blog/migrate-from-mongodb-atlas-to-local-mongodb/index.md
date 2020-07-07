---
title: "Migrate from MongoDB Atlas to a local MongoDB database"
published: true
date: "2020707"
tags: ["mongodb", "dokku", "mongodump"]
---

## Introduction

I have been using MongoDB Atlas for my hobby projects for over a year now. Usually I deploy all my hobby projects to my VPS where I run `dokku`. I wanted to try to migrate one of my databases to `dokku` to be run on the server where my application resides. I wanted to do this mainly because that way I could have all the app data centralized. For a hobby project this is not an issue but an advantage, as you don't have to log in to different systems to check stuff up.

mongodump and mongorestore cannot be part of a backup strategy for 4.2+ sharded clusters that have sharded transactions in progress, as backups created with mongodump do not maintain the atomicity guarantees of transactions across shards.

## Hands-on

### Backing up existing Atlas database

So, I practically needed to create a duplicate of my existing database to be able to import it later into the new database service on `dokku`. It can achieve this by using `mongodump`. I ran this directly on my project server as I needed the dump file there.

If you don't have MongoDB on your server, you will have to install `mongodump` sepately in your system. This was also my case, as I was planning to run MongoDB, but I wanted to run it in a container using `dokku`, so I didn't have `mongodump` available outside of it. I think I could have entered the container and use run `mongodump` from there, but I chose to do it the other way this time.

You can follow [these](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/) instructions to install `mongodump` on your server if you are running Ubuntu as I am. Note that `mongodump` is part of `mongodb-org-tools` package. You don't need to install MongoDB completely.

> Note! As it says in the MongoDB manual, if you are running a newer version than 4.2 of MongoDB database in your Atlas, you can't use `mongodump`. I am not familiar of the backup strategies for those databases.

To find the specific `mongodump` command you need to run, you can navigate to _Command Line Tools_ section of your Atlas database and find it available there. In my case the command is:

```
mongodump --host <CLUSTERNAME>-shard-0/<CLUSTERNAME>-shard-00-00-jomgh.mongodb.net:27017,<CLUSTERNAME>-shard-00-01-jomgh.mongodb.net:27017,<CLUSTERNAME>-shard-00-02-jomgh.mongodb.net:27017 --ssl --username admin --password <PASSWORD> --authenticationDatabase admin --db <DATABASE> --archive=db.dump --gzip
```

I have added `--archive` and `--gzip` commands there myself as they are not the by default. `--archive` makes the output to be a single file instead of a bunch of `.bson` files and `--gzip` compresses the file.

Now you should have a file called `db.dump` in the directory where you ran the command (for example, ~/db.dump).

### Creating MongoDB service for dokku

There is an official dokku plugin for MongoDB. You can install it by running:
`dokku plugin:install https://github.com/dokku/dokku-mongo.git mongo`

When you create a new service, the plugin defaults to installing the 3.6.15 version of MongoDB. We can change this by setting a couple of environments variables:

```
export MONGO_IMAGE="mongo"
export MONGO_IMAGE_VERSION=4.2"
```

Now we can run the create command and a MongoDB service of 4.2.8 version is created:

```
dokku mongo:create myproject-mongo
```

Here `myproject-mongo` is just a naming scheme that I use for the dokku services. You can use whatever name you prefer.

Now the service is created. Now we still have to link it to an application by running:

```
dokku mongo:link myproject-mongo myproject
```

An environment variable called `MONGO_URL` is automatically created within `myproject` and that can be used in the app to connect to the database.

### Importing dump file into the local MongoDB service

This is the easiest step, if the preparations have been done correctly.

I just ran:

```
dokku mongo:import playerfan-mongo < db.dump
```

`db.dump` must correspond to the path where you saved the dump of your database.

You will get an error, if you didn't use `--gzip` flag when executing `mongodump`. The import function requires the dump file to be gzip compressed.

## Conclusion

It is just stunningly easy to handle databases with dokku. Not only MongoDB, but also other databases like PostgreSQL. Dokku has a very nice plugin library and on a single cheap hobby server you can run various applications and services easily in their containers and when you don't need them anymore there is usually a destroy command to do all the cleanup for you.

That's all today!
