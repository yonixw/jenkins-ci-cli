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

Since there are a lot of options for a CI solution out there ([see this](https://github.com/ligurio/awesome-ci)), I had to find a few iron-cloud creteria which will help me to focus on a few.
And I have decided on these:

* Ability to run tasks locally - crutial to being cross platform
* Top (giant) player - Github, Gitlab, etc. Ignoring even a 4k stared repo. I just can't let myself find out in a few years my choise was abondant. 
  * Sub requirment - Good documentation
* Open source when possible
* Docker based when possible

With the above, I ended up with 2 options: Gitlab and Jenkins. Here is why:

* Github and Azure Devops have no self hosted version for free, so they are out of the race.
* Drone, GoCD (and many others such as TeamCity) has no way to run yaml locally with 1 line of shell/docker. You have to have a server up with a lot of stuff configures. Some options are also only available through the UI.

Please note that Github does have some tools to run a workflow locally, but they are not official and bring me back to my requirement of "Top player". 
Gitlab is not fully open-sourced but it is open sourced enough for me to get the fully featured CI runner as a docker.

Between my two finalists, I dabbled in gitlab but then realy wanted to try Jenkins as a solution. As, one, it is realy open sourced. and, two, I saw it being used mutliple times in very repectables companies. 

