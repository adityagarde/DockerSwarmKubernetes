# Docker | Swarm | Kubernetes

## 1. Docker Concepts and CLI Commands 

- Docker Containers are not mini-virtual machines - they are just a process.

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
- **PRUNE** -                                                    
  - to clean up images, volumes, build cache, and containers.    
                                                                 
 ```shell                                                        
$ docker system prune // cleans up everything - Nuking           
$ docker image prune // cleans up just dangling images           
$ docker image prune -a // removes all images you are not using. 
$ docker container prune                                         
$ docker system df // fetches all the data - disk usage          
```                                                              
```text                                                          
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE        
Images          29        0         3.195GB   3.195GB (100%)     
Containers      0         0         0B        0B                 
Local Volumes   37        0         2.864GB   2.864GB (100%)     
Build Cache     29        0         18.43MB   18.43MB            
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
 Name:  search
 Address: 172.23.0.2
 Name:  search
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

## 3. Docker Images and Dockerfiles

```text                                                                                                    
Image ====docker run=====> Running Container ---exit---> Stopped Container ====docker commit====> New Image
```                                                                                                        
- App binaries and dependencies.
- Metadata about the image and how to run the image.
- Images does not have complete OS or kernel modules like drivers.
- The host provides the OS.
- There are just enough binaries required to execute the instructions.

- **Images Layers**
    - Union file system format
    - Images are bundled to form the final image
        - Ex - Debian + Apt install + Environment changes + mySQL ===> Final Image which is a mixture of multiple images.
    - Individual Images are cached and thus saves time and space.
    - Unique SHA to identify the exact images it needs. SHA match between Dockerhub and local cache.
    - Only one copy of individual images is stored.

- History command shows layers of images / task building up the image.
- If 2 containers run dependent on a comman image - only differentiating factor between them will be what has actually happened on the separate containers.
```shell
$ docker history <IMAGE:TAG>
$ docker history nginx:latest
```
- Copy On Write (COW) - Changing base files etc by a running container.
    - In such case - the changed file is copied from the image and stored in the container layer.

- `docker inspect < IMAGE >`
    - returns the JSON metadata about the image.

- Images are made up of file system changes and metadata.
- Each layer is uniquely identified and only stored once on a host.
- This saves storage space on host and time on push / pull.
- A container is just a single read / write on top of an image.

- **Image Tags** -
    - `docker image tag` or `docker tag`
    - assigns one or more tags to an image
    - docker images don't have a name - thus we uniquely identify them by `< user >/< repo >:< tag >` OR the image_id(SHA).
    - REPOSITORY in the docker images output => <user>/<repo>
        - Official repositories - don't specify the <user> tag
        - they live at the "root namespace", so they don't need account name in the front of the repo.
      
    - One image can have multiple tags - but in essence they are the same image.
    - They are not stored multiple times - just one image is cached locally on the host.

    - `docker image tag <image> <new_tag>`
    - `docker image push <image_id>` or `docker push <image_id>`
        - after `docker login`
        - `.docker/config.json` - adds the authentication key here
    - public / private repositories on dockerhub

**Dockerfiles - Building Images**

- `docker build -f <dockerfile name>`
- Ordering matters in Dockerfiles. These are key commands -
    1. `FROM` - minimal installation image (required)
    2. `ENV` - Environment variables - To inject properties as Key value pairs. (optional)
    3. `RUN` - set of commands to run inside the container - updates, installation, running shell scripts, update any internal files. 
        - Writing logs to stdout so that docker can consume them from there. 
    4. `EXPOSE` - Expose said ports on docker virtual network. (optional)
    5. `CMD` - Run this command when container is launched or restarted. (required)

```shell
$ docker image build -t <myImageName>
$ docker image build -t customnginx .  // . to specify that build in this repository
```
- `-f` (alias for `--file`) is used to denote Dockerfile's name. By default, it is `Docerfile` only and thus `-f` is not required in default case.
- `-t` is used to tag images.

- Ordering in Dockerfile -
    - When a line on the code changes in the dockerfile, the remaining steps are executed.
        - So it makes sense to keep things in the top of the docker file that change less and keep things which change the most in the bottom.

- Extending Official Images
    - When we are using an Image in the FROM statement - we inherit everything (FORM, EXPOSE etc.) from the Dockerfile.

```shell
$ docker image build -t new_nginx . 
$ docker image tag new_nginx:latest adityagarde/new_nginx
```
Example -
```shell
$ docker build -t testnode .
$ docker container run --rm -p 80:3000 testnode:latest
$ docker tag testnode adityagarde/testing-node
$ docker push adityagarde/testing-node
$ docker image rm adityagarde/testing-node
$ docker container run --rm -p 80:3000 adityagarde/testing-node:latest
```

#### Some Dockerfile Examples can be checked here -

1. [DockerFile Example 1](https://github.com/adityagarde/DockerSwarmKubernetes/tree/main/dockerfile-q0)
2. [DockerFile Example 2](https://github.com/adityagarde/DockerSwarmKubernetes/tree/main/dockerfile-q1)

## 4. Volumes - Container Lifetime and Persistent Storage

- Containers are meant to be Immutable and Ephemeral Containers.
    - i.e. unchanging, disposable, temporary.
- Immutable infrastructure -
    - If there are any changes we don't change existing containers - we only re-deploy containers from images.
- Trade-off - What about the unique data you that the containers might have generated (DB, files, key-value pairs etc.)
- Separation of Concerns - Ideally docker should not mix our unique data with the application files.

- Persistent Data - problem with the unique data in the docker container.
- Handled in two ways -
    - **Volumes** - Make special location outside of the container filesystem UFS to store unique data.
    - **Bind Mounts** - Link container path to host path.

**1. Persistent Data : Volumes**

- check Dockerfile
    VOLUME /data - This is used to explicitly put data in this location and this is not removed when the container is removed.
- Volumes requires separate step to remove it.
- Volume = A running container getting its own unique location on the host to store the data and in the background it is mapped to the container.

Ex. -
- `docker inspect mysql`
```text
"Image": "sha256:a717583fdcec69e6c839d2647da972980b6dcce80cad9bcdce9c760c10e222ba", 
"Volumes": {                                                                        
    "/var/lib/mysql": {}                                                            
},                                                                                  
"WorkingDir": "",                                                                   
"Entrypoint": [                                                                     
    "docker-entrypoint.sh"                                                          
],
```
```shell
$ docker run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True mysql
```
- After running the container -
```text
"Mounts": [                                                                                                         
    {                                                                                                               
        "Type": "volume",                                                                                           
        "Name": "c2bd8d01fe315344761178b7019c1c4ae0db8da2721029617db20384dc6052a7",                                 
        "Source": "/var/lib/docker/volumes/c2bd8d01fe315344761178b7019c1c4ae0db8da2721029617db20384dc6052a7/_data", 
        "Destination": "/var/lib/mysql",                                                                            
        "Driver": "local",                                                                                          
        "Mode": "",                                                                                                 
        "RW": true,                                                                                                 
        "Propagation": ""                                                                                           
    }                                                                                                               
],                                                                                                                  
```
- **Source** - Container is writing to this location
  & **Destination** - The data is actually being stored here.
- On Mac and Windows - the data is actually in a Linux VM - so this path cannot be accessed directly.

- Named Volumes - easy to identify the volumes.
```shell
$ docker run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v /var/lib/mysql mysql
```
- This does the same thing as what our VOLUME command does in Dockerfile.
```shell
$ docker run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql-db:/var/lib/mysql mysql
```
- This is named volume - the tag mysql-db will appear in the docker volume ls command.
    - This tag now can be used when restarting the container or starting a new container.

- docker volume create -
    - Required to do this before "docker run" to use custom drivers and labels.

**2. Persistent Data : Bind Mounting**

- Maps / attaches a host file or directory to a container file or directory.
- Basically just two locations pointing to the same file(s).
- This skips UFS as well - i.e. it is not wiped when container is removed.
- Can't use in Dockerfile, must be at container run time.
```shell
$ docker run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v /Users/adityagarde/localdump:/var/lib/mysql mysql
$ docker container run -d --name nginxtest -p 80:80 -v $(pwd):/usr/share/nginx/html nginx
```
Example - Named Volumes - Database upgrade with containers.

```shell
$ docker container run -d --name postgresql -v postgresql:/var/lib/postgresql/data postgresql:9.6.1
$ docker logs -f postgresql
$ docker container run -d --name postgresql2 -v postgresql:/var/lib/postgresql/data postgresql:9.6.1
```

## 5. Docker Compose

- Configure relationships between containers
- Save our docker container settings in easy-to-read yaml files
- Good for dev, testing and local setup, not for production.
 
- The following CLI command and yaml configuration are same.
```shell
$ docker run -p 80:4000 -v $(pwd):/site bretfisher/jekyll-serve
```
```yaml
services:
  jekyll:
    image: bretfisher/jekyll-serve
    volumes:
      - .:/site    
    ports:    
      - '80:4000'
