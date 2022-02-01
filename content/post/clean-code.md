---
categories:
- ppl
date: "2020-05-12T07:23:21+07:00"
description: ""
draft: false
images:
- https://upload.wikimedia.org/wikipedia/commons/thumb/d/df/ASR-33_at_CHM.agr.jpg/800px-ASR-33_at_CHM.agr.jpg
tags:
- ""
title: Clean Code 101
toc: false
---

{{<figure
  alt="Photograph of Veil Nebula"
  src="https://upload.wikimedia.org/wikipedia/commons/thumb/6/6e/Veil_Nebula_-_NGC6960.jpg/1024px-Veil_Nebula_-_NGC6960.jpg"
  title="Veil Nebula (NGC 6960), in the constellation Cygnus"
  attr="Ken Crawford"
  attrlink="https://en.wikipedia.org/wiki/Ken_Crawford_(astrophotographer)"
>}}

Here is another post in PPL series. In this post I will try to explain "Clean
Code" concept and some tips on how to apply it.

<!--more-->

## What is Clean Code?

Most of clean code's popularity in software engineering discipline can be
attributed to Robert C. Martin's *Clean Code: A Handbook of Agile Software
Craftsmanship*. This is the de facto resource material you should be reading to
learn clean code instead of this post (lol). However, the idea behind clean code
might have been around well before Uncle Bob's book. If I have to take an
example the classic KISS principle, "Keep It Simple, Stupid", comes to mind.

Enough with the chitchat. This is the best explanation I can find about clean
code.

{{<blockquote
  text="Clean code is code that is easy to understand and easy to change."
  cite="Carl Vuorinen"
  citelink="https://cvuorinen.net/2014/04/what-is-clean-code-and-why-should-you-care/"
/>}}

I couldn't agree more with this one sentence summary of what clean code is. The
horror of incomprehensible black box code that refuses to change has been
subject to many programming creepypastas like this one sample below.

{{<figure
  alt="a code comment"
  src="https://miro.medium.com/max/1400/1*NT1um6PYhJU4q9E26cQUew.png"
>}}

Whether such thing is genuine or not, you get the point. This is the very
problem clean code tries to solve. On another note, you can also check
[this][quotes] goodread's compilation of quotes from the *Clean Code* book. The
quotes sound poetic and have some of the most powerful clean code messages.
They would certainly make good wallpaper material *wink*. Below is one example.
You're welcome.

{{<blockquote
  text="One difference between a smart programmer and a professional programmer is that the professional understands that clarity is king. Professionals use their powers for good and write code that others can understand."
  cite="Robert C. Martin, Clean Code: A Handbook of Agile Software Craftsmanship"
  citelink="https://www.goodreads.com/quotes/7030076-one-difference-between-a-smart-programmer-and-a-professional-programmer"
/>}}


## Applying Clean Code

Thanks. But, that still doesn't explain how to apply clean code. Of course you
better read the full guides on Uncle Bob's book rather than having me
half-assedly paraphrase them for you. Alternatively, I find a quick
[summary][summary] that can help you navigate different topics on clean code.
However, for the sake of completeness I will also demonstrate how we apply some
of clean code concepts to our project.

There is a time in our development phase where we need to move our backend app's
enviroment multiple times.
Sure, its 2020 and container technology easily handles some packaging problemes.
However, its still a too steep of a learning curve for our teammates.
So we still need to handle these settings on project level.
Thus, our Django's settings module also grows larger as we need to account for
different scenarios such as local development, staging, and production
enviroment.

This one is quite simple.

```python
if DEBUG:
    ALLOWED_HOSTS = ['*']
else:
    ALLOWED_HOSTS = os.getenv('ALLOWED_HOSTS', 'localhost').split(';')
```

Mutating state, anybody?

```python
if DEBUG:
    INSTALLED_APPS.extend([
        'django_extensions',
    ])
```

This one is quite dangerous if not properly configured on the production
enviroment.
The missing `SECRET_KEY` security warning will be silenced by the default value.

