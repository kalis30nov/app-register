Register-App

<img width="1355" alt="Screenshot 2024-03-18 at 9 41 17 PM" src="https://github.com/kalis30nov/app-register/assets/129021438/6e896220-6c4b-4912-ad5f-8f98c5b498cb">


*****************************************************************************************************************************
Tools Used / Prerequisites :
---------------------------

Git        : Version control system for efficient code management and collaboration.                                                   
Jenkins    : Auotmation server for continous integration and delivery pipelines.                                                       
Maven      : Build automation tool for managing Java project dependencies.                                                             
Sonarqube  : Code quality and security platform for continous code inspection.                                                         
Dockerhub  : Repository manager for centralized artifact management and distribution.                                                  
Trivy      : Vulnerability scanner for docker images.                                                                                  
ArgoCD     : Declarative continuous delivery tool for Kubernetes. I                                                                    
Kubernetes : Container orchestration platfrom for automating deployment and scaling.                                                   
Slack      : To get notification on status of CICD build.                                                                              
*****************************************************************************************************************************
Steps

1. Install the necessary Jenkins plugins:
   1.1 Git plugin
   1.2 Maven Integration plugin
   1.3 Pipeline plugin
   1.4 Docker plugin
   1.5 Kubernetes Continuous Deploy plugin

3. Create a new Jenkins pipeline:
   2.1 In Jenkins, create a new pipeline job and configure it with the Git repository URL for the Java application.
   2.2 Add a Jenkinsfile to the Git repository to define the pipeline stages.

4. Configure the pipeline stages:
    Stage 1: Use the Git plugin to check out the source code from the Git repository.
   
    Stage 2: Use the Maven Integration plugin to build the Java application.
   
    Stage 3: Use the JUnit and Mockito plugins to run unit tests.
   
    Stage 4: Use the SonarQube plugin to analyze the code quality of the Java application.
   
    Stage 5: Use the Maven Integration plugin to package the application into a JAR file.
   
    Stage 6: Push the image to DockerHub.
   
    Stage 7: Manifest file repo is updated with new tag.
   
    Stage 8: Use ArgoCD to promote application to dev environment.
   
    Stage 9: Run user acceptance tests on the deployed application.
   
    Stage 10: Promote the application to a production environment using Argo CD.

6. Set up Argo CD:
    Install Argo CD on the Kubernetes cluster.
    Set up a Git repository for Argo CD to track the changes in the Manifest file repo.
    Deploy the changes to the cluster added.

8. Configure Jenkins pipeline to integrate with Argo CD:
   6.1 Add the Argo CD API token to Jenkins credentials.
   6.2 Update the Jenkins pipeline to include the Argo CD deployment stage.


*****************************************************************************************************************************

# Deatailed Step by Step Installation Procedure
1) Install and Configure the Jenkins-Master & Jenkins-Agent
2) Integrate Maven to Jenkins and Add GitHub credentials to Jenkins
3) Install and Configure the SonarQube
4) Setup DockerHub
5) Setup Bootstrap Server for eksctl and Setup Kubernetes using eksctl
6) ArgoCD Installation on EKS Cluster and Add EKS Cluster to ArgoCD

# 1) Install and Configure the Jenkins-Master & Jenkins-Agent 
---------------------------------------------------------

## Jenkins-Master
--------------
    $  Provision EC2 instance with 15Gb disk and run the following commands.                                                        
    $ Enable Public IP while creation.
    $ Enable port 8080 in SG of EC2 instance to access the Jenkins

## Install Java

    $ sudo apt update
    $ sudo apt upgrade
    $ sudo nano /etc/hostname
    $ sudo init 6
    $ sudo apt install openjdk-17-jre
    $ java -version

## Install Jenkins by checking weekly release in Jenkins

    $ curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
      /usr/share/keyrings/jenkins-keyring.asc > /dev/null
    $ echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
      https://pkg.jenkins.io/debian binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
  
    $ sudo apt-get update
    $ sudo apt-get install jenkins
    $ sudo systemctl enable jenkins       //Enable the Jenkins service to start at boot
    $ sudo systemctl start jenkins        //Start Jenkins as a service
    $ systemctl status jenkins
    $ sudo vi /etc/ssh/sshd_config        //uncomment PublicKeyAuthentication and AuthorizedKey file
    $ sudo service sshd reload
    $ ssh-keygen 
    $ cd .ssh

