# Docker Quickstart

* Launch an existing container from docker hub
* Launch a container that runs in the background
* Execute commands inside a container
* Connect our containers to the external network

## Launch container

If you're going through these labs using Docker Machine (recommended), please first launch a machine and make sure your Docker client is set to use this:

```
docker-machine create -d virtualbox docker-lab
eval $(docker-machine env docker-lab)
```

Launch a container!

```
docker run busybox echo Hello World!
```

Note that this image is not stored locally, so docker downloads all necessary images before running it.

Notice that the container dies immediately after it is finished.

## Launch long-running container

Let's launch a long-running command instead, and use the -d flag to run our container in "detached" mode.

```
docker run -d --name bbox busybox /bin/sh -c "while true; do date; sleep 1; done"
```

> In this case, we assigned our own name to the container. Since we are keeping it around for a while, we want to be able to refer to it by this name, instead of the random name that Docker comes up with

Notice that this outputs the UUID of the resulting container, and returns us to our prompt instead of keeping us in the container while it loops - this is "detached" mode.

We can check on the status of the container with ```docker ps```:

```
docker ps
```

This shows us some high-level details of the container, including the UUID, which we'll use for the next few commands.

We can also see what the container has written to stdout with ```docker logs```:

```
docker logs bbox
```

Since we executed an infinite loop inside this container, it will never end naturally - so lets kill and remove it:

```
docker kill bbox
docker rm bbox
```

Confirm that nothing is running with one last ```docker ps```.

## Execute commands inside container

Let's restart that long-running container so we can do some other things inside of it:

```
docker run -d --name bbox busybox /bin/sh -c "while true; do date; sleep 1; done"
```

When you pass a command into "docker run", that command or process is running as PID 1 within the PID namespace allocated to that container.

```docker exec``` allows us to run additional commands or processes inside a container, as long as that container is running (remember, the container runs as long as PID 1 is running). We can actually use ```docker exec``` to check on this with the ```ps``` command:

```
docker exec bbox ps -ea
```

Note that the ouput shows our original command as PID 1. Also, the container is still running, even though the "ps" command executed and finished, because that's not what's keeping the container alive, that's PID 1.

```
docker ps
```

We can even get an interactive shell within this running container:

```
docker exec -t -i bbox /bin/sh
```

The ```-t``` flag allocates a psuedo-TTY for this container, and the ```-i``` flag runs this container in "interactive" mode, which keeps STDIN open even if not attached.

At this point, the prompt changes to ```/ #```, to indicate that we're now interactively executing commands within the container's shell. Run ```ps -ea``` to see the same output as before from this perspective.

```
/ # ps -ea
```

Finally, ```exit``` to get out of this container, and kill and remove it as we did before to get rid of it.

```
docker kill bbox
docker rm bbox
```

## Expose port to container on host

Normally, while containers are able to access the outside networks by default, all containers are NATted at the docker0 bridge interface address, and therefore outside entities cannot reach them. This is much like a residential internet connection performing source NAT overload at the internet router.

We can map a port on the host to a port in the container, to allow outside connections to a process within our container.

```
docker run -d -p 80:5000 training/webapp python app.py
```

We're running a simple web server in detached mode, and we're mapping port 5000 on the container, to port 80 on the outside host.

Thus, we should be able to curl to localhost and get the web page:

```
curl $(docker-machine ip docker-lab)
```

## Kill running containers

Let's kill all of the containers that we have started in this lab, so we can move on to the next one:

```
docker kill $(docker ps -q)
```

## Navigation

[Next Lab - Docker Images](https://github.com/Mierdin/intro-to-docker-workshop/tree/master/labs/02-docker-images)
