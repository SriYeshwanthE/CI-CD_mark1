 
Real-Time DevOps Project | Deploy to Kubernetes Using Jenkins | End-to-End DevOps Project | CICD
Build a complete CI/CD pipeline using Jenkins, Maven, Docker, and the Elastic Kubernetes Services (EKS)
 
Project Description
Build a complete CI/CD pipeline using Jenkins, Maven, Docker, and the Elastic Kubernetes Services (EKS)
// In the CI/CD pipeline if the developer makes any changes to the ---> application repository on GitHub then it will trigger the CI job of the Jenkins ---> which will pull the code from the application repository and it will build and test the artifact using Maven ---> then we will do the static code analysis of the artifact using SonarQube ---> then we will build the Docker image and we will push that Docker image to the Docker hub, ---> After that we will scan the docker image using Trivy scan, here our CI job will end
\** Once our CI job has been completed our CD job will automatically get trigger ---> and our CD job will first update the build number or the release number in the deployment yaml file on the git ops repository on the GitHub --->* and then ArgoCD will pull the manifest file, ---> updated manifest file from the GitHub repository and it will deploy the resources on the EKS cluster ---> and once our CD job is finished it will send the notification over the slack ---> or it can send the email also, so let's start building our project.//
YouTube Link For This Project
*** STAGES ***
1.	Install and Configure the Jenkins-Master and Jenkins-Agent
2.	Integrate Maven to Jenkins and Add GitHub Credentials to Jenkins
3.	Create Pipeline Script(Jenkinsfile) for Build & Test Artifacts and Create CI Job on Jenkins
4.	Install and Configure the SonarQube
5.	Integrate SonarQube with Jenkins
6.	Build and Push Docker Image using Pipeline Script
7.	Setup Bootstrap Server for eksctl and Setup Kubernetes using eksctl
8.	ArgoCD Installation on EKS Cluster and Add EKS Cluster to ArgoCD
9.	Configure ArgoCD to Deploy Pods on EKS and Automate ArgoCD Deployment Job using GitOps GitHub Repository
10.	DONE - Verify CI/CD Pipeline by Doing Test Commit on GitHub Application Repo
Install and Configure the Jenkins-Master and Jenkins-Agent
Create One EC2 instance t2. micro Ubuntu and 15GB Storage Configuration
 
Copy the public IP of this instance and Connect via Mobaxterm
 
 
After connecting update the machine and upgrade
sudo apt update -y
sudo apt upgrade -y
 
 
 
 
sudo vim /etc/hostname
Delete old name and Type: Jenkins-Master
 
After that reboot system sudo init 6
 
 
 
sudo wget -O /usr/share/keyrings/jenkins-keyring.aschttps://pkg.jenkins.io/debian/jenkins.io-2023.key echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]https://pkg.jenkins.io/debian binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null sudo apt-get update sudo apt-get install jenkins
 
 
You can enable the Jenkins service to start at boot with the command:
COPY
COPY
sudo systemctl enable jenkins
You can start the Jenkins service with the command:
COPY
COPY
sudo systemctl start jenkins
You can check the status of the Jenkins service using the command:
COPY
COPY
sudo systemctl status jenkins
 
Go to instance security group and allow port no. 8080 for jenkins
 
 
 
Now launch another instance with the name of Jenkins-Agent
 
 
 
EBS volumes will be select 15GB
 
Click on Launch Instance and Connect this instance with the help of mobaxterm
 
 
After connecting the server first we will update and upgrade the server with the help of the following command
sudo apt update
 
sudo apt upgrade -y
 
To change hostname --> sudo vim /etc/hostname
 
Jenkins-Agent ---> new hostname
 
After that reboot instance ---> sudo init 6
 
Now install java on this instance sudo apt install openjdk-17-jre -y
 
Now install docker on this server --> sudo apt-get install docker.io -y
 
Give full rights of current user sudo usermod -aG docker $USER
 
sudo vim /etc/ssh/sshd_config ---- make some changes
 
Remove # from this line and save the file
 
