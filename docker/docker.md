### 1. Commands

docker ps 

docker ps -a // 

docker logs 

docker inspect

docker stop 

docker rm

docker container prune -f //  remove all containers without asking

docker run -d (-detach) // running in background

docker run alpine ping www.docker.com

docker run -d alpine ping www.docker.com

docker logs 6bdc

docker logs --since 10s 6bdc

docker stop 789b

docker rm 789b

docker ps -a

docker run -d -p 8085:80 nginx // listen on 8095 port and route it to 80
