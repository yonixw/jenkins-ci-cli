# jenkins-ci-cli

## Intro

I wanted to have a cross platform CI tool to build, test and even deploy stuff.
Currently the big players all have some solution but they need a server or saas account to run the scripts (pipelines).
For example, I can use Github Action or Gitlab CICD. Each of them have a yaml you need to configure, and on the next push they will run them. (They give some free build time each month).

## The issues

As I become more and more familiar with the top leading tools I realized that they are not cross platformed compatible. You would think that since you just run code, you can create a few
bash scripts, and just configure to run them. And to local test would be to just run them locally. That's it. But I soon relized the truth to be far from it.

There are a lot of assumptions writing a CI pipeline in a specifiec system:
* Where do you get ENV? Secrets?
* How to share ENV between steps?
* How to share files between steps?
* What context do you run under? OSes (ubuntu, windows, etc)? Dockers?

There are also a lot of "native" features that you can put in your bash scripts. And are tied to specifiec implementation. For example:
* How to specify you want 2 stages to run in parallel?
* How to manually wait for user without spending build times?
* How to publish HTML reports? (Such as test coverage)
* Plugins and Addons

With all of the above, once you invest time in a specifiec platfrom CI system, you are tied to it pretty heavily.
But I don't want that! and that is the main issues.

Just over the past year (2022-2023) we had many Github Actions outages, And the current system I use the most which is Azure Devops is billed
seperetly from my Gitlab self-host on AWS. Not to mention we chose most of them mainly because of their generous free tier, but as my company grow, I might need other features or
will have other price constrains. All of those might cause a trigger to move between CI pipelines.

And after thinking about all of the above. I decided I must find some cross platform way to plan my CI pipeline that will not stop me in the future.

## The possibilities
