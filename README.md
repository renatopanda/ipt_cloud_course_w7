# Cloud Computing and Virtualization - W7 - Docker

Set of examples to ilustrate how to use docker containers.

Requirements:
* Docker Desktop (https://www.docker.com/products/docker-desktop/)
* Under Win10/11: WSL2 + Docker Integration:
    * Use the WSL 2 based engine (Docker Desktop -> Settings -> General)
    * Activate WSL integration in the WSL2 you plan to use (Docker Desktop -> Settings -> Resources -> WSL integration)
    
Check if docker cli is working under your WSL2 distro:
```bash
$ docker -v
Docker version 23.0.5, build bc4487a
```
Alternatively, the commands should work directly under windows using cmd/powershell/gitbash, test at your own risk.

Finally, clone this repo and follow along.

## Docker cli usage
Start by reading all the options available under docker cli:
```bash
$ docker --help

# Expected output:
Usage:  docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Common Commands:
  run         Create and run a new container from an image
  exec        Execute a command in a running container
  ps          List containers
  build       Build an image from a Dockerfile
  pull        Download an image from a registry
  push        Upload an image to a registry
  images      List images

(...)

Run 'docker COMMAND --help' for more information on a command.

For more help on how to use Docker, head to https://docs.docker.com/go/guides/
```


Notice that you can get extra documentation for a specific command, example for the `docker run` command:
```bash
docker run --help

Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Create and run a new container from an image

Aliases:
  docker container run, docker run

Options:
      --add-host list                  Add a custom host-to-IP mapping (host:ip)
  -a, --attach list                    Attach to STDIN, STDOUT or STDERR
      --blkio-weight uint16            Block IO (relative weight), between 10 and 1000,
                                       or 0 to disable (default 0)
      --blkio-weight-device list       Block IO weight (relative device weight) (default [])
      --cap-add list                   Add Linux capabilities
      --cap-drop list                  Drop Linux capabilities
      --cgroup-parent string           Optional parent cgroup for the container

(...)
```


## Hello, world!
Run your first containerized app:
```bash
$ docker run hello-world

# expected output:
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

Read the output, you should understand all the concepts by now (slides).

Check the docker images available, notice the tag, image id and so on:
```bash
$ docker image ls
REPOSITORY                         TAG            IMAGE ID       CREATED         SIZE
hello-world                        latest         9c7a54a9a43c   7 weeks ago     13.3kB
...
```

Check the running containers:
```bash
docker container ls # same as docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

# to check all containers use -a
docker container ls -a
docker ps -a
CONTAINER ID   IMAGE                    COMMAND             CREATED         STATUS                     PORTS     NAMES
ec69e1e7615c   hello-world              "/hello"            5 minutes ago   Exited (0) 5 minutes ago             strange_gates
```

Try rerunning the above commands and notice that a new container is created from the same image.

## BusyBox

BusyBox combines tiny versions of many common UNIX utilities into a single small executable (https://hub.docker.com/_/busybox).

First pull the image from the DockerHub Registry:
```bash
# Pull the image tagged as "latest"
$ docker pull busybox

# Pull a specific image
$ docker pull busybox:1.34

# check the local images again
$ docker image ls

# to remove an image use
$ docker rmi busybox:1.34
```

Now run the image:
```bash
docker run busybox
```
Nothing seems to happen. In reality the container is created, executed and just terminates without outputting anything (it just runs `sh`). You can use the busybox image to run commands inside the container.

```bash
$ docker run --help
# Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

# Run an echo inside a busybox container:
$ docker run busybox echo "Hello, CNV students!"
Hello, CNV students!


# Using the -it flags attaches us to an interactive tty in the container.
# Here, run sh inside busybox and attache STDIN/OUT to it 
$ docker run -it busybox sh
# now you are running sh inside the container:
> / whoami
root
> / uptime
 14:38:00 up 2 days,  4:22,  0 users,  load average: 0.00, 0.15, 3.49
> / ls
bin    dev    etc    home   lib    lib64  proc   root   sys    tmp    usr    var
> / exit
```

### Removing terminated containers

If you check the existing containers, you will see that each `docker run` creates a new container from the specified image, executes, and then terminates, leaving them there in the exited state.

```bash
$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS                     PORTS     NAMES
6cc134dadd5f   busybox   "echo 'Hello, CNV st…"   4 seconds ago   Exited (0) 2 seconds ago             blissful_hopper
ef686873a44f   busybox   "sh"                     9 seconds ago   Exited (0) 7 seconds ago             exciting_leakey
(...)

# Add "--rm" to remove containers when they terminate:
$ docker run --rm busybox echo "Hello, CNV students!"

# One can also remove containers by id using docker rm <ids>
$ docker rm 6cc ef68

# finally, we can prune all terminated containers with:
$ docker container prune
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Deleted Containers:
ef686873a44fddebe2dccd13abcfc4921263c97b625a60f45a731a1df3e38b52

Total reclaimed space: 0B
```

## Acessing container services

Many times you want to use containers and make the services available to other containers or the outside world. To this end you will need a network.

First pull the nginx image (https://hub.docker.com/_/nginx) and experiment with it.

```bash
docker pull nginx

# run a container with nginx
$ docker run --rm nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
(...)
```
The container runs in the foreground, hence the output to console. Trying to open http://localhost results in a timeout. Why?

### Inspect on a second WSL2 terminal

```bash
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
87ea2cdd155c   nginx     "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   80/tcp    angry_kowalevski

# it is running on port 80 but we cannot access it from the outside
```

Terminate the running nginx container with either `ctrl + C` at the terminal, or using:

```bash
$ docker stop 87ea2 # partial id
87ea2 # docker outputs the partial id when succeeded
```

### Exposing ports
In order to access a service, you must expose the internal port to the host:
```bash
# run nginx exposing the container port in one a random port of the host
# we are also setting the container name to cnv_nginx
docker run --rm -P --name cnv_nginx nginx

# in another terminal check the port with:
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS
         NAMES
e8d43359bf86   nginx     "/docker-entrypoint.…"   5 seconds ago   Up 5 seconds   0.0.0.0:32771->80/tcp   cnv_nginx
```

You can see that it is using 32771 -> 80 (host -> container)
Try opening http://localhost:32771.

Instead of using a random port, we can define it with `-p`:
```bash
docker run --rm --name cnv_nginx -p 8080:80 nginx
```
Verify that the default page is now available at http://localhost:8080

### Docker Networks
In brief, by default docker has 3 networks: bridge, host and none. You can create new networks to isolate containers.

Check and create networks with:
```bash
$ docker network ls
NETWORK ID     NAME             DRIVER    SCOPE
e482b6117c49   bridge           bridge    local
3deb3ef9af3f   host             host      local
6b6bb13efc6f   none             null      local

$ docker network create cnv_network
827be4a238047ffc67e6f4650f8deea6e46959361d9b057b64a16c29f79f13f5
$ docker network rm cnv_network
cnv_network
```

By default, a new container is connected to the `bridge` network. For instance, our cnv_nginx container is listed there:
```bash
$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "e482b6117c493c6ca1fb07e49656bc8849231acb679471c2a997a308e53698b6",
        "Created": "2023-06-19T11:44:02.362346851Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },

        (...)

        "Containers": {
            "e8d43359bf86625c8741ad106c6a29865df5f5420b3fb504b442b95342b97e01": {
                "Name": "cnv_nginx",
                "EndpointID": "8d8c2aa998b7e8ef45b7148158449f05d6548055eba9d5c1aa145ab1bbf70cdd",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        }
        (...)
    }
]
```
Each container gets an IP and is accessible by other containers using its name (internal DNS resolution) - this might be limited in the default `bridge` (see slides).

### Detaching STD IO

To run the container in the background we can use the -d flag. After that you can attach to a container, check logs or open the terminal (if possible):
```bash
# run dettached
$ docker run --rm --name cnv_nginx -p 8080:80 -d nginx
8cc2e8e84e4dea79b6cd52dcb728e1c35fca8a0581056f2540f767334ac2604c

