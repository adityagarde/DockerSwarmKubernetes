#### Dockerfile 101

- Containerizing a simple Node.js application
- ```docker image build -t adityagarde/myNodeApp:0.0.1 .```
- ```docker container run --rm -p 8080:8080 adityagarde/myNodeApp:0.0.1```