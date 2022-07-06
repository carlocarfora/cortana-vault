tags: [linux, pcoip]
4
# Getting PCOIP running on POP!
Used the instructions from pcoip website but they weren't working properly. They seem to be outdated so the modified instructions below are what I eneded up using successfully.

Install Docker. I had to use docker-ce not docker
```
$ sudo apt-get install docker-ce

$ xhost +local:docker
```

Add docker to sudo so don't need sudo commands. Not required though.
```
$ sudo groupadd docker

$ sudo usermod -a -G docker  $USER
```

Create a directory for the dockerfile to live in first, cd to it and then build.

```
$ cd <directory with Dockerfile>

$ docker build -t pcoip-client .
```

Dockerfile for docker container. Note that user name has been changed and user/group id's were already 1000 for me.
```
FROM ubuntu:18.04

# Use the following two lines to install the Teradici repository package
RUN apt-get update && apt-get install -y wget && apt-get install -y curl
#RUN wget -O teradici-repo-latest.deb https://downloads.teradici.com/ubuntu/teradici-repo-bionic-latest.deb --no-check-certificate
#RUN apt install ./teradici-repo-latest.deb

RUN curl -1sLf https://dl.teradici.com/DeAdBCiUYInHcSTy/pcoip-client/cfg/setup/bash.deb.sh | sh=ubuntu codename=bionic bash

# Uncomment the following line to install Beta client builds from the internal repository
#RUN echo "deb [arch=amd64] https://downloads.teradici.com/ubuntu bionic-beta non-free" > /etc/apt/sources.list.d/pcoip.list

# Install apt-transport-https to support the client installation
RUN apt-get update && apt-get install -y apt-transport-https

# Install the client application
RUN apt-get install -y pcoip-client

# Setup a functional user within the docker container with the same permissions as your local user.
# Replace 1000 with your user / group id
# Replace myuser with your local username
RUN export uid=1000 gid=1000 && \
    mkdir -p /etc/sudoers.d/ && \
    mkdir -p /home/carlo && \
    echo "carlo:x:${uid}:${gid}:carlo,,,:/home/carlo:/bin/bash" >> /etc/passwd && \
    echo "carlo:x:${uid}:" >> /etc/group && \
    echo "carlo ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/carlo && \
    chmod 0440 /etc/sudoers.d/carlo && \
    chown ${uid}:${gid} -R /home/carlo

# Set some environment variables for the current user
USER carlo
ENV HOME /home/carlo

# Set the path for QT to find the keyboard context
ENV QT_XKB_CONFIG_ROOT /user/share/X11/xkb

ENTRYPOINT exec pcoip-client
```

Command to run the docker container. Changed again to my user name.
```
$ docker run -d --rm -h myhost -v $(pwd)/.config/:/home/carlo/.config/Teradici -v $(pwd)/.logs:/tmp/Teradici/$USER/PCoIPClient/logs -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=$DISPLAY pcoip-client
```

TODO: alias to a command in bash_aliases.rc for easy running 