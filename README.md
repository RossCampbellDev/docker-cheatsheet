# Docker
Docker allows for easy setup and recreation of specific environments.
 
For example, if our software requires a specific version of several different services (PHP, mySQL, Java etc) then we can create a docker container for this configuration
we can then use Docker to simply start a container and run our software inside of it.
Docker is similar to a Virtual Machine, however it is generally more light weight.
Docker is 1 layer of abstraction higher than a VM, in that a VM simulates the Operating System layer, while Docker containers share the underlying OS between multiple containers
commands
sudo service docker start 
^ may be needed to start the docker service before beginning

`sudo docker run docker/whalesay hello-there!`
`sudo docker run --name rossApp nginx`
^ start docker and run the container "docker/whalesay" with the parameter 'hello-there!'
--name flag much come before the name of the container
`docker ps -a`
^ shows us currently running docker containers.  the -a flag shows us ones that have been run but are now closed
`docker stop \<container name\>`
^ kills the container.  use docker ps to find container name
`docker rm \<container name\>`
^ permanently remove a container (returns name of container if successful)
`docker images / docker rmi \<img name\>`
^ lists all installed images.  rmi removes one
`docker pull \<img name\>`
^ downloads and installs a docker image
a container only lasts as long as its task is incomplete
`docker run ubuntu` ends instantly because there is no task running inside of it, just the OS
`docker run ubuntu sleep 5` would run ubuntu OS and then use the OS sleep for 5 seconds, and THEN the container would end

to run a command on a running container:
`docker ps -a
-\> docker exec \<container name\> \<command\>
-\> docker exec ubuntu curl bbc.co.uk`

Docker containers are attached to the current terminal window like a venv
in order to detach, do a CTRL+C

We can provide -d flag to run a container in the background instead

## docker run
`docker run redis:4.0`
redis is the name of the container running a redis service
the ":" character gives it a "tag"
the tag can allow us to specify the version - pulling the different version if necessary
"latest" is the default assumed tag if you do not give one
dockerhub.com will list all available tags (versions)

docker container does not listen to standard input from the terminal.  if you are going to provide input, you must map input to the container using the -i parameter for interactive mode.

`docker run -i redis`
`docker run -it redis`
the addition of the t flag gives us the output of the terminal that the app is working with.  Without it, we are not attached to that terminal so we do not see output

### Port Mapping
2 options.  1 is to use the IP of the container which it gets by default.  Is an internal IP only accessible within the docker host (eg me on my pc i can browse to this IP)

other option, use the docker host's IP and then map the port inside the container to a free port on the docker host.  ex:  map port 80 on the host to 5000 on the container
we can map multiple host ports to the same container port

`docker run -p 80:5000 \<container name\>`

### volume mapping
we can map data to a location on the host, rather than have it inside the container.  this prevents the data being lost if the container ends

`docker run -v /fld/dir/mysql-data mysql`

### inspect container
get info about the specified container in a JSON format with docker inspect \<name\>
we can get logs about a container with:
`docker logs \<name|id\>`

### Environment Variables
use the -e flag to pass the name(and value) of an environment variable that may be used in an app within a container
`docker run -e ENV_VAR=test123 \<name\>`

docker inspect can be used to find a list of env variables that are necessary for a container.

## Create own image
decide what you need, eg for a flask server OS -\> apt updates -\> dependencies like Python, MySQL -\> copy source code to the right folder -\> run the server

## Dockerfile - docker build
`docker build . -t app-name`
the period refers to 'this directory' (build requires us to be in same context with the Dockerfile)
-t shows that we are giving a name (and potentially a tag too - name:tag)

in the config file we use "instructions" which are all in caps.
FROM - defines base OS (eg FROM ubuntu).  your image must be based off another image.  find OS images on dockerhub. all docker files must start with a FROM instruction
RUN - instructs the container to run these commands (eg RUN apt-get update)
COPY - copy files from local media to the image.  COPY . /destination/ (the full stop = current location)
ENTRYPOINT - specify a command that will be run when the image is run as a container

