---
layout: post
title: How to Enable Docker Remote API, Usage and Risks 
date: 2020-9-14
summary: |
 In various situations you may in a need to control Docker outside of your host environment. Let's say you're deploying to a server of yours and to remote deploy you need a connection to your Docker Daemon.
tags: docker devops
excerpt_separator: <!--more-->
categories:
  - Docker
  - DevOps
---

In various situations you may in a need to control Docker outside of your host environment. Let's say you're deploying to a server of yours and to remote deploy you need a connection to your Docker Daemon. <!--more--> In that scenario Docker REST API connection comes to your rescue but with a little security cost. We'll talk about that later in this post.
But first let's enable it. Assuming that you already have a server that has Docker installed on it.   

## Enabling Docker Remote API

Go to `/lib/systemd/system` and edit the file `docker.service` with your favourite terminal editor. Doesn't matter whether you're a *super user* or not. Find the line below and edit it like this:

```terminal
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2376 --containerd=/run/containerd/containerd.sock
```

This way Docker engine server will be binded to Unix socket and TCP Port 2376. You can change the port however you want but be sure that it's not being used by any other service on your machine.   

After editing fire up these commands:

```terminal
sudo systemctl daemon-reload
sudo service docker restart
```

Now your remote API is enabled. Let's test it.

## Testing Docker Remote API Connection

Step outside from your server to anywhere else and now, you should be able to run these commands.

### With Docker

```
docker -H YOUR_SERVER_IP:2376 version
```

### With cURL

```
curl http://YOUR_SERVER_IP:2376/version
```

### With a REST Client

```
http://YOUR_SERVER_IP:2376/version
```

## Benefits and Advanced Usage

As dangerous as this can be (ops, spoilers for next chapter), this connection also offers some undeniable benefits. For example, from your CI agent you could say:

```terminal
docker -H YOUR_SERVER_IP:2376 pull YOUR_IMAGE
docker -H YOUR_SERVER_IP:2376 run
        --name 
        -it
        -e SAMPLE_ENV=SAMPLE_ENV_VARIABLE
        -p 80:80
        -d $YOUR_IMAGE
```

Honestly, this opens up so many options and enhances your reach on environments.  

## Risks and Side Effects

Take a moment the notice that the definition of the IP address we gave in the *docker.service* file. `0.0.0.0` means that **any person** who has the IP of your server and port of the Docker can connect to your Docker engine without hesitation. Opened ports are easily findable via various easy techniques thus your environment can be instantly vulnerable to attack. Take a look at this [article](https://www.riccardoancarani.it/attacking-docker-exposed-api/) about attacking exposing Docker API's to have a grasp on the situation.  

There are various solutions such as [HTTP Basic Authentication with Nginx](https://ahmet.im/blog/docker-http-basic-auth/), adding a [TLS client certificate verification](https://gist.github.com/kekru/974e40bb1cd4b947a53cca5ba4b0bbe5) to Docker API. Personally I came up with another solution. I'll edit this part when I wrote the article about it.

## Resources

* [https://success.docker.com/article/how-do-i-enable-the-remote-api-for-dockerd](https://success.docker.com/article/how-do-i-enable-the-remote-api-for-dockerd)
* [https://docs.docker.com/develop/sdk/examples/](https://docs.docker.com/develop/sdk/examples/)
* [https://medium.com/@sudarakayasindu/enabling-and-accessing-docker-engine-api-on-a-remote-docker-host-on-ubuntu-16-04-2c15f55f5d39](https://medium.com/@sudarakayasindu/enabling-and-accessing-docker-engine-api-on-a-remote-docker-host-on-ubuntu-16-04-2c15f55f5d39)
* [https://scriptcrunch.com/enable-docker-remote-api/](https://scriptcrunch.com/enable-docker-remote-api/)