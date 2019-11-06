# Docker

## Docker-Engine:: Client Server Application 

* dockerd -- Server  
* Docker CLI - Client

dockerd can be accessed by CLI and API

<pre>[abhi@Dock docker]$ docker run hello-world
Unable to find image &apos;hello-world:latest&apos; locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete 
Digest: sha256:c3b4ada4687bbaa170745b3e4dd8ac3f194ca95b2d0518b417fb47e5879d9b5f
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the &quot;hello-world&quot; image from the Docker Hub.
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
</pre>

docker daemon is installed on the Docker_HOST and it runs containers.
Docker_HOST is administered through the docker client (CLI), which could be on the same host or a remote machine. 

Docker hosts can be 10's of thousands of numbers and they are administered by the UCP - Universal Control Plane.

## Namespaces and Cgroups 
Docker uses NameSpaces and Cgroups to provide isolation and limit resources used by the containers---- 

Docker uses 
1. User
2. Network
3. Process
4. Mount
5. IPC
NameSpaces to provide isolation inside the containers running on the host OS

Cgroups are for controlling resource utilization - in particular, CPU and memory

Docker uses Cgroups to limit:
* CPU limits
* CPU reservations
* Memory limits
* Memory reservations


Docker Control groups - docker run -c (for CPU limitations) and -m (for Memory limitations)

systemd is controlled by systemctl command  
`docker images | awk '{print $3}' | grep -v IMAGE`        #Prints all image ids  
`docker ps | awk '{print $1}' | grep -v CONTAINER`        #Prints running container ids




# Docker  Swarm
https://docs.docker.com/engine/swarm/swarm-mode/

 Initialize Docker Swarm using - 

`docker swarm init`  

 * To initialize docker swarm with a different overlay address pool -  

`$ docker swarm init --default-addr-pool <IP range in CIDR> [--default-addr-pool <IP range in CIDR> --default-addr-pool-mask-length <CIDR value>]  
`

 This initializes swarm with the node as a Manager node: 

```
[abhi@manager ~]$ docker swarm init
Swarm initialized: current node (2vr72nfkoyeotsngj80porph8) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-48316sjc047oe4i7iulwftt9j4oygmrcjm27yzfijak54eqqx1-1kmsl8k4peck67vuwmqb59cwl 192.168.122.39:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

* On the Worker Node:

```
[abhi@worker ~]$ docker swarm join --token SWMTKN-1-48316sjc047oe4i7iulwftt9j4oygmrcjm27yzfijak54eqqx1-1kmsl8k4peck67vuwmqb59cwl 192.168.122.39:2377
This node joined a swarm as a worker.
```

 To retrieve the join command including the join token for worker nodes, run:

`docker swarm join-token worker`

[abhi@manager ~]$ docker swarm join-token worker  
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-48316sjc047oe4i7iulwftt9j4oygmrcjm27yzfijak54eqqx1-1kmsl8k4peck67vuwmqb59cwl 192.168.122.39:2377

To view the join command and token for manager nodes, run:

`docker swarm join-token manager`
```[abhi@manager ~]$ docker swarm join-token manager
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-48316sjc047oe4i7iulwftt9j4oygmrcjm27yzfijak54eqqx1-9qn4l8gnjd9617q54b6mc1wpv 192.168.122.39:2377
```

`docker node ls`    

Lists the nodes of docker swarm:

```
[abhi@manager ~]$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
jl9acswyh10ad7uqf0numc4bn *   manager             Ready               Active              Leader              19.03.4
ysmvp8yv1nrzuichfhcpbb7t9     worker              Ready               Active                                  19.03.4
yd3idenuf18s3xcd88wc4etap     worker2             Ready               Active                                  19.03.4


[abhi@worker ~]$ docker node ls
Error response from daemon: This node is not a swarm manager. Worker nodes can't be used to view or modify cluster state. Please run this command on a manager node or promote the current node to a manager.

