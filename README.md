# REDDEMO
This is a brief nginx demo to illistrate how to mount containers to NFS for shared persistance.

## NFS Share Prep:
The NFS server will have this mapping in the /etc/exports file:
```
/nfs/marriott *(rw,sync,no_subtree_check,all_squash,anonuid=65534,anongid=65534)
```
* Note, I'm using Ubuntu, under RHEL or CentOS you'll map the UID/GID for nfsnobody.

The above share's the mount point /nfs/marriott to the world, makes it writable, commits the io before acknowledgment, skips host file checks, changes "all" users that write to the NFS filesystem to user nobody:nogroup (nfsnobody:nfsnogroup).

* Note, I am making the NFS share writable since I'll be coping in the files to be shared among the containers separtly; we'll mount them into the container as read-only (see the docker-compose.yml file).

## NFS Export Prep
Now I'll chown the /nfs/marriott directory and all the files contained there in to nobody:nogroup and list them all out for you. I elected for chown 775; however, 755 would have worked as well since the share will be read-only.
```
shaker@nfs1:/$ ls -ld /nfs/marriott/
drwxrwxr-x 3 nobody nogroup 4096 May  3 07:54 /nfs/marriott/
```
In this share I've copied in an easy index.html file and a couple logos to display
```
shaker@nfs1:/$ ls -l /nfs/marriott/
total 8
drwxr-xr-x 2 nobody nogroup 4096 May  3 07:54 img
-rw-r--r-- 1 nobody nogroup 1137 May  3 07:54 index.html

shaker@nfs1:/$ exportfs -ra
shaker@nfs1:/$ exportfs
/nfs/marriott   <world>
```

Build your test container or use mine as noted in the docker-compose.yml (image: shaker242/dlogo:red1).
```
docker build -t repo/image:tag -f redweb/Dockerfile .
```

Now launch the server... 

I loaded my admin client bundle and ran a docker stack deploy... you can also just paste the docker-compose.yml file into your UCP > Stack > Create... 

DockerMac:reddemo $ docker stack deploy -c docker-compose.yml reddemo
Creating service reddemo_demo

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
        com.docker.ucp.mesh.http.80-11: "external_route=http://marriott.apps.docker.ee,internal_port=80"
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
