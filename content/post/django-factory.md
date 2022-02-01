---
categories:
- ppl
date: "2020-05-12T07:36:35+07:00"
description: ""
draft: false
images:
- https://upload.wikimedia.org/wikipedia/commons/thumb/d/df/ASR-33_at_CHM.agr.jpg/800px-ASR-33_at_CHM.agr.jpg
tags:
- ""
title: Factory Boy
toc: false
---

{{<figure
  src="https://upload.wikimedia.org/wikipedia/commons/thumb/c/cf/The_Story_of_Mankind_-_The_Factory.png/800px-The_Story_of_Mankind_-_The_Factory.png"
  alt="Drawing of The Factory from The Story of Mankind"
  title="The Factory"
  attr="Hendrik Willem Van Loon"
  attrlink="https://commons.wikimedia.org/wiki/File:The_Story_of_Mankind_-_The_Factory.png"
>}}

Good Morning! Ever wonder how much ez life you can have if you don't have to
instantiate and type tiresomely each parameters one by one of that one Django
Model with 17++ `null=False` fields for your cute little unit tests? Sorry,
that's a strange thing to ask. Regardless, I have found the solution to those
miserable musing of mine. I'm pleased to introduce you to
[`factory_boy`][factory-boy].

<!--more-->

## Who is This Good Boy?

It's a tool to replace hard to maintain fixtures with easier to use _Factory_
for creating complex objects. It's ripped off description from the readme, but
I still can't come up with a better one.

As a bonus, factory_boy supports using [faker][faker] library to generate
realistic random values such as names, email addresses, phone number and many
more. You will also get a factory that's well integrated to popular Python's
ORM so it can support related objects and vice versa. These reasons are enough
for me to choose factory_boy from other alternatives such as
[django-seed][django-seed].

Even better, the faker library also supports `id_ID` locale. That means we can
have the dummy user profile with name such as "Bambang", and phone number
prefixed with "+62". Here is an example of the generated objects.

```python
Python 3.8.3 (default, May 17 2020, 14:48:56) [GCC] on linux
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> from main.factories import UserFactory
>>> a = UserFactory()
>>> a.__dict__
{'_state': <django.db.models.base.ModelState object at 0x7f813916dc10>, 'id': None, 'password': '', 'last_login': None, 'is_superuser': False, 'first_name': 'Viktor', 'last_name': 'Fujiati', 'is_staff': False, 'is_active': True, 'date_joined': datetime.datetime(2020, 6, 13, 0, 9, 14, 600439, tzinfo=<UTC>), 'email': 'sihombinggangsa@cv.mil.id', 'is_verified': False}
>>> a.profile.__dict__
{'_state': <django.db.models.base.ModelState object at 0x7f8139125700>, 'id': None, 'user_id': None, 'body_weight': 102.335683582417, 'id_card_no': '#################', 'birthplace': 'Bogor', 'birthdate': date(1910, 3, 1), 'sex': 'F', 'profession': 'Therapist, horticultural', 'blood_type': 'A-', 'married_status': 'CM', 'address': 'Jalan Ciumbuleuit No. 609\nKota Administrasi Jakarta Pusat, SG 82767', 'city': 'Probolinggo', 'district': 'Palu', 'village': 'Padangpanjang', 'phone_no': '0829827678', 'work_address': 'Jalan Laswi No. 128\nBau-Bau, PB 74579', 'work_email': 'caturprabowo@ud.or.id', 'work_phone_no': '+62 (288) 755-3935'}
```

## factory_boy for Unit Tests

This is one such cases where factory_boy solves my initial problem. Here I don't
really care about the value of other fields beside "sex" and "blood_type" fields
of the related "Profile" model.

```python
    def test_profile_str(self):
        user = UserFactory(
            profile__sex=Sex.FEMALE,
            profile__blood_type='O+')

        self.assertEqual(str(user.profile), '(F, O+)')
```

## factory_boy for Data Seeding

Now we can also combine factory_boy with management command to create easy to
use data seeding tool.

This is the content of `stok_darah/factories.py`.

```python
import factory
import factory.fuzzy

from .models import Darah


class DarahFactory(factory.DjangoModelFactory):
    class Meta:
        model = Darah

    tipe = factory.Iterator(Darah.GOLDAR, getter=lambda c: c[0])
    jumlah_stok = factory.fuzzy.FuzzyInteger(low=33, high=666)
```

While this is the content of
`stok_darah/management/commands/stok_darah_seeder.py`.

```python
from django.core.management.base import BaseCommand
from stok_darah.factories import DarahFactory


class Command(BaseCommand):
    help = 'Seeds the database.'

    def handle(self, *args, **options):
        for _ in range(8):
            DarahFactory.create()

        self.stdout.write(self.style.SUCCESS('Successfully seeds the database.'))
```

This way the seeding process becomes super easy.

```bash
(env) $ ./manage.py stok_darah_seeder
Successfully seeds the database.
```

## Notes on Handling Migration

So, not too long ago we had an unexpected error on our staging environment that
wasn't cought on our tests. It was almost the time for sprint review, so the
merge process is a bit chaotic. The unexpected error mentions about missing
column from this table blah blah blah. We made sure to `migrate` our database to
the latest version of our application but the problem persisted. It turned out
that some migration files have been replaced with the newer one making the live
database and the model out of sync.

It's a bit tricky because the new model doesn't have the same primary key as the
previous version, so there is no alter table or `migrations.RunPython` method we
can employ to save the existing data.

Thankfully we are able to reset the table and have them in sync again with our
migration files. We used the `./manage.py migrate [app_label] zero` command to
reset the table, migrate them over again and use the data seeding tool to
repopulate the missing data. Its very fortunate this doesn't happen on our
production database as losing customers data is bound to be catastrophic.

[django-seed]: https://github.com/Brobin/django-seed
[factory-boy]: https://github.com/FactoryBoy/factory_boy
[faker]: https://github.com/joke2k/faker
