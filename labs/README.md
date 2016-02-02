# READ ME FIRST

## Prerequisites

This lab is heavily built on Docker Machine. I tested this on OSX, but any system that supports Docker Machine should also work.

I recommend installing the [Docker Toolbox](https://www.docker.com/products/docker-toolbox) for working with these labs, especially if this is your first experience with Docker.

## Useful Commands

Remove containers by image:

```
docker rm $(docker ps -a | grep "busybox" | awk '{print $1}')
```

Set shell context to Swarm instead of a particular host:

```
eval $(docker-machine env --swarm swarm-master)
```

Set context to a particular docker machine (useful for querying a host directly instead of through the swarm)

```
eval "$(docker-machine env dev)"
```