## Installing docker
* From Docker: https://store.docker.com/search?type=edition&offering=community

## Verifying docker

```shell
$ docker version
```

By running the `docker version` command we'll get an output similar to the following.

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

### Our first docker container

```
$ docker run bash:latest echo 'hello world'
```

```
Unable to find image 'bash:latest' locally
latest: Pulling from library/bash
4fe2ade4980c: Pull complete
ec6d9ca5c66a: Pull complete
d8685fbd86ca: Pull complete
Digest: sha256:8634afcddefc8a10565b22d685df782058b096712a91bf45d75633f368dda729
Status: Downloaded newer image for bash:latest
hello world
```

Lets see another example

```
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

Let's run the example again:

```
$ docker run ruby:2.4 ruby -e "puts :hello"
```

It only shows the output! That's because we already downloaded the image so it's cached locally. This is a strength of Docker it makes subsequent runs faster.
```
hello
```

Let's break apart the two commands we ran to see the similarities between the two container runs

```
$ docker run bash:latest echo 'hello world'
$ docker run ruby:2.4 ruby -e "puts :hello"
```

```
$ docker run [OPTIONS] <image>:<tag> <command>
```

So if there is caching what all is left on my system?

First let's look at what is running on our system
```
$ docker ps
```

Now let's list all the containers including stopped ones
```
$ docker ps -a
```

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                   PORTS               NAMES
ee0237d3741d        ruby:2.4            "ruby -e 'puts :hell…"   9 hours ago         Exited (0) 9 hours ago                       gracious_pasteur
b97d3f4758c3        ruby:2.4            "ruby -e 'puts :hell…"   9 hours ago         Exited (0) 9 hours ago                       relaxed_swanson
```

We can clean up our containers by running `$ docker rm <container id> [<container id2> ...]`

We can also

# Let's generate a new Rails app

`$ mkdir new_rails_app`
`$ cd new_rails_app`

`$ docker run -it --rm -v ${PWD}:/usr/src/app ruby:2.4 bash`

Inside the container move into the app directory and install rails `gem install rails`

`$ rails new myapp --skip-test --skip-bundle`

Let's see what was generated

`$ ls -la`

Ok now let's use what we know to run this "app"

`$ docker run -it --rm -p 3000:3000 -v ${PWD}:/usr/src/app ruby:2.4 bash -c "apt-get update -yqq && apt-get install -yqq --no-install-recommends node js && cd /usr/src/app && gem install bundler && bundle install && bin/rails s -b 0.0.0.0"`

Wow that is terrible

Let's make it a little better with a Dockerfile

```
FROM ruby:2.4
RUN apt-get update -yqq
RUN apt-get install -yqq --no-install-recommends nodejs

COPY . /usr/src/app/ 7
WORKDIR /usr/src/app

RUN bundle install
```

Break down this file, explain each line and what is going on

To build the image from the file use `$ docker build .`

`$ docker images` shows us the images we have locally and we can run using a specific one by specifying the image id

`$ docker run -p 3000:3000 4ea99c4f5025 bin/rails s -b 0.0.0.0`

Much shorter but still a ton to remember. Let's make it a little less confusing by adding some tagging

`$ docker tag <image id> mkecc`

You can have multiple tags that point to an image

`$ docker tag mkecc mkecc:1.0`

Then we can run our command specifying a tag

`$ docker run -p 3000:3000 mkecc:1.0 bin/rails s -b 0.0.0.0`

Ok that's cool, let's get some house keeping out of the way. Because each layer is a cache of the state of the image, we're going to want to make sure we keep it lean and mean. We can use a `.dockerignore` file to ignore some things from being cached.

```
.git
.gitignore

log/*

tmp/*

*.swp
*.swo
```

Then we can build it again and we'll see the size output, if we dig back to our last build we would see this should be smaller

```
$ docker build -t mkecc .
```

### Gotcha with cache
If the code in the `Dockerfile` doesn't change it's not going to run again. So for something like the `apt-get update` we'll want to chain it with our `apt-get install`. This also means that if we change files in our Rails app it's going to copy and re-bundle! Let's fix the `apt-get` and then make the rails app not re-bundle unless there is a change to the gemfile.

```
RUN apt-get update -yqq && apt-get install -yqq --no-install-recommends nodejs

COPY Gemfile* /usr/src/app/
WORKDIR /usr/src/app

RUN bundle install

COPY . /usr/src/app/
```

Now we see that we don't end up bundling every time we build!

One more adjustment, before we were starting up the Rails server by specifying the comand when we ran the container with `$ docker run -p 3000:3000 mkecc:latest bin/rails s -b 0.0.0.0` let's add a default command to our `Dockerfile`


```
CMD ["bin/rails", "s", "-b", "0.0.0.0"]
```
Now we can get it down to the following command!

`$ docker run -p 3000:3000 mkecc:latest`

# Docker Compose


# Let's adapt an existing Rails app