Do this settings same on Jenkins-Master server also
 
 
And Reload sshd service ---> sudo service sshd reload
on both instance same command
 
Now on Jenkins-Master server generate secrete key ---> ssh-keygen -t ed25519
 
Go to cd .ssh/ path and it will generated public key and private key id ed25519 and id ed25519.pub
Now copy the private key and copy this key on Jenkins-Agent authorized_keys folder and save it.
 
 
 
 
cat authorized_keys
It will shows copied key.
 
Now we need to access jenkins so copy the Jenkins-Master server public ip and paste it in browser <PublicIP>:8080
 
It will show jenkins dashboard
Go to given path for password
 
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
It will show password copy it
 
And paste it on Administrator password and click on continue
 
Click on Install Suggested plugins
 
 
Click on save and finish
 
create first admin user and click on save and continue
 
Click on Start using Jenkins
 
This is Jenkins Dashboard.
 
Go to Manage Jenkins --> Nodes
 
Click Built-In Node
 
Configure ---> 0 and save
 
Again go to Manage Jenkins ---> Nodes
 
Click on New Node
 
Node name: Jenkins-Agent
Type: Permanent Agent and click on create
 
Number of executors: 2
Remote root directory: /home/ubuntu
Labels: Jenkins-Agent
 
Host: Paste Private IP of Jenkins-Agent
 
Click on Jenkins for adding credetials
 
 
Copy This Private Key and Paste it in Key value
 
 
 
 
 
Now go to Jenkins Dashboard and Click on Create a Job
 
Enter job name: Test--> Click on Pipeline and click on ok
 
Go to Pipeline-->Pipeline Script-->Hello World
 
Click on Build now and see test is successful.
 
 
Integrate Maven to Jenkins and Add GitHub Credentials to Jenkins
Go to Manage Jenkins-->Plugins
 
Available Plugins>maven
 
Available Plugins>eclipse install them
 
Manage Jenkins--> Tools
 
Maven Installations-->Add Maven--> Maven3
 
JDK installations -->java17--> install from adoptium.net
 
Version -- jdk-17.0.5+8
 
Manage Jenkins-->Credentials
 
Add Credentials
 
Kind: Username with password
GitHub username: thoratsunil
For password we want to create token
 
Go to GitHub account setting
 
Developer settings
 
Personal access tokens --> Token (classic)
 
Generate new token (classic)
 
It will ask password so give password and confirm
name for Note: gitHub and generate token
 
 
Copy this token
 
And paste it in Password section and give id name gitHub
 
so github credentials are created
 
Create Pipeline Script(Jenkinsfile) for Build & Test Artifacts and Create CI Job on Jenkins
Go to GitHub repositories --> CI-CD_mark1

and create Jenkinsfile in master or main branch
 
Start writing groovy syntax with the help of the Hello World script

Save file with commit changes
 
Go to Jenkins Dashboard and create new job
 
register-app-ci
 
In the pipeline option select Pipeline script from SCM --> Git --> Paste Repository URL --> Select Credentials and click apply save.
 
Select branch main or master
Script Path: Jenkinsfile
 
Now click on Build Now

Job will success
  
 
 
Install and Configure the SonarQube
For install SonarQube Create new EC2 instance : SonarQube -- Ubuntu --
 
Instance Type: t3.medium
 
Volume 15GiB and click on launch instance
 
Select SonarQube instance and copy public IP
 
Connect SonarQube with the help of Mobaxterm
 
Update the server sudo apt update
 
sudo apt upgrade -y
 
Now install PostgreSQL with the help of following commands
 
 
 
Add PostgresSQL repository
$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' $ wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/pgdg.asc &>/dev/null
 
Install PostgreSQL
$ sudo apt update
$ sudo apt-get -y install postgresql postgresql-contrib
$ sudo systemctl enable postgresql
 
Create a Database for Sonarqube
$ sudo passwd postgres
$ su - postgres
$ createuser sonar
$ psql
$ ALTER USER sonar WITH ENCRYPTED password 'sonar';
$ CREATE DATABASE sonarqube OWNER sonar;
$ grant all privileges on DATABASE sonarqube to sonar;
$ \q $ exit
 
