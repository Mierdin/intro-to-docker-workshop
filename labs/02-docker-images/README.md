# Docker Images

* Work with existing images
* Build custom images with Dockerfiles
* Sharing our images using Docker Hub

## Look at existing images on system

We can list the existing images on the system:

```
docker images
```

You'll recognize some of these - for instance, the "busybox" image that we used in lab 01.

You will also notice the user/repo format, which is similar to Github for some of these (i.e. training/webapp).

Also notice that the "tag" for all of these images is "latest", which is self-explanatory. We'll work with different tags shortly.

## Pull Images from Docker Hub

In our last example, we simply ran a docker image using ```docker run```, but if you recall, before we could use an image for the first time, we had to download it. This was done for us, but what if we wanted to download an image explicitly? And where do they come from?

By default, when you perform an action that requires that Docker download an image, it does so from [Docker Hub](https://hub.docker.com), a Docker-run docker image repository.

Here is an example pull (check out the [corresponding Docker Hub page](https://hub.docker.com/r/training/docker-fundamentals-image/))

> These pulls don't have to finish if they're taking too long - we don't really use these images in any meaningful way

```
docker pull training/docker-fundamentals-image
docker images
```

As mentioned before, tags are a way to specify the "version" of an image. (Note in the last pull that "latest" is the default if unspecified). These are defined by the maintainer of the image, so the version depends on how they have things set up. For instance, the ubuntu image uses tag names closely tied to the actual version of ubuntu that they represent:

```
docker pull ubuntu:14.04
docker images
```

We'll be working a bit more with Docker Hub after the next section.

## Building Custom Docker images

Many of the images we've been experimenting with have been built off of other base images. Docker has a domain-specific language for specifying how to build a Docker container, called a Dockerfile. We're going to walk through writing our own Dockerfile.

Generally, Dockerfile instructions follow the following format:

```
INSTRUCTION arguments
```

The instruction doesn't need to be capitalized, but this is the accepted convention for keeping instructions and arguments clearly separate.

We won't be using all of the instructions in our example, but we'll use the common ones, and the rest are well documented in the [Dockerfile reference online](https://docs.docker.com/engine/reference/builder/).

I am going to walk through a typical Dockerfile, and use it to build a container to run a really basic Go app.

First, let's create a "src" directory, and download our Go application from the repository. Don't worry about the contents of this file, it's not the point of the workshop. It is a basic web server.

```
mkdir src
wget -O src/main.go https://raw.githubusercontent.com/Mierdin/intro-to-docker-workshop/master/labs/02-docker-images/src/main.go\?token\=AECM-3_pGYU5QYdZfTClij6wNOu6qqXGks5Wti38wA%3D%3D
```

> We will be using the image "golang:1.5" in the following example. This is not required, but for the sake of the demonstration, it may be useful to "pull" this image in advance while we're putting together our own Dockerfile.

Let's edit a new file in our environment, and call it "Dockerfile":

```
vi Dockerfile
```

For the first line, it's appropriate to specify what base image you're building off of. This is done using the FROM instruction. I like to use specific tags for base images in my Dockerfile, so I always end up with an expected result, even if the base image gets new versions. 

```
FROM golang:1.5
```

This particular image will be necessary in order for us to compile our Go application. Note that the tag corresponds with the version of Go that's installed.

It's also useful to specify MAINTAINER near the top as well:

```
MAINTAINER Matt Oswalt <matt@keepingitclassless.net> (@mierdin)
```

Next we want to add some meaningful metadata to the container. Docker does this through the LABEL instruction, which follows the format of key/value pairs. What we add here will depend on what uses this metadata - but generally you can't go wrong with a simple description.

```
LABEL description="A very simple Go application"
```

In this case, the key for this label is "description", and the value for that key is "A very simple Go application". We'll see where this metadata is viewable once we've built our container.

We can set environment variables within the container. Since we are going to compile our Go program into /go/bin, it's a good idea to add this to the path inside our container:

```
ENV PATH /go/bin:$PATH
```

Next, we can use the ADD instruction to get our source code into the container:

```
ADD ./src /go/src
```

Now that the source code is there, we can use the RUN instruction to tell Go to compile our application. This is similar to the ```docker run``` command we used in the last lab - the command provided here is run inside the container.

> For you power users out there, you shouldn't compile Go apps inside the same container you provide the binaries in - the binaries end up being quite small but the Go installation is quite large. Best practice is to compile the binaries outside the container and copy just those binaries. We're doing it this way in this workshop for simplicity.

```
RUN cd /go/src && go build -o ../bin/hellogo
```

We can also provide some specific metadata about which ports this container is listening on, using the EXPOSE keyword. Note that this doesn't automatically map an external host port to this container, as we did in the last lab. We still need to do this when we run the container, using the "-p" flag.

```
EXPOSE 80
```

Finally, for ease of use, we should provide this container with an entrypoint. This will be the default command that is run when the container is started.

```
ENTRYPOINT ["hellogo"]
```

The parameter to the ENTRYPOINT instruction is actually an array of strings, in case the command has any arguments you wish to pass in. In our case, we only want to run the binary without arguments.

Finally, save this file and get back to a shell prompt. The ```docker build``` command will create our container for us based on this Dockerfile.

```
docker build -t mierdin/hellogo:1.0 .
```

Note that I am specifying username (for dockerhub purposes - not strictly required), repo name, and tag. The dot indicates I want to build in this directory (so this is where docker will look for the Dockerfile)

We can see all kinds of detailed information about our container using the ```docker inspect``` command:

```
docker inspect mierdin/hellogo:1.0
```

Finally, we can run our container using the -d and -p flags ("d" for detached mode - remember, this is a web server! Also, "-p" allows us to map host TCP ports to our container.)

```
docker run -d --name hellogo -p 80:80 mierdin/hellogo:1.0
```

Note also that we're not specifying a command, as we need when we used ```docker run``` in the last lab. This is because we have specified a default entrypoint - by simply starting the container, we can rest assured that our webserver is running.

To prove it, curl once more to our docker machine:

```
curl $(docker-machine ip docker-lab)
```

## Sharing to Docker Hub

It's fairly easy to make this container available to others, by pushing this container to docker hub.

> You can of course use your own Docker registry, but I leave that to you.

We first need to generate docker hub credentials:

```
docker login
```

Now that our local config has been generated, we can push our container to docker hub.

```
docker push mierdin/hellogo:1.0
```

Finally, it's worth noting that the command ```docker commit``` can be used to update a container, in the event you've made changes to it's metadata, or its contents. However, in my experience, this is not used. Since Docker containers are generally built as the result of a CI/CD pipeline, the "docker build" process is going to overwrite the container anyways - so just use proper tags to manage this.

## Kill running containers

Let's kill all of the containers that we have started in this lab, so we can move on to the next one:

```
docker kill hellogo
docker rm hellogo
```

## Navigation

[Previous Lab - Docker Quickstart](https://github.com/Mierdin/intro-to-docker-workshop/tree/master/labs/01-docker-quickstart) | [Next Lab - Docker Volumes](https://github.com/Mierdin/intro-to-docker-workshop/tree/master/labs/03-docker-volumes)
