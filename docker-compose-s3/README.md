###Building Images in docker-compose files

- Images can be built using docker-compose files.
- It searches for the "image" in local cache and if not found builds using the "build" instructions.
- Explicit build can be done using - ```docker-compose build``` OR ```docker-compose up --build```
- The /html folder has a static HTML webpage which can run using ```docker-compose up --build``` and if the image is not found locally it builds from ```nginx.Dockerfile```.

```
  proxy:
    build:
      context: .
      dockerfile: nginx.Dockerfile
    image: nginx-custom
```