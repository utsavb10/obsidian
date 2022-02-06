# Docker Compose

- Why: configure realtionship between containers
- Why: save our docker container run settings in easy-to -read file
- Why: create one-liner developer enivronment setups
- Comprised of 2 separate but related things:
	1. YAML file that describes our solution options for:
	<ul>
	<li> containers </li>
	<li> networks </li>
	<li> volumes </li>
	</ul>
	2. CLI tool docker-compose used for local dev/test automation with those YAML files


## docker-compose.yml
- Compose YAML format has its own versions: 1,2,2.1,3,.31
- YAML file can be used with docker-compose command for local docker automation or..
- With **docker** direclty used in production with **Swarm**
- ```docker-compose --help```
- docker-compose.yml is the default file name, but any can be used with docker-compose -f 

```
version: '3.1' # if no version is specified then v1 is assumed. Recommend v2 minimum  
  
services:  # containers. same as docker run  
 servicename: # a friendly name. this is also DNS name inside network  
 image: # Optional if you use build:  
 command: # Optional, replace the default CMD specified by the image  
 environment: # Optional, same as -e in docker run  
 volumes: # Optional, same as -v in docker run  
 servicename2:  
  
volumes: # Optional, same as docker volume create  
  
networks: # Optional, same as docker network create
```

## docker-compose cli
- download separately. Use for non-production uses.
- two most common commands:
  - ```docker-compose up```  setup volumes/networks and start all containers
  - ```docker-compose down``` stop all containers and remove volumes and networks
- 