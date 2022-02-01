---
categories:
- technology
date: "2021-11-15T06:53:05+07:00"
description: ""
draft: false
images:
- https://upload.wikimedia.org/wikipedia/commons/thumb/d/df/ASR-33_at_CHM.agr.jpg/800px-ASR-33_at_CHM.agr.jpg
tags:
- computer
- kubernetes
- gcp
title: Kubernetes the Hard Way
toc: false
---

Hi! Last weekend I finally tried out the famous [Kubernetes The Hard
Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) by [Kelsey
Hightower](https://twitter.com/kelseyhightower). It's a long overdue considering
how long it's been since I started learning Kubernetes, but I finally made it.
Here I want to share some of the gotchas that I found while trying it out for
the first time.

<!--more-->

## Gotchas

### Google Cloud Quotas

Google Cloud Platform enforces quotas of cloud resources that you can use. On
all my previous attempts this has been the biggest blocker because the starting
quotas are 8 CPUs and 8 compute instances per region. There's no way to increase
these quota except contacting sales or capping the quota for some time which
isn't realistic for studying/lab purpose. Fortunately, the company I'm working
on does let me use their dev GCP project to try things out. OMEGA Cool!

### Google Cloud Excessive Public Firewall

Another thing to note is that when creating firewall rule to allow ssh and ICMP
from the internet, GCP actively tries to patch my firewall and I had to allow
them back every now and then.

### Bug with Google Cloud APIs

A minor bug with Kelsey's script, `REGION=$(curl -s -H "Metadata-Flavor: Google"
\
http://metadata.google.internal/computeMetadata/v1/project/attributes/google-compute-default-region)`
retrieved other region instead of us-west1 as set in the beginning of the lab.

### IP Address Clean Up

Another minor bug where Google Cloud turns out already clean up the IP address
with the instances and no longer need another command.

## Lessons Learned

This is definitely a great resource for beginner to learn Kubernetes fundamental
components. I also got to learn how to implement Kubernetes Networking model
using Google Cloud routes. Besides that, I learned the above gotchas and some of
Google Cloud undocumented (or just my ignorance) behavior / characteristic.
