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