`docker history \<img name\>` - shows the sizes of stuff pulled for the container

## Entrypoint vs CMD
Entrypoint defines a command which will be executed at run time - but you can give parameters in the 'docker run' command.
e.g: ENTRYPOINT ["sleep"] -\> docker run sleep-app 10 -\> will run, and sleep for 10 seconds
CMD sleep 10 -\> will simply run this cmd at runtime

the entrypoint can be overridden with the --entrypoint flag
## Docker Networks
the default setup is a Bridge, where all containers fall under 172.17.XXX.XXX
they can communicate with eachother
there is a --network=none|host flag to change the configuration

if i use `--network=host` on my squadventurers container, i can then navigate to the IP of the VM hosting the container

we can use docker network create `/ --driver=bridge / --subnet=182.18.0.0/16 / nwrk-name`
to create our own custom bridge within a container (eg if we want multiple networks)

`docker network` ls to show them all

Docker has a built in DNS so that you can simply use the container names instead of IP addresses!

docker network inspect ls
## Docker File System
Containers are made up from layers, starting with the lowest (eg Ubuntu) and working upwards (eg Python -\> source code)
Container data is read/write, but disappears when the container STOPS.

to create a volume of data inside the container:
`docker volume create rossFolder`
`docker run -v rossFolder:/var/lib/mysql imgName`
this creates 'rossFolder' in the /var/lib/mysql directory

volume create command allows the data to persist
## Docker compose
use YAML syntax to define a runnable composition of containers.  saves us running a bunch of docker run commands every time we want to use a certain configuration
eg OS, app, db, messaging platform containers all in 1

## Docker link
connect two containers together
--link python:python-app

this creates an entry in the hosts file of the container that needs to interact with the python container.  It maps the IP for python-app container to the name

--link python:python-app --link redis:redis-app

deprecated
--version: 2 # required in 2+
services:
\<name\>:
image: vote-app
build: ./vote-app # can replace image with this, and give dir of Dockerfile
ports:
- 5000:80
links:
- python
- redis
\<name\>:
image: mysql
depends_on: vote-app

don't need to use links with version 2+ of docker compose file
in 2+, docker attaches to default bridged network, so links are not required because it automatically does it (the hosts file additions)

version 2 and 3 have "services" section

you can use docker compose to start up several containers simultaneously, and have them automatically on the same bridged network
we can use depends_on: to specify if any of the images MUST wait for another image to start up before they can be started

we can use networks: to specify which network(s) the container is attached to

version 3 
## Docker prune -a
get back a shitload of disk spce

## Remote Docker Registries
docker.io/nginx/nginx
[registry]/[user]/[image]

you can have private registries which are not open to the public.

`docker login private-registry.io`
`docker run private-registry.io/apps/internal-app`

how to deploy your own?
docker run -d -p 5000:5000 --name registry registry
the above runs the private registry container
docker image tag my-image localhost:5000/my-image
then we can push the image to this registry container. this means that the image is available anywhere on the specified network (for pulling/pushing) but it is not available outside of that
`docker push localhost:5000/my-image`
`docker pull localhost:5000/my-image`
`docker pull 192.168.56.100:5000/my-image` (can use IP instead)

## Docker Engine
is what you install on a host machine.  Docker engine is made up of:
- docker CLI
- REST API
- Docker Daemon

the CLI requires the REST API to interact with the daemon
you can run a container on a remote host:
`docker -H=192.168.0.69:1234 run \<imgname\>`

cgroups - control groups.  can restrict the amount of hardware allocated to the container
`docker run --cpus=.5 \<imgname\>`
`docker run --memory=100m \<imgname\>`
limit CPU to 50%, or RAM to 100mb
	
#CLI #cheatsheet #server #docker
