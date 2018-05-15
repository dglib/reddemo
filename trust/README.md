# Adding security to docker images and deployments
### Step 1, the Dockerfile
The step here is to set the user as 'nginx' and chown it's directories to that username.
```
RUN chown nginx /var/log/nginx && chown nginx /run/nginx

USER nginx
```
This means we when run the container it runs as uid 100 (nginx) and gid 101 (nginx). The user will then be run as this nonprivilaged user and you wont be able to su into root. Also, we'll listen on port 8080 inside the container.

```
DockerMac:trust $ docker run -d --rm -p 8080:8080 shaker242/dlogo:trust
0af342973636e4a508f64da868281c075de3c1b1ee1dc579062dde17710b13f5
DockerMac:trust $ docker ps
CONTAINER ID        IMAGE                   COMMAND             CREATED                  STATUS              PORTS                    NAMES
0af342973636        shaker242/dlogo:trust   "nginx"             Less than a second ago   Up 2 seconds        0.0.0.0:8080->8080/tcp   dreamy_zhukovsky
DockerMac:trust $ docker exec -ti 0af342973636 sh
/ $ id
uid=100(nginx) gid=101(nginx) groups=82(www-data),101(nginx)
/ $ su
su: must be suid to work properly
/ $
```

In kurbernetes, we'll futher extend security capabilities by dropping all Linux root level privilages for this nginx (uid=100) user. 

```
        securityContext:
          runAsNonRoot: true
          runAsUser: 100
          capabilities:
            drop:
              - all
```
If you so desired, in this use case you could disable logging or set a writeable log volume and then enable this service to be read-only, yet the volume read-write.

```
          readOnlyRootFilesystem: false
```
If you test this out; I've pinned it to a node in my lab, you'll want to remove or edit this out.

This was deployed under Docker EE 2.0/ UCP 3.0.0 via kubectl:
```
kubectl apply -n default -f trust-demo.yml
```