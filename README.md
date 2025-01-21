# Assignement 1: Manage multiple containers

docker container run --publish 80:80 --detach --name nginx nginx
docker container run --publish 8080:80 --detach --name httpd httpd
docker container run --publish 3306:3306 --detach --name mysql -e MYSQL_RANDOM_ROOT_PASSWORD=yes mysql

## Commands to check

docker container ls
docker container ps --all 
docker container logs mysql --tail all
docker container rm {{ container_name }

Start up a NEW image and run the `bash` command shell :
docker container run --tty --interactive --name proxy nginx bash

Get into an EXISTING running image:
docker container exec -it mysql bash

