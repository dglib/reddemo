FROM alpine:latest as base
RUN apk upgrade --update-cache --available
RUN apk add --update nodejs
WORKDIR /app
COPY package.json ./

FROM node:carbon
WORKDIR /usr/src/app
RUN npm i npm@latest -g 
RUN npm i express 
COPY --from=base /app/package*.json ./
COPY server.js .
EXPOSE 8080
CMD [ "npm", "start" ]
