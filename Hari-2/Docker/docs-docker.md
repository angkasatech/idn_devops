# Docker

## Install Docker
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-pluginsudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

## Docker command
docker --help

docker image ls
docker images

docker container ls
docker ps
docker ps -a

docker info

## Docker practices
docker pull ubuntu

docker run -it ubuntu

cat /etc/os-release
free -m
hostname -i
exit

docker rename old-name-ct new-name-ct

docker start ct-name

docker exec -it ct-name cat /etc/os-release
docker exec -it ct-name cat /bin/bash

apt update
apt install nginx
apt list --installed | grep nginx
nginx -g "daemon off;"

docker commit -p ct-ubuntu ubuntu-nginx

docker run -dit -p 8001:80 --name ct-nginx ubuntu-nginx
docker exec -it ct-nginx nginx -g "daemon off;"

docker save -o output-file ct-name
docker load -i file

## Dockerfile
vim Dockerfile
docker build -t image-name path

## Docker Registry
docker login
docker tag image-name repository/image-name
docker push repository/image-name

## Docker Volume
docker volume create volume-name
docker run -d -p host:container -v volume-name:destination image-name

## Docker Network
docker network create --subnet 192.168.100.1/24 net-A
docker run -d -p host:container -v volume-name:destination --network net-A image-name

## Docker Logging and Troubleshooting
docker inspect ct-name
docker logs ct-name
docker top ct-ubuntu
docker stats
docker image history image-name
docker system df

## Docker Dashboard
docker volume create portainer_data
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest

https://localhost:9443

## Docker Compose
vim docker-compose.yml
docker compose build
docker compose up -d
docker compose down