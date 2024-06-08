
-----Repo URL-------

CICD pipeline Demo

Create EC2 instance
Install Jenkins
Install Java
install git
install maven
install docker

Note : Create Ubuntu Machine with t2.small
22,80,8080,8081
Configure Storage : 20

1.)login using command 
========================== 
ssh -i cicd-instancekeys.pem ubuntu@3.86.5.44

ssh -i cicd-instancekeys.pem ubuntu@54.163.81.88

============================
and make sure EC2 security group allows port 80 and 8080 and 8081(application port)

ssh -i CICDKeyPair.pem ubuntu@52.66.135.30

2.) install java
sudo apt-get update -y
sudo apt-get install openjdk-11-jdk -y
sudo apt-get install openjdk-17-jdk -y
sudo apt-get install openjdk:17-jdk-slim -y
3.) install docker
https://get.docker.com/
or below 2 commands

curl -fsSL https://get.docker.com -o install-docker.sh
sudo sh install-docker.sh
docker ps(Permission denied): Will check it later.

4.) 
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null


echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl status jenkins : to verify jenkins is running
                              : Press Q to exit

//Allow to add jenkins to add in the docker //group							  
* sudo usermod -a -G docker jenkins	

//cmd to add permission to add our usr(docker ps)
* sudo usermod -aG docker $USER	
* newgrp docker
* sudo service jenkins restart					  


go to browser and hit ip:8080 you will see jenkins home page, it will give you path for security token and path will be like /var/jenkins_home/secrets/initialAdminPassword  make sure you use sudo command like this   "sudo vim /var/jenkins_home/secrets/initialAdminPassword "

sudo less /var/lib/jenkins/secrets/initialAdminPassword
* sudo vim /var/lib/jenkins/secrets/initialAdminPassword
* http://3.86.5.44:8080/

sudo less /var/lib/jenkins/secrets/initialAdminPassword


5.) install awscli on ec2 instance  "Note: Use this command:
 sudo apt install awscli"
 
 sudo curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt-get install unzip
sudo unzip awscliv2.zip
sudo ./aws/install
then aws --version it should show version 2

6.) install git using command "sudo apt install git -y"
7.) install maven using command "sudo apt install maven -y"
8.) Create Repo in ECR with name 4
    Amazon Elastic Container Registry
	create the public repo
	Linux or Arm64 : choose Karo if not created.

9)
click on install suggested plugins
after that install docker and docker pipeline plugin, 
install git plugin also

10) Create the role below deatils.
    Uploading Docker images into AWS ECR
   a) AmazonEC2ContainerRegistoryPowerUser
   b) AmazonEC2ContainerRegistoryFullAccess
   c) AmazonElasticContainerRegistoryPublicFullAccess

aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/h6a5w0v8
docker build -t my-cicd-repo .
docker tag my-cicd-repo:latest public.ecr.aws/h6a5w0v8/my-cicd-repo:latest
docker push public.ecr.aws/h6a5w0v8/my-cicd-repo:latest
-------------------Jenkins Script------031870154778--

pipeline {
    agent any
    environment {
        registry = "<>.dkr.ecr.us-east-1.amazonaws.com/my-cicd-repo"
    }

    stages {
        stage('Cloning Git') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/tejsingh1977/JenkinsCICDAWSUbuntuProject']]])
            }
        }

        // Building the JAR file
        stage('Building JAR') {
            steps {
                sh 'mvn clean package'
            }
        }

       

        // Uploading Docker images into AWS ECR
       stage('Pushing to ECR') {
            steps{  
                script {
                    sh 'aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/h6a5w0v8'
					sh 'docker build -t public.ecr.aws/h6a5w0v8/my-cicd-repo:latest .'
					sh 'docker push public.ecr.aws/h6a5w0v8/my-cicd-repo:latest'
                }
            }
        }

        // Stopping Docker containers for cleaner Docker run
        stage('Stop previous containers') {
            steps {
                sh 'docker stop myjavaContainer || true'
                sh 'docker rm myjavaContainer || true'
            }
        }

        // Running Docker container
        stage('Docker Run') {
            steps {
                sh 'docker run -d -p 8081:8081 --name myjavaContainer public.ecr.aws/h6a5w0v8/my-cicd-repo:latest'
            }
        }
    }
}

===========================

```
Port Enable 80 for http, 9000 for sonarqube

sudo apt-get update -y
sudo apt-get install openjdk-11-jdk -y

sudo apt install unzip
sudo adduser sonarqube
su - sonarqube
 wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
unzip *
chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
./sonar.sh start

54.163.81.88
http://54.163.81.88:9000
Once you login to sonarqube follow the below steps: 
1) go to Account
2) go to security tab
3) write jenkins and generate token
4) go to the jenkins
4.1-> got to manage jenkins, got credentials
4.2 ->  got to domain-> global-> add credentials
4.3 -> kind will be secret text
4.4-> secret will be sonarqube token and id will be sonarqube
and save.
  

```
 stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://<>:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'cd accounts && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
	
	---------------------------------final running pipeline below================
	
pipeline {
    agent any
    environment {
        registry = "<>.dkr.ecr.us-east-1.amazonaws.com/accounts"
    }

    stages {
        stage('Cloning Git') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/Prem12/cicd-pipeline-project.git']]])
            }
        }
        stage('Build and Test') {
           steps {
			    sh 'ls -ltr'
                sh 'cd accounts && mvn clean package'
            }
        }
        stage('Static Code Analysis') {
          environment {
            SONAR_URL = "http://<>:9000"
          }
          steps {
            withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
              sh 'cd accounts && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
            }
          }
        }
        stage('Pushing to ECR') {
            steps{  
                script {
                    sh 'aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/h6a5w0v8'
					sh 'cd accounts && docker build -t public.ecr.aws/h6a5w0v8/accounts:latest .'
					sh 'docker push public.ecr.aws/h6a5w0v8/accounts:latest'
                }
            }
        }
    }
}
