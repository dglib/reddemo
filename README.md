# REDDEMO
This is a brief nginx demo to illistrate how to mount containers to NFS for shared persistance.

To build your images "docker build -t repo/image:tag -f redweb/Dockerfile ."

Use this image in docker-compose.yml or mine if you'd like shaker242/dlogo:red1

If you're not going to use/test HRM you can ignore the labels for lb: these are for Docker 2.0 (UCP 3.0.0+)
```
      labels:
        com.docker.lb.hosts: marriott.apps.docker.ee
        com.docker.lb.network: ucp-hrm
        com.docker.lb.port: "80"
```
If you'd like to convert HRM to support Docker UCP version 2.2.x, change the labels to this:
```
      labels:
          com.docker.ucp.mesh.http.80-81: "external_route=http://marriott.apps.docker.ee,internal_port=80","external_route=http://marriott.apps.docker.ee,internal_port=80"
```

In the docker-compose file we mount the nfs volume and specify it as read-only (ro)
```
# NFS mount 
volumes:
  redvol1:
    driver: local
    driver_opts:
      type: nfs
      o: addr=172.16.1.5,ro
      device: ":/nfs/marriott"
```
