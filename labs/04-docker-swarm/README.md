# Docker Swarm

* Get a Swarm Cluster Up and Running
* Check on cluster health
* Deploy containers to cluster

## Getting a Swarm Started

This is a summary of the instructions on the [getting started guide for Swarm](https://docs.docker.com/swarm/install-w-machine/). Basically, [install Docker Toolbox](https://www.docker.com/products/docker-toolbox) and use docker machine to set up a Swarm cluster.

[This script](https://gist.github.com/Mierdin/a1d803f501bd34137517) should get a Swarm cluster up and running using Docker Machine on OSX. Note that the commands to destroy the cluster are also provided (commented out).

The rest of the workshop will take place on top of this Swarm cluster - in production, we're typically dealing with some kind of cluster, and not one discrete host.

## Cluster Health Check

Asssuming you have a working Swarm cluster, you can run ```docker info``` to check on your cluster.

```
docker info
```

Take care to note the fact that each node is running at least one container. However, we can still list our containers as if it was all one host:

```
docker ps -a
```

## Swarm Management and Scheduling

The idea of "scheduling a container" is not unlike what an operating system does to schedule processes onto a CPU. Ultimately, it comes down to how you want your resources to be consumed.

A cluster of Docker hosts is very similar to this paradigm - and [Swarm offers serveral scheduling strategies](https://docs.docker.com/swarm/scheduler/strategy/) to assign containers to hosts when they are called upon. We can see that the default scheduling strategy for our Swarm is "spread" by running ```docker info```:

```
docker info
```

> One of the biggest "missing features" from Docker Swarm is the ability to restart containers after a host failure. To my knowledge, [this is still a WIP](https://github.com/docker/swarm/issues/1488).

You can run "swarm manage" from within the Swarm manager, in order to change the strategy, if you wanted to. However, I would recommend keeping this strategy, and use the other tools we'll discuss next. For now, just be aware that you can manage your swarm by running ```swarm manage``` from within the container:

```
docker exec swarm-master/swarm-agent-master /swarm manage --help
```

You can also manage the high-availability rules of the swarm manager itself from this interface.

One common problem when running a cluster of container hosts is constraining certain containers to a set of hosts that would run that workload optimally. For instance, one workload may require SSDs in order to run properly, but perhaps not all of your hosts have this capability.

If you run ```docker info``` once more, you can see that each host has a set of labels attached to it. Some of these are automatically assigned, but others have been manually placed there as flags when the daemon started (if you look at my docker machine bootstrap script, you'll notice I've passed these flags through docker machine):

```
docker info
```

When instructing Swarm to run a container, we can use these labels to constrain that container to a subset of hosts, based on these labels, using [Docker constraint filters](https://docs.docker.com/swarm/scheduler/filter/). For instance, if we simply run a container without any other instruction, the "spread" strategy takes effect, and tries to schedule containers over the cluster as evenly as possible:

```
docker run -d busybox sleep 900     #run this a few times
```

Notice that these containers are pretty evenly spread (the node they're running on is indicated under the "NAMES" column):

```
docker ps
```

Kill all of those containers with ```docker kill $(docker ps -q)``` and then try this again - this time appending a constraint to only start these containers on hosts with a label of "storage==ssd":

```
docker run -d -e constraint:storage==ssd busybox sleep 900     #run this a few times
```

These containers should all be constrained to the one host that has this parameter (naturally in production you'd have multiple hosts that have this label).

```
docker ps
```

Sometimes container placement should be chosen based off of the presence of other containers. For this, you should use affinity rules. A very common example is to have a front-end application running nginx, and a database container for that application. It may be useful to tell Docker to NOT run these on the same host (this is commonly referred to as an anti-affinity).

```
docker run -d -p 80:80 --name frontend nginx
docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=demo -e affinity:container!=frontend mysql
```

Note that these two containers are not on the same host:

```
docker ps
```

There are several other types of filters, but constraints and affinities will get you started. We'll come back to these concepts when we talk about Docker compose.

Don't forget to clean up our mess!

```
docker kill $(docker ps -q)
```

## Navigation

[Previous Lab - Docker Volumes](https://github.com/Mierdin/intro-to-docker-workshop/tree/master/labs/03-docker-volumes) | [Next Lab - Docker Multi-Host Networking](https://github.com/Mierdin/intro-to-docker-workshop/tree/master/labs/05-multihost-networking)
