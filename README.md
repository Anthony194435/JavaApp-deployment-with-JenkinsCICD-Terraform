# CICD with jenkins-maven-sonarqube-docker-Eks-Terraform-Promethues-Grafana-newfile

![Screenshot 2025-01-21 144703](https://github.com/user-attachments/assets/4f8cfab8-d841-4596-a7e5-3ee365a7f9f8)


![CICD for JavaApp](https://github.com/Ikechukwu980/cicd-deployment-with-JavaApp-Terraform/assets/106882590/81e366bf-68fc-479f-9627-6a3d202316a0)


# Step 1 ; launch a Jenkins server using AWS EC2 instance
 	- Ubuntu 2
	- Jenkins and Sonarqube servers
	- Keypair
	- Security group : SSH / Jenkins port 8080 and Sonarqube port 9000
	- Instance type: T2 large for Jenkins and T2 medium for sonarqube
	- Attach an IAM Role for EC2 : permission AdminFullAccess
	- Launch the service and login to the instance via SSH

# Step 2 ; installation of required software

 `sudo hostname Jenkins`
 
 `sudo apt update`
 
 `sudo apt install maven -y`
 
#### Jenkins setup **

`curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
- sudo apt-get update
- sudo apt-get install jenkins
- systemctl status jenkins 
#### then access the Jenkins server via the server public_dns_name:8080
- Get the initial password from the below file

sudo cat /var/lib/jenkins/secrets/initialAdminPassword
	- Copy and paste in the browser
	- Click install plug-ins
	- Create username and password 
	- Click save and finish.

# Step 3: install AWS CLI 
  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
 - sudo apt install unzip 
 - sudo unzip awscliv2.zip 
 - sudo ./aws/install
 - aws --version

# Step 4: installing docker on the Jenkins server
   # installing docker on the Jenkins server 

   - sudo apt install docker.io -y 
   - sudo usermod -aG docker $USER 
   
  Exit and log back in
  
  - sudo systemctl start docker 
  - sudo systemctl enable docker 
  - sudo systemctl status docker 

#### add jenkins user to docker group 

 - sudo usermod -a -G docker jenkins 
 - sudo service jenkins restart 
 - sudo systemctl daemon-reload 
 - sudo service docker stop 
 - sudo service docker start 
	


# Step 5: install plug-ins inside your Jenkins UI
     Navigate to manage Jenkins  and click manage plug-ins
	- Select Available 
	- Search for docker( select docker and docker pipeline)
	- Search for Kubernetes ( Kubernetes cli )
	- search for slack (slack notification)
	- Install without restart.

# Step 6: start building the Jenkins pipeline
	 - Create maven3 variable under global tool configuration in Jenkins( to build the java file )
	 - Navigate to manage Jenkins and click on global tool configuration
	 - Scroll down and click on add maven
	 - Name: maven3
	 - Maven_home : /usr/share/maven
	 - Click save and apply
 #### Pipeline creation
    - Click on **New Item**
    - Enter an item name: **app-infra-pipeline** & select the category as **Pipeline**
    - Now scroll-down and in the Pipeline section --> Definition --> Select Pipeline script from SCM
    
# Step 7: Credentials setup(Slack):
   ## Navigate to your slack app and create a new channel and click create a channel
       1) Name: Ikechukwu-devops
         - Discription: Ikechukwu-devops
	 - Click on create
	 you can Add users by Name or Email Address
	 
       2) Click on the Slack Name on the Top left corner
       	  - Select Administration and then select manage App
	  - Search for Jenkins CI and click Add
	  - click on the dropdown, select your channel and click Add Jenkins CI integration             

     3)  #### Configure system:
            - Click on Manage Jenkins --> Configure System
            1)  - Go to section Slack
                - Workspace: **devops-fully-automated
                - Credentials: click on Add >> Jenkins >> select secret text >> under secret, copy and pasta the Token from the slcak setup page
		- ID: slack-token
		- Description: slack-token
		- click Add and then select secret text.
		- Default channel: Ikechukwu-devops, test connection and the click save and apply.

#### Create Credentials for connecting to Kubernetes Cluster using kubeconfig **

   	  After the cluster is created check the cluster
	  
	  `Kubectl get nodes`
	 
	  copy and store in a txt-file
	  
	  Navigate to Jenkins dashboard and click on manage Jenkins
	  
	  Manage credentials click on add credentials
	  
	  Select Secret file and select the file we just cpy
	  
        Add id and description k8s and k8s
	  
#### Access the Sonarqube server with the public IP:9000 and click login

	Username: admin
	
	Password: admin
	
	- select create a project 
	
	- project: JavaWebApp
	
	- Enter a name for your token: Jenkins and click generate and click continue
	
	- select java and select maven
	
	- copy the generated command and paste it on the Jenkinsfile at the sorna-scanner stage
	
	
### Step 8: GitHub webhook

1) #### Add jenkins webhook to github
    - Access your repo **cicd-deployment-with-JavaApp-Terraform** on github
    - Goto Settings --> Webhooks --> Click on Add webhook 
    - Payload URL: **htpp://REPLACE-JENKINS-SERVER-PUBLIC-IP:8080/github-webhook/**             (Note: The IP should be public as GitHub is outside of the AWS VPC where Jenkins server is hosted)
    - Click on Add webhook

2) #### Configure on the Jenkins side to pull based on the event
    - Access your jenkins server, pipeline **app-infra-pipeline**
    - Once pipeline is accessed --> Click on Configure --> In the General section --> **Select GitHub project checkbox** and fill your repo URL of the project devops-fully-automated.
    - Scroll down --> In the Build Triggers section -->  **Select GitHub hook trigger for GITScm polling checkbox**

Once both the above steps are done click on Save.
	
	
	
# Step 8, After the cluster is created we can setup the prometheus and grafana.

![promethue new](https://github.com/Ikechukwu980/cicd-deployment-with-JavaApp-Terraform/assets/106882590/e90cbae6-5786-4371-8085-a0822b463282)

#### install helm chart to the Jenkins server **
   
	`curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3`
	
	`sudo chmod 700 get_helm.sh`
	
	`sudo ./get_helm.sh`
	
	`helm version --client`
	
#### to add the helm sable chart to my local **

	`helm repo add stable https://charts.helm.sh/stable`
	
#### Add the prometheus repo **

	`helm repo add prometheus-community https://prometheus-community.github.io/helm-charts`
	
	`helm search repo prometheus-community`
	
#### create prometheus namespace **

	`kubectl create namespace prometheus`
	
#### install the kube-prometheus-stack **

	`helm install stable prometheus-community/kube-prometheus-stack -n prometheus`
	
	`kubectl get pods -n prometheus`
	
	`kubectl get svc -n prometheus`
	
#### Edit the prometheus and grafana service **

	`kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus`
	
	change the service to 'LoadBalancer'
	
	`kubectl edit svc stable-grafana -n prometheus`
	
	`kubectl get svc -n prometheus`

# STEP 9, Access the Grafana UI in the browser
	 - copy the LoadBalancer URL and paste it on the browser
	 - UserName: admin
	 - password: prom-operator
 
#### create a dashboard in grafana
	 - Click '+' button on left panel and select ‘Import’.
	 - Enter 12740 dashboard id under Grafana.com Dashboard.
	 - Click ‘Load’.
	 - Select ‘Prometheus’ as the endpoint under prometheus data sources drop down.
	 - Click ‘Import’.

#### Create POD Monitoring Dashboard
	 - Click '+' button on left panel and select ‘Import’.
	 - Enter 6417 dashboard id under Grafana.com Dashboard.
	 - Click ‘Load’.
	 - Select ‘Prometheus’ as the endpoint under prometheus data sources drop down.
	 - Click ‘Import






	

	


	





