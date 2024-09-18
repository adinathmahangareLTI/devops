## Step 1: Set Up Jenkins ##
1. Launch an EC2 Instance:
- Go to the AWS Management Console and launch an EC2 instance.
- Choose an Amazon Machine Image (AMI), such as Ubuntu.
- Configure security groups to allow HTTP, HTTPS, and SSH access.
2. Install Jenkins:
- SSH into your EC2 instance.
- Install Java:
```
sudo apt update
sudo apt install openjdk-11-jdk -y
```
3. Add Jenkins repository and install Jenkins:
```
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y
```
4. Start Jenkins:
```
sudo systemctl start jenkins
sudo systemctl enable jenkins
```
5. Access Jenkins:
- Open a web browser and go to http://<your-ec2-public-ip>:8080.
- Follow the instructions to unlock Jenkins and complete the setup.


## Step 2: Configure GitHub Repository ##
1. Create a GitHub Repository:
- Create a new repository on Github for your Java web application.
2. Add Webhook:
- Go to your GitHub repository settings.
- Add a webhook with the Jenkins URL: http://<your-ec2-public-ip>:8080/github-webhook/.


## Step 3: Build a Sample Java Web App Using Maven ##
1. Create a Maven Project:
- On your local machine, create a new Maven project:
```
mvn archetype:generate -DgroupId=com.example -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false
```

2. Push to GitHub:
- Initialize Git and push the project to your GitHub repository:

```
git init
git add .
git commit -m "Initial commit"
git remote add origin <your-github-repo-url>
git push -u origin master
```


## Step 4: Configure Jenkins Pipeline ##
1. Install Plugins:
- Install the following Jenkins plugins: GitHub, Maven Integration, Docker, Kubernetes, Amazon ECR.
2. Create a Pipeline Job:
- In Jenkins, create a new pipeline job.
- Configure the pipeline script (Jenkinsfile) to include stages for building, testing, and deploying the application.


## Step 5: Build and Push Docker Image to ECR ##
1. Create an ECR Repository:
- Go to the AWS Management Console and create a new ECR repository.
2. Configure Jenkinsfile:
- Add stages to build the Docker image and push it to ECR:

```
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your-repo-url.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    dockerImage = docker.build("your-ecr-repo-url:latest")
                    docker.withRegistry('https://your-ecr-repo-url', 'ecr:us-east-1:aws-credentials') {
                        dockerImage.push()
                    }
                }
            }
        }
    }
}
```

## Step 6: Deploy to Kubernetes ##
1. Set Up EKS Cluster:
- Create an EKS cluster using the AWS Management Console or AWS CLI.
2. Configure kubectl:
- Install and configure kubectl to interact with your EKS cluster.
3. Create Kubernetes Deployment:
- Create a deployment YAML file using the ECR image:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: <your-ecr-repo-url>:latest
        ports:
        - containerPort: 8080
```

4. Apply Deployment:
- Apply the deployment to your EKS cluster:
```
kubectl apply -f deployment.yaml
```

## Step 7: Verify Deployment ##
1. Check Pods:
- Verify that the pods are running:
```
kubectl get pods
```

2. Access the Application:
- Expose the deployment as a service and access the application using the service URL.
  