$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                  NAMES
8cc2e8e84e4d   nginx     "/docker-entrypoint.…"   19 seconds ago   Up 18 seconds   0.0.0.0:8080->80/tcp   cnv_nginx

# check container ports
$ docker port cnv_nginx
80/tcp -> 0.0.0.0:8080

# check container logs
$ docker longs cnv_nginx
 docker logs cnv_nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
(...)

# attach to the running container
$ docker attach cnv_nginx
# after attaching open http://localhost:8080 to see some output in the STDOUT (terminal)
# CTRL + C to terminate the container/process
# alternatively, use docker stop cnv_nginx in a second terminal
```

## Create our first docker image

One of the major goals with docker is to build new images containing our software and required libraries. The next example uses nginx official docker image to build a simple "web page image".

Lets continue with the previous example:
```bash
docker run --rm --name cnv_nginx -p 8080:80 -d nginx
```
By opening http://localhost:8080 we see the default nginx web page. How can we change that?

On a traditional server we would:
* connect to the server
* edit or upload the html page
* if needed, install extra libraries and restart or reload nginx

We can do that here with containers:
```bash
# open a terminal to the container:
$ docker exec -it cnv_nginx bash

# do the changes inside the container:
> ls
# see image details at https://hub.docker.com/_/nginx

# edit the index.html 
> echo "Hello World!" > /usr/share/nginx/html/index.html
# or use vi / nano
> vi /usr/share/nginx/html/index.html
# no vi or nano?
apt update
apt install vim -y
vim /usr/share/nginx/html/index.html
exit
```
Now check the page at http://localhost:8080. It works... but what happens if you destroy the container? Why are we using images and containers after all?

### Creating a Dockerfile

We can create a docker image that starts with nginx, and has an extra layer (or several) with our changes (or app). To this end we use a Dockerfile that describes the changes.

Check folder `first_image`, it contains the Dockerfile and index.html. We will use the Dockerfile to build an image that:
* starts with nginx:1.25
* copies the local index.html to the folder /usr/share/nginx/html/
* exposes port 80

 ```Dockerfile
