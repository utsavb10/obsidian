### Volumes
[Storage Overview](https://docs.docker.com/storage/)
#### Container Lifetime & Persistent Data
- Containers are **usually** immutable and ephemeral
- "immutable infrastructure": only re-deploy containers, never change
- This is the ideal scneario, but what about databases, unique data ?
- Docker gives us feature to esure these "separation of concerns"
- This is known as **persistent data**
- Two Ways: Volumes and Bind Mounts
- **Volumes**: make special location outside of container UFS
- **Bind Mounts**: link container paths to host path
	[12 Factor App](https://12factor.net/)

#### Permanent Data Volumes
Use Volume argument in Dockerfile
```
VOLUME /varlib/mysql
```

pull a mysql image from dockerHub to see a volume used as default
```
docker conatiner inspect <image>
docker volume ls
docker volume inspect <volumeID>
```

Volumes are outside the lifecycle of containers, they don't get destroyed when containers are destroyed

**Named vomules** are just an easy way to assign vols to containers

```
docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True            -v /var/lib/mysql mysql
```
Above `-v` option is equivalent to writing VOLUME in dockerfile

```
docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True            -v named-volume:/var/lib/mysql mysql
```

```
docker volume create --help this command can be used to create volumes independently
```

#### Persistent Data Bind Mounting
- Maps s host file or directory to a container file or directory
- Basically just two locations pointing to the same file(s)
- Again, skips UFS, and host files overwrite any in container
- Bind mounts are host specific so they can't be used in Dockerfile, must be used during `container run`
-  ... `run -v /Users/user/stuff:/path/container` (mac/linux)
-  ... `run -v //c/Users/user/stuff:/path/conatiner` (windows)

sample -> 
```
docker conatiner run -d --name nginx -p 80:80 
-v $(pwd):/user/share/nginx/html nginx
```

Now after spinning the container we can put new web apps in $pwd and access the app from the running container
