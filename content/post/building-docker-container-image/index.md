+++
title = "Building Docker Container Image"
date = 2020-04-28T07:21:24+07:00
description = ""
draft = false
toc = false
categories = ["ppl"]
tags = ["computer", "container", "docker"]
images = [
  "https://upload.wikimedia.org/wikipedia/commons/thumb/d/df/ASR-33_at_CHM.agr.jpg/800px-ASR-33_at_CHM.agr.jpg"
] # overrides the site-wide open graph image
+++

Hi! We are back to another post in PPL series. Here, I will explain about how
our team build small and efficient docker image that is used to ship and deploy
our server side application. Our server side application is Django-based app. So
this article can also serve as reference on how to build another similar python
based server side application. I will skip most of the container vs VM
explanation. So, you are expected to be already familiar with docker fundamental
concepts such as container, container image, its layers, and container registry.

<!--more-->

Explaining things in top-down approach is faster to write down. So, without
further ado, this is our team Django app Dockerfile.

```dockerfile
FROM python:3.8-alpine as builder

RUN apk add --no-cache postgresql-dev gcc python3-dev musl-dev libffi-dev

WORKDIR /app/
COPY requirements.txt ./
COPY requirements-prod.txt ./
RUN pip wheel --no-cache-dir \
              --wheel-dir /app/wheels \
              -r requirements-prod.txt


FROM python:3.8-alpine

RUN apk add --no-cache libpq postgresql-client

WORKDIR /app
COPY --from=builder /app/wheels/ /wheels
RUN pip install --upgrade pip
RUN pip install --no-cache /wheels/*

COPY . /app/

EXPOSE 8000

ENV DATABASE_URL 'sqlite:///db.sqlite3'
ENV DJANGO_SETTINGS_MODULE 'dblood.settings.production'
ENV DEBUG 'False'

CMD ["gunicorn", "dblood.wsgi", "--bind", "0.0.0.0:8000"]
```

You like fast and small things right? No? But production environment does. With
this configuration we can achive docker container image of size __176MB__. It's
still pretty big compared to docker container image of critical systems such as
nginx. But, hey, its a python based app so we can overlook this once. Here is
the comparison to other versions of our Dockerfile.

```sh
$ docker images giovanism/dblood
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
giovanism/dblood    single              0861531f0246        59 seconds ago      336MB
giovanism/dblood    multi               64fdd2ca2bd8        5 minutes ago       176MB
giovanism/dblood    staging             02b3ae30dc17        2 days ago          1.13GB
```

From here on is for nerds who want to know which line does what. To achieve such
result we employed several techniques. Well, not all techniques explained here
will give significant results, but I hope they are usefull to keep in mind.

## Alpine Base Image

One of the easiest thing to do to reduce your docker container image size is
to change your base image to alpine. [Alpine][alpine] is one of many GNU/Linux
distros that is available on official dockerhub container image library. Its
meant to be lightweight by using `musl` libc and `busybox` instead of the more
popular GNU based variant. Using `musl` libc can be quiet tricky when your app
depends on compiled binary because they have to be compiled and run against the
same libc.

Moreover `musl` is not a silver bullet. By trading `glib` with `musl` you have
to give up on locale support and many improvements and fixes that `glib`
developers have work over the decades they are being maintained. Point is, your
milage may vary, but `musl` will work most of the time.

## Multi Stage Build

Here we also employ multi stage build. You can notice this from __multiple__
`FROM` command in our Dockerfile. We have a build stage and the unnamed final
stage. In a multi stage build, only the final stage will be tagged as our
`-t <tag-name>` docker container image. Whats special is the fact that it
doesn't directly depend on the previous stages layers. Instead, we use `COPY`
command to move artifacts from previous stages to our final image.

So, thats the basic idea. Here in our python app we use the build stage to build
build dependency for some packages in `requirements-prod.txt` file. They are not
installed; however, they are used to build python wheel packages as artifact.
What's special about python wheel packages is that they no longer need further
building or compilation like some packages straight from [PyPI][pypi]. Thus we
can install them on our final stage without their build dependency. However,
this doesn't apply to runtime dependency. You still have to install them.

## Separete Your Dev And Prod Packages

Well, this is obvious. But there aren't many decent dependency management for
python packages. So, we maintain 3 requirements.txt file one for common, dev,
and prod dependencies. Other alternative such as `pipenv` comes to mind but it
introduces lots of dependency in itself.

## RUN Sequence Matters

This is more of a tip rather than technique. You can write your Dockerfile `RUN`
commands in a way that utilize your cache more efficiently. Example such as
installing from requirements.txt first before copying your app sources. This way
you are less likely to install your dependency over and over again because of
changes in your source files.

Okay, this is about what I can explain here with my already short time.
Thank you and see you again in another posts.

## P.S.

Okay here is some additional content about our container orchestration.
Currently, we run and serve our applications inside containers, except for some
proxy logics which is run by normal Nginx daemon. This is configured using plain
old docker-compose.yml file. `api` container is the one I explained on how to
build the docker container image previously.

```

                    +-----------------------------------+
 +------------------+               NGINX               +-----------------+
 |                  +-----------------------------------+                 |
 |          +--------^     +--^                                           |
 | 8000:8000|              |8080:8000                                     |
 | +---------------------------+  +-------------------------------------+ |
 | |        default net        |  |            internal net             | |  <--+  container network
 | +---------------------------+  +-------------------------------------+ |
 |         ^              ^           ^            ^               ^      |
 | +----------------+ +--------------------+ +------------+ +-----------+ |
 | |      web       | |         api        | |     db     | |  memcache | |  <--+  containers
 | +----------------+ +--------------------+ +------------+ +-----------+ |
 |      ^ /media_path       ^ /media_path      ^ /data/postgresql         |
 |    +------------------------------------------------------------+      |
 +----+                                                            +------+
      |                        HOST VOLUMES                        |
      +------------------------------------------------------------+
```

[alpine]: https://www.alpinelinux.org/
[pypi]: https://pypi.org/