# which base image?
FROM nginx:1.25

# set a working directory inside the image
WORKDIR /usr/share/nginx/html/

# copy all the files to that working dir
COPY index.html .

# define the port number the container should expose
EXPOSE 80
```
More could be added, from installing extra packages (using the package manager of the base image, e.g., apt or apk), changing configurations, defining environment variables and so on.

Now build an image by running (make sure you are in the right folder):
```bash
# builds image described in the Dockerfile of the current folder ".", tagged with panda/cnv_first_image
run: docker build -t panda/cnv_first_image .

# check images and run a container using our new image
docker image ls
docker run -p 8888:80 --rm panda/cnv_first_image
```
Check http://localhost:8888

Now imagine you want to create a new version, just change the app (in our case, a simple index.html), and rebuild with an updated tag version
```bash
# change index and rebuild, use tags
$ docker build -t panda/cnv_first_image:1.1 . # (use multiple tags if needed)
# also, note that docker caches layers and only updates the ones that changed
[+] Building 1.9s (8/8) FINISHED
 => [internal] load .dockerignore                                                          0.0s
 => => transferring context: 2B                                                            0.0s
 => [internal] load build definition from Dockerfile                                       0.0s
 => => transferring dockerfile: 275B                                                       0.0s
 => [internal] load metadata for docker.io/library/nginx:1.25                              1.6s
 => [1/3] FROM docker.io/library/nginx:1.25@sha256:593dac25b7733ffb7afe1a72649a43e574778b  0.0s
 => [internal] load build context                                                          0.0s
 => => transferring context: 78B                                                           0.0s
 => CACHED [2/3] WORKDIR /usr/share/nginx/html/                                            0.0s
 => [3/3] COPY index.html .                                                                0.1s
 => exporting to image                                                                     0.1s
 => => exporting layers                                                                    0.1s
 => => writing image sha256:4fc097524af3a30afc7bf8bf6c405e7f9cb6c89dea0b0d6f951b3cc63beab  0.0s
 => => naming to docker.io/panda/cnv_first_image:1.1                                       0.0s
