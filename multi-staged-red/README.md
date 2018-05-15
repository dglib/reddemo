# Multi-Stage Docker Example
This primary applicaiton for illistration is the Marriott nodejs app, it's simple but shows how you can combine build jobs. The other build is an even more simple, but layered multi-staged app to further show how complexity can be resolved through this technique. 

The primary Docker file includes 2 parts, the first part...
```
FROM alpine:latest as base
RUN apk upgrade --update-cache --available
RUN apk add --update nodejs
WORKDIR /app
COPY package.json ./
```
pulls in alpine and tags the layer as "base" then rolls in updates and adds the nodejs package so all the required files for package.json/package-lock.json are there. Next it copies in the modified package.json with the express dependancy.

The second part...
```
FROM node:carbon
WORKDIR /usr/src/app
RUN npm i npm@latest -g 
RUN npm i express 
COPY --from=base /app/package*.json ./
COPY server.js .
EXPOSE 8080
CMD [ "npm", "start" ]
```
pulls down node:carbon, set the working directory, installs the latest npm and requried express package, the copies all the built /app/packages* files and puts them in it's /usr/src/app working directory. Finally it copies in our server.js file, exposes the port 8080 and triggers the command execution phase at runtime.

We also create a .dockerignore file to ignore all the extra modules and logs files from being in our final image.

To run this build we execute:
```
docker build -t red .
```
To execute the container:
``` 
docker run -it --rm -p 8080:8080 red

> docker_web_app@1.0.0 start /usr/src/app
> node server.js

Running on http://0.0.0.0:8080

```
Open a web browser and visit http://localhost:8080

You should be presented with 
```
Success! This site is running Marriott's nodejs app
```

The second test is under "Dockerfile.basic"

```
FROM alpine:3.7 as base
RUN apk upgrade --update-cache --available
RUN apk add --no-cache curl

FROM ubuntu:16.04 as hello
RUN apt-get update 
RUN echo hello > /hello

FROM alpine:latest as world
RUN echo world > /world

FROM base
COPY --from=hello /hello /hello
COPY --from=world /world /world
```
You'll build this and just exec into it...
```
docker build -t basic -f Dockerfile.basic .
```
Then run the container to search the directories:
```
$ docker run -ti basic
/ # ls /
bin    dev    etc    hello  home   lib    media  mnt    proc   root   run    sbin   srv    sys    tmp    usr    var    world
/ #
```

### More information on Multi-Stage builds and build options
https://docs.docker.com/develop/develop-images/multistage-build/

