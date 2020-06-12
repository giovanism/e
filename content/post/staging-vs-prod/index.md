+++
title = "Staging vs Prod and More..."
date = 2020-05-12T07:34:07+07:00
description = ""
draft = false
toc = false
categories = ["ppl"]
tags = [""]
images = [
  "https://upload.wikimedia.org/wikipedia/commons/thumb/d/df/ASR-33_at_CHM.agr.jpg/800px-ASR-33_at_CHM.agr.jpg"
] # overrides the site-wide open graph image

[[resources]]
  name = "requirements"
  src = "requirements.png"
  title = "Multiple requirements.txt Files"

# external resources
# 1. https://upload.wikimedia.org/wikipedia/commons/4/41/Space_Shuttle_Columbia_launching.jpg
+++

{{<figure
  alt="First launch of Space Shuttle Columbia "
  src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/41/Space_Shuttle_Columbia_launching.jpg/1024px-Space_Shuttle_Columbia_launching.jpg"
  title="What can still go wrong?"
  attr="NASA"
  attrlink="https://commons.wikimedia.org/wiki/File:Space_Shuttle_Columbia_launching.jpg"
>}}

Hi! This time, I want to talk more about software environments. Even though our
code has been tested using TDD, there are still many more factors that we have
to take into account when we are deploying our application, whether it goes to
staging or straight to production environment. Previously I had touched on the
subject briefly in my [clean code][clean-code] post. However, that's only
limited to the application level. There are also other system configuration
that's invisible to the application.

<!--more-->

## Application

On application level, the configuration for different environments is placed on
separate settings modules. This way we can define things separately and even
reuse some codes defined in `common.py`. Here is some lines from
`production.py`.

```python
from .common import *

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = os.getenv('SECRET_KEY')

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = os.getenv('DEBUG', 'True') != 'False'

ALLOWED_HOSTS = os.getenv('ALLOWED_HOSTS', 'localhost').split(';')
```

While this one comes from the `staging.py`.

```python
from .production import *

SECRET_KEY = os.getenv('SECRET_KEY', '__secret_high_entropy_random_bytes__')

DEBUG = True

ALLOWED_HOSTS = ['*']


# Application definition

INSTALLED_APPS.extend([
    'django_extensions',
])
```

I think some obligatory explanation is needed here for context, contrary to the
same example I had given on the [clean code][clean-code] post where I just
shoved these codes to the readers.

So, `staging.py` settings module is used mainly for development purposes. Local
development environment, automated tests environment, and staging environment
use this setting module. This way, it provides convenient default value while
still not compromising on security risks. `staging.py` settings module also
includes [`django_extensions`][django-extensions] package that has more
convenient debugging tools than the default one. One such feature that I use
often is the `shell_plus` management command. It has the addition of python3
shell-esque method of accessing expressions history with arrow up and arrow down
keys that I find very useful.

## Packaging and Dependency

Now, both of this topic can be explained in one section. I can also hear your
mind thinking something along the line "I know your next line you are going to
say is 'docker all the things'.". Well, it's true that I use docker but, it
doesn't solve every _staging vs production_ problem.

First, as shown on the previous `staging.py` snippet, the combination of
packages being used varies between environment. This also calls  multiple
version of `requirements.txt` file, one for development/staging and another one
for production. Here are what our project uses.

{{<resfigure
  alt="Three different requirements.txt files"
  src="requirements"
  title="Multiple requirements.txt Files"
>}}

Pardon the usage of screenshot. It's prettier to look at this way. Before
talking about docker, I want to talk about python dependencies first. As you
can see, there are other development dependencies beside `django_extensions`.
The difference between them and `django_extensions` is the fact that they don't
need Django's initialization process to work.

Another key difference between the `-dev` and `-prod` version is the usage of
`psycopg2-binary` and `psycopg2` package. Both provides Django's PostgreSQL
database backend adapter, but both have their own quirks. The `-binary` version
is made to be easy for development process as they bundled the compiled C
extension binary into the python wheel package. However, it has several
[issues][psycopg2-binary-issues] that made it unsuitable for production use.
In turn, the non `-binary` version also requires PostgreSQL development header
files during its build process. This will translate to requiring a package
called `postgresql-dev` or `postgresql-devel` into the system. This is less than
ideal for developer's development environment.

TL;DR. We need both packages, so we maintain both version of requirements.txt
files.

Now that the difference has reached system packages level, this is the perfect
time to introduce container as the solution for this problem. Docker provides
a packaging format, essentially a container image, that allows us to ship system
dependencies together with our application.

Here is our production `Dockerfile`, also previously explained in the [building
docker container image][building-docker-container-image] post.

```Dockerfile
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
ENV STATIC_URL '/api/static/'

RUN ["python", "manage.py", "installtasks"]

CMD ["gunicorn", "dblood.wsgi", "--bind", "0.0.0.0:8000"]
```

Here is the `dev.Dockerfile` version that is used on our staging environment.

```Dockerfile
FROM python:3.8-buster

RUN apt-get update -q && \
    apt-get install -y libpq-dev python3-dev cron

WORKDIR /app
COPY requirements.txt /app/
COPY requirements-dev.txt /app/

RUN pip install -r requirements-dev.txt

COPY . /app/
EXPOSE 8000

ENV DATABASE_URL 'sqlite:///db.sqlite3'
ENV DJANGO_SETTINGS_MODULE 'dblood.settings.staging'
ENV STATIC_URL '/staging/api/static/'

RUN ["python", "manage.py", "installtasks"]

CMD ["gunicorn", "dblood.wsgi", "--bind", "0.0.0.0:8000"]
```

At this point we are almost done on our journey in the CI (Continuous
Integration) process. Both the staging and production version of our application
container image will be pushed to a single repository on [DockerHub][docker-hub]
with different tags. This push events will trigger our CD (Continuous Delivery)
process by calling our registered webhooks from DockerHub.

## Host, Container Orchestration and Proxies

So, the major difference between our staging and production environments end at
the previous paragraph. This topic won't have major difference between
staging and production enviroment beside some environment variables, access
control, and backup mechanism.

So, I think I will put this section on "To be continued ..." for now. So,
without further ado.

To be continued ...

[building-docker-container-image]: /post/building-docker-container-image/
[clean-code]: /post/clean-code/
[django-extensions]: https://pypi.org/project/django-extensions/
[docker-hub]: https://hub.docker.com/
[psycopg2-binary-issues]: https://www.psycopg.org/articles/2018/02/08/psycopg-274-released/
