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

> NOTE: This article assume that the reader already has some familiarity with git
> version control system


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
| commit    | 168 |
| log       | 130 |
| switch    | 126 |
| checkout  | 119 |
| clone     |  72 |
| rebase    |  45 |
| pull      |  40 |
| reset     |  37 |
| fetch     |  30 |
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

Notice the decorations added in some commit hashes. It will show you not only
your current local branch but all branches tracked by git. This command is very
usefull for me to keep tracks of my local changes against whatever is in the
`origin/master` branch or what my team member has pushed to their own branches.

Now you may ask, "Why don't we use the GUI version?". There aren't any real
difference with the GUI version. I use git cli more than any other git client.
Even despite BitBucket and GitLab very beautiful graph view mode. The only
exception is when VS Code's interactive `--patch` operation comes in really
handy that I will avoid using the cli version.

Here's the honorable mentions. Another variants I want to share with you is the
little `git log -<n>` version. Not the `git log -n <n>` variants. It will saves
you a few keystrokes without relying on aliases. Another one I often check is
the `git log --show-signature` command. In simple explanation, this command will
show you the verified gpg sign of your commit. This one became a habit since
all CNCF organizations have CLA (Contributors License Aggreement) that mandates
every commit to be gpg signed.

## Git Fundamentals: Anantomy of a commit

Okay, so lets get to the fundamental. This topic is different from before that
I rarely care about this nitty-gritty details. However, by understanding this
you will gain mechanical sympathy. Mechanical Sympathy? What new viral virus is
that?

> To be continued
