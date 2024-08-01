# Jenkins Pipeline for Java based application using Maven, Docker, SonarQube, Kubernetes, and Argo CD

![Screenshot 2023-03-28 at 9 38 09 PM](https://user-images.githubusercontent.com/43399466/228301952-abc02ca2-9942-4a67-8293-f76647b6f9d8.png)


## Continious Deployment of Java Web App

## Project Description

This project provides step-by-step instructions for setting up a CI/CD pipeline using Jenkins, Docker, SonarQube, on an AWS EC2 instance and deployment of the Java Web Application to a Kubernetes Cluster via ArgoCD . The setup includes installing necessary tools, running SonarQube as a Docker container, configuring Jenkins with required plugins and credentials, and deploying the application to a Kubernetes cluster managed by ArgoCD.

Basic knowledge of installation and configuration of the below tools are a pre-requisite to implementing this project.

## Tools
- Jenkins
- Git
- EC2 Instance
- Docker
- Kubernetes (Minikube)
- SonarQube
- Argo CD

## Steps to Launch and Configure

### 1. Launch a t2.large EC2 Instance (Ubuntu)

### 2. Update System Cache
```bash
sudo apt update
```

### 3. Install Java-17
```bash
sudo apt install openjdk-17-jre-headless -y
```

### 4. Install Jenkins
```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins -y
```

### 5. Enable Jenkins Service to Start at Boot
```bash
sudo systemctl enable jenkins
```

### 6. Start Jenkins Service
```bash
sudo systemctl start jenkins
```

### 7. Check Status of Jenkins Service
```bash
sudo systemctl status jenkins
```

### 8. Install Docker
```bash
sudo apt update
sudo apt install docker.io -y
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu
sudo chmod 666 /var/run/docker.sock
sudo systemctl restart docker
```

### 9. Run SonarQube as a Container
```bash
docker run -d -p 9000:9000 --name sonarqube sonarqube:lts-community
```

### 10. Install Plugins on Jenkins Server
Log into Jenkins server and download the following plugings:

- SonarQube Scanner
- Docker Pipeline

### 11. Generate Authentication Token Named `jenkins` on SonarQube Server
This is to enable jenkins have access to the Sonarqube server to store report from `static code` analysis.

### 12. Configure Credentials on Jenkins Server
- `sonarqube` (token)
- `docker-cred` (username & password)
- `github` (token)

### 13. Configure Jenkins Pipeline
- Update `Static Code Analysis` stage with `SonarQube` server URL
- Include Repository URL `https://github.com/slimboi/Continious-Deployment-of-Java-Web-App.git`
- Add path to Jenkinsfile `java-maven-sonar-argocd-helm-k8s/spring-boot-app/JenkinsFile`

### 14. Restart Jenkins
```bash
http://server_ip:8080/restart
```

### 15. Start Kubernetes Cluster Locally
```bash
minikube start
```

### 16. Install Argo CD Operator
```bash
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.28.0/install.sh | bash -s v0.28.0
```

### 17. Create Argo CD Controller
```yaml
apiVersion: argoproj.io/v1beta1
kind: ArgoCD
metadata:
  name: example-argocd
  labels:
    example: basic
spec: {}
```
```bash
kubectl apply -f argocd-basics.yml
```
```bash
kubectl create -f https://operatorhub.io/install/argocd-operator.yaml
```

### 18. Change Argo CD Server from ClusterIP to NodePort
```bash
kubectl edit svc example-argocd-server
```

### 19. Use Minikube to Generate a URL to Access Argo CD via Browser
```bash
minikube service example-argocd-server
```

### 20. Get Initial Admin Password
```bash
kubectl edit secret example-argocd-cluster
```

### 21. Decrypt Password
```bash
echo 'password' | base64 -d
```

### 22. Log into Argo CD with the Username Admin and Decrypted Password

### 23. Configure Argo CD to Fetch Manifests from Repo and Deploy Application to Kubernetes Cluster

### 24. Enable port forwarding so app can be accessible via browser
```bash
kubectl port-forward --address 0.0.0.0 svc/spring-boot-app-service 9090:80
```
### 25. View deployed application on browser
```bash
http://host_ip:9090/
```
---
By following these instructions, you will have a fully functional CI/CD pipeline. This end-to-end Jenkins pipeline will automate the entire CI/CD process for a Java application, from code checkout to production deployment, leveraging Jenkins, Docker, SonarQube, and ArgoCD for continuous integration, static code analysis, and continuous deployment to a Kubernetes cluster.
