# See list of docker virtual machines on the local box
$ docker-machine ls
NAME      ACTIVE   URL          STATE     URL                         SWARM   DOCKER   ERRORS
default   *        virtualbox   Running   tcp://192.168.99.100:2376           v1.9.1 

# Note the host URL 192.168.99.100 - it will be used later!

# Build an image from current folder under given image name
$ docker build -t gleb/demo-app .

# see list of build images
$ docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
gleb/demo-app        latest              506bf31537d4        17 minutes ago      904.5 MB
node                 5.0                 c4f955829812        10 weeks ago        642.2 MB

# Note both the original Node:5 and the built images are there

# Let us run the image under name 'demo'
$ docker run --name demo -p 5000:1337 -d gleb/demo-app
9f9f3ae62038805504c3c23cce4e9229008ba6bd9ea16b560a7a9e1cfa932e57

# A good security practice is to run the Docker image (if possible) in read-only mode
# by adding --read-only flag
# See "Docker security" talk at mark 22:00 https://www.youtube.com/watch?v=oANurUSaOFs

# Run docker image with a folder from HOST machine mounted
$ docker run -v /usr/source:/destination --name demo -d gleb/demo-app
# inside the container /destination folder will be pointing at /usr/source from the HOST

# Note: you can pass environment variable values to the `docker run` command
# docker run -e USER=name
# or, if the USER is already an environment var
# docker run -e USER
# or put all env variables into a file and pass its name
# docker run --env-file=<filename>
# You can check all the options passed into the running container
# docker inspect demo

# It prints the long container ID, but we can use our name "demo"
# We also mapped outside port 5000 to container's exposed port 1337
# Let us see running containers
$ docker ps
CONTAINER ID    IMAGE             COMMAND             CREATED              STATUS              PORTS                    NAMES
9f9f3ae62038    gleb/demo-app     "node server.js"    About a minute ago   Up About a minute   0.0.0.0:5000->1337/tcp   demo

# Let us make a dummy request to the app running inside the container
# We will use the virtual machine's IP and outside port 5000
$ curl 192.168.99.100:5000
Not Found

# Let us see the app's console log to confirm that it has received our request
$ docker logs demo
listening on port 1337 { subdomainOffset: 2, proxy: false, env: 'development' }
started server
  <-- GET /
  --> GET / 404 6ms -
# you can follow the logs along using -f (--follow) option

# Jump into the running container to run any commands
# -i option means bind STDIO from the current shell
docker exec -it demo bash
root@9f9f3ae62038:/usr/src/demo-server# ls
... list of files
root@9f9f3ae62038:/usr/src/demo-server# exit

# If you want to quickly list files in the docker container (default folder)
# you can use `docker exec`
$ docker exec -t demo ls
# ... list of files in /usr/src folder

# We can even copy files from the container back into our system
docker cp demo:/usr/src/demo-server/file.txt file.txt
# look at file.txt locally

# Running Docker container in terminal
# If you want to see the output / enter commands *inside the container* right from the terminal
# just skip -d option (-d is for detached mode)
$ docker run --name demo -it gleb/demo-app

# if the container stops (like the command terminates due to error)
# a nice trick is to run the container with dummy infinite command
# then shell into the container and find the problem
$ docker run --name demo -d gleb/demo-app tail -f /dev/null
$ docker exec -it demo bash

# Nice!
# To stop the running container
$ docker stop demo