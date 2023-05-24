# Install Docker

### Spesifikasi Server
Operating System: Ubuntu
CPU: 4 Core
Memory: 8 GB
Disk: 30 GB

### Setup Repository
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

sudo apt-get update
sudo apt-get install ca-certificates curl gnupg

### Docker Installation
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin