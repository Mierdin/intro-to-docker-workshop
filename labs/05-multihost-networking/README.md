# Docker Multihost Networking

* Examine default/legacy networking behavior
* Create a multi-host overlay network
* Attach containers to this network and test connectivity

## Default/Legacy Docker Networking

Remember in the very first lab, whenever we wanted to expose a container to the outside word, we had to start that container with the "-p" flag to indicate port mappings? This is because the default network, ```bridge``` works by NATing each container's IP address under the docker0 bridge.

We can list the networks available to us:

```
docker network ls
```

Note that we're still running on our Swarm, so we're seeing the [3 default networks](https://docs.docker.com/engine/userguide/networking/dockernetworks/): ```bridge```, ```null```, and ```host```, multiplied by three nodes in our cluster.

Do not worry about the ```null``` and ```host``` networks for now - they are built into Docker and not the focus of this exercise.

You may have heard about Docker's ["link" feature](https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks/). This was a way for two containers on the same host to be able to share information about each other, and therefore connect securely to each other without using the port mapping mechanism. This feature is still supported for backwards compatibility reasons, but the present and future of inter-container communication is the Docker Networking framework brought to us by libnetwork - so we will not be discussing linking here.

## Multi-Host Overlay Networking

Docker 1.9 introduced multi-host networking.

FYI, our [Swarm script](https://gist.github.com/Mierdin/a1d803f501bd34137517) configures the k/v store necessary for docker networking. With that script you can just simply start creating networks.

We have a ```docker network``` command now. See all of the available options:

```
docker network create --help
```

The default network driver is actually "overlay", so we can simply create a new network, and Docker will assume this is what we want:

```
docker network create my-net
```

Note that the default network, ```overlay``` is chosen for us, when we list networks again:

```
docker network ls
```

We can also see more advanced information about this network by running ```docker network inspect```:

```
docker network inspect my-net
```

## Attach Containers to Overlay Network

Lets start a webserver, and specify that we want to attach it to our new overlay network:

```
docker run -d --name=web --net=my-net nginx
```

Next, lets run a busybox image that will allow us to run "wget" to retrieve this new web page over our overlay network. Note the anti-affinity rule (you may recognize this from the Swarm lab) that ensures that these two containers are not scheduled to the same host, which means they have to use the overlay network:

```
docker run --rm --net=my-net --env="affinity:container!=web" busybox wget -O- http://web
```

You should see output containing the HTML of the page on our webserver, indicating that we've successfully communicated over the overlay network. Note that we didn't have to provide port mappings for this to happen - this is the beauty of the overlay.

> We know this took place over the overlay because these two containers were on different hosts, and we did not create any port mapping rules. This was all "intra-cluster" communication. This means, only traffic actually leaving or entering the cluster from the outside requires this port mapping now. You'll still want to provide port mappings if the entity connecting to a container can't participate in the overlay (i.e. isn't a Docker host, like a user). However, this mechanism makes it a lot easer for application-to-application connectivity within the datacenter.

Also note that we're connecting to the web server by name: "web". This is because Docker manages these (and other) files through an [overlay storage mount](https://docs.docker.com/engine/userguide/networking/default_network/configure-dns/)) Check it out:

```
docker run -it --rm --net=my-net --env="affinity:container!=web" busybox /bin/sh
mount
cat /etc/hosts
```

Finally, let's kill that nginx container we created earlier!

```
docker kill web
docker rm web
```

## Navigation

[Previous Lab - Docker Swarm](https://github.com/Mierdin/intro-to-docker-workshop/tree/master/labs/04-docker-swarm) | [Next Lab - Docker Compose](https://github.com/Mierdin/intro-to-docker-workshop/tree/master/labs/06-docker-compose)