```

- `docker-compose up` - setup volumes / networks and start all containers
- `docker-compose down` - stop all containers and remove containers / volume / network etc.

- `docker-compose ps`
- `docker-compose top`

- `docker-compose down -v` => To removes the associated volumes as well.

- Adding Image Build to Compose Files
    - Compose can build your custom images at runtime.
    - Will build images with docker-compose up if not found in the cache.
    - It will not build the images everytime, will build only if it does not find it locally.
    - docker-compose build OR docker-compose up --build - to rebuild images
```yaml
services:
  proxy:
    build:
      context: .
      dockerfile: nginx.Dockerfile
    ports:
      - '80:80'
```

- The `ports:` key publishes the particular service on whatever port you specify, and is the docker run equivalent to the -p flag.
- The `ports:` key in a compose file does NOT do the same thing as the EXPOSE stanza in a Dockerfile.

#### Some Docker Compose Examples can be checked here - 

1. [Docker Compose Example 1](https://github.com/adityagarde/DockerSwarmKubernetes/tree/main/docker-compose-q1)
2. [Docker Compose Example 2](https://github.com/adityagarde/DockerSwarmKubernetes/tree/main/docker-compose-q2)
3. [Docker Compose Example 3](https://github.com/adityagarde/DockerSwarmKubernetes/tree/main/docker-compose-s3)
4. [Docker Compose Example 4](https://github.com/adityagarde/currency-conversion-microservice/blob/main/docker-compose.yml)