```

Swarm management commands like docker node ls only work on manager nodes.

# Deploy a service to the swarm #

After you create a swarm, you can deploy a service to the swarm

`docker service create --replicas 1 --name helloworld alpine ping docker.com`

* The `docker service create` command creates the service.
* The `--name` flag names the service helloworld.
* The `--replicas` flag specifies the desired state of 1 running instance.
* The arguments `alpine ping docker.com` define the service as an Alpine Linux container that executes the command ping docker.com.

```
[abhi@manager ~]$ docker service create --replicas 1 --name helloworld alpine ping docker.com
zd87w9skys1iv7p5i4n0krvyb
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>] 
verify: Service converged 
[abhi@manager ~]$ 
```

This deployed an Alpine container on one of the worker nodes:

```
[abhi@worker2 ~]$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
8fec43904753        alpine:latest       "ping docker.com"   20 seconds ago      Up 19 seconds                           helloworld.1.qz2fdbqihvs43bhf51u72844v
[abhi@worker2 ~]$ 
```

`docker service ls` lists all running services:

```
[abhi@manager ~]$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
zd87w9skys1i        helloworld          replicated          1/1                 alpine:latest       
[abhi@manager ~]$ 
```

`docker service rm <service id>` removes the service
`docker service inspect <service id> --pretty ` provides details about the service and if JSON output is needed, --pretty flag is not required.

```
[abhi@manager ~]$ docker service inspect helloworld --pretty 

ID:		x2qcujpjo8jp69fnbu7e48156
Name:		helloworld
Service Mode:	Replicated
 Replicas:	3
Placement:
UpdateConfig:
 Parallelism:	1
 On failure:	pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Update order:      stop-first
RollbackConfig:
 Parallelism:	1
 On failure:	pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Rollback order:    stop-first
ContainerSpec:
 Image:		alpine:latest@sha256:c19173c5ada610a5989151111163d28a67368362762534d8a8121ce95cf2bd5a
 Args:		ping docker.com 
 Init:		false
Resources:
Endpoint Mode:	vip
```


`docker service ps <service id>` provides with the node details where the container is running:

```
[abhi@manager ~]$ 
[abhi@manager ~]$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
x2qcujpjo8jp        helloworld          replicated          1/1                 alpine:latest       
[abhi@manager ~]$ 
[abhi@manager ~]$ docker service ps helloworld
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
szjxdbo59ao9        helloworld.1        alpine:latest       manager             Running             Running 3 minutes ago                       
[abhi@manager ~]$ 
[abhi@manager ~]$ 
[abhi@manager ~]$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
0058b93eb425        alpine:latest       "ping docker.com"   3 minutes ago       Up 3 minutes                            helloworld.1.szjxdbo59ao9v5w4zt5nmh5da
[abhi@manager ~]$ 

```


* To scale up/scale down the running containers for a running service :

`docker service scale <SERVICE-ID>=<NUMBER-OF-TASKS>`

```
[abhi@manager ~]$ docker service scale helloworld=5
helloworld scaled to 5
overall progress: 5 out of 5 tasks 
1/5: running   [==================================================>] 
2/5: running   [==================================================>] 
3/5: running   [==================================================>] 
4/5: running   [==================================================>] 
5/5: running   [==================================================>] 
verify: Service converged 
[abhi@manager ~]$ 
[abhi@manager ~]$ 
[abhi@manager ~]$ docker service ps helloworld
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
szjxdbo59ao9        helloworld.1        alpine:latest       manager             Running             Running 5 minutes ago                       
6d39nelkboh9        helloworld.2        alpine:latest       worker2             Running             Running 9 seconds ago                       
waxsgiiiyqef        helloworld.3        alpine:latest       worker2             Running             Running 9 seconds ago                       
ah1c7vrwxu9c        helloworld.4        alpine:latest       manager             Running             Running 9 seconds ago                       
q4kuibkiyubl        helloworld.5        alpine:latest       worker              Running             Running 9 seconds ago                       
[abhi@manager ~]$ 
```

