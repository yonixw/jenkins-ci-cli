# jenkins-ci-cli

## Intro

I wanted to have a cross platform CI tool to build, test and even deploy stuff.
Currently the big players all have some solution but they need a server or saas account to run the scripts (pipelines).
For example, I can use Github Action or Gitlab CICD. Each of them have a yaml you need to configure, and on the next push they will run them. (They give some free build time each month).

## My problems

As I become more and more familiar with the top leading tools I realized that they are not cross platformed compatible. You would think that since you just run code, you can create a few
scripts, and just configure to run them. And a local test would be to run them locally. That's it. But I soon relized the truth to be far from it.
