---
layout: post
title: Why is Batect written in Kotlin?
tags: kotlin golang batect open-source
comments: true
date: 2022-02-07 09:19 +1100
---
After quite a hiatus, I'm going to try to post here more regularly... let's start with two topics close to my heart: Kotlin and Batect. 

A fairly frequent question I get is: why is [Batect](https://batect.dev/) written in Kotlin? And often the question behind the question is:
why isn't it written in Golang?

To answer those questions we need to go back in time to 2017, when I first started working on Batect. 

I had the idea for what would become Batect after working with a couple of different teams who were all trying
to use Dockerised development environments. And after going through the pain of trying to once again reinvent the same less-than-ideal setup
we'd had on a previous team, I decided to build a proof of concept for my idea.

At the time, all I wanted to do was quickly build a proof of concept. I was more concerned about building it quickly and having fun and learning
while I built it than anything else: I wasn't expecting the proof of concept to be anything more than some throwaway code. (Famous last words.) 
I had been dabbling in Kotlin for a while and thought this would be a great opportunity to play with a language I really liked.

Fast forward a bit and we come to late 2017. I'd finished the proof of concept in Kotlin and demoed it to my team for feedback. They gave some really
positive feedback and that encouraged me to seriously consider turning Batect into something more than a proof of concept.

At this point, I had a  codebase that could run a sample project, a working interface to Docker that invoked the `docker` command to pull images, create
containers etc., and some basic tests. I had two choices: continue with this largely working app, or throw it away and start afresh. 

As part of considering whether or not to start afresh, I thought for a while about whether to continue in Kotlin or switch to Golang. 
There were four things that were going through my mind:

**I had something that worked** and a codebase that was in relatively good shape. 

Switching to Golang would require starting from scratch. Given I was
doing this entirely on my own time, that didn't seem like a great use of time, especially for something that still didn't have any active users.

The main argument in favour of Golang was the fact that **the vast majority of the Docker ecosystem is written in Golang**.

If Batect was written in Golang, I could take advantage of things such as the Docker client library for Golang, rather than write my own for Kotlin.
I was using the `docker` CLI to communicate with the Docker daemon from Kotlin and changing to use [the HTTP API](https://docs.docker.com/engine/api/v1.41/)
didn't seem _that_ difficult if needed later.

Another argument in favour of Golang was **the ability to build a single self-contained binary** for distribution, rather than requiring users to install
a JVM. 

However, [earlier that year](https://blog.jetbrains.com/kotlin/2017/04/kotlinnative-tech-preview-kotlin-without-a-vm/), JetBrains had
announced the first preview of Kotlin/Native, which would allow the same thing for Kotlin code. Younger, naïve-r me assumed that would be good enough
and easy enough to adopt in the future should the use of a JVM really turn out to be a problem.

The last thing was simply **personal preference** and how much I enjoyed working with the language.

Again, this was something I was doing on my own time and had no active users yet, so personal enjoyment was one of the main priorities for me. 
At the time, I was working on a production Golang system and was finding the language somewhat lacking in comparison to Kotlin. Dependency management in
Golang was a pain (this would be fixed with the introduction of [modules in Go 1.11](https://go.dev/doc/go1.11#modules) in August the next year), and I found 
Kotlin's syntax and type system enabled me to write expressive, safe code.

So I chose to continue in Kotlin.

Looking back on that decision, there are definitely some things that remain true today, and others where the situation turned out to be
a bit different:

**Not rewriting Batect from scratch** meant I was able to continue adding new features and incorporating feedback. This meant that Batect was ready to
introduce to a new team in late 2017. This team chose not only to take a risk and adopt a completely unproven tool, but continue to this day to be some 
of its strongest advocates. If I'd stopped to rewrite Batect in Golang, I would have missed that opportunity.

**Using the `docker` CLI worked reasonably well for quite some time**, but eventually the performance hit of spawning new processes to interact with the Docker
daemon was starting to have a noticeable impact.

Switching to use the API directly was, sadly, not as straightforward as I hoped. In particular, I had failed to consider the client-side complexities of some 
of Docker's features, such as connection configuration management, image registry credentials and managing the terminal while streaming I/O to and from the daemon. 

This has played out over and over again, and has been a significant drain on my time, especially when it came to adding support for BuildKit, which is largely 
undocumented, requires extensive client-side logic to implement correctly and relies on 
[a number of Golang idiosyncrasies](https://github.com/batect/batect/commit/98262d74c3e26b36b9d89eebb2838c48365e68d5).

**Kotlin/Native sadly hasn't matured as quickly as I expected.** So Batect still requires a JVM, and this adds a small barrier to entry for some people. Having
said that, JetBrains is still actively developing Kotlin/Native and significant progress has been made in the last 12 months or so, so I remain hopeful that
removing the need for a JVM is still an achievable goal. This will not be painless -- Batect has dependencies on some JVM-only libraries at the moment --
but it certainly seems within reach.

The last point will always be a matter of personal opinion, but **Kotlin remains my favourite language to this day.**

Every now and then I question my choice and whether it was the right decision to continue building Batect in Kotlin. While some things may well have been easier had
I chosen to use Golang, Batect has hundreds of active users who love using it, and I still enjoy working on it after all this time, and those are the two things that 
matter most to me.

_Thank you to Andy Marks, Inny So and Jo Piechota for providing feedback on a draft version of this post._
