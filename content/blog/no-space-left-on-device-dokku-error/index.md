---
title: "No space left on device dokku error"
published: true
date: "20201015"
tags: ["dokku", "deployment"]
---

I have a few projects running on [dokku](http://dokku.viewdocs.io/dokku/). One day I deployed a new release and unexpectedly was greeted with an error message at the end of the commit: `remote: fatal: cannot create directory at 'frontend/src': No space left on device` and `! [remote rejected] master -> master (pre-receive hook declined)`.

The error message is clear. The is not enough space for the new build. The question is _how_ is there _not_ enough space? I have only _four_ apps running each of which shouldnt be more than a gig with the container and everything included.

Time to find out what was robbing the space.

## How to check what directories take up the space?

```shell
du -sh /*
```

`du` is a standard Unix command. I suppose it means _disk usage_. The flags after it are `-s` for getting the total space usage of the directory instead of single files and `-h` for _human-readable_ formatting. The last argument `/*` is the path.

After running the command on the root level, I noticed that `/var` was easily the biggest directory. The output was something like this:

```
15M     /bin
|
|
|
965M    /usr
20G     /var
```

The next step was drilling down to `/var` and run the same command again:

```shell
du -sh /var/*
```

The process needs to continue until you find the root cause. In my case that was found in `/var/lib`. Docker folder was the culprit. Probably something wrong with the containers.

Another common cause could be `/var/log`. You might have some excessive logging going on.

## Checking the health of the Docker daemon

```shell
docker system df
```

`docker system df` shows the amount of disk space used by the Docker daemon. The output in my case was this:

```
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              47                  18                  9.575GB             5.974GB (62%)
Containers          52                  5                   290.7MB             290.2MB (99%)
Local Volumes       2                   1                   0B                  0B
Build Cache         0                   0                   0B                  0B
```

A **whopping** 47 images and 48 containers. That can't be right. I have only 4 apps!

```shell
docker container ls -a
```

`docker container ls -a` gives you a list of all docker containers present in the system. My ouput looked something like this:

```
CONTAINER ID        IMAGE                            COMMAND                  CREATED             STATUS                     PORTS               NAMES
715de26d52cf        ee77575140d7                     "/bin/sh -c 'npm run…"   2 hours ago         Exited (254) 2 hours ago                       infallible_sammet
b3b8648b5d62        ba4d32a75345                     "/bin/sh -c 'npm ins…"   2 hours ago         Exited (228) 2 hours ago                       trusting_goldberg
cb59f6185361        a6d8a8a9fea0                     "/bin/sh -c 'npm ins…"   6 hours ago         Exited (1) 6 hours ago                         nostalgic_austin
f8fc8512def0        dokku/snw:latest                 "node index.js"          2 days ago          Up 2 days                  5000/tcp            snw.web.1
952a363d7e43        dokku/playerfan-landing:latest   "/start web"             3 days ago          Up 3 days                                      playerfan-landing.web.1
57fd98cfb3d3        b91d2e5651dd                     "/bin/bash -c 'mkdir…"   3 days ago          Exited (0) 3 days ago                          relaxed_kilby
461a99095b4e        906276725301                     "/bin/bash -c 'mkdir…"   3 days ago          Exited (0) 3 days ago                          stoic_tharp
7fb994596c1a        d74c56ccc252                     "/bin/bash -c 'mkdir…"   3 days ago          Exited (0) 3 days ago                          blissful_curran
|
|
|
5d68cd37084e        165031171fbc                     "/build"                 2 weeks ago         Exited (0) 2 weeks ago                         nervous_elgamal
db3db0614c35        5c23278731f7                     "/bin/bash -c 'cat >…"   2 weeks ago         Exited (0) 2 weeks ago                         busy_mendeleev
034ed44c8617        bc547b6cd03e                     "/bin/bash -c 'mkdir…"   2 weeks ago         Exited (0) 2 weeks ago                         lucid_tharp
ca88cc2fcf37        gliderlabs/herokuish:latest      "/bin/bash -c 'mkdir…"   2 weeks ago         Exited (0) 2 weeks ago                         boring_meninsky
```

The output was full of exited containers that had been left running there. We will get rid of those later. Let's run the next command first.

```shell
docker image ls
```

`docker image ls` lists all the docker images present in the system. The output was something like this:

```
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
<none>                    <none>              ee77575140d7        2 hours ago         943MB
<none>                    <none>              ba4d32a75345        2 hours ago         944MB
<none>                    <none>              a6d8a8a9fea0        6 hours ago         1.21GB
dokku/snw                 latest              2d15588b1d6e        2 days ago          1.46GB
dokku/playerfan-landing   latest              29e8b4a9dfad        3 days ago          1.47GB
dokku/playerfan           latest              47350dbae961        11 days ago         2.04GB
<none>                    <none>              c1aa2ac74fc8        11 days ago         2.04GB
<none>                    <none>              1453b7f3ee55        11 days ago         2.04GB
<none>                    <none>              4f906504e071        11 days ago         2.04GB
|
|
|
<none>                    <none>              0178b0b978df        12 days ago         1.09GB
<none>                    <none>              a646869a410c        12 days ago         1.09GB
<none>                    <none>              1d361291e72d        12 days ago         1.09GB

```

Also here, the system seemed to be full of useless `<none>` images that take up 2GB space each!

It was clear that Docker was leaving some development related cruft behind and a serious cleanup was needed.

## Cleaning up the system

Dokku comes with a one useful command: `dokku cleanup`. This command should delete all dangling images and exited containers. However, it doesn't always do the job perfectly. You may have to delete some containers and images manually. This was my case.

After running `dokku cleanup` I reran `docker container ls -a`. There was still some exited containers. I reran `docker image ls` and still found a lot of dangling `<none>` images. At this point, I ran `docker system prune`. Running this is risky. You may delete something you didn't intend to delete. When you ran the command, it gives you this warning:

```
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all dangling images
  - all dangling build cache
```

So, if you have, for example, an app container stopped for whatever reason, that will get deleted. You may not want that.

There is a way to delete the containers and images one-by-one, which is a lot safer. For containers, you can use this command:

```shell
docker container rm CONTAINER_ID1 CONTAINER_ID2...
```

The container ids' can be found in the output of `docker container ls -a`.

For the images you maybe use the following command:

```shell
docker image rm IMAGE_ID1 IMAGE_ID2...
```

The image ids' can be found in the output of `docker image ls`.

## Wrap it up

I had a while back when deploying to dokku. The problem was that dokku was leaving behind some cruft from the development process. The commands used to do the cleanup, especially `docker system prune` must be used with care. If you don'g have a gazillion images and container to delete, just delete them one by one by using `docker container rm` and `docker image rm`.