Add Adoptium repository
$ sudo bash $ wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
$ echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
 
 
Install Java 17
$ apt update
$ apt install temurin-17-jdk
$ update-alternatives --config java
$ /usr/bin/java --version $ exit
 
 
 
Linux Kernel Tuning
Increase Limits
$ sudo vim /etc/security/limits.conf //Paste the below values at the bottom of the file sonarqube - nofile 65536
sonarqube - nproc 4096
 
 
Increase Mapped Memory Regions
sudo vim /etc/sysctl.conf //Paste the below values at the bottom of the file vm.max_map_count = 262144
 
 
Restart the server --> sudo init 6
 
Go to the SonarQube instance and allow port number 9000 because sonarqube will run on 9000 port
 
 
 
Now install SonarQube on This Instance
 
Sonarqube Installation
Download and Extract
$ sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.0.65466.zip
$ sudo apt install unzip
$ sudo unzip sonarqube-9.9.0.65466.zip -d /opt
$ sudo mv /opt/sonarqube-9.9.0.65466 /opt/sonarqube
 
Create a user and set permissions
$ sudo groupadd sonar
$ sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar
$ sudo chown sonar:sonar /opt/sonarqube -R
Update Sonarqube properties with DB credentials
$ sudo vim /opt/sonarqube/conf/sonar.properties //Find and replace the below values, you might need to add the sonar.jdbc.url sonar.jdbc.username=sonar sonar.jdbc.password=sonar sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
 
 
Create service for Sonarqube
$ sudo vim /etc/systemd/system/sonar.service //Paste the below into the file [Unit] Description=SonarQube service After=syslog.target network.target
[Service] Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
User=sonar Group=sonar Restart=always
LimitNOFILE=65536 LimitNPROC=4096
[Install] WantedBy=multi-user.target
 
 
Start Sonarqube and Enable service
$ sudo systemctl start sonar
$ sudo systemctl enable sonar
$ sudo systemctl status sonar
 
Watch log files and monitor for startup
$ sudo tail -f /opt/sonarqube/logs/sonar.log
 
Now go to sonarqube instance and copy PublicIP address for access sonarqube
 
Go to Browser and paste publicip with port number 9000
Give admin admin username and password and login
 
Update your passwoprd
 
This is the Sonar Qube dashboard
 
Integrate SonarQube with Jenkins
Integrate SonarQube with Jenkins
So go to Myaccount in that go to security
 
Click on generate token copy it
 
Go to Manage Jenkins --> Credentials
 
Add Credentials
 
Kind: Secret text
 
jenkins-sonarqube-token created
 
Now go to Manage Jenkins --> Plugins
 
INstall sonarqube plugins
 
 
After Installation of plugins restart the Jenkins
 
After restart go to Manage Jenkins --> System Configure
 
To remove this popup Go to Configure which of these warnings are shown
 
Go to Hudden Security warnings and uncheck this two options
 
Now go to SonarQube Servers in the Manage Jenkins --> System
In that Paste Private Ip Address of SonarQube here
 
 
 
 
 
Go to jenkinsfile and add one more stage SonarQube Analysis
 
 
 
 
Now click on build now and check all stages
 
 
Go to the sonarqube dashboard and see all stages are green means no error
 
Now add Jenkins in sonarqube pipeline
Go to Administration -- Configuration -- Webhooks
 
Click on Create
 
Name sonarqube-webhook ---> paste Private Ip of Jenkins-server
 
sonarqube-webhook created
 
Go to Jenkins file and add quality gate stage
 
After that click on Build Now we will see all stages will be successfully done
 
Build and Push Docker Image using Pipeline Script
Now add one more stage for Build and Push Docker Image using Pipeline Script
so go to the Manage Jenkins --> Plugins
 
Install docker plugins ---> 6 Plugins
 
 
After installation restart the Jenkins
 
 
After login go to Manage Jenkins- Credentials
 
