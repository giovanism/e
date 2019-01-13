+++
title = "Arkavidia 5 CTF"
date = 2019-01-12T21:45:12+07:00
description = ""
draft = false
toc = false
categories = ["write up"]
tags = ["ctf", "computer", "security", "competition"]
images = [
  "images/arkavidia5ctf.svg"
] # overrides the site-wide open graph image

+++

{{< figure 
  src="images/arkavidia5ctf.svg"
  alt="CTF logo"
  title="Arkavidia 5 CTF"
>}}

These are write-ups for some of the challenges that I solved during the Quals.
Also, huge thanks to my teammates [@bungamaku][bungamaku] and 
[@crosscheese][crosscheese] for inviting me to join in their team.

<!--more-->

## Reverse Engineering

### Sanca

We were given a .pyc file, [sanca.pyc][sanca.pyc]. This is a
straightforward RE challenge, the program will output "Correct!" if we 
enter the correct flag. I used uncompyle6 to decompile the .pyc.

With python's handy assignment, We could just turn all the string conditionals
into list assignments.

```python
#!/usr/bin/env python
data = raw_input('Flag:')
data = list('Arkav5{                    }')  # The initial list with len 28
data[-2] = 'n'
data[10] = '3'
data[::-2] = list('_otp5ar}3l3333')
data[::-3] = list('_hpvrtls3r')
data[::-5] = list('_yat3v')
data[::-7] = list('_{}s')
data[::4] = list('rr_tk{h')
data[::7] = list('r3Ap')

print ''.join(data)
```

This gave us `r3v3r3s3_l33t}Arkav5{python_`, which was a pretty good
approximation to our desired flag.

Flag: `Arkav5{python_r3v3rs3_l33t}`

## Forensics

### YaQueen

We were given a .jpg file, [YaQueen.jpg][YaQueen.jpg]. This file looked
like a normal jpg, but after a little inspection using binwalk revealed that
the image file also had a zip file. The zip file contained 625 other jpgs, each
with 11x11 size and either white or black color.

We could quickly deduce that these images might have been a QR code or some
sort. So I wrote a little python script that will print empty space or ascii
block character in my terminal based on the color numbered images.

```python3
#!/usr/bin/env python3
from PIL import Image

data = []
for i in range(1, 626):
    im = Image.open("data/um_%d.jpg" % i)
    fr = im.load()

    temp =fr[1,1][0]
    data.append(temp // 255)

    # do something to im

print(u'██' * 27)
for i in range(25):
    print(u'██', end='')
    for j in range(25):
        print(u'██' if data[i * 25 + j] else '  ', end='')
    else:
        print(u'██')
print(u'██' * 27)
```

I also needed to place a few padding here and there so that the printed
characters will be recognizable by online QR code reader. It also depends
on your terminal emulator settings how nice the final rendered version will
be.

Output was:

```
██████████████████████████████████████████████████████
██              ████  ████████  ██  ██              ██
██  ██████████  ██    ██    ██      ██  ██████████  ██
██  ██      ██  ████      ████    ████  ██      ██  ██
██  ██      ██  ██    ██  ██        ██  ██      ██  ██
██  ██      ██  ██████  ██  ████    ██  ██      ██  ██
██  ██████████  ██  ██  ██████      ██  ██████████  ██
██              ██  ██  ██  ██  ██  ██              ██
██████████████████████████  ██    ████████████████████
██          ██          ██        ██  ██  ██  ██  ████
████          ██      ████████████        ██    ██████
██  ██████████  ██    ██    ██      ██      ██      ██
██████████  ████          ████    ██    ██████████████
██████  ████      ██  ██        ██      ██      ██  ██
██  ████  ██  ██    ██  ██  ████      ██  ██  ██  ████
██  ██      ██    ██    ██████    ██      ██████    ██
██  ██████  ██████      ██████  ██          ████    ██
██  ████  ██    ████████  ██    ██          ██  ██  ██
██████████████████    ██  ██████    ██████  ██████  ██
██              ██  ██  ██████  ██  ██  ██          ██
██  ██████████  ████  ██  ████  ██  ██████    ████████
██  ██      ██  ██  ████  ██                ██  ██████
██  ██      ██  ██    ██████  ██████    ████████    ██
██  ██      ██  ██    ██████    ████████  ████████  ██
██  ██████████  ██    ██  ██████            ██████  ██
██              ██  ████  ████  ████    ██  ██      ██
██████████████████████████████████████████████████████
```

Flag: `Arkav5{McQueenYaQueeeen___}`

## Web

### Fancafe

We were given the source code of web service running in 
http://18.223.125.143:10011/, [fancafe.zip][fancafe.zip]. The source code 
contained:

```
.
├── app
│   └── web
│       └── main.go
├── database
│   ├── database.go
│   └── mysql.go
├── entity
│   ├── entity.go
│   └── post.go
├── fancafe.go
├── handler
│   ├── handler.go
│   └── home.go
├── service
│   ├── post.go
│   └── service.go
└── view
    └── home.html

7 directories, 11 files
```

The most interesting part of the source code is the Search function.

```go
func (p *PostService) Search(keyword string) ([]entity.Post, error) {
    // We only support one keyword at the moment
    keyword = strings.Fields(keyword)[0]
    query := "SELECT * FROM posts WHERE is_deleted = false AND content LIKE '%" + keyword + "%'"
    log.Println(query)
    posts := []entity.Post{}
    err := database.MySQL.Select(&posts, query)
    if err != nil {
        return nil, err
    }
    return posts, nil
}
```

We could see that the only filter/escaping here was to only allow a single
keyword from the search field. To execute our injection we need to compose our
payload without using whitespace character, therefore we can use inline comment
instead such as `/**/` to bypass the `strings.Fields()` function.

payload:

```sql
%'/**/OR/**/is_deleted/**/=/**/true;#
```

Flag: `Arkav5{SQLi_adalah_jalan_ninjaku}`

## Misc

### geet

We were given a zip file, [geet.zip][geet.zip]. As the name suggest the zip
file contained a git repository containing a file called `flag`. But, that
will be too easy, right? Another thing to note was that the commit history even
numbered 731 commits. We can use simple bash command to search through all
commit history;

```bash
$ for i in {1..731}; git diff HEAD~$i >> out
$ cat out | grep Arkav5{
```

Flag: `Arkav5{git_s4ve_y0uR_h1st0ri3s}`


[bungamaku]: https://github.com/bungamaku
[crosscheese]: https://github.com/crosscheese

[sanca.pyc]: files/sanca.pyc
[YaQueen.jpg]: files/Yaqueen.jpg
[fancafe.zip]: files/fancafe.zip
[geet.zip]: files/geet.zip
