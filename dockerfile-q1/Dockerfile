# This dir contains a Node.js app, you need to get it running in a container
# No modifications to the app should be necessary, only edit the Dockerfile.


# Instructions from the app developer
# - you should use the 'node' official image, with the alpine 6.x branch (node:6-alpine)
  #  yes this is a 2-year old image of node, but all official images are always
  #  available on Docker Hub forever, to ensure even old apps still work.
  #  It is common to still need to deploy old app versions, even years later.
FROM node:6-alpine
# - this app listens on port 3000, but the container should launch on port 80
  #  so it will respond to http://localhost:80 on your computer
EXPOSE 3000
# - then it should use alpine package manager to install tini: 'apk add --update tini'
RUN apk add --update tini
# - then it should create directory /usr/src/app for app files with 'mkdir -p /usr/src/app'
RUN mkdir -p /usr/src/app
# - Node uses a "package manager", so it needs to copy in package.json file
WORKDIR /usr/src/app
COPY package.json package.json
# - then it needs to run 'npm install' to install dependencies from that file
# - to keep it clean and small, run 'npm cache clean --force' after above
RUN npm install && npm cache clean --force
# - then it needs to copy in all files from current directory
COPY . .
# - then it needs to start container with command '/sbin/tini -- node ./bin/www'
CMD ["tini", "--", "node", "./bin/www"]
# - in the end you should be using FROM, RUN, WORKDIR, COPY, EXPOSE, and CMD commands