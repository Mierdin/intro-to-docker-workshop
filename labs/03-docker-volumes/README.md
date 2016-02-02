# Docker Volumes

* Discuss Basics of Volumes
* Overview Volumes Documentation (Concepts, and Drivers/Plugins)
* Identify a container that requires data volumes, and work with those volumes locally

## Volumes Documentation

Volumes are a simple concept but why they're needed is a little more complicated (you have to think about stateless vs stateful applications).

[Volumes 101](https://docs.docker.com/engine/userguide/dockervolumes/)

Volumes are plugin-based, meaning the back-end storage for volumes doesn't have to be local disk (default). Could be an NFS mount, iSCSI, etc - the plugins are responsible for integrating this with Docker and ensuring the back-end storage is there.

[Volume Plugins](https://docs.docker.com/engine/extend/plugins_volume/)


## Pull a container that requires data volumes

We haven't had a need for data volumes up to this point, so if we check the local system now, there aren't any volumes set up. Docker 1.9 introduced the "docker volume" command:

```
docker volume ls
```

Let's use an example where this concept is needed. We can see the ["mysql" Dockerfile](https://github.com/docker-library/mysql/blob/master/5.7/Dockerfile) mounts a volume.

This means that if you're writing a Dockerfile for a container that will require a volume, you need to make sure you create this volume in the build, and make sure the application writes changes to that mountpoint - otherwise, the data will be placed in the *image*, not the *volume*. 

Let's create a mysql container and see what this does to our volumes.

```
docker run --name mysql-demo \
    -p 3306:3306 \
    -e MYSQL_ROOT_PASSWORD=demo \
    -e MYSQL_USER=demo \
    -e MYSQL_PASSWORD=demo \
    -d mysql:latest
```

You'll see that - because of this container's specification, a data volume was created on our host:

```
docker volume ls
```

We can see more detailed info about our volumes using a context-specific "inspect":

```
docker volume inspect < volume ID >
```


What if we kill and remove the mysql-demo container we just ran? What happens to our volume?

```
docker kill mysql-demo
docker rm mysql-demo
docker volume ls
```

Despite our deletion of the mysql container, the data volume is still around! This is because Docker does not do any garbage collection of volumes - it assumes you will want to re-mount this data volume from another container.

From Docker's documentation:

> Data volumes are designed to persist data, independent of the container’s life cycle. Docker therefore never automatically delete volumes when you remove a container, nor will it “garbage collect” volumes that are no longer referenced by a container.

Unused volumes can take up a lot of space if you let them. I've run into this issue a LOT, especially during test/dev when I destroy and recreate containers frequently. (discuss mysql example)

The "volume ls" command has a filter argument that allows us to only display "dangling" volumes - that is, volumes that no longer belong to a container. We can use this to periodically clean up unused data volumes, if needed.

```
docker volume rm $(docker volume ls -qf dangling=true)
```

We'll be using different docker machines for all future labs, so let's clean up our docker-lab machine:

```
docker-machine rm docker-lab
```

## Navigation

[Previous Lab - Docker Images](https://github.com/Mierdin/intro-to-docker-workshop/tree/master/labs/02-docker-images) | [Next Lab - Docker Swarm](https://github.com/Mierdin/intro-to-docker-workshop/tree/master/labs/04-docker-swarm)