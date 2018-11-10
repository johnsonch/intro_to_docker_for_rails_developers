footer: Chris Johnson :: @johnsonch :: Intro to Docker for (Rails) Developers
autoscale: true

# Intro to Docker for (Rails) Developers

---
## Who am I

### Chris Johnson
* Dad and Husband
* Most places @johnsonch
* Ops Manager @ healthfinch
* Into IoT, Hockey, Racing, Food and Beer

---
## What are we going to talk about today?

* Docker
* Some of my experiences
* Possibly Docker Compose

![full](https://images.pexels.com/photos/753331/pexels-photo-753331.jpeg)

---
## What are we *not* going to talk about today?

* Production deployments
* Cupcakes
* Things I don't want to talk about

![full](https://images.pexels.com/photos/8148/kitchen-cookies-work-cake-8148.jpg)

---
## Ground rules

* These are my experiences and opinions, your mileage may vary
* I'm not here to argue, if you want to do that buy me a beer afterwards
* Ask questions, but again I'm not looking for a fight

![full](https://images.pexels.com/photos/209954/pexels-photo-209954.jpeg)

---
# What is Docker?

![full](https://pbs.twimg.com/profile_images/1012762345238454272/Q9jiI1pL_400x400.jpg)

---
### What is Docker?

> Docker is used to run software packages called "containers". Containers are isolated from each other and bundle their own tools, libraries and configuration files; they can communicate with each other through well-defined channels. All containers are run by a single operating system kernel and are thus more lightweight than virtual machines. Containers are created from "images" that specify their precise contents. Images are often created by combining and modifying standard images downloaded from public repositories.

![full](https://images.pexels.com/photos/684385/pexels-photo-684385.jpeg)

---
## TL;DR;
### Docker is a company that makes it easier to work with containers

---
## Well then smarty pants, what the h-e-double-hockey-sticks is a container

* An isolated execution environment
* Specifically an isolated process that doesn't know about the rest of the system
* Different from VMs in that they don't require a hypervisor and entire OS instead they piggy-back on a single Kernel

^ Think of it as a box for a single product, one that makes transporting and storing consistent and universal

![full](https://images.pexels.com/photos/40861/jeans-pants-clothing-blue-40861.jpeg)

---
## What's an image?

* The blueprint for the container
* It's like a factory
* What we distribute

![full](https://images.pexels.com/photos/297648/pexels-photo-297648.jpeg)

---
## Container vs Image

You run a *container* and share an *image*

![full](https://images.pexels.com/photos/257552/pexels-photo-257552.jpeg)


---
## Installing docker
* From Docker: https://store.docker.com/search?type=edition&offering=community

![full](https://images.pexels.com/photos/536/road-street-sign-way.jpg)

---
# Demo Getting Started

![full](https://images.pexels.com/photos/878352/pexels-photo-878352.jpeg)

---
## Verifying docker

```shell
$ docker version
```

---
### By running the `docker version` command we'll get an output similar to the following.

```shell
Client:
 Version:           18.06.1-ce
 API version:       1.38
 Go version:        go1.10.3
 Git commit:        e68fc7a
 Built:             Tue Aug 21 17:21:31 2018
 OS/Arch:           darwin/amd64
 Experimental:      false

Server:
 Engine:
  Version:          18.06.1-ce
  API version:      1.38 (minimum version 1.12)
  Go version:       go1.10.3
  Git commit:       e68fc7a
  Built:            Tue Aug 21 17:29:02 2018
  OS/Arch:          linux/amd64
  Experimental:     true
```
---
## Our first docker container

```shell
$ docker run shell:latest echo 'hello world'
```

Gives us the following

```
Unable to find image 'shell:latest' locally
latest: Pulling from library/shell
4fe2ade4980c: Pull complete
ec6d9ca5c66a: Pull complete
d8685fbd86ca: Pull complete
Digest: sha256:8634afcddefc8a10565b22d685df782058b096712a91bf45d75633f368dda729
Status: Downloaded newer image for shell:latest
hello world

```

---
## Lets see another example

```shell
$ docker run ruby:2.4 ruby -e "puts :hello"
```

Here we can see that there is more to the Ruby image and it takes a little long to pull down.

```
Unable to find image 'ruby:2.4' locally
2.4: Pulling from library/ruby
bc9ab73e5b14: Pull complete
193a6306c92a: Pull complete
e5c3f8c317dc: Pull complete
a587a86c9dcb: Pull complete
72744d0a318b: Pull complete
31d57ef7a684: Pull complete
c8c56b2e15ce: Pull complete
926f1d0d6e72: Pull complete
Digest: sha256:3c9ac52e86ebdb9cf0d42a459ccb016ebfba347c0a5d4621fa8d70a5ede72c7b
Status: Downloaded newer image for ruby:2.4
hello
```

---
## Let's run the example again:

```shell
$ docker run ruby:2.4 ruby -e "puts :hello"
```

It only shows the output!

^ That's because we already downloaded the image so it's cached locally. This is a strength of Docker it makes subsequent runs faster.

---
Let's break apart the two commands we ran to see the similarities between the two container runs

```shell
$ docker run bash:latest echo 'hello world'
$ docker run ruby:2.4 ruby -e "puts :hello"
```

Breaks down into

```
$ docker run [OPTIONS] <image>:<tag> <command>
```

^ We'll see later how complex these can get!

---
## So if there is caching what all is left on my system?

First let's look at what is running on our system
```
$ docker ps
```

---
## Now let's list all the containers including stopped ones
```
$ docker ps -a
```

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                   PORTS               NAMES
ee0237d3741d        ruby:2.4            "ruby -e 'puts :hell…"   9 hours ago         Exited (0) 9 hours ago                       gracious_pasteur
b97d3f4758c3        ruby:2.4            "ruby -e 'puts :hell…"   9 hours ago         Exited (0) 9 hours ago                       relaxed_swanson
```

---
## We can clean up our containers by running

`$ docker rm <container id> [<container id2> ...]`

^ There are other commands or chains of commands out there, this is unix afterall, that clean up all images without you having to type out a list but that's a bit more advanced than what we're going to get into.

---
# Demo generating a new Rails app

![full](https://images.pexels.com/photos/72594/japan-train-railroad-railway-72594.jpeg)

---
## Let's generate a new Rails app

`$ mkdir new_rails_app`
`$ cd new_rails_app`

`$ docker run -it --rm -v ${PWD}:/usr/src/app ruby:2.4 bash`

Inside the container move into the app directory and install rails `gem install rails`

`$ rails new myapp --skip-test --skip-bundle`

Let's see what was generated

`$ ls -la`

---
## Ok now let's use what we know to run this "app"

`$ docker run -it --rm -p 3000:3000 -v ${PWD}:/usr/src/app ruby:2.4 bash -c "apt-get update -yqq && apt-get install -yqq --no-install-recommends node js && cd /usr/src/app && gem install bundler && bundle install && bin/rails s -b 0.0.0.0"`

---
# That is a terrible command and it takes for ever for the app to boot!

---
# Bring on the Dockerfile

---
## Dockerfile

Inside of our Rails app we'll create a file called `Dockerfile` with the contents

```
FROM ruby:2.4
RUN apt-get update -yqq
RUN apt-get install -yqq --no-install-recommends nodejs

COPY . /usr/src/app/
WORKDIR /usr/src/app

RUN bundle install
```

^ Here we have a small example of a Dockerfile. Each line has an action or actvity. Those are in all caps `FROM`, `RUN` and `COPY` are examples.

---
## Let's use this Dockerfile to build an image!

`$ docker build .`

---
## Then verify it

`$ docker images`

^ Shows us the images we have locally and we can run using a specific one by specifying the image id

---
## Now we can run our app again

`$ docker run -p 3000:3000 4ea99c4f5025 bin/rails s -b 0.0.0.0`

^ Much shorter but still a ton to remember. Let's make it a little less confusing by adding some tagging

---
## Tags

* They are references to a specific point in time
* Can be used again
* There are no 'magic' tags - latest isn't allways the latest

---
## Tags

Let's tag our image

`$ docker tag <image id> mkecc`

---
## Tags
You can have multiple tags that point to an image

`$ docker tag mkecc mkecc:1.0`

---
## Tags
Then we can run our command specifying a tag

`$ docker run -p 3000:3000 mkecc:1.0 bin/rails s -b 0.0.0.0`

^ Ok that's cool, let's get some house keeping out of the way. Because each layer is a cache of the state of the image, we're going to want to make sure we keep it lean and mean. We can use a `.dockerignore` file to ignore some things from being cached.

---
## Caching

* Each line in the `Dockerfile` is a new layer
* It stores information about the state of the image at that point in time, files and all

---
## Caching

We can avoid some of our app being cached by adding a `.dockerignore` file, which works much like a `.gitignore` file for Git.

This will take out some of the known Rails wasted spaces

```
.git
.gitignore

log/*

tmp/*

*.swp
*.swo
```
Then we can build again!

```shell
$ docker build -t mkecc .
```

---
## Gotcha with cache
If the code in the `Dockerfile` doesn't change it's not going to run again. So for something like the `apt-get update` we'll want to chain it with our `apt-get install`. This also means that if we change files in our Rails app it's going to copy and re-bundle! Let's fix the `apt-get` and then make the rails app not re-bundle unless there is a change to the gemfile.

---
## Gotcha with cache

```
RUN apt-get update -yqq && apt-get install -yqq --no-install-recommends nodejs

COPY Gemfile* /usr/src/app/
WORKDIR /usr/src/app

RUN bundle install

COPY . /usr/src/app/
```

---
## Gotcha with cache

Let's build it again (maybe even a few times)

```shell
$ docker build -t mkecc .
```
Now we see that we don't end up bundling every time we build!

---
## One final adjustment

Before we were starting up the Rails server by specifying the comand when we ran the container with `$ docker run -p 3000:3000 mkecc:latest bin/rails s -b 0.0.0.0` let's add a default command to our `Dockerfile`

```
CMD ["bin/rails", "s", "-b", "0.0.0.0"]
```

---
## One final adjustment

Now we can get it down to the following command!

`$ docker run -p 3000:3000 mkecc:latest`


---
## Wrap up
We learned:
* What Docker is
* The difference between a container and an Image
* How to make a `Dockerfile`
* How to run a Rails application from a container

![full](https://images.pexels.com/photos/860620/pexels-photo-860620.jpeg)

---
# Questions

![full](https://images.pexels.com/photos/355952/pexels-photo-355952.jpeg)

---
![full](http://johnsonch.com/2018-milwaukee-code-camp-speaker-intro-image-part-one.png)
