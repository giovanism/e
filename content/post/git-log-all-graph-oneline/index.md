+++
title = "git log --all --graph --oneline"
date = 2020-03-09T18:01:17+07:00
description = "Giovan growing his little trees"
draft = false
toc = false
categories = ["ppl"]
tags = ["computer", "git"]
images = [
  "https://upload.wikimedia.org/wikipedia/commons/thumb/d/df/ASR-33_at_CHM.agr.jpg/800px-ASR-33_at_CHM.agr.jpg"
] # overrides the site-wide open graph image

[[resources]]
  name = "little-wire-tree"
  src = "images/Wire-Tree-Sculpture-Metal-Bonsai-8.jpg"
  title = "Metal Wire Bonsai"
  [resources.params.meta]
    creator = "Metal Bonsai | https://metalbonsai.com/"
    sameAs = "https://mymodernmet.com/wp/wp-content/uploads/2017/03/Wire-Tree-Sculpture-Metal-Bonsai-8.jpg"

[[resources]]
  name = "git-log-output"
  src = "images/git-log-output.png"
  title = "git --log --graph --oneline"

[[resources]]
  name = "winnie-the-pooh"
  src = "images/winnie-the-pooh.jpg"
  title = "Winnie The Pooh Approve's `git switch`"

[[resources]]
  name = "gigas-chedar"
  src = "images/gigas-chedar.webp"
  title = "[REVIEW] Pembahasan Sword Art Online Alicization Episode 4: Tumbangnya Pohon Gigas Cedar"

[[resources]]
  name = "trunk1b"
  src = "images/trunk1b.png"
  title = "Scaled Trunk-Based Development"

+++

{{<resfigure
  alt="Little Tree"
  src="little-wire-tree"
  title="Metal Wire Bonsai"
  attr="Metal Bonsai"
  attrlink="https://metalbonsai.com/"
>}}

You may be wondering, "Hey? Why do you give this post a git command as the
title?". The reasons's are clicks, views, and sweet-sweet engagement metrics! I
hope by using weird idiosyncratic nerdy title will get people to read this post.
So, what do I really want to talk about here? You're already 1 minute in reading
the post but still has no clue what the heck this is all about. This series of 
article (yup, there will be more) is written for my Software Engineering course
and this one will primarily explain git.

<!--more-->

