+++
title = "Testing and Test Driven Development"
date = 2020-04-14T04:29:39+07:00
description = ""
draft = false
toc = false
categories = ["ppl"]
tags = ["computer", "qa", "testing", "sonarqube"]
images = [
  "https://upload.wikimedia.org/wikipedia/commons/thumb/d/df/ASR-33_at_CHM.agr.jpg/800px-ASR-33_at_CHM.agr.jpg"
] # overrides the site-wide open graph image

[[resources]]
  name = "testing-lifecycle"
  src = "Software_Testing_Life_Cycle.jpg"
  title = "Software Testing Life Cycle"

+++

{{<resfigure
  alt="Sequential Software Testing Life Cycle Created by Software Testing Engineer "
  src="testing-lifecycle"
  title="Software Testing Life Cycle"
  attr="Perfect Happiness"
  attrlink="https://commons.wikimedia.org/wiki/File:Software_Testing_Life_Cycle.jpg"
>}}

Hi! Now, I'm back again to another post on PPL series. This time I will share
my thoughts about testing and Test Driven Development (TDD) on my software
engineering project. This post will only dive a little as to show how we
implemented it. So, don't put so much expectation, mkay?

<!--more-->

## Testing

Testing has always been an important part of software engineering disciples.
Through executing the system under test in specific ways, it tries to uncover
bugs lurking in the corner and verify that the system is running correctly. I've
put a nice image of Software Testing Life Cycle at the top of the post, but it
only serves its purpose as hero image. I won't delve to the topic of testing
lifecycle because its pretty standard and I think everyone had already use it
subconciously. In turn, I will tell you about testing methods that we use in
PPL. They are user acceptance test, unit test, lint test and static analysis.

User acceptance test (UAT) is done as a part of sprint review per Scrum
framework by letting the tester, in this case user or any relevant stakeholder,
interact with the system. In turn, the rest of the tests is automated as part of
Continous Integration (CI) or anytime anyone would like to run them. Running
those tests as part of CI makes sure that every version is up to standard. This
is especially true for my team as we are required to implement TDD.

## Test Driven Development

For the uninitiated, TDD is a software development methodology that requires
developer to turn technical requirements into specific test cases first before
making changes that will fulfill those requirements. This test-first programming
concept is not new however, as similar concept is also made in Extreme
Programming (XP). Even [Kent Beck][kent-beck] himself, one of the most known
figure in XP and TDD, attributed his work to only
[*rediscovering*][kent-beck-quora] TDD as similar concept has been used way way
before in some ancient programming books and subconciously by senior
programmers.

### TDD Cycle

The modern TDD cycle nowadays looks like this:

1. **Add tests**. Turn the requirement into runnable test cases.
2. Run test to see if the **tests fail** (bonus point to put this version into
   version control first).
3. Write the required **functionality**.
4. Run tests to see if the **tests pass**.
5. **Refactor** the code base as needed. One such cases is when making small
   changes becomes too tedious as the units (class/function/method) becomes too
   large and requires too many code/test changes.

Merely repeat these steps to perform TDD.

### TDD's Promises

The advertised benefits for implementing TDD includes encouraging developers to
define how to use their class/function/method and their expected output/side
effect by writing unit test. Some even argue that these "definitions" can also
serve as a kind of document that developer can look up to. TDD obviously also
produces better test coverage that in turn proportional to higher code quality,
to a certain extent.

Another interesting effect TDD does is how it encourages developer to make
changes just enough to pass the test. This tendency to keep changes small,
simple, and limited to only related units is claimed to produces cleaner and
cleaner design than is achieved by other methods.

## Where TDD falls short

Of course oppossing arguments exist too. Most people claiming against TDD will
most likely complain about it being too slow or something along the way. Its
event worse for those who just started or learning. Performing TDD requires the
developer to be familiar with testing tools such as assertion and mocking on top
of the tools needed to implement the actual requirement.

Other problems that may arise when performing TDD is when the requirements can
change often. Thus, Developer needs to rewrite their tests again as long as the
requirement is not final. Though it seems normal, but this can be very
inefficient for design that needs a lot of prototyping. One such case can be
made to compare framework user and library developer. For framework user much of
the coding/testing pattern has been estahblished so there are already lots of
samples online and its fairly easy to write tests. Meanwhile, library developers
more often needs to deal with introducing new level of abstraction, or larger
structure bigger than a unit that needs multiple times of code refactoring to be
implemented TDD way.

## Our Experience

Most of my claims above is made observation on my own projects and other
blogposts online. I feel firsthand how nice it is to perform TDD on my Django
project using the provided testing library. Though, it kinda feel more like
integration test more than unit test as Django's testing library already mocks
client request, database connection and vice versa. On the other hand there is
also a component in my project that I write as a library albeit still in one
repository with the rest of the code. This library is written without TDD most
of the time because it required me some time to do prototyping.

So, what? TDD is a nice dicipline to follow, but it doesn't perfect for every
situation in existance. Rules are meant to be broken anyway. Knowing when to
break them however is essential.

[kent-beck]: https://en.wikipedia.org/wiki/Kent_Beck
[kent-beck-quora]: https://www.quora.com/Why-does-Kent-Beck-refer-to-the-rediscovery-of-test-driven-development
