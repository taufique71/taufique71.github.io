---
layout: post
title: Cloning Private Git Repository at Docker Image Build Time
date: 2017-10-09
modified: 2017-11-30
category: tech
permalink: cloning-private-git-repository-at-docker-build-time
---

Recently I faced a problem while building a Docker image for my work purpose.
A private git repository was being cloned at the Docker image build time.
But problem was that, to clone a private repo authentication was required.
For this purpose, github password of my co-worker was included in the Dockerfile.
He was the person who wrote the Dockerfile initially and couldn't manage much time to design it generic due to time constraint.
I could easily use the Dockerfile to build the Docker image I needed.
But I thought that this is not good practice to use some other person's password while I am the person who is building the image.
So I wanted to come up with a solution to this problem. 
I was a complete newbie to what Docker is and how it works.
After some research I came up with three options.
I am going to go over these three possible solutions.  
<br>
<br>

First, through Docker `'ADD'` command.  
<br>

I use ssh keys for Github authentication. So I have one residing my machine. 
My first idea was to make this ssh key accessible to docker.
`'ADD'` instruction of Dockerfile was there to serve my purpose.  
<br>

Dockerfile was like this -
```
...
# Install necessary packages like Git
ADD ~/.ssh/ssh_key /tmp/
RUN eval $(ssh-agent)
RUN ssh-add /tmp/ssh_key
RUN git clone <repository>
RUN rm /tmp/ssh_key
...
```  
<br>

Problem is, every instruction of Dockerfile generates a layer in the Docker image and every layer of the image leaves some footprint. 
So merely `'rm'` command was not enough to get rid of that footprint of the ssh key.
[`docker-squash`](https://github.com/jwilder/docker-squash) came to the rescue in this case.
It is a tool to squash multiple Docker layers into one. 
In the process it gets rid of the files those were removed at later stages of Docker build.  
<br>

Finally, Docker build process was like following -  
```
# Build the original Docker image first
$ docker build -t <original-image-name> .

# Squash the image
$ docker save <original-image-name> | docker-squash -t <squashed-image-name> | docker load

# Remove the original image
$ docker rmi <original-image-name>
```  
<br>
<br>

Second, using a local HTTP server.  
<br>

Idea was borrowed from another [blog post](https://farazdagi.com/2016/using-ssh-private-keys-securely-when-building-docker-images/). 
Solution was to run a simple HTTP server from the location of the local ssh-key then from Dockerfile using one `'RUN'` instruction to download the ssh key, clone the repository and then remove the key.  
<br>

I used Python's SimpleHTTPServer. You can use any.
```
$ cd ~/.ssh/
$ python -m SimpleHTTPServer
```  
<br>

Dockerfile was like following -
```
...
RUN \
  wget -O /tmp/ssh_key http://<IP_ADDRESS>:<8000>/ssh_key && \
  chmod 600 /tmp/ssh_key && \
  eval $(ssh-agent) && \
  ssh-add /tmp/ssh_key && \
  git clone <repository>  && \
  rm /tmp/id_rsa
...
```  
<br>
<br>

Third, using `'ARG'` instruction of Dockerfile.  
<br>

This solution doesn't use ssh keys. 
It takes Github username and password as argument to the docker build command and uses those arguments for authentication.
[Github access token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/) instead of password can also be used.
In my case I used access token.  
<br>

Dockerfile was like following-
```
...
ARG GITHUB_USERNAME
ARG GITHUB_TOKEN
RUN git clone https://$GITHUB_USERNAME:$GITHUB_TOKEN@github.com/<organization-name>/<repository-name>.git
...
```  
<br>

Docker build command was like following -
```
$ docker build -t <image-name> 
               --build-arg GITHUB_USERNAME=<github-username> 
               --build-arg GITHUB_TOKEN=<github-access-token> .
```  
<br>
<br>

In my case I used the third solution to solve my problem as it seemed like that would be most portable one.  
<br>
<br>

<br>
#### Reference:
1. [Pulling Git into a Docker image without leaving SSH keys behind](http://blog.cloud66.com/pulling-git-into-a-docker-image-without-leaving-ssh-keys-behind/)
2. [Using SSH private keys securely when building Docker images](https://farazdagi.com/2016/using-ssh-private-keys-securely-when-building-docker-images/)
3. [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#arg)