```python
SECRET_KEY = os.getenv('SECRET_KEY', '__secret_high_entropy_random_bytes__')
```

It's apparent that, so many if/else over our settings module became too
repetitive and not easy to change. Its not *clean*. As one of the clean code
design rules says:

> Prefer polymorphism to if/else or switch/case.

Thus, we need to *refactor* our settings module.
Thankfully, this is a common problem for many people using Django and their
solution is to use multiple settings files. The `settings.py` file is removed
and instead we have settings directory under our project directory.

```bash
$ tree dblood
dblood
├── asgi.py
├── __init__.py
├── settings
│   ├── common.py
│   ├── production.py
│   └── staging.py
├── urls.py
└── wsgi.py
```

This way we can define things separately and even reuse some codes defined in
`common.py`. Here is some lines from `production.py`.

```python
from .common import *

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = os.getenv('SECRET_KEY')

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = os.getenv('DEBUG', 'True') != 'False'

ALLOWED_HOSTS = os.getenv('ALLOWED_HOSTS', 'localhost').split(';')
```

This one comes from the `staging.py`.

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

Things already looks better. However, there are also some changes needed on
`manage.py` and `wsgi.py`. This is required to let know both the `./manage.py
runserver` command and the wsgi server where to look for Django's settings
module.

```diff
diff --git a/dblood/wsgi.py b/dblood/wsgi.py
--- dblood/wsgi.py
+++ dblood/wsgi.py
@@ -11,6 +11,6 @@ import os
 
 from django.core.wsgi import get_wsgi_application
 
-os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'dblood.settings')
+os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'dblood.settings.production')
 
 application = get_wsgi_application()
diff --git a/manage.py b/manage.py
--- manage.py
+++ manage.py
@@ -5,7 +5,7 @@ import sys
 
 
 def main():
-    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'dblood.settings')
+    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'dblood.settings.staging')
     try:
         from django.core.management import execute_from_command_line
     except ImportError as exc:
```

These changes although seemed small, they have already saved us quite some time
while we moved our backend app to new enviroment, now behind a troublesome proxy
wall.

## Linter and Formatter

Besides Uncle Bob's *Clean Code* teaching, there are also ways you can use to
help applying some clean code principle easily.
Developer often employs linter and formatter to catch those readability issues
early. So, what are they?

> lint, or a linter, is a tool to analyze source code to flag programming
> errors, bugs, stylistic errors, and suspicious constructs.

Yup, that one is a blatant ripoff from Lint [wikipedia page][lint-wk]. Meanwhile
a formatter, or otherwise called *prettier*, or *beautifier* is used to
automatically format code to some set of rules. I personally think calling
them "formatter" is more correct as they are mainly used to avoid pointless
discussion about style at all. They give you results that are sensible enough to
be easily read while not trying hard to be pretty.

For example, our backend Django app uses [flake8][flake8] linter with django
[plugin][flake8-django] and no formatter. We use flake8 because Django Framework
itself uses it. The additional Django plugin only a few but usefull rules that
understand Django's specific semantic. We ditch formatter because having Lint
Test job on our CI is enough for now to catch the occasional rule violations.

As an example I added an empty model called `FooModel` on `main/models.py`.

```python
class FooModel(models.Model):
    pass
```

Here is the flake8 output.

```bash
(venv) $ flake8                              
./main/models.py:100:1: DJ08 __str__ method should be present in all db models
./main/models.py:100:1: DJ09 Model: FooModel must define class Meta
```

Yup, that's all about *Clean Code* concept and tips I have.

[flake8]: https://github.com/PyCQA/flake8
[flake8-django]: https://github.com/rocioar/flake8-django
[lint-wk]: https://en.wikipedia.org/wiki/Lint_(software)
[quotes]: https://www.goodreads.com/author/quotes/45372.Robert_C_Martin
[summary]: https://gist.github.com/wojteklu/73c6914cc446146b8b533c0988cf8d29