```

Now you can even run both versions simultaneously on different ports (do not forget to stop the previous containers):
```bash
docker run -p 8888:80 --name cnv_nginx_new -d panda/cnv_first_image:1.1
docker run -p 9999:80 --name cnv_nginx_old -d panda/cnv_first_image:latest

# check running containers and inspect default network
docker ps
docker network inspect bridge
```

Now, in a CI/CD scenario, you probably would want to run tests and push the new image to the Docker registry (DockerHub or a private one). To this end there are commands such as `docker login` and `docker push`.

## Isolating different apps
As explained, when running several applications on the same host, it is recommended to isolate them in different networks (check slides).

A simple example, this time using Python 3.8 and Flask to output a "Hello, World!".
```bash
# navigate to second_image
~/first_image $ cd ..
~/ $ cd second_image/
~/second_image $ ls
Dockerfile  app.py  requirements.txt

# check the Dockerfile (also check requirements.txt and app.py is you never used Flask)
~/second_image $ cat Dockerfile
```
```dockerfile
FROM python:3.8 # start with https://hub.docker.com/_/python

# set a directory for the app
WORKDIR /usr/src/app

# copy all the files in current folder to the container WORKDIR
# tip: Dockerfile is not needed there, just to simplify
COPY . .

# RUN command inside the container to install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# define the port number the container should expose
# 5000 because of app.py
EXPOSE 5000

# Finally, run our app when the container starts
CMD ["python", "./app.py"]
```

Now just build the image and launch a container:
```bash
$ docker build -t panda/cnv_second_image:1.0 -t panda/cnv_second_image:latest .
$ docker image ls
$ docker run -p 8080:5000 panda/cnv_second_image
# check that it is running at http://localhost:8080

$ docker network inspect bridge
# all containers run by default on bridge
```

### Custom networks and Volumes
To conclude, we will experiment with networks and volumes. To this end, we will run a container with the our first image and:
* Connect it to a new `cat_network`
* Create a bind_mount volume, so our host and container share a folder
  * This means we can change files locally (host) and see them change on the container - think of it like /vagrant shared folder, but for development using containers.

Instructions:
```bash
# create the network
$ docker network create cat_network
$ docker network ls

# check the docker run help
docker run --help

# make sure the current folder is /third_demo
$ cd ..
$ cd third_demo

# run the container:
# * connect it to our new network
# * add a volume setting a bind mount between the current local folder ".", and the "/usr/share/nginx/html/" inside the container.
docker run --rm --network=cat_network -v .:/usr/share/nginx/html/ --name cat_site1 -p 7777:80 -d panda/cnv_first_image:1.1
```
Now open http://localhost:7777. Edit the `third_demo/index.html` page and and refresh the browser - changes should be instantaneous.

Inspect the networks:
```bash
docker network inspect bridge
docker network inspect cat_network
```

### DNS resolution
As a final test, try `curl` from one container to another inside the same and different networks. Example:
```bash
# run a second copy
docker run --rm --network=cat_network --name first_image_app -p 7778:80 -d panda/cnv_first_image:1.1

# now there are two containers inside our cat_network
# open terminal to one of the containers and try reaching the other by name or ip (inspect network). Example:
$ docker network inspect cat_network
$ docker exec -it cat_site1 bash
> curl first_image_app
```

## Final notes

Check the remaining tutorials (W7 slides). This was just an introduction, you should explore:
* named volumes (persist data)
* docker compose (multi-image apps)
* docker swarm
* eventually K8s