Even if the main driver for me to write this article is this course's credit and
the ever aproaching deadline, I will try my best to have something more,
something of value that you will only find here and nowhere else (or atleast I'm
aware of as I'm writing this article). Oh, and before you are annoyed by this
unstructured dialogue and narative writing style, I can only hope that you would
bear with me on this wild ride. Deciding which part to tell first is as hard as
deciding what I want to write about. So I dont really plan what I'm going to
write.

> NOTE: This article assume that the reader already has some familiarity with
> git version control system.
> You can go read [A Tale of Three Trees](three-little-trees) or my friend's
> awesome [Git Handbook](git-handbook) in Indonesian first.

## git log

Back to the title. Why this particular command? What does it do? I put this
command because this is my favorite git command, but not the most frequent by
any means. I run `git status` number of times way more than `git log`. I check
the number of times I run `git status` command by running `grep "git status"
~/.zsh_history | wc -l`. So you can imagine what I'm talking about, you can see
in the nice rendered table below to see how my git commands usage statistics
compare.

| command   | n   |
| --------- | --- |
| status    | 582 |
| add       | 266 |
| commit    | 168 |
| log       | 130 |
| switch    | 126 |
| checkout  | 119 |
| clone     |  72 |
| rebase    |  45 |
| pull      |  40 |
| reset     |  37 |
| fetch     |  30 |
| remote    |  26 |
| merge     |  25 |
| init      |   2 |

Interesting to say the least. So it takes a paragraph and a table to explain why
I chose it over any other command and even the new one that just came out, the
`git switch` command. But wait, I haven't even shown you what this variant with
it's three flag does.

```bash
$ git log --all --graph --oneline
```

The `--all` flag tell git to show commits from local and every remote branches.
The `--graph` flag tell git to show commits as graph, connecting commits with
their parrents. And lastly, the `--oneline` flag will tell git to show commit
logs in oneline pretty format. The result of running this command against this
site's repo is this.

{{<resfigure
  alt="`git log`'s Output"
  src="git-log-output"
  title="git log --all --graph --oneline"
>}}

Notice the decorations added after some commit hashes. It will show you not only
your current local branch but all branches tracked by git. This command is very
usefull for me to keep tracks of my local changes against whatever it is in the
`origin/master` branch or what my team member has pushed to their own branches.

Now you may ask, "Why don't we use the GUI version?". There aren't many real
difference with the GUI version. I use git cli more than any other git client.
Even despite BitBucket and GitLab very beautiful graph view mode. The only
exception is when VS Code's interactive `--patch` operation comes in really
handy that I will avoid using the cli version.

{{<resfigure
  alt="Winnie The Pooh meme formatted git switch vs git checkout"
  src="winnie-the-pooh"
  title="Classy Winnie The Pooh Approving `git switch` command"
>}}

Here's some of the the honorable mentions. Another variants I want to share with
you is the little `git log -<n>` version. Not the `git log -n <n>` variants. It
will saves you a few keystrokes without relying on aliases. Another one I often
check is the `git log --show-signature` command. In simple explanation, this
command will show you the verified gpg sign of your commit. This one became a 
habit since all CNCF organizations have CLA (Contributor License Aggreement)
that mandates every commit to be gpg signed.

## Trunk Based Development

Have I told you how much I love Trunk Based Development compared to the usual
Git Flow branching model?

{{<resfigure
  alt="[REVIEW] Pembahasan Sword Art Online Alicization Episode 4: Tumbangnya Pohon Gigas Cedar"
  src="gigas-chedar"
  title="Fallen Gigas Chedar Tree"
  attr="Dunia Games"
  attrlink="https://redeem.duniagames.co.id/news/9862-review-pembahasan-sword-art-online-alicization-episode-4-tumbangnya-pohon-gigas-cedar/"
>}}

Impressive trunk isn't it? The boring explanation is this. Trunk Based
Development is a source-control branching model, where the main development,
commits, and pull requests occurs at a single branch called the "trunk". This
development model promises avoiding merge hell, broken build, and nicer output
when using the `git log --all --graph --oneline` (Yup, I do care about this
aspect).

This way feature branches are short-lived and become subject to code reviews
and build checking using CI, but not the artifact creation or publication such
as pushing to package repository or container registry. One of the arguments is
faster feedback loop through code review and faster integration back to the
trunk with the use of Feature Flags (Feature Toggle). This is useful to develop
large work in progress feature together alongside the other feature in the trunk
branch.

{{<resfigure
  alt="Scaled Trunk-Based Development Graph"
  src="trunk1b"
  title="Scaled Trunk Based Development"
  attr="trunkbaseddevelopment.com"
  attrlink="https://trunkbaseddevelopment.com/"
>}}

Some rule of thumbs of when to integrate Pull/Merge Request to the trunk is:
1. It is possible to squash+merge it back to the trunk
2. It is already reviewed
3. It passed automated tests against the trunk

To avoid breaking changes and conflict, developer will need to learn best ways
to sync (integrate) changes from the trunk to feature branch. Learn what rebase
is, when to rebase, and how to do it when appropriate. This does demand even
more from the developer's skillset besides implementing feature, creating
rigorous testing, that is writing maintainable and backward compatible code.

## What we actually do

Unfortunately, we dont implement Trunk Based Development in our software
engineering course. In my software engineering course we try to implement SCRUM
agile software development methodology. There are some requirements for the
scoring system and the lecturer team mandates every group to implement a variant
of Git Flow branching model. Our project are split into PBIs (Project Backlog
Item). PBI is a unit of tangible progress/product that can be used by the
user/client. These PBIs are broken down even more into tasks that can tackled
down by one person.

There are a lot of things that don't go with my ideal, there are lots of 'em. I
want to stop at the previous sentence, but people won't get what I meant unless
I written them down here.

Fear of reverting Merge Request if they don't comply with client's standard? Why
bother? This will only apply with breaking changes. Incremental changes doesn't
need to be reverted. I want to merge into staging and share my core for all I
care.

I want to get rid of feature dependency across PBIs. Why? Each sprint can have
multiple PBIs. Previously we work on different PBIs concurrently. We might have
done it wrong, though. How about we work on one PBI before moving to another
PBI? This way we can implement "small" Trunk Based Development using each PBI's
branch as the trunk.

Okay, now you are tired of me ranting here instead of actually giving you
content and entertainment. Sorry, this will part will be end soon. I believe
there are difference between compromises vs half-assed attempt. So, wish me luck
on my compromise attempt.

## Git Fundamentals: Anantomy of a commit

Okay, to next content. Now lets get to the fundamental. This topic is different
from before that I rarely care about this nitty-gritty details on my daily
`git commit` and `git rebase` daily routine. However, by understanding this
concept I hope you will gain the so-called mechanical sympathy. Mechanical
Sympathy? What new viral virus is that?

{{<blockquote
  text="You don't have to be an engineer to be a racing driver, but you do have to have Mechanical Sympathy."
  cite="Jackie Stewart, racing driver "
  citelink="https://wa.aws.amazon.com/wat.concept.mechanical-sympathy.en.html"
/>}}

{{<blockquote
  text="Mechanical sympathy is when you use a tool or system with an understanding of how it operates best. When you understand how a system is designed to be used, you can align with the design to gain optimal performance. For example, if you know that a certain type of memory is more efficient when addresses are multiples of a factor, you can optimize your performance by using data structure alignment."
  citelink="https://wa.aws.amazon.com/wat.concept.mechanical-sympathy.en.html"
/>}}


[three-little-trees]: https://speakerdeck.com/schacon/a-tale-of-three-trees
[git-handbook]: https://medium.com/@reyhanhamidi/buku-saku-git-cheatsheet-git-bahasa-indonesia-3af42e42156e
