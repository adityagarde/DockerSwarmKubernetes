# Docker | Swarm | Kubernetes

## 1. Docker Concepts and CLI Commands 

- Docker Containers are not mini-virtual machines - they are just a process.

```text
Image ====docker run=====> Running Container ---exit---> Stopped Container ====docker commit====> New Image
```

```shell
$ docker container run --publish 80:80 --detach nginx // or just -d
$ docker container stop <container_name or id> // stop a running container
$ docker container ls -a // list all (-a) containers
$ docker container logs <container_name> // check logs
$ docker container rm <container_name> // remove a container
```

- Docker container commands like rm, stop etc can accept multiple values if space separated.
- rm -f = for force = removes even if the container is running
- docker container ls (older version is docker ps) when used with -a flag returns all containers, running or stopped.

- `--publish HOST:CONTAINER` OR just `-p HOST:CONTAINER` used to publish ports in docker container.
- `name` is used to explicitly name containers.
- `--env` to pass container's environment variables as key value pairs.

```shell
$ docker container run -p 80:80 -d --name proxyserver nginx
$ docker container run -p 8080:80 -d --name httpdserver httpd
$ docker container run -p 3306:3306 -d --name mysqlserver --env MYSQL_RANDOM_ROOT_PASSWORD=yes mysql
```

- A few commands very handy to get started as well as troubleshooting.
```shell
$ docker container top <container> // Display the running processes.
$ docker container inspect <container> // Display detailed information.
$ docker container stats <container> // Display a live stream of container resource usage statistics
```

- `-it` almost always used with docker run.
    - `-t` - allocates a pseudo tty  
    - `-i` - Interactive
- `exec` to run additional commands in existing running container.

```shell
$ docker container run -it --name proxy nginx bash
$ docker container run -it --name myUbuntu ubuntu_1 bash
$ docker container start -ai myUbuntu
```

## 2. Docker Networks

- Each container connected to a private virtual network "bridge".
- Each Virtual network routes through NAT firewall on host IP.
- All containers on a virtual network can talk to each other without -p  .
- Best practice is to create a virtual network for each app.
- Skip virtual networks and use host IP (--net=none)

```shell
$ docker container inspect --format '{{.NetworkSettings.IPAddress}}' proxyserver
$ docker container port <container>
```

- Docker Network commands -
    - Show networks - `docker network ls`
    - Inspect a network - `docker network inspect`
    - Create a network - `docker network create --driver`
    - Attach a network to a container - `docker network connect`
    - Detach network from container - `docker network disconnect`
 
- Network options -  
	- `--network bridge`
		- Default docker virtual network which is NAT'ed behind the HOST IP.
	- `--network host`
		- It gains performace by skipping virtual networks but sacrifices security of container
	- `--network none`
		- Removes eth0 and only leaves you with localhost interface in container.
    
```shell
$ docker network inspect bridge

$ docker network create my_app_net
$ docker network ls
```
- The CLI output.
```text
NETWORK ID     NAME             DRIVER    SCOPE
747eea8c4dd6   my_app_net       bridge    local
```

- by default network driver is bridge
- network driver - built-in or 3rd party extensions that give you virtual network features.

```shell
$ docker container run -d --name new_nginx --network my_app_net nginx
$ docker network connect <container_id_1> <container_id_2>
``` 

- Docker Network - Default Security
    - Create your apps so frontend / backend sit on the same Docker network
    - Their inter-communication never leaves host
    - All externally exposed ports closed by default
    - And ports exposed only with -p are open.

- Docker Networks - DNS
    - How containers find each other.
    - Static IPs and using IPs for talking to containers is an anti-pattern. Try and avoid it.
    - Containers keep on shrinking and growing, re-starting, going down when not needed etc.
    - Thus, IPs keep changing as well.

- Docker DNS
    - Docker daemon has a built-in DNS Server that containers use by default.
- DNS Default Names
	- Docker defaults the hostname to the container's name, but you can also set aliases.

- This works because both the containers are in the same network.
```shell
$ docker container run -d --name my_nginx --network my_app_net nginx:alpine
$ docker container exec -it my_nginx ping new_nginx
```

- default bridge network does not have built in DNS resolution.
    - we can use --link to overcome this.

- For intercommunication between containers - IPs should not be relied on - DNS / custom networks are suggested.

Example 1 -
   1. Ubuntu
```shell
$ docker contaienr run --rm -it --name myUbuntu ubuntu_1 bash
$ apt-get update && apt-get install -y curl
```
   2. Centos
```shell
$ docker container run --rm -it --name myCentos centos:7 bash
$ yum update curl
```

Example 2 - DNS RR Test
```shell
$ docker network create my_network
$ docker container run -d --net my_network --net-alias search elasticsearch:2
$ docker container run -d --net my_network --net-alias search elasticsearch:2
```
- CLI Output
```text
CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS          PORTS                   NAMES            
661c5052c462   elasticsearch:2   "/docker-entrypoint.…"   11 seconds ago   Up 8 seconds    9200/tcp, 9300/tcp      agitated_hamilton
801fffb00534   elasticsearch:2   "/docker-entrypoint.…"   2 minutes ago    Up 2 minutes    9200/tcp, 9300/tcp      upbeat_pike      
```

```shell
$ docker container run --rm --net my_network alpine nslookup search
```
```text
 Server:    127.0.0.11   
 Address:   127.0.0.11:53
                    
 Non-authoritative answer:
 Name:	search
 Address: 172.23.0.2
 Name:	search
 Address: 172.23.0.3
```
```shell
$ docker container run --rm --net my_network centos curl -s search:9200
```
- We get Names in Round Robin Fashion.

```json
{
   "name":"Blacklash",
   "cluster_name":"elasticsearch",
   "cluster_uuid":"CTrvPae0QzKcoAA4pKMmxQ",
   "version":{
      "number":"2.4.6",
      "build_hash":"5376dca9f70f3abef96a77f4bb22720ace8240fd",
      "build_timestamp":"2017-07-18T12:17:44Z",
      "build_snapshot":false,
      "lucene_version":"5.5.4"
   },
   "tagline":"You Know, for Search"
}
```
```json
{                                                             
   "name":"Grizzly",                                          
   "cluster_name":"elasticsearch",                            
   "cluster_uuid":"IwCPsDSKRGOdlGYFcTre0w",                   
   "version":{                                                
      "number":"2.4.6",                                       
      "build_hash":"5376dca9f70f3abef96a77f4bb22720ace8240fd",
      "build_timestamp":"2017-07-18T12:17:44Z",               
      "build_snapshot":false,                                 
      "lucene_version":"5.5.4"                                
   },                                                         
   "tagline":"You Know, for Search"                           
}                                                             
```
```shell
$ docker container rm -f 661c5052c462 upbeat_pike my_nginx f7df0fcdafa6 b170ca67a2a8
```

## 3. Docker Images

- App binaries and dependencies.
- Metadata about the image and how to run the image.
- Images does not have complete OS or kernel modules like drivers.
- The host provides the OS.
- There are just enough binaries required to execute the instructions.

<br />