## Jenkins-Agent
--------------
    $ Provision EC2 instance with 15Gb disk and run the following commands.
    $ Enable Public IP while creation.

## Install Java

    $ sudo apt update
    $ sudo apt upgrade
    $ sudo nano /etc/hostname
    $ sudo apt install openjdk-17-jre
    $ java -version
    $ sudo init 6

## Install Docker and setup up Master/Agent connection

    $ sudo apt-get install docker.io   // installs docker and docker group is created                                              
    $ sudo usermod -aG docker $USER    // add current user to docker group                                                          
    $ vi /etc/ssh/sshd_config          // uncomment PublicKeyAuthentication and AuthorizedKey file                                  
    $ sudo service sshd reload                                                                                                      
    $ vi ~/.ssh/authorized_keys        // updatee the id_rsa.pub key of Jenkins Master                                              
    $ Login to Jenkins console and make No. executor nodes to "0".                                                                  
    $ Add Jenkins Agent in Nodes, with Jenkins Master's Private key and Private IP as new node and test Hello World pipeline job. 
    
# 2) Integrate Maven to Jenkins and Add GitHub credentials to Jenkins
--------------------------------------------------------------------

## Install Plugins:

    $ Maven Integration                                                                                                             
    $ Pipeline Maven                                                                                                                
    $ Ecplise Temurin Installer                                                                                                         
## Configure Tools

    Maven --> Maven3                                                                                                                
    JDK --> Java17                                                                                                                  
    // This config will be referred in Jenkins pipeline.                                                                            

## Github Credential

   Create personal access token and save in Jenkins Credentials as username and password.                        

## Copy the Application code to Git Repo and create a Jenkinsfile with CI pipeline job.                                              

-      PollSCM everyminute
-      Poll script from SCM
-      Provide the URL of the Application Repo
-      Select the Github credential that was added earlier.
-      Branch: Main

# 3. Install and Configure the SonarQube 
 -----------------------------------------

    $ Provision EC2 instance with 15Gb disk and run the following commands.
    $ Enable Public IP while creation.
    $ Enable port 9000 in SG of SonarQube instance.
 
## Update Package Repository and Upgrade Packages
    $ sudo apt update
    $ sudo apt upgrade
    
## Add PostgresSQL repository
    $ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
    $ wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/pgdg.asc &>/dev/null
    
## Install PostgreSQL
    $ sudo apt update
    $ sudo apt-get -y install postgresql postgresql-contrib
    $ sudo systemctl enable postgresql
    
## Create Database for Sonarqube
    $ sudo passwd postgres
    $ su - postgres
    $ createuser sonar
    $ psql 
    $ ALTER USER sonar WITH ENCRYPTED password 'sonar';
    $ CREATE DATABASE sonarqube OWNER sonar;
    $ grant all privileges on DATABASE sonarqube to sonar;
    $ \q
    $ exit
    
## Add Adoptium repository
    $ sudo bash
    $ wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
    $ echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
    
 ## Install Java 17
    $ apt update
    $ apt install temurin-17-jdk
    $ update-alternatives --config java
    $ /usr/bin/java --version
    $ exit 
## Linux Kernel Tuning
   # Increase Limits
    $ sudo vim /etc/security/limits.conf
    //Paste the below values at the bottom of the file
    sonarqube   -   nofile   65536
    sonarqube   -   nproc    4096

    # Increase Mapped Memory Regions
    sudo vim /etc/sysctl.conf
    //Paste the below values at the bottom of the file
    vm.max_map_count = 262144

## Sonarqube Installation ##
## Download and Extract
    $ sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.0.65466.zip
    $ sudo apt install unzip
    $ sudo unzip sonarqube-9.9.0.65466.zip -d /opt
    $ sudo mv /opt/sonarqube-9.9.0.65466 /opt/sonarqube
    
## Create user and set permissions
     $ sudo groupadd sonar
     $ sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar
     $ sudo chown sonar:sonar /opt/sonarqube -R
     
## Update Sonarqube properties with DB credentials
     $ sudo vim /opt/sonarqube/conf/sonar.properties
     //Find and replace the below values, you might need to add the sonar.jdbc.url
     sonar.jdbc.username=sonar
     sonar.jdbc.password=sonar
     sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
     
## Create service for Sonarqube
$ sudo vim /etc/systemd/system/sonar.service
//Paste the below into the file
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

