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

There are also a lot of "native" features that you cant put in your bash scripts. And are tied to specifiec implementation. For example:
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

Between my two finalists, I dabbled in gitlab but then realy wanted to try Jenkins as a solution. As, one, it is really open sourced. and, two, I saw it being used mutliple times in very repectables companies. 
Which gave me an apataite to make it work. And for the final reason, I understand that a gitlab CI file cannot add steps dinamiclly (like per item in an input list) which is a downgrade compared to Azure Devops and Jenkins.

## Trying out Jenkins

Testing period was May 5, 2023 until May 22, 2023 ([under this project](https://github.com/yonixw/LivestreamDockerRecorder)). Unfortunately, I abonded it. And here I will explain why.

It started very good. There is an [offical docker image](https://github.com/jenkinsci/jenkinsfile-runner) and a project allowing you to run a "Jenkinsfile". And it support they entire "Decleretive Pipeline" so you can pass variables
between stages just like in a real jenkins server etc.

Learning Jenkins was surely not straight-forwared but I guessed it all was worth it once I migrated to it. I even made a custom bash script to store my learning phase as a common functions inside it (pull, run, lint etc.).
And as I digged deeper I found a lot of unducumented stuff, But again, thought it was worth the "long way". Here are more examples of unique issues I came across:

* You had to build a custom docker to have your plugins inside it (not just mount it to the docker)
  * i.e. plugins are "instance" level, and not in a pipeline level (so same pipeline might produce diff result)
* You had to mount your workspace in specifiec scheme to applease some plugins
* The built in bash did not know how to just add "lint" to the entrypoint, and you had to redifine it
* You had to know Java conventions of env vars to enable "verbose" mode for plugins. (Since Jenkins is written in Java)
* I needed to learn how to print the version of each component as it is not a built in feature

You might think that this list is absured to handle on top of the official docker as it might be out-dated in their next realese, but for me it looked easy to do with a (200 LOC) custom  script and a little of bash stuff even if it sure made me worry about the solution I chose.
But then came a problem that was just not possible to solve.

You see, Jenkins is a software from 2011, which is the very very start of the CICD movement (Before docker in 2013 and Github actions in 2018). Which means its set of assumptions is also very old. It start with the fact
that docker delegation is not even a built in feature, but a plugin. And it ends, unfortunately with the bad assumption of how it produces, integrate and collects logs.

As I understand it, each agent/worker send the console log of its process back to the master node. This step alone is not consistent, as logs of "starting a parallel job" that comes from jenkins are not stored in the same
place with the process log (console text). And then you have 2 options:

1] Get the dump of all the logs that the Jenkins master collected, i.e. "Console Text". which is good for simple linear cases but become unreadable fast in other situation ([see here how it looks after the user help](https://stackoverflow.com/a/58050883/1997873)) mainly because all context is lost, and even a simple row cannot restore its stage name!

2] Get a structred log, which is more "Jenkinsfile language nodes" oriented, i.e you can see the console text of an action. But stage in a stage plannig? No logs, as the system logs and context of the parse process can only seen in the log dump (i.e the full console text from option 1]) from my experience. [Here is examples how jenkins log structure works i.e. action nodes](https://i.imgur.com/5A1W998.jpg)

You can see those problems of log inconssitancy here: https://i.imgur.com/bsUmt8x.png 

And this time, I had reached a wall. Why? Because in every other CICD both the system logs and logs of steps are collected at the stage level out of the box. Here I will either give this feature up or will have to write a low level plugin to handle it. (if even possible, see the inconssitencies in existing plugins in my image example above).

But why not just take the console text that IS seperated? Again, because to me, it's too much to give up. As inner parsing of a CI file can cause problem too. And if I don't see any way to debug it, that it is just a ticking bomb until I will find this bug in the future. Which I really don't want.

Why so many big companies use it then? Great question! Maybe because the worker pool is already there. And maybe their build flow is more straight-forwared. But if I find problems and inconsistencies even in my evaluation phase... then it is just too much for me and I will search for alternatives.

## Moving to gitlab runner

Gitlab runner docker is also an officialy supported solution to run gitlab CI file. There are some caveats. You can only run 1 stage at a time (since the runner will do the same under a gitlab instance) but a simpe linear foreach in a bash script can fix this. which will give us a very similar feeling to the real solution.

What is mostly missing:
* There are no loops and dynamic stages, which is the real downgrade in my opnion. But seeing it as the only option I don't have much to do about it.
* You can't build a docker in one step and use it in another unless you push it to a registry. (For local running, I think tagging it will be enough)

On the plus side, whenever I go, It will easy to split the stages in such a way the platform except.

There is also no problem with logs (so far) and plugins are just dockers (so we can also use github action plugins).

There is also no need to a huge custom bash script, just a loop on stages and mounting to the correct place, and we are good to go on any platform!

## Conclusion

Even tough I still wonder If "I can fix her" i.e. fix Jenkins to give me the logs,  I think that 1 month of work just to find you need to touch core features next is a no go and a solution, even though less sophisiticated in its structure will
be enough for me. As I mainly saw my plugin system moving around dockers and not a native one like jenkins have (or Azure Devops which is propitery)


