FROM alpine:3.7 

LABEL maintainer="shaker242@gmail.com"

RUN apk add --no-cache nginx && mkdir -p /run/nginx

COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80

CMD ["nginx"] 
