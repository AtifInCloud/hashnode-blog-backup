---
title: "How to Implement DevSecOps for WANDERLUST: A Complete Deployment Guide"
seoTitle: "Implementing DevSecOps: A Complete Guide"
seoDescription: "Comprehensive guide on implementing DevSecOps for Wanderlust with Jenkins, SonarQube, OWASP, and Trivy for secure and efficient CI/CD deployment"
datePublished: Thu Aug 15 2024 06:12:51 GMT+0000 (Coordinated Universal Time)
cuid: clzuvvy3z000209kw2i5rfvtp
slug: how-to-implement-devsecops-for-wanderlust-a-complete-deployment-guide
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1723701853300/16e78130-3490-4104-9d9b-8f202aef4840.jpeg
tags: owasp, sonarqube, jenkins, devsecops, trivy, cicd-complete-proccess

---

The goal of this project was to implement a robust CI/CD pipeline using Jenkins to automate the build, test, and deployment processes for the Wanderlust application. This pipeline was designed to ensure quality and security at every stage, from code commit to production deployment.

The key stages of the pipeline include:

* **Cloning the Code:** The code is automatically pulled from a GitHub repository.
    
* **SonarQube Quality Analysis:** The code undergoes quality checks to identify bugs, vulnerabilities, and code smells.
    
* **OWASP Dependency Check:** This stage scans for known vulnerabilities in third-party libraries.
    
* **Trivy Vulnerability Scanning:** The final security check scans the application’s Docker images for vulnerabilities.
    
* **Deployment:** The application is deployed using Docker Compose.
    

Throughout the project, several challenges were overcome, such as handling node module dependency issues, integrating SonarQube with Jenkins, and resolving Docker Compose version conflicts. This setup improved deployment efficiency and ensured consistent application quality by automating the entire process.

### Key Technologies

* **Jenkins:** Automates the CI/CD process, from code commit to deployment.
    
* **SonarQube:** Ensures code quality by identifying bugs, vulnerabilities, and code smells.
    
* **OWASP Dependency Check:** Scans third-party libraries for known vulnerabilities.
    
* **Trivy:** Scans Docker images for vulnerabilities.
    
* **Docker & Docker Compose:** Containerizes and deploys the application.
    
* **Node.js, MongoDB, Redis:** The core technologies of the Wanderlust application.
    

### Step 1: Setting Up the EC2 Instance

To get started, an EC2 instance was set up with an allocated elastic IP address. Docker and Docker Compose were then installed:

```bash
sudo apt-get update && sudo apt-get install -y docker.io
sudo apt-get install docker-compose -y
sudo usermod -aG docker $USER
```

### Step 2: Setting Up Jenkins

Jenkins was installed and configured on the EC2 instance to serve as the CI/CD orchestrator. The installation steps include adding the Jenkins package repository and installing Jenkins:

```bash
sudo apt update
sudo apt install fontconfig openjdk-17-jre
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723700145214/093b15cc-eaaa-45d9-a489-9022ac5a7852.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723700155724/afce9a7f-c80b-4ee2-803f-962979c48cda.png align="center")

### Step 3: Setting Up SonarQube

SonarQube was set up in a Docker container to handle the code quality checks:

```bash
docker run -itd --name sonarqube-server -p 9000:9000 sonarqube:lts-community
```

SonarQube is an open-source tool that helps in maintaining clean, secure, and maintainable code by analyzing the codebase for potential issues.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723700255801/9e926c7a-e33b-4406-b12d-0794d0bfcb22.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723700267413/bfb73d26-7817-4573-b225-a709bfe725b0.png align="center")

### Step 4: Setting Up Trivy

Trivy, a vulnerability scanner for containers, was installed to ensure the security of Docker images:

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release -y && \
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | \
gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null && \
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | \
sudo tee /etc/apt/sources.list.d/trivy.list && \
sudo apt-get update && sudo apt-get install trivy -y
```

Trivy was used to scan Docker images for vulnerabilities before deployment

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723700309926/2c86caff-f281-4760-a558-7cf92011c2cd.png align="center")

### Step 5: Integrating Jenkins and SonarQube

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723700432339/77b000a0-940a-4a37-97e1-e18210526b1d.png align="center")

To ensure that our CI/CD pipeline effectively integrates SonarQube for continuous code quality checks, we need to configure both SonarQube and Jenkins to communicate seamlessly.

#### Step 1: Configuring SonarQube

1. **Navigate to Administration**: Start by logging into your SonarQube instance. Navigate to the **Administration** tab to access various settings.
    
2. **Set Up Webhook**: In the **Configuration** section, locate the **Webhooks** option. Here, you'll need to create a new webhook that will notify Jenkins whenever a SonarQube scan completes. This allows Jenkins to take further action based on the scan results.
    
3. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723700549372/46a5f8ef-109a-48bc-a2c3-911018b2fc06.png align="center")
    

#### Step 2: Generating a Security Token in SonarQube

1. **Create a Token**: Since Jenkins requires secure communication with SonarQube, you must generate a security token. In the SonarQube **Administration** section, go to the **Security** tab.
    
2. **Generate and Save the Token**: Click on the option to create a new token. Enter a descriptive name for the token, generate it, and make sure to save it securely. This token will be used in Jenkins to authenticate the connection.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723700562041/4c150071-99a9-4325-a6ee-7f2b66decab4.png align="center")
    

#### Step 3: Configuring Jenkins

1. **Integrate with SonarQube**: In Jenkins, use the SonarQube token you generated to establish a secure connection. This ensures that Jenkins can trigger SonarQube scans and receive the results directly within the CI/CD pipeline.
    

With this setup, SonarQube is now fully integrated with Jenkins, allowing automatic code quality analysis during the CI/CD process.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723700572483/31fab34f-7654-4008-a9e5-7f217713d93a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723700580421/40f004ae-0b42-419a-808f-10a3514e2326.png align="center")

### Integrating OWASP Dependency Check

Following the successful integration of SonarQube with Jenkins, the next step is to add OWASP Dependency Check to the pipeline. This tool helps in identifying vulnerabilities in third-party libraries used by your application.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723700588316/48e042ed-4f4f-41aa-91a5-bc2418ffc025.png align="center")

### Step 6: Setting Up the Jenkins Pipeline

With Jenkins and SonarQube integrated, it was time to set up the Jenkins pipeline that would orchestrate the entire CI/CD process. Two types of Jenkins pipelines were considered: Scripted (Static) and Declarative. Given the project's needs, a Declarative Pipeline was chosen due to its structured and easy-to-maintain syntax.

### Jenkins Pipeline

```bash
groovyCopy codepipeline {
    agent any
    environment {
        SONAR_HOME = tool "Sonar"
    }
    
    stages {
        // Stage 1: Clone Code from GitHub
        stage("Clone Code from Github") { 
            steps {
                git url: "https://github.com/krishnaacharyaa/wanderlust.git", branch: "devops"
            }
        }

        // Stage 2: SonarQube Quality Analysis
        stage("SonarQube Quality Analysis") {
            steps {
                withSonarQubeEnv("Sonar"){
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=wanderlust -Dsonar.projectKey=wanderlust -Dsonar.sources=./"
                }
            }
        }

        // Stage 3: OWASP Dependency Check
        stage("OWASP Dependency Check") {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        // Stage 4: Sonar Quality Gate Scan
        stage("Sonar Quality Gate Scan") {
            steps {
                timeout(time: 2, unit: java.util.concurrent.TimeUnit.MINUTES) {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        // Stage 5: Trivy File System Scan
        stage("Trivy File System Scan") {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }

        // Stage 6: Deploy using Docker Compose
        stage("Deploy using Docker Compose") {
            steps {
                sh "docker-compose up -d"
            }
        }
    }
}
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723700772355/04550c0d-70ec-45ec-b7b3-02ed78411876.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723700817260/6c9eccee-3397-40df-bde1-98e234c12431.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723700821757/e32dfe4d-e8bc-43d7-aee7-8833ab04108b.png align="center")

![The landing page of the Wanderlust app, inviting users to explore travel stories and create posts.](https://cdn.hashnode.com/res/hashnode/image/upload/v1723700860140/8aecbce6-21d9-4a6a-a1c9-1f89e6d71698.png align="center")

### Key Learnings & Insights

1. **Challenge:** **Integrating SonarQube with Jenkins**
    
    * The integration required secure communication, which was achieved by setting up a webhook in SonarQube and generating a security token. This ensured seamless quality analysis within the CI/CD pipeline, though the process involved navigating complex configurations.
        
2. **Challenge:** **Scaling Jenkins Resources**
    
    * As the project grew, the initial EC2 instance (t2.medium) became insufficient, leading to slow builds and frequent timeouts. Scaling up to a t3.large instance provided the necessary resources to handle the increased load, resulting in improved performance and reliability of the pipeline.
        

### Conclusion

This project successfully demonstrated the power of integrating DevSecOps practices into the CI/CD pipeline for the Wanderlust application. By automating code quality checks, vulnerability scans, and deployment processes, we ensured that the application not only met high standards of quality but also maintained robust security. The integration of tools like Jenkins, SonarQube, OWASP Dependency Check, and Trivy into the pipeline proved essential in achieving these goals.

If you're interested in exploring the project further or want to see more of my work, feel free to check out my GitHub account [AtifinCloud](https://github.com/AtifInCloud).