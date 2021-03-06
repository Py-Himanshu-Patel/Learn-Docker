# Working with Docker Containers

Run below command to verify the docker is running
```bash
~ $ docker version
Client: Docker Engine - Community
 Version:           20.10.12
 API version:       1.41
 Go version:        go1.16.12
 Git commit:        e91ed57
 Built:             Mon Dec 13 11:45:34 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.12
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.16.12
  Git commit:       459d0df
  Built:            Mon Dec 13 11:43:42 2021
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.12
  GitCommit:        7b11cfaabd73bb80907dd23182b9347b4245eb5d
 runc:
  Version:          1.0.2
  GitCommit:        v1.0.2-0-g52b36a2
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

### Listing/searching for an image
The docker search command lets you search an image on a Docker registry.

```bash
~ $ docker search --limit 5 alpine
NAME                         DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
alpine                       A minimal Docker image based on Alpine Linux…   8389      [OK]       
anapsix/alpine-java          Oracle Java 8 (and 7) with GLIBC 2.28 over A…   478                  [OK]
alpine/git                   A  simple git container running in alpine li…   193                  [OK]
yobasystems/alpine-mariadb   MariaDB running on Alpine Linux [docker] [am…   106                  [OK]
byrnedo/alpine-curl          Alpine linux with curl installed and set as …   35                   [OK]
```

The convention for an image name is `<user>/<name>`, but it can be anything.

To list images that got more than 20 stars and are automated, run the following command:
```bash
docker search --filter is-automated=true --filter stars=20 alpine
```

The `--insecure-registry` option for the Docker daemon is provided, which allows us to search/pull/commit images from an insecure registry.

### Pulling an image

To pull an image from the Docker registry, you can either run the following:
```
docker image pull [OPTIONS] NAME[:TAG|@DIGEST]
```
Or the legacy command:
```
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

The `pull` command downloads all layers from the Docker registry that are required to
create that image locally.

To pull an image with a specific tag:
```
$ docker image pull centos:centos7
```
By default, the image with latest tag gets pulled. To pull all images corresponding to all
tags, use the following command:
```
docker image pull --all-tags alpine
```
Once an image gets pulled, it resides on the local cache (storage), so subsequent pulls will
be very fast.

### Listing images
We can list the images that are available on the system by running the Docker daemon.
These images might have been pulled from the registry, imported through the `docker
image pull` command, or created through a Dockerfile.

```bash
docker image ls
docker images
```

### Starting a container
Second command is preferred and is expected way to call container management
```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
docker container run [OPTIONS] IMAGE [COMMAND] [ARG...]
```
Here is an example of using the docker container run command:
```bash
docker container run -i -t --name mycontainer ubuntu /bin/bash
```

By default, Docker picks the image with the latest tag:
- The `-interactive` or `-i` option starts the container in interactive mode by
keeping the STDIN open.
- The `--tty` or `-t` option allocates a pseudo-tty and attaches it to the standard
input.  
Start a container from the `ubuntu:latest` image, attach `pseudo-tty`, name it mycontainer, and run the `/bin/bash` command.

#### How it works
- Merge all the layers, that makeup that image using UnionFS.
- Allocate a unique ID to a container, which is referred to as the Container ID.
- Allocate a filesystem and mount a read/write layer for the container. Any changes on this layer will be temporary and will be discarded if they are not committed.
- Allocate a bridge network interface.
- Assign an IP address to the container.
- Execute the process specified by the user.

With the default Docker configuration, it creates a directory (with the container's ID
inside `/var/lib/docker/containers`), which has the container's specific information
such as hostname, configuration details, logs, and `/etc/hosts`.

To exit from the container, press Ctrl + D or type exit. It is similar to exiting from a shell, but this will stop the container. Alternatively, **to detach from the container**, press `Ctrl + P + Q`. The detached container will detach itself from the terminal, give control back to the Docker host shell, and wait for the docker container attach command to attach back
to the container.

The container can be started in the background, and then we can attach to it whenever needed. We need to use the `-d` option to start the container in the background:
```bash
docker container run -d -i -t ubuntu /bin/bash
```

The preceding command returns the container ID of the container, which we can attach to
later, as follows:
- Using ID
```bash
ID=$(docker container run -d -t -i ubuntu /bin/bash)`
docker attach $ID
```

- Using container name
```bash
docker attach my_container_name
```

- Using `exec` command
```bash
docker exec -it my_container_name bash
```

### Listing containers
To list the containers, either run the following command:
```
docker container ls [OPTIONS]
```
Or run the following legacy command:
```
docker ps [OPTIONS]
```

This commands gives following data

- The container ID
- The image from which it was created
- The command that was run after starting the container
- The details about when it was created
- The current status
- The ports that are exposed from the container
- The name of the container

To list both running and stopped containers, use the `-a` option.  
To return just the container IDs of all containers, use the `-aq` option.  
Look for help like this `docker container ls --help`.  

### Looking at the container logs

```bash
docker container logs [OPTIONS] CONTAINER
```

```bash
~ $ docker container run -d ubuntu /bin/bash -c "while [ true ]; do date; sleep 1; done"
78823c1ace9ff62682ef50963d310cc67eb0e620fda615e43361595fe1f64a43
~ $ 
~ $ docker logs strange_keller 
Fri Jan 28 20:53:05 UTC 2022
Fri Jan 28 20:53:06 UTC 2022
Fri Jan 28 20:53:07 UTC 2022
Fri Jan 28 20:53:08 UTC 2022
Fri Jan 28 20:53:09 UTC 2022
```

Docker will look at the container's specific log file from `/var/lib/docker/containers/<Container ID>/<Container ID>-json.log` and show the results.

### Stopping a container
To stop a container, run the following command:
```
docker container stop [OPTIONS] CONTAINER [CONTAINER...]
```
Or run the following legacy command:
```
docker stop [OPTIONS] CONTAINER [CONTAINER...]
```

This will move the container from a running state to a stop state by stopping the process running inside the container. A stopped container can be be started again, if needed.

- To stop a container after waiting for some time, use the `--time/-t` option.
- To stop all running containers, run the following command:
	```
	$ docker stop $(docker ps -q)
	```

### Removing a container
We can remove a container permanently, but before that, we have to stop the container or use the force option.

Use the following command:
```
$ docker container rm [OPTIONS] CONTAINER [CONTAINER]
```
Or run the following legacy command:
```
$ docker rm [OPTIONS] CONTAINER [CONTAINER]
```

### Removing all stopped containers
We can remove all stopped containers with a single command.
```bash
docker container prune [OPTIONS]
```
The Docker daemon will iterate through containers that are not running and will remove them.

### Setting the restart policy on a container
You can set the restart policy using the following syntax:
```
$ docker container run --restart=POLICY [OPTIONS] IMAGE[:TAG] [COMMAND] [ARG...]
```
Here is an example of using the preceding command:
```
docker container run --restart=always -d -i -t ubuntu /bin/bash
```
There are three restart policies to choose from:
- `no`: This does not start the container if it dies
- `on-failure`: This restarts the container if it fails with a nonzero exit code
- `always`: This always restarts the container without worrying about the return
code

You can also get an optional restart count with the on-failure policy as follows:
```
docker container run --restart=on-failure:3 -d -i -t ubuntu /bin/bash
```

### Getting privileged access inside a container
Linux divides privileges that are traditionally associated with superusers into distinct units, known as capabilities (run man capabilities on a Linux-based system), which can be
independently enabled and disabled.

Docker starts containers with limited capabilities. With privileged access inside the container, we allocate more capabilities to perform operations that are normally done by the root user.

Inside a container
```
/home # mount --bind /home/ /mnt/
mount: permission denied (are you root?)
```

To use privileged mode, use the following command:
```
docker container run --privileged [OPTIONS] IMAGE [COMMAND] [ARG...]
```
Now, let's try the preceding example with privileged access:
```
docker container run --privileged -i -t ubuntu /bin/bash
```
This provides almost all capabilities inside the container. This mode causes security risks as containers can get root-level access on the Docker host. two new flags called `--cap-add` and `--cap-del` have been added to give fine-grained control inside a container. For example, to prevent any `chown` inside the container, use the following command:
```bash
docker container run --cap-drop=CHOWN [OPTIONS] IMAGE [COMMAND][ARG...]
```

### Accessing the host device inside a container
we can give access of the host device to a container with the `--device` option, following the run command.

You can give access of a host device to the container using the following syntax:
```
docker container run --device=<Host Device>:<Container DeviceMapping>:<Permissions> [OPTIONS] IMAGE [COMMAND] [ARG...]
```
Here is an example of using the preceding command:
```
$ docker container run --device=/dev/sda:/dev/xvdc -i -t ubuntu /bin/bash
```

### Injecting a new process into a running container
You can inject a process inside a running container with the following command:
```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```
Let's start an ubuntu container and then inject bash into it:
```bash
ID=$(docker container run -d redis)
docker container exec -it $ID /bin/bash
```
The exec command enters the namespace of the container and starts the new process.

### Reading a container's metadata
To inspect a container, run the following command:
```bash
docker container inspect [OPTIONS] CONTAINER [CONTAINER...]
```
We'll start a container and then inspect it, like so:
```bash
ID=$(docker container run -d -i ubuntu /bin/bash)
docker container inspect $ID
```

The following command will give us an IP address for the container:
```bash
docker container inspect --format='{{.NetworkSettings.IPAddress}}' container_name
172.17.0.2
```

### Labeling and filtering containers
Labels attached to images also get applied to containers that are started using those images. We can also attach labels to containers while starting them. Having labeled an image or a container, the labels can later be used for filtering or selection purposes.

Run a normal container without label
```bash
~ $ docker run --name=redis_normal redis date
Sat Jan 29 10:55:28 UTC 2022
```

Run a container with label
```bash
~ $ docker run --name=redis_label --label com.example.container=redis-container redis date
Sat Jan 29 10:56:07 UTC 2022
```

Check both containers
```bash
~ $ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                      PORTS     NAMES
9821948a72d9   redis     "docker-entrypoint.s…"   2 seconds ago    Exited (0) 1 second ago               redis_label
bbe44961b6a1   redis     "docker-entrypoint.s…"   41 seconds ago   Exited (0) 40 seconds ago             redis_normal
```

Let's check the meta data of both the containers
```bash
~ $ docker container inspect --format '{{json .Config.Labels}}' redis_normal
{}
~ $ docker container inspect --format '{{json .Config.Labels}}' redis_label
{"com.example.container":"redis-container"}
```

Filter the containers based on label
```bash
~ $ docker container ls -a --filter label=com.example.container=redis-container
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS                     PORTS     NAMES
9821948a72d9   redis     "docker-entrypoint.s…"   3 minutes ago   Exited (0) 3 minutes ago             redis_label
```

### Reaping a zombie inside a container
On Linux (and all Unix-like) operating systems, when a process exits, all the resources associated with that process are released with the exception of its entry in the process table. This entry in the process table is kept until the parent process reads the entry to learn about the exit status of its child. This transient state of a process is called a zombie. As soon as the parent process reads the entry, the zombie process is removed from the process table, and
this is called reaping. If the parent process exits before the child process, the init process (PID 1) adopts the child process (PID 1) and it eventually reaps adopted child processes when they exit:

```
~ $ pstree
systemd─┬─ModemManager───2*[{ModemManager}]
        ├─NetworkManager───2*[{NetworkManager}]
        ├─accounts-daemon───2*[{accounts-daemon}]
        ├─acpid
        ├─apache2───2*[apache2───26*[{apache2}]]
```
`systemd` is a variant of the `init` system and is adopted by many Linux distributions. `systemd` is the `init` process or `PID 1`.

The Docker engine creates a new **PID namespace** for each docker container, thus the first process inside the container is mapped to **PID 1**. Docker is designed to run one process per container and usually the process
running inside the containers doesn't create child processes. However, if the process inside the container creates child processes, then an `init` system is needed to reap zombie processes. In this recipe, we will look at how to configure an `init` process for our container to reap zombie processes.

You can launch a container with the init process using the `--init` option of the Docker container run command, as shown in the following syntax:
```
docker container run --init [OPTIONS] IMAGE [COMMAND] [ARG...]
```

Without `--init` flag the process are invoked directly.
```bash
~ $ docker container run --rm alpine pstree -p
pstree(1)
~ $ docker container run --rm alpine sh -c "pstree -p"
pstree(1)
```

With `--init` flag the process are invoked are headed by init process.
```bash
~ $ docker container run --rm --init alpine pstree -p
docker-init(1)---pstree(8)
~ $ docker container run --rm --init alpine sh -c "pstree -p"
docker-init(1)---pstree(8)
```