## Start Sonarqube and Enable service
     $ sudo systemctl start sonar
     $ sudo systemctl enable sonar
     $ sudo systemctl status sonar

## Watch log files and monitor for startup
     $ sudo tail -f /opt/sonarqube/logs/sonar.log

## Sonar Webhook setup
    $ Login to sonarqube with id admin:admin
    $ Myaccount --> Security --> GenerateToken
    $ Add generated token in Jenkins as secrettext.
    $ Create webhook : Admin -> Config -> Webhook -> <jenkins_url>/sonarqube-webhook

## Plugins Install:
    $ SonarQube Scanner for Jenkins
    $ Sonar Quality Gates Plugin
    $ Quality Gates Plugin

## Add Sonarqube EC2 instance in Jenkins
    $ Manage Jenkins -> System -> SonarQube servers -> Sonarserver URL with port 9000 -> Sonar authentication

## SonarScanner for MSBuild installations
    $ Manage Jenkins -> Tools -> SonarScanner for MSBuild installations -> Select Version

# 4) Setup DockerHub
-----------------------

## Plugins Install:
    $ CloudBees Docker Build and Publish plugin
    $ Docker API Plugin
    $ Docker Commons Plugin
    $ Docker Pipeline
    $ Docker plugin
    $ docker-build-step

## DockerHub link with Jenkins
    $ DockerHub -> MyAccount -> Security -> CreateNew Access Token
    $ Manage Jenkins - > Credentials -> Username and Password

# 5) Setup Bootstrap Server for eksctl and Setup Kubernetes using eksctl 
 ----------------------------------------------------------------------
 
## Create EC2 instance

## Create IAM Role with eks access and assing to this EC2.

## Install awscli on the above EC2
$ sudo su
$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
$ apt install unzip 
$ unzip awscliv2.zip
$ sudo ./aws/install


## Installing kubectl
$ sudo su
$ curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl
$ chmod +x ./kubectl  //Gave executable permisions
$ mv kubectl /bin   //Because all our executable files are in /bin
$ kubectl version --output=yaml

## Installing  eksctl
$ curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
$ cd /tmp
$ sudo mv /tmp/eksctl /bin
$ eksctl version

## Setup Kubernetes using eksctl
$ eksctl create cluster --name nutmeg_demo \
--region us-east-1 \
--node-type t2.small \
--nodes 3 \

$ kubectl get nodes

# 6) ArgoCD Installation on EKS Cluster and Add EKS Cluster to ArgoCD
-------------------------------------------------------------------

## First, create a namespace
    $ kubectl create namespace argocd

## Next, let's install ArgoCd using Helm.
    $ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
    $ chmod 700 get_helm.sh
    $ ./get_helm.sh
    $ helm version 
    $ kubectl create ns argocd
    $ helm repo add argo https://argoproj.github.io/argo-helm
    $ helm install argocd-release argo/argo-cd --namespace argocd

## Now we can view the pods created in the ArgoCD namespace.
    $ kubectl get pods -n argocd

## To interact with the API Server we need to deploy the CLI:
    $ curl --silent --location -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.4.7/argocd-linux-amd64
    $ chmod +x /usr/local/bin/argocd

## Expose argocd-server
    $ kubectl patch svc argocd-release-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

## Wait about 2 minutes for the LoadBalancer creation
    $ kubectl get svc -n argocd

## Get pasword and decode it.
    $ kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
    $ echo U0dVT3U3ZFdOOHc5WVl5WQ== |base64 -d

## Add EKS Cluster to ArgoCD
    login to ArgoCD from CLI
    $ argocd login a2a336b7c390c47b890ea6a75aa7129f-80549556.us-east-1.elb.amazonaws.com

## List cluster and verify 
     $ argocd cluster list

## Below command will show the EKS cluster
     $ kubectl config get-contexts

## Add above EKS cluster to ArgoCD with below command
     $ argocd cluster add i-04afe7b218bd3f3ce@nutmeg-demo.us-east-1.eksctl.io --name nutmeg-demo-eks-cluster

## $ kubectl get svc

# 7) Cleanup 
------------

$ kubectl get all
$ kubectl delete deployment.apps/app-register-deployment                            //it will delete the deployment                 
$ kubectl delete service/app-register-service                                       //it will delete the service                    
$ eksctl delete cluster nutmeg-demo.us-east-1.eksctl.io --region us-east-1          //it will delete the EKS cluster





