# Install Sonarqube

### Setup Database
su - postgres

createuser sonar
psql
ALTER USER sonar WITH ENCRYPTED password 'sonar';
CREATE DATABASE sonarqube OWNER sonar;
grant all privileges on DATABASE sonarqube to sonar;

\l
\du
\q

exit

### Setup User
groupadd sonar
useradd -s /bin/bash -d /opt/sonarqube -g sonar sonar

### Download Sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.1.69595.zip
unzip sonarqube-9.9.1.69595.zip

mv sonarqube-9.9.1.69595 /opt/sonarqube
chown sonar:sonar /opt/sonarqube/ -R

### Config Sonarqube
vim /opt/sonarqube/conf/sonar.properties
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube


vim /opt/sonarqube/bin/linux-x86-64/sonar.sh
RUN_AS_USER=sonar

### Start Sonarqube
sudo su sonar
cd /opt/sonarqube/bin/linux-x86-64/

./sonar.sh start
./sonar.sh status
./sonar.sh stop

tail /opt/sonarqube/logs/sonar.log

### Configure systemd service
vim /etc/systemd/system/sonar.service
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target

systemctl start sonar
systemctl enable sonar
systemctl status sonar

### Access Sonarqube
http://server_IP:9000