add credentials
 
Go to dockerhub account settings
 
Security --> New Access Token
 
Type anything in the description ; project5
 
Go to jenkins credentials and paste password section newly created token
 
dockerhub credentials ready
 
Now add environment variables in stages in groovy syntax
 
 
 
there is no new repository on docker hub account
 
Click on Build Now
All stages passed successfully
 
Refresh the docker hub page we will see the register-app-pipeline image be added here
 
 
Now one more stage add for trivy scan
 
this stage will remove the previously create docker image
 
Setup Bootstrap Server for eksctl and Setup Kubernetes using eksctl
 
Go to AWS and create one instance with the name of EKS-Bootstrap-Server
Ubuntu Image --t2.micro instance
 
Copy Public IP of EKS-Bootstrap-Server and access it from local
 
Open mobaxterm and connect instance
 
update instance: sudo apt update
 
sudo apt upgrade -y
 
To change hostname of this machine sudo vim/etc/hostname
 
Now reboot the instance : sudo inti 6
 
After restarting install AWS cli on this server
Install AWS Cli on the above EC2
Refer--https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html $ sudo su
$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
$ apt install unzip,
$ unzip awscliv2.zip
$ sudo ./aws/install
 
 
Installing kubectl
Refer--https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html $ sudo su
$ curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl
$ ll ,
$ chmod +x ./kubectl //Gave executable permisions
$ mv kubectl /bin //Because all our executable files are in /bin
$ kubectl version --output=yaml
 
 
 
Installing eksctl
Refer---https://github.com/eksctl-io/eksctl/blob/main/README.md#installation $ curl --silent --location "[https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_](github.com/weaveworks/eksctl/releases/lates..
[$(uname](github.com/weaveworks/eksctl/releases/lates.. -s)_amd64.tar.gz" | tar xz -C /tmp
$ cd /tmp
$ ll
$ sudo mv /tmp/eksctl /bin
$ eksctl version
 
 
For cluster we want to create one IAM user so go to AWS and select IAM
 
Click on Role --> Create Role
 
AWS service --> EC2
 
Perminissions Policeies --> AdministratorAccess
 
Roll Name: eksctl_role Click on Create role
 
Go to Instance --Actions--Security--ModifyIAM user
 
select eksctl_role and Update IAM role
 
Setup Kubernetes using eksctl
Refer--https://github.com/aws-samples/eks-workshop/issues/734
$ eksctl create cluster --name virtualtechbox-cluster --region ap-south-1 --node-type t2.small --nodes 3 \
 
 
It will take 15 - 20 minutes
 
After creating creating cluster
kubectl get nodes
 
ArgoCD Installation on EKS Cluster and Add EKS Cluster to ArgoCD
First, create a namespace
$ kubectl create namespace argocd
 
Next, let's apply the yaml configuration files for ArgoCd
$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
 
Now we can view the pods created in the ArgoCD namespace.
$ kubectl get pods -n argocd
 
To interact with the API Server we need to deploy the CLI:
$ curl --silent --location -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.4.7/argocd-linux-amd64
$ chmod +x /usr/local/bin/argocd
 
Expose argocd-server
$ kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
 
Wait about 2 minutes for the LoadBalancer creation
$ kubectl get svc -n argocd
 
Get pasword and decode it.
$ kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
$ echo QVpTbXZ0QnM1LXlDNXp4WA== | base64 --decode
 
 
Add EKS Cluster to ArgoCD
login to ArgoCD from CLI
$ argocd login a2255bb2bb33f438d9addf8840d294c5-785887595.ap-south-1.elb.amazonaws.com --username admin
Copy this password for Argocd login
 
kubectl get svc -n argocd
copy argocd server url and paste it in browser for login argocd
 
This is the login dashboard of argocd
Type username: admin and Password -> will be generate above
 
This is the Argocd dashboard
 
Go to User Info --> UPDATE PASSWORD
 
SAVE NEW PASSWORD
 
 
Logout and Login with new set password
 
 
On server type argocd login and paster url and --username admin
 
Type password --> Login successfully
 
Go to argocd Dashboard -- Settings -- Clusters
 
See this is default cluster
 
This will also show in terminal by typing following command
argocd cluster list
Below command will show the EKS cluster
$ kubectl config get-contexts
Add above EKS cluster to ArgoCD with below command
$ argocd cluster add i-08b9d0ff0409f48e7@virtualtechbox-cluster.ap-south-1.eksctl.io --name virtualtechbox-eks-cluster
 
$ kubectl get svc
 
Refresh the argocd page we will see the virtualtechbox eks cluster created
 
Configure ArgoCD to Deploy Pods on EKS and Automate ArgoCD Deployment Job using GitOps GitHub Repository
Go to github create gitops-register-app repository
 
and connect this repository to argo cd
so go to argo cd settings -- Repositories
 
click --> + CONNECT REPO
 
Paste here url of github
 
connected successfully
 
in the deployment.yaml file image tag should be same
 
Now go to argo cd Applications --> +NEW APPS
 
General --register-app --> project name:default
 
paste here gitops-register-app git url here
 
click on create
 
Application are created in argocd
 
Go to terminal and type kubectl get pods
It will show running pods
 
kubectl get svc
copy the url and paste it on browser for access the application
 
 
<url>:8080/webapp/
 
Argocd job will healthy
 
Now go to Jenkins and create new job "gitips-register-app-cd"
 
 
 
 
 
 
 
 
 
Go to the AWS and copy PublicIP DNS of Jenkins-Master
 
As well as create new stage "Trigger CD pipeline"
 
Go to github username-Configure
 
Create API Token
 
 
Copy is only supported with a secure (HTTPS) connection
 
 
And this credentials in Jenkins Credentials
 
 
 
JENKINS_API_TOKEN credentials created
 
Now add these credentials in environment variables of jenkins file on github
 
 
 
 
Verify CI/CD Pipeline by Doing Test Commit on GitHub Application Repo
Now go to register-app-ci ---> configure
 
 
Build Triggers --> Poll SCM --> schedule
 
Now click on Build Now all stages will be completed ( More Issues will occur with this stage search on the internet for a solution)
If I make some changes in the GitHub repository it will automatically trigger the CI job and after completing CI Job CD Job will be Trigger successfully and our changes will be made to our application.
CI Job will run successfully
 
And CD Job also ran successfully
 
Application Changed with the CI CD Pipeline
 
Argocd job also healthy
 
SonarQube Scanning also green
 
Done Seccessfully ...!!!
1
Subscribe to my newsletter
Read articles from Sunil Thorat directly inside your inbox. Subscribe to the newsletter, and don't miss out.
SUBSCRIBE
Devopsci-cdJenkinsArgoCDsonarqube
Written by
  
Sunil Thorat
I'm Sunil Thorat, an enthusiastic DevOps Engineer with a passion for technology and problem-solving. With a strong foundation in supporting applications and a keen interest in DevOps tools, I thrive on ensuring seamless operations and top-notch user experiences. My journey has led me to delve into the realm of AWS cloud engineering, where I leverage my skills to optimize cloud solutions and enhance scalability. Eager to explore new challenges and contribute to innovative projects, I'm dedicated to continuous learning and staying updated with the latest trends in DevOps and cloud technologies.
Follow
 
Share this
MORE ARTICLES
  
Sunil Thorat
 
AWS Project
Steps for this Project:----> Step 1. A high-level look at the pipeline we’re building Step 2. How mu…
  
Sunil Thorat
 
Jenkins Pipeline Script Tutorial
Complete the CI CD Pipeline Project with the help of Groovy Syntax ---> Tools Used In This Project -…
  
Sunil Thorat
 
Jenkins Declarative CI/CD Pipeline for DevOps
YouTube Link For Complete Project GitHub Link 1 for source code Source Code for Jenkins Pipeline Je…
©2024 Sunil Thorat
Archive·Privacy policy·Terms
Write on Hashnode
Powered by Hashnode - Home for tech writers and readers

