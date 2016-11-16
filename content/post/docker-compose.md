+++
date = "2016-11-15T17:00:00"
draft = false
tags = ["docker", "docker-compose", "git", "microservices", "oms"]
title = "Setting up a docker-compose environment for the OMS"
math = false
+++

# Introduction

At first I used this file only as a help for myself, but I thought I could also share my thoughts. The project was to set up a docker environment for a microservice architecture consisting of the following parts. This was specifically done for the [OMS](https://oms-project.atlassian.net/wiki/display/GENERAL/Project+description)

1. OMS-Core, consisting of
  * nginx
  * php-fpm
  * A workspace container with php artisan, composer, etc
  * postgres
  * A data container for the whole repo
2. OMS-Events, consisting of
  * node.js backend
  * nginx frontend for static serving and reverse proxy
  * mongodb
  * (Not yet) A data container for media files

# Design decisions
I had to do some design decisions in setting this up, most importantly

1. How to get the code into the container
2. How to get the configuration into the container
3. How to link up the containers
4. How far can/should I split everything
5. Sharing API-Keys

## How to get the code into the container
This seemed to be my first real task. I wanted to have it set up in a way that you ideally just run docker-compose up and everything is set up, AND the code from each of the modules is linked via a volume mapping to the host so you can just edit the files on the host with a text editor and it will end up iin the container right away. However, now is the question on when the code for oms-core and oms-events will be pulled and how it can be moved into the container. There were basically 3 possibilities



1. Create a git repo with the docker-compose.yml (and some other files) and add oms-neo-core and oms-events as submodules, then COPY the files into the containers.
2. Create a git repo with the docker-compose.yml (and some other files) and add oms-neo-core and oms-events as submodules, then mount the files as volumes into the containers.
3. Git clone inside the Dockerfiles, but still mount the volumes to harddisk during run.

At first I read [this](https://forums.docker.com/t/best-practices-for-getting-code-into-a-container-git-clone-vs-copy-vs-data-container/4077) article, which basically has the same 3 options, and where the majority pledged for option 2. The advantages which were described there were better CI integration and more lightweight containers. Especially the last argument is strong, as replicating the container wouldn't really work, as the code would be cloned twice. However, I also had some points for option 3. It would make setting up everything a lot more transparent, as the configuration files (which definetely needed to be changed) could have been added in the docker build process. However, I trusted that forum post and went for number 2 - number 1 was never really an option.

The file-system now looks like this:
```
.
├── docker
├── oms-core
├── oms-events
└── oms-events-frontend
```
Whereas the docker folder holds the docker-compose.yml and some dockerfiles.



## How to get the configuration into the container

As I decided for number 2 in the above question, I was expecting some new problems with this one. The common way to mount configuration files into a docker container is to just use the -v switch (or on docker-compose the volumes: list). However, I already have a volume mount to the whole codebase, including the configuration file and that would mean intersecting mounts. I personnally had no idea if that actually was something bad, so I tried it aaaand it worked! =) It actually left the file in `oms-events/lib/config/configFile.json` intact but to the container the contents of `docker/omsevents/configFile.json` (which I added via volume mounts) were visible. Great!

However, not so easy. The configuration might be in there, but what about bootstrapping the apps? The problem is that the volume mounts are not there during docker build, and there is no way to get them there.

For **oms-events** I would have to run `npm install` at some point in time, as it's based on nodejs. This would have been nice to run in the docker build process, so I checked on how this [should be done](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/) - copying the package.json file in advance enables npm install to run and it doesn't need any app code for that. However currently the package.json file lives outside of the build context, in the git repo with the other code and docker doesn't allow copying that. My fix for this was putting the raw url from github for that specific file into the container:  
```
ADD https://raw.githubusercontent.com/AEGEE/oms-events/dev/package.json /usr/app/package.json
RUN npm install
```

Great, node works. However the **omscore** seemed to have far more heavy dependencies and I personally didn't know so much about how the bootstrapping works and if it could actually do something before having the codebase present. The laradock image also didn't give many hints on this, as they seem to leave this as something the user can do by hand. I thought it might be neat to have a configuration container, which starts, does the configuration work and then exits again. I believe that this also was the concept behind the workspace container, that laradock introduced. However, that container would have to be designed in a way that it can be run every time on a startup, as having optional containers is [not yet](https://github.com/docker/compose/issues/1896) implemented in docker-compose. I decided to use the workspace for just that purpose and tried to set something up which can be run any time without breaking app. However, in the middle of the process I noticed that I need a working db connection, which made it impossible to set this up during the docker build process. Resultingly, I decided to place a small bootstrap script inside the workspace container and tell the user to manually run it when he runs the app the first time. Not beautiful, but for omsevents I need the same, see [Sharing API-Keys](#sharing-api-keys)

## How to link up the containers

"Linking up" actually is a pretty vague term, we need to differentiate between linking in **network** terms and linking in **volume** terms. 

Regarding the **volumes**, there basically only was the design decision to have a data container or not. Some services, like the nginx and the workspace of omscore need the same data, so we need to distribute that somehow. A dirty version which could be thinkable would be to copy all the data into all the containers, but that would go against our requirement of having a volume mount for live code-reloading. The [docker tutorial](https://docs.docker.com/engine/tutorials/dockervolumes/) recommends to insert data containers, meaning images like the `tianon/true` image on dockerhub with a volume mount linked up to them. Another possibility would be to link the folders with a volume switch everywhere and not put up a data container. That would also be thinkable and actually work, just with the problem that the services might spam up our development folders, which we want to keep as clean as possible. In the end I actually went this way with the omsevents module, however planning to change that in the future. For the omscore it was already set up with the laradock configuration, so I kept it.

The **network**-linking can be pretty straightforward, but might still be worth a thought. Docker supports multiple networks between the containers, which would in this case be a really good idea as the omsevents and omscore service are relatively independent and could be linked up with just a proxy. However, docker also supports mashing all of them into the same network. Let's not do this in production, but for development I think this is fine, and I decided I'd rather spend that time making dinner than trying to seperate the networks.


## How far should/can I split everything

If you read any article about microservices, you will find the word "split" or a synonym in every second sentence. Splitting things is good, as it provides isolation so faults or exploits can't propagate too easy, allows easier scaling, better fault tolerance and much more. The major reason why we actually need it in this size is splitting the teams working on different parts of the application. The core can be developed completely independent of the events module, and the events module frontend is almost independent of the backend, and so on. This is just pretty flexible. When I started setting up the docker for oms-events, I found that I would have to create a container with both node and nginx, as that module uses node for the backend and nginx for statically serving the frontend. At the beginning of the development, I had the node backend also serving the frontend files, but I discarted that solution for any production use, as serving frontend files would put unnecessary load on the backend. So at this point, I decided to fully split the two, and have two different containers for frontend and backend. That also meant introducing a new github repo for the frontend, as it felt just wrong having them in the same. Fancy extra: it looks super cool starting up 9 containers and seeing them appear after another :D

## Sharing API-Keys

This is actually a difficult question, as the API-Keys should not get leaked but need to be distributed to the containers. In development this might be of less danger, but we should get this one straight from the beginning on, so we don't mess up in production. The talk [Docker Security](https://www.youtube.com/watch?v=A32Yjizt2_s) by Adrian Mouat addressed that issue, and pointed out 4 possibilities how to solve it.

1. Hardwire the secret into the images
2. Pass it via command-line parameters/environment variables
3. Create a file containing the secret and mount it everywhere needed
4. Use a secret key/value store like [vault](https://www.hashicorp.com/blog/vault.html)

1 and 2 are obviously flawed, also because in our setup the omscore generates the token itself, so I found that 3 could be a good option. I don't yet have experience with vault or other secure key/value stores, but maybe I will try to implement that in the future. The rest then was pretty straight forward. Create a named volume, mount it read-only into the omsevents module (and have the service read from the file) and mount it into the omscore-nginx (and have the service write to it). Well, actually both things in bracket meant code changes, so this took a while, but the idea is straight forward. Remember you don't even need to specify a host folder for this, a named volume should be enough as it is written by omscore during runtime. Like this, only the necessary containers have access to the api-key, so the chance of it getting leaked is pretty low. One could also think of introducing a data-container for persistence.