# Pre install Sonarqube
Sonarqube: versi sonarqube-9.9.1
Java: versi 17
Postgresql: versi 15

### Spesifikasi Server
Operating System: Ubuntu
CPU: 4 Core
Memory: 8 GB
Disk: 30 GB

### Java Installation
apt install openjdk-17-jdk openjdk-17-jre -y
sudo update-alternatives --config java

### Database Installation
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get -y install postgresql

### Tuning
vim /etc/sysctl.d/99-sonarqube.conf
vm.max_map_count=524288
fs.file-max=131072

/etc/security/limits.d/99-sonarqube.conf
sonarqube   -   nofile   131072
sonarqube   -   nproc    8192

grep SECCOMP /boot/config-$(uname -r)
CONFIG_HAVE_ARCH_SECCOMP_FILTER=y
CONFIG_SECCOMP_FILTER=y
CONFIG_SECCOMP=y

grep SECCOMP /boot/config-$(uname -r)
CONFIG_HAVE_ARCH_SECCOMP_FILTER=y
CONFIG_SECCOMP_FILTER=y
CONFIG_SECCOMP=y

sysctl --system
ulimit -n 131072
ulimit -u 8192