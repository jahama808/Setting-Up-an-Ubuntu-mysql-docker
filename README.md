# Setting-Up-an-Ubuntu-mysql-docker

This is a guide on setting up a development environment for python coding with communication to a mysql database.

I'll be using Docker-Compose to make this quick and painless, but will go through the creating of the containers
using the regular docker command.

## The UBUNTU 18.04 Container

### What we want in this image

1. Ubuntu 18.04 (or later)
2. python 3
3. pip3
4. mysql client to allow python to talk to the database
5. Create a directory called "/home/jay" for us to work in


### The Dockerfile

We'll build this first to help inform us on how to build the docker-compose.yml file later.

```
From ubuntu:18.04

RUN apt-get update -y && apt-get upgrade -y && apt-get update

RUN apt-get install python3 -y
RUN apt-get install python3-pip -y
RUN apt-get install bash -y
RUN apt-get install nano -y

RUN apt-get install python3-dev libmysqlclient-dev -y
RUN pip3 install mysqlclient

WORKDIR /home/jay
```
That creates our ubuntu 18.04 docker image with python3 and pip3, and mysql support.

#### What we want the container to do

1. Run in interactive mode
2. Present a bash interface
3. Map the "/home/jay" directory in the container to a volume on the host server.

To create the container:
```
docker run -it  -v $(pwd):/home/jay --name ubuntuDevEnv test bash

```

## The mariadb/mysql Container

Setting up the mysql/mariadb container is pretty straight forward.   You can have it do as little or as
much as you want.  For this exercise, we'll have it build our root password and then stop.  The 
rest of the database buildout can be done from an interactive session and then using the standard
mysql command line:

For this, we can pretty much define everything in the "docker run" command.

```
docker run --name ubuntuSQLDevEnv -e MYSQL_ROOT_PASSWORD=theROOTpasswordisNULL -d mariadb:latest
```

### What's being built:

1. --name
  * Assign a custom name to this container
2. -e
  * Set an environment variable, which is the mysql root password.
   * This is the password you use when you do a "mysql -u root -p" command in the cli
3. -d
  * Run in daemon mode
4. -mariadb:latest
  * Grab the latest mariadb image from docker hub


### Accessing our mariadb container

To access the container, we'll use the "exec" command:

```
docker exec -it ubuntuSQLDevEnv bash
```

### What's being built:

1. -it
  * interactive mode
2. ubuntuSQLDevEnv
  * the name of our container
3. bash
  * Give us a bash prompt

## Creating a private network for our environment

By default docker puts the containers on a default network, but you'll need to know the IP addresses
of each container if you want them to talk.  So if you're python script needs to talk to the database, you
have to use the IP.  That can be found by using the "network inspect" command

```
docker network inspect bridge
```
which give the output
```
"Containers": {
            "d283eaaedbfa696c0d3c120e72b89160ed8dbb89b52a223c4451945cbd0e225d": {
                "Name": "ubuntuSQLDevEnv",
                "EndpointID": "e6a72b94f3378474788abade4dc837862bb1ddaced76595b8c1476754fc88079",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
```

The problem with this is that if you rebuild this container on another system the address could be different.
To solve that, we create our own docker network (which includes the build in DNS feature).

```
docker network create ourDevNetwork
```

To get our containers on our network, we need to stop our current containers (and delete them) and then add the "--network" option:

```
docker container rm -f ubuntuDevEnv
docker container rm -f ubuntuSQLDevEnv

docker run --name ubuntuSQLDevEnv -e MYSQL_ROOT_PASSWORD=theROOTpasswordisNULL -d --network ourDevNetwork mariadb:latest
docker run -it  -v $(pwd):/home/jay --name ubuntuDevEnv --network ourDevNetwork test bash

```
That's it.  

With this setup, you can edit the code in the directory you mapped the /home/jay to in the docker run command instead of doing it in the cli of the container.  You'll still need to execute the code from within the container though.

Also important, when you build the python code and have to define the host of the mysql database, use the name of the mysql container you built.  The docker network you built will resolve that to the proper IP address automatically!

