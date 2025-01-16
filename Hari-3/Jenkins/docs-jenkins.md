# Jenkins

## Install Jenkins
### Jenkins Debian Packages
This is the Debian package repository of Jenkins to automate installation and upgrade. To use this repository, first add the key to your system: 
```
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null
```

Then add a Jenkins apt repository entry: 
```
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
```

Update your local package index, then finally install Jenkins: 
```
sudo apt-get update
sudo apt-get install fontconfig openjdk-17-jre
sudo apt-get install jenkins
```
Cek status jenkins dan cek password
```
systemctl status jenkins
> copy password, contoh:
Jan 15 17:03:59 ubuntu jenkins[6727]: *a0a0abda60714afa9a6b89b6c9be3678*
```
login ke browsers
```
http://172.23.10.24:8080/
>install suggest plugin
```

Pipeline and Jenkinsfile
```
pipeline {
    agent { label 'devops1-amar' }

    stages {
        stage('Pull SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/bazigan/simple-apps.git'
            }
        }
        
        stage('Build') {
            steps {
                sh'''
                cd app
                npm install
                '''
            }
        }
        
        stage('Testing') {
            steps {
                sh'''
                cd app
                npm test
                npm run test:coverage
                '''
            }
        }
        
        stage('Code Review') {
            steps {
                sh'''
                cd app
                sonar-scanner \
                    -Dsonar.projectKey=Simple-Apps \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://172.23.8.20:9000 \
                    -Dsonar.login=sqp_afd306fec629a29b3636919ec4f4aaeff8f70ac4
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                sh'''
                docker compose up --build -d
                '''
            }
        }
        
        stage('Backup') {
            steps {
                 sh 'docker compose push' 
            }
        }
    }
}
```

PRACTICE: Pipeline and Jenkinsfile 
```
pipeline {
    agent { label 'devops1-rmdhn'}
    tools { nodejs 'NodeJs' }
    

    stages {
        stage('Pull SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/angkasatech/simple-apps.git'
            }
        }

        stage('Build') {
            steps {
                sh'''
                cd apps
                npm install
                '''
            }
        }
        
        stage('Testing') {
            steps {
                sh'''
                cd apps
                npm test
                npm run test:coverage
                '''
            }
        }
        
        stage('Code Review') {
            steps {
                sh'''
                cd apps
                sonar-scanner \
                    -Dsonar.projectKey=Simple-Apps \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://172.23.10.14:9000 \
                    -Dsonar.login=sqp_99bb646e1faee6597a4bc23b1ee231fa5bb6af3f
                '''
            }
        }
        
        stage('Deploy Compose') {
            steps {
                sh'''
                docker compose up --build -d
                '''
            }
        }
    }
}
```
>option backup to docker hub stages
```
stage('Backup') {
            steps {
                 sh 'docker compose push' 
            }
        }
```
