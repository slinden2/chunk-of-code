---
title: "Deploying a monorepo with backend and frontend directory to Heroku using Git Subtree"
published: true
date: "20190708"
tags: ["deployment", "heroku", "git", "subtree"]
---

Deploying to [Heroku](https://heroku.com/) is not difficult, if you follow the basic convention that they expect you to follow. This convention is that you need to have your project directly in the repository root folder. If you follow that convention, everything is simple and straight-forward. But what if you have structured your project differently and do not want to comply with a service that may change in the future?

In this blog post I will show how I solved this problem. I like to have a single repository for both, frontend and backend. I want them to be in separate folders. I do not want to have anything related to frontend in the backend folder. Maybe I am a purist, but that is how I am.

## Project structure

```
project
│   .git
│   .gitignore
│   LICENCE
│   ...
│
└───backend
│   │   server.js
│   │   ...
│
└───docs
│   │   README.md
│   │   todo.md
│   │   ...
│
└───frontend
    │   index.js
    │   ...

```

This is my basic project structure for a project that has a backend and a frontend. If the project is not huge, I don't see good reasons for separating backend and frontend into separate repositories/services.

## Problems with Heroku's standard approach

As I said before, Heroku requires you to have your project in the root directory. In our case, that would mean that I would need to have a the stuff in the backend folder in the project root. I would also need to have the _build_ folder in the root folder within all my backend stuff. Alternatively, we could have a frontend folder containing build folder in the project root. This is the worst way in my opinion. You would have all your frontend development files within the backend files. That seems messy.

## Preparing for deployment

I am going to assume that you are somewhat familiar with Heroku to keep this post to the point. I also assume that the frontend project is created with CRA (create-react-app). This is a really barebones example of how I did my build setup. Of course, I need to assume that you already know how to configure your backend for the production build and that you have already a frontend that is ready to go.

Let's get started.

First thing we need is a `Procfile` in our backend folder. In this case the contents of the file are:

```
Procfile
web: node server.js
```

Make sure that your `package.json` file uses the same script for starting the server:

```json
{
  "scripts": {
    "start": "node server.js"
  }
}
```

Then you may build your frontend by running the build command in the frontend folder: `npm run build`. This creates a build directory in the frontend folder.

Now, we have the pieces, but we need to glue them together somehow. The answer is a deployment script.

## How to glue backend and frontend together?

Before delving to the deployment script, we still need to figure out one thing. Where to store the deployment files and how to manage them? I wanted a separate folder in my project repository that contained the production versions of both, frontend and backend. The solution for this problem is _Git subtree_.

Git subtree basically allows you to manage multiple repositories in one. The main repository would have the following structure:

```
project
│   .git
│   .gitignore
│   LICENCE
│   ...
│
└───backend
│   │   server.js
│   │   ...
│
└───docs
│   │   README.md
│   │   todo.md
│   │   ...
│
└───frontend
│   │   index.js
│   │   ...
│
└───web
    │   server.js
    │   ...
    │
    └───build
        │   index.js
        │   ...
```

Notice that there is a new directory _web_ at this point. That is the directory that will contain the production version of the frontend and the backend.

So, the idea is to push only the _subrepo_ to Heroku, while the main repository with all its subfolders (including _web_) will be pushed to [GitHub](https://github.com/).

You should have Heroku installed and you should be logged in at this point. Navigate to your project root and run `heroku create`.

Now, in order to use a _subtree_ in our Heroku remote, we would run `git subtree push --prefix web heroku master`. However, we are not done yet. We still need a script that handles the deployment for us.

## Deployment script

Let's do the script in Python. You could use any scripting language or even Node, but I am quite familiar with Python so I am doing the example with it.

Let's create a file called `heroku-deploy.py` in the project root folder.

The file will be like this:

```Python
import os
import shutil
import sys

FILE_PATH = os.path.dirname(os.path.realpath(__file__))


def init_deploy():
    shutil.rmtree(os.path.join(FILE_PATH, 'web'))


def copy_backend():
    shutil.copytree(
        os.path.join(FILE_PATH, 'backend'),
        os.path.join(FILE_PATH, 'web'),
        ignore=shutil.ignore_patterns('node_modules', '.env', 'db.json', '.eslintrc.js', 'tests'))


def build_frontend():
    os.system('cd frontend && npm run build')


def copy_build_folder():
    shutil.copytree(
        os.path.join(FILE_PATH, 'frontend', 'build'),
        os.path.join(FILE_PATH, 'web', 'build'))
    shutil.rmtree(os.path.join(FILE_PATH, 'frontend', 'build'))


def push_to_heroku():
    os.system('git add .')
    os.system('git commit -m "Commit from heroku-deploy.py for deployment"')
    os.system('git push origin master')
    os.system('git subtree push --prefix web heroku master')


def main():
    init_deploy()
    copy_backend()
    build_frontend()
    copy_build_folder()
    push_to_heroku()


if __name__ == "__main__":
    main()

```

If you are familiar with Python (to be honest, even if you are not...), the code is quite self-explanatory.

First, we initialize the building process by deleting the existing `web` folder by calling `shutil.rmtree(os.path.join(FILE_PATH, 'web'))`. `FILE_PATH` refers to the location where the script file lives.

Then we copy the contents of the backend folder to a new web folder. Note that here you should ignore all the files that are not used in the production version. In my case those are:

- node_modules
- .env
- db.json
- eslintrc.js
- tests

After that we create the actual build of the frontend. This may take a while, but as the script takes care of everything, we don't have to worry about it.

Then we copy the build folder from the frontend folder in to the web folder, and finally we remove the build folder from the frontend folder.

At this point, web folder is completely isolated from our development files. Now we can just push it to Heroku. The actual git commands to run are:

```git
git add .
git commit -m "Commit from heroku-deploy.py for deployment"
git push origin master
git subtree push --prefix web heroku master
```

So, we stage the changes, create a commit with a standard commit message, push everything to [GitHub](https://github.com/), and finally push _only_ the web folder to Heroku.

That is actually all there is to it. I use this build setup daily in one of my projects and currently I am pleased with it.
