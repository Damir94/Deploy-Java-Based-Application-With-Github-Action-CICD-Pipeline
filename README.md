# Deploy a Java-Based Application With GitHub Action CI/CD Pipeline

<img width="1012" height="362" alt="Screenshot 2026-10-10 at 10 39 52 AM" src="https://github.com/user-attachments/assets/8d7712cd-ad5d-4085-9b97-b7d88a4f8f97" />

## Introduction
- In this document, I will demonstrate how to deploy a Java-based application using GitHub Action Pipeline. In this, we have the concept of slave called runner. There will be some kind of server or virtual machine where the application is going to be built and whatever commands, we are going to be running will get executed.

- In GitHub Action, we have two options for runners. One option is shared runner and the second option is private runner.

- The benefit of shared runner is that it is completely free. It will be provided by GitHub, we do not have to do anything, all we need to do is to provide a name. For example, I want to run my commands on ubuntu. I just have to write “ubuntu-latest”. That means all my commands on a runner that is of configuration ubuntu latest. If I want to run on Windows, I will define “windows-latest”. The advantage of shared runner is that it is completely free and the disadvantage is that you won’t be having access to backend of it.

- In Private runner, we bring our own virtual machine and then add the virtual machine as a runner. The disadvantage here is that you have to pay for it. And the benefit is that it will be complete isolated and private to you only and your company. Secondly, you will be having complete control or complete access over it. So, most companies prefer private runner because they can use their own virtual machine and have complete control over it and make sure that is completely isolated. 

In this project, we are going to set up our own virtual machine and we are going to work with it. We are going to follow these steps in this project.
  - Set up the GitHub repository
  - Create a Virtual Machine and add as a runner
  - Create the CI/CD Pipeline
  - Security checks with Trivy
  - Test application with Maven
  - Build application and publish the artifacts
  - Build and scan Docker image
  - Push the docker image to Docker hub
  - Deploy the application to Kubernetes cluster

### Create a Workflow and add .yml file

- We have to create a workflow and add the .yml pipeline configuration file. Workflows in GitHub Action is like Pipeline in Jenkins. This involves creating a folder called “.github”, and then create another folder called “workflows” in the “.github” folder. Then create a workflow .yml file called cicd.yml.
- Let us create the folder “.github” in our GitHub repository.
- Click on the drop down on “Add File"
- Select “create new file”
- Enter the folder name “.github” by entering “.github”
- Enter the name of the next folder called “workflows” by entering “/workflows”
- Then create the workflow file called “cicd.yml” by entering “/cicd.yml”
- Then commit the changes by clicking on “Commit Changes”
- Click on “Commit Changes” again

<img width="1906" height="295" alt="Screenshot 2026-10-10 at 10 52 07 AM" src="https://github.com/user-attachments/assets/a0184c2c-fb0b-4f2a-a3d3-c14095850052" />

- Go back to our repository

<img width="1194" height="430" alt="Screenshot 2026-10-10 at 10 52 39 AM" src="https://github.com/user-attachments/assets/2734b126-0b64-485a-b7b2-f99f27484050" />

### Adding Jobs to the Pipeline Configuration file

- Now, we can add the jobs. Jobs are like steps that are used to perform particular tasks. Our pipeline can have multiple jobs. Each job will be running in an isolated environment. We will add the jobs namely build, test and deploy to the pipeline file. We can now edit our workflow file cicd.yml.
- Click on the Pipeline folder .github
- Then, open the “cicd.yml” file by clicking on it.
- We can now start editing the file. Click on “Edit”
- Yaml is indentation sensitive. So, it is advisable to write the code of the workflow file on VSCode

### Setting up the Pipeline

- Here we will give the pipeline a name and add the events that will trigger the pipeline.
- We can now start populating the file with code. First, we will give the Pipeline a name. We will call it “CICD Pipeline”
```bash
name: CICD Pipeline
```

### Add a Branch

- Then we have to add an event that will trigger the workflow. Our event will be “push”, whenever you push a code to our “master” branch, this will trigger the workflow for the Pipeline to start running.
- So, we have to add the main branch to our code as follows:
```bash
on:
 push:
 branches: ["main"]
```

### Start adding Jobs to the Pipeline file

- We will add the jobs of the pipeline here.
- Now, let us add the code for our first job called “compile”. Then we assign the shared runner, in this case we are using “ubuntu”. So, we will assign “ubuntu-latest” to use the latest version of ubuntu.
```bash
jobs:
 compile:
 runs-on: ubuntu-latest
```
- The first step in this job is to compile our source code using the actions “actions/checkout@v4” as first action which pulls a copy of the GitHub repository, “actions/setup-java@v4” as the second action in our first job which sets up environment for installing Java JDK.

```bash
steps:
 - uses: actions/checkout@v4
```

- In the second step in this job, we define the version of JDK we want to install, in this case we want to install JDK 17 from temurin.
```bash
- name: Set up JDK 17
 uses: actions/setup-java@v4
 with:
 java-version: '17'
 distribution: 'temurin'
 cache: maven
```
- The last thing in this job is to define the tool used for the build. Since it is a Java-based project, we will use Maven. So, we add the line with “name: Build with Maven” and add the command to compile with maven “run: mvn compile”
```bash
jobs:
  compile:
    runs-on: ubuntu-latest

 steps:
 - uses: actions/checkout@v4
 - name: Set up JDK 17
   uses: actions/setup-java@v4
   with:
     java-version: '17'
     distribution: 'temurin'
     cache: maven
 - name: Build with Maven
   run: mvn compil
```
- We have added the stage ti compile our source code

### Add the “security-check” job

- In the second job, we will perform the security check. We will call this job “security-check”. This job will run after the first job, they do not have to run simultaneously. That is the jobs will run sequentially. It will need the first job to be completed first, so we add the line “needs: compile”
```bash
security-check:
 runs-on: ubuntu-latest
 needs: compile
```
- Then since we are running the jobs separately, we have to check out the code again. We have to add the step to check out the code.
```bash
steps:
 - uses: actions/checkout@v4
```
- The next step in this job is to install Trivy, since Trivy might not be installed in the shared runner. We will call the step “Trivy Installation”. Then, the next part of this step is to run the command to install Trivy. Since it is multiple lines of command, we use the pipe (|).
```bash
- name: Trivy Installation
        run: |
          sudo apt-get install -y wget apt-transport-https gnupg lsb-release
          wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
          echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
          sudo apt-get update -y
          sudo apt-get install -y trivy
```

- This is going to install Trivy on our shared runner. The second step in this job is to scan the Java code and its dependencies for vulnerabilities using Trivy. We will call this step “Trivy FS Scan” and the command to scan the code will be “run: trivy fs”.

```bash
- name: Trivy FS Scan
 run: trivy fs --format table -o fs-report.json .
```

- The next step is to perform Gitleaks. Gitleaks is a tool that is going to find in your source code if you have any kind of sensitive data hard coded for example API tokens, secret key, access key which should not be hard coded in the source code. So, we have to install Gitleaks.
- We will call this step “Gitleaks Installation”, the run the command to install Gitleaks “run: sudo apt install gitleaks -y”
```bash
- name: Gitleaks Installation
  run: sudo apt install gitleaks -y
```

- Finally, we will add the step is to perform scanning with Gitleaks. We will call the step “Gitleaks Code Scan” and run the command using this line of code “run: gitleaks detect source . -r gitleaks-report.json -f json”
```bash
- name: Gitleaks Code Scan
  run: gitleaks detect source . -r gitleaks-report.json -f json
```

### Add the “test” job

- In the third job, we will perform the test. We will call this job “test”. The test will be done with Maven. This job will run after the second job, they do not have to run simultaneously or parallelly. That is the jobs will run sequentially. It will need the first job to be completed first, so we add the line “needs: security-check”
```bash
test:
 runs-on: ubuntu-latest
 needs: secur
```
- Then since we are running the jobs separately, we have to check out the code again. We have to add the step to check out the code
```bash
steps:
   - uses: actions/checkout@v4
```
- In the second step in this job, we define the version of JDK we want to install, in this case we want to install JDK 17 from temurin.
```bash
 - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven
```

<img width="1874" height="899" alt="Screenshot 2026-10-10 at 12 06 49 PM" src="https://github.com/user-attachments/assets/82675de5-a459-472e-bea3-2ee0af724225" />

<img width="1875" height="579" alt="Screenshot 2026-10-10 at 12 06 55 PM" src="https://github.com/user-attachments/assets/cb38e1ea-29e7-426d-b53c-2e48990b94d8" />

### Create a “runner”
- We have to create an Ubuntu EC2 instance and SSH connect to it.
- We will launch an ubuntu EC2 instance called “runner”. Go to EC2 dashboard
- Click on “Launch Instance”
- Give the instance a name, I will call it “runner”
- On “AMI”, I will select “ubuntu”
- On “Instance Type”, select “t2.medium”


- Scroll down to “Key Pair”
- Click on “Create new key pair”
- Enter the name of the key pair, we will call it “runner-key”
- Then click on “create key pair”


- Then scroll down to “Network settings” and select “Allow SSH traffic from”, “Allow HTTPS traffic from internet”, “Allow HTTP traffic from internet”


- One “Configure storage”, make it “20” GiB
- Then click on “Launch Instance”
- Click on “Instances”
- You can see that the instance is initializing. Wait for it to pass the “2/2 check”
- You can see that it has passed the “2/2 check”

### SSH Connect to the Instance

- We have to SSH connect to the EC2 instance we just created
- Select the instance
- Click on “Connect”
- Copy the command:
```bash
ssh -i "runner-key.pem" ubuntu@ec2-174-129-114-216.compute-1.amazonaws.com
```
- Open Terminal and navigate to where your Key Pair .pem file is save. In my case it is saved in the Downloads folder.
- Paste the copied command here and press enter
- Enter “yes” and press “Enter”
- We are now connected to the instance.

### Install Tools

- We have to install all the tools we need in this project as shown on the architecture.

#### Install Maven
- We have to first update the package using the command:
```bash
sudo apt update
```
- Then install Maven using the command:
```bash
sudo apt install maven -y
```

#### Install unzip
- We have to install unzip but ley us first update the package using the command:
```bash
sudo apt-get update
```
- Then, run the command to install unzip
```bash
sudo apt-get install unzip
```

### Adding Private Runner

- In this step we are going to add a virtual machine to serve as our private runner. To do this, we have to go to our GitHub repository
- Get the commands from the GitHub Repository
- Click on “Settings”
- Click on the drop down on “Actions”
- Select “Runners”
- Click on “New self-hosted runner”
- We are going to use “Linux”, so select “Linux”
- Now, we have to run these commands on a virtual machine. So, we have to launch an EC2 instance to server as our virtual machine

#### Download

```bash
# Create a folder
$ mkdir actions-runner && cd actions-runner
# Download the latest runner package
$ curl -o actions-runner-linux-x64-2.329.0.tar.gz -
L https://github.com/actions/runner/releases/download/v2.329.0/actions-runnerlinux-x64-2.329.0.tar.gz
# Optional: Validate the hash
$ echo "194f1e1e4bd02f80b7e9633fc546084d8d4e19f3928a324d512ea53430102e1d actionsrunner-linux-x64-2.329.0.tar.gz" | shasum -a 256 -c
# Extract the installer
$ tar xzf ./actions-runner-linux-x64-2.329.0.tar.gz
```

#### Configure
```bash
# Create the runner and start the configuration experience
$ ./config.sh --url https://github.com/ebotsidneysmith/Java-Based-GithubAction --
token BVAKISJXU3CWNDDV6MEXZF3I7BZ42
# Last step, run it!
$ ./run.sh
```

#### Using your self-hosted runner
```bash
# Use this YAML in your workflow file for each job
runs-on: self-hosted
```
- We have to now run these commands on the terminal of our EC2 instance “runner”
- We will first update the package using the command:
```bash
sudo apt update
```
- Create a folder called “actions-runner” using the command:
```bash
mkdir actions-runner
```
- Navigate to the created folder using the command:
```bash
cd actions-runner
```
- Download the latest runner package using this command:
```bash
curl -o actions-runner-linux-x64-2.329.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.329.0/actionsrunner-linux-x64-2.329.0.tar.gz
```
- Extract the installer using this command:
```bash
tar xzf ./actions-runner-linux-x64-2.329.0.tar.gz
```
- List the content of the folder using the command:
```bash
ls
```
- I am going to remove the action package using the command:
```bash
rm actions-runner-linux-x64-2.329.0.tar.gz
```
- Run the command to check the content again
```bash
ls
```
- Create the runner and start the configuration experience
```bash
./config.sh --url https://github.com/ebotsidneysmith/Java-Based-GithubAction --token BVAKISJVV6D35BC3FBVSI33I7FQMW
```
- In case you have this error, it means your runner is offline and it is not running. To resolve this go back to your runner on your GitHub.
- You can see the runner is offline. Click on “New self-hosted runner”
- Select “Linux”
- Copy the above configure command:
```bash
./config.sh --url https://github.com/ebotsidneysmith/Java-BasedGithubAction --token BVAKISIBMQSCEWTHG6XYTZDI7RFTW
```
- Then run this command on your “runner” terminal
- We do not have a runner group, so just press “Enter”
- For the name of the runner, enter “Runner-1” and press “Enter”
- For label, we will use “self-hosted”. Type “self-hosted” and press “Enter”
- We will use the default work folder, so just press “Enter”
- The runner is saved. So, we have set up our runner. Let us now start the runner using the command:
```bash
./run.sh
```
- We are now connected to GitHub. We can now go to our pipeline code and change all the “ubuntulatest” to “self-hosted”.
- Commit the changes by clicking on “Commit changes”
- Click on “commit changes” again
- Then click on “Actions”
- Click on the updated pipeline “Update cicd.yml"
- You can see that the jobs have started running. Wait for it to run all the jobs
- You can see that the three jobs are successful. Head back to our EC2 instance terminal
- You can see that the runner is able to pick up our jobs

### Adding the remaining jobs / stages

- Now, we have to continue adding the remaining jobs to our Pipeline code. These jobs will be for code analysis with SonarQube, build and push docker image with Docker and finally deploying image to Kubernetes.

#### Adding “build_project_and_sonar_scan” job

- We will now add the job to build the code using Maven and perform code analysis with SonarQube. To do this, we will head back to VScode
- Then let us add the code for the code analysis job. We will call the job “build_project_and_sonar_scan” that will run on our self-hosted runner, so we add the line “runs-on: self-hosted” and this job will need the test job. So, we add the line “needs: test”.
```bash
build_project_and_sonar_scan:
 runs-on: self-hosted
 needs: test
```
- Now, let us add the steps of this job. As usual, the first step is to fetch or check out the code using the line of code:
```bash
steps:
 - uses: actions/checkout@v4
```

- The next step is to install Java JDK 17 using the code
```bash
- name: Set up JDK 17
  uses: actions/setup-java@v4
  with:
    java-version: '17'
    distribution: 'temurin'
    cache: maven
```

- Now, let us add the code of the step to build the code using Maven
```bash
- name: Build Project
  run: mvn package
```

- Next, we are going to create another action which will upload an artifact. So, we will be uploading the artifact, while in the next job we will be downloading the artifact.
```bash
- name: Upload JAR artifact
  uses: actions/upload-artifact@v4
    with:
    name: app-jar
    path: target/*.jar
```
- Before we add the code for code analysis with SonarQube, we have to first of all Launch an EC2 instance called “sonar-server” for SonarQube and install SonarQube on it.

### Creating SonarQube Virtual Machine
- We have to create a Ubuntu EC2 instance for SonarQube, SSH Connect to it and install SonarQube
- We will call the instance “sonar-server”, AMI will be “ubuntu”, Instance type will be “t2.medium” and will enable port 9000 on the Network settings. Then configure storage will be “20” GiB.
- The sonar-server has been created, let us now add port 9000. Select the instance “sonar-server”

- Click on “Security” tab
- Click on the “security Group” url
- Click on “Edit inbound rules”
- Click on “Add rule”
- Enter “9000”
- Click on the drop down and select “Anywhere-IPv4”
- Click on “Save rules”
- Port 9000 has been added

### SSH Connect to the Instance
- We will now SSH connect to the sonar-server instance
- Select the instance
- Click on “Connect”
- Copy the command:
```bash
ssh -i "runner-key.pem" ubuntu@ec2-54-224-99-24.compute-1.amazonaws.com
```
- Open Terminal and navigate to where your Key Pair .pem file is save. In my case it is saved in the Downloads folder.
- Paste the copied command here and press enter
- Enter “yes” and press “Enter”
- We are now connected to the instance.


### Install Docker
- Let us install SonarQube now but we have to first update the package using the command:
```bash
sudo apt update
```
- After this we will install Docker which will help set up SonarQube inside a Docker container. Run the command:
```bash
docker
```
- To install docker, we will run the command
```bash
sudo apt install docker.io -y
```
- By default, only root user has permission to execute docker commands. So, once you have installed docker, you need to make sure that the user has permission to execute docker commands. For that we have to add a user using the command:
```bash
sudo usermod -aG docker $USER
```
- We need to run a command which will make sure these changes are applied and they are reflecting:
```bash
newgrp docker
```

### Install SonarQube
- Then we run the command to set up docker container with the name “sonar”
```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

### Access SonarQube on Browser and configure it
- Let us now try to access SonarQube on the browser using: That is http://<Public IPv4 address>:9000

- Let is enter the username and password. Username is “admin” and password is also “admin”
- Click on “Log in”
- Let us modify the password
- Click on “update”
- We have to generate the SonarQube token.
- Click on “Administration” tab
- Click on the drop down on “security” tab
- Select “users”
- Click on “update token”
- Enter the token name, I will call it “sonar-token”
- Click on “Generate”
- Copy the key and save it somewhere
- Then click on “Done”

### Create the file “sonar-project.properties” in our GitHub repository
- We have to create a file called “sonar-project.properties”
- Click on the drop down on “Add file”
- Select “create new file”
- Enter the file name as “sonar-project.properties”
- Enter the following code:
```bash
sonar.projectKey=GC-Bank
sonar.projectName=GC-Bank
sonar.java.binaries=.
```
- Commit the changes by clicking on “commit changes”
- Click on “commit changes” again
- We have added the file.

### Adding Actions related to SonarQube
- Let us now go to our pipeline and add the job for SonarQube. First search on google for “SonarQube Action”
- Click on “SonarSource/sonarqube-scan-action”
- Scroll down
- Copy this and add on your code and add to your pipeline code
```bash
- uses: actions/checkout@v4
  with:
    # Disabling shallow clones is recommended for improving the relevancy of reporting
    fetch-depth: 0
- name: SonarQube Scan
  uses: SonarSource/sonarqube-scan-action@v6.0.0
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
    SONAR_HOST_URL: ${{ vars.SONAR_HOST_URL }}
```

- You can get the latest version by going to
```bash
https://github.com/marketplace/actions/official-sonarqube-scan
```
- You can see the latest version is 6.0.0. We have to change the version on the pipeline code to 6.0.0. So, we modify the code as follows:
```bash
- uses: actions/checkout@v4
 with:
   # Disabling shallow clones is recommended for improving the relevancy of reporting
   fetch-depth: 0
- name: SonarQube Scan
 uses: SonarSource/sonarqube-scan-action@v6.0.0 # Ex: v4.1.0, See the latest version at https://github.com/marketplace/actions/official-sonarqube-scan
 env:
   SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
   SONAR_HOST_URL: ${{ vars.SONAR_HOST_URL }}
```

### Adding the Action / Steps “SonarQube Quality Check”
- Now, let us add a step for SonarQube quality gate check. Go to google and search for “SonarQube Quality Check”
- Click on “SonarSource/sonarqube-quality-gate-action”
- Copy the code for Quality Gate check
```bash
- name: SonarQube Quality Gate check
  id: sonarqube-quality-gate-check
  uses: sonarsource/sonarqube-quality-gate-action@master
  with:
   pollingTimeoutSec: 600
  env:
   SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
   SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }} #OPTIONAL
```
- We can now transfer this code to our Pipeline file in the GitHub repository and commit the changes

### Adding SonarQube Secret and Variables on GitHub Action
- Now we have two important things to add, that is Token and URL. We have to add “SONAR_TOKEN” and “SONAR_HOST_URL” to the settings in our GitHub repository.

#### Part 1: Adding the secret “SONAR_TOKEN”
- Go to our GitHub repository
- Click on “settings”
- Click on the drop down on “Secrets and Variables”
- Select “Actions”
- Click on “New repository secret”
- On “name”, enter “SONAR_TOKEN”
- On “Secret”, enter the SonarQube key we copied above.
- Then click on “Add Secret”
- We have added a secret.

### Adding the Variable “SONAR_HOST_URL”
- Let us add the variable that is “SONAR_HOST_URL”.
- Select the “Variable” tab
- Click on “New repository variable”
- On “name”, enter “SONAR_HOST_URL”
- On “Value”, go to the SonarQube browser
- Copy the highlighted part and paste on the “value” field in GitHub
- Click on “Add Variable”
- We have added our variables.

### Test the jobs that have been added
- Let us test the jobs that we have added so far by running the pipeline.
- We can now transfer this code to our Pipeline file in the GitHub repository and commit the changes
- Click on “Actions”
- Click on “Update cicd.yml”, make sure the runner is running
- You can see that the pipeline is successful. Go to SonarQube browser and click on “Projects”
- Click on “GC-Bank”
- You can see that the analysis has been done and it works fine.

### Adding the remaining jobs
- We will be adding the remaining jobs in this step

#### Adding “build_docker_image_and_push” job
- We will now add the job to build and push the docker image using Docker. To do this, we have to first install Docker on our runner.
- Go to the terminal of our “runner”.
- Check if docker is install by using the command:
```bash
docker
```
- You can see that Docker has not been installed on our “runner”.

### Install Docker
- Let us now install docker.
- Go to google and search for “docker install ubuntu”
- Click on “Install Docker Engine on Ubuntu”
- Copy these commands
```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o
/etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
# Add the repository to Apt sources:
echo \
 "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc]
https://download.docker.com/linux/ubuntu \
 $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
 sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
- Run the copied commands on the terminal of our runner EC2 instance
- We will now run this command to install Docker
- Then copy the above command
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
- Run it too on the terminal of the runner EC2 instance
- Type “Y” and press “Enter”
- Docker has been installed; we have to set permissions so that it can execute docker commands. We do this by running the command:
```bash
sudo usermod -aG docker ubuntu
```
- In order to apply changes, we run the command:
```bash
newgrp docker
```
- Now, we have to add the code of the job to build and push the docker image. We head back to VSCode
- Adding the job to build and push docker image. Search for “actions build and push docker image”
- Click on “Build and push Docker images – Actions”
- Scroll down
- Copy this code and modify it
- he name of the job is “build_docker_image_and_push”. It will run on our self-hosted runner, so we will add the like “runs-on: self-hosted”. This job depends on the previous job that is “build_docker_image_and_push”, so we add the line “needs: build_project_and_sonar_scan”.
```bash
build_docker_image_and_push:
    runs-on: self-hosted
    needs: build_project_and_sonar_scan
```
- The next thing is to add the steps. The first step is check out or fetch our code
```bash
steps:
      - uses: actions/checkout@v4
```
- The second step is to download the Jar artifacts that was uploaded in the previous stage / job.
```bash
- name: Download JAR artifact
        uses: actions/download-artifact@v4
        with:
          name: app-jar
          path: app  # this will download JAR to ./app folder
```
- The next step is to login to the docker hub.
```bash
 - name: Login to Docker Hub
   uses: docker/login-action@v3
   with:
      username: ${{ vars.DOCKERHUB_USERNAME }}
      password: ${{ secrets.DOCKERHUB_TOKEN }}
```
- Then we set up qemu
```bash
 - name: Set up QEMU
   uses: docker/setup-qemu-action@v3
```
- Then we set up docker build
```bash
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3
```
- Then we build and push the docker image to my docker hub with the repository name “bankapp” and my docker hub account is your dockerhub username.
```bash
- name: Build and Push Docker image
  uses: docker/build-push-action@v6
  with:
    context: .
    push: true
    tags: dockerhubusername/bankapp:latest
    file: ./Dockerfile
```
- We have completed writing the code for this job. Let us copy the code from IntelliJ and paste on our Pipeline file in GitHub.
- Then, commit the changes by clicking on “commit changes”
- Confirm by clicking on “commit changes” again
- Click on “Actions”
- And click on “Update cicd.yml”
- You can see that the pipeline is not running. This is because our “runner” is offline. Let us go and start the runner now.

- First SSH connect to our runner instance
- Navigate to the “actions-runner” folder using the command
```bash
cd actions-runner
```
- Then start the runner using the command:
```bash
./run.sh
```
- You can see that the runner has started running and the jobs have started running. We expect it to fail because we have not added our Docker login name and password.
- It failed as expected. Go back to the GitHub repository. This is because we have not yet added Docker username and password as variable and secret respectively on the GitHub Action

### Adding Secret and Variables oof Docker on the GitHub Action
- We are going to add added Docker username and password as variable and secret respectively on the GitHub Action.
- Click on “Settings”
- Then click on “secret and variables”
- Select “Actions”
- Now, we have to add our docker login password. Click on “New repository secret”
- For “name”, enter “DOCKERHUB_TOKEN”
- And for “secret”, enter your docker password “xxxxxxx”
- Click on “Add secret”
- Now, let us add the docker login username. This is a variable. So, click on the variable tab above
- Then click on “New repository variable”
- On “name”, enter “DOCKERHUB_USERNAME"
- And on “value”, enter “ebotsidneysmith”
- Then click on “Add variable”
- You can now go and re-run the pipeline. Click on “Action”
- Click on the “update cicd.yml”
- Click on re-run jobs and select “Re-run all jobs”
- Click on “Re-run jobs”
- The jobs have started running
- You can see that the build is successful. Go now to the docker hub and check if the image is there
- You can see our docker image

### Adding “deploy_to_kubernetes” job
- We will now add the job to deploy the docker image on Kubernetes. To do this, we will start by setting up EKS cluster using Terraform. For this we will create a new EC2 instance called “eks-server”.

#### Create Ubuntu EC2 instance for “eks-server”
- We will call the instance “eks-server”, AMI will be “ubuntu”, Instance type will be “t2.medium” and the configure storage will be “20”. Then SSH connect to the “eks-server”.
- Select the “eks-server” instance

### SSH Connect to the EC2 instance
- Now, let us SSH connect to the EKS server. Select the instance
- Click on “Connect”
- Copy the command above:
```bash
ssh -i "runner-key.pem" ubuntu@ec2-98-91-200-7.compute-1.amazonaws.com
```
- Then open terminal and navigate to where the Key pair file .pem file is saved. It is saved in our Downloads folder
- Then run the command:
```bash
ssh -i "runner-key.pem" ubuntu@ec2-98-91-200-7.compute-1.amazonaws.com
```
- Type “yes” and press “Enter”
- We are now connected to our EKS server. We have to install two things, namely AWS CLI and Minkube.

### Install AWS CLI
- Let us install AWS CLI, to do this we will first update the package using the command:
```bash
sudo apt update
```
- Now, let us install AWS CLI using the command:
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
```
- AWS CLI has been installed. We are going to use the AWS CLI to connect to our account. To do this we will first create a “Security Credential”. Go to AWS Management console
- Click on your username
- Select “Security Credentials”
- Click on “Create Access Key”
- Select “Command Line Interface (CLI)”
- Check the box “I understand the above recommendation and want to proceed to create an access key”
- Click on “next”
- Click on “Create access key”
- Click on “Download .csv file” to download the file containing your access and secret keys
- Then click on “Done”
- Now, let us connect to our AWS account using the command:
```bash
aws configure
```
- Enter the Access key generated in the downloaded file
```bash
aws configure
```
- Enter the Access key generated in the downloaded file
- Then enter the secret key
- Then enter your region, my region is “us-east-1”
- For “default output format [None]”, press “Enter”
- The next thing we have to do is to install Terraform. To install Terraform on an Ubuntu EC2 instance, follow these steps:

### Install Terraform
- Update and upgrade system Packages:
```bash
sudo apt update && sudo apt upgrade -y
```
- Install Required Dependencies:
```bash
sudo apt install -y software-properties-common gnupg2 curl
```
- Add HashiCorp GPG Key:
```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o
/usr/share/keyrings/hashicorp-archive-keyring.gpg
```
- Add HashiCorp Repository:
```bash
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg]
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee
/etc/apt/sources.list.d/hashicorp.list
```
- Update Package List Again:
```bash
sudo apt update
```
- Install Terraform:
```bash
sudo apt install terraform -y
```
- Verify Installation:
```bash
terraform -version
```
- The next thing to do is to create EKS cluster. We will create the EKS cluster on AWS using Terraform. To do this we will create another GitHub repository called “EKS-Terraform”

### Create EKS Cluster GitHub Repository
- Go back to GitHub and create this repository.
- Click on “Create Repository”

### Upload files to the Repository
- Let us now upload the files of this project from our local machine to the GitHub repository. The project files are located at:
```bash
https://github.com/ebotsidneysmith/EKS-Terraform
```
- Open terminal and We will clone the repository by using the command
```bash
git clone https://github.com/ebotsidneysmith/EKS-Terraform.git
```
- Let us move into the repository using the command:
```bash
cd EKS-Terraform
```
- Let us Initialize the directory containing Terraform using the command:
```bash
terraform init
```
- Then let us set up the EKS cluster on the AWS account by using the command:
```bash
terraform apply --auto-approve
```
- The EKS cluster has been created. You can verify this by checking on your AWS
- You can see that the cluster has been created and it is “Active”

### Install Kubectl
- Next thing to do is to install kubectl to enable us to access the cluster using the commands:
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s
https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -L -s
https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```
- We have to now connect to our cluster. To do this we have to set up the kubeconfig file by using the command:
```bash
aws eks --region us-east-1 update-kubeconfig --name devopsshack-cluster
```
- Then verify if you can see node by using the command:
```bash
kubectl get nodes
```
- You can see our nodes.
- We will now have to add the Kubernetes secret on our GitHub Action. To do this, open the kubeconfig file by using the command:
```bash
cat ~/.kube/config
```
- We will copy the code in this file
- Then head back to the GitHub Action repository
- Click on “Settings”
- Then click on “Secrets and Variables”
- Click on “Actions”
- Then click on “New repository secret”
- Then for “name”, enter “EKS_KUBECONFIG” that we will use in the code of the job
- The for “secret”, enter the code we copied from our kube config file
- Then click on “Add secret”
- The secret has been added.

### Adding the job
- Now, let us go ahead to add our job to deploy the image to EKS cluster. We will call this job “deploy_to_kubernetes”. It will run on our private runner, so we add the line “runs-on: self-hosted”. For this job to start, we will need the previous job “build_docker_image_and_push”, so we will add the line “needs: build_docker_image_and_push”. 
```bash
deploy_to_kubernetes:
    runs-on: self-hosted
    needs: build_docker_image_and_push
```
- We now have to start adding the steps / action. Our first step will be to check out of fetch the code from the GitHub repository. This will be named “Checkout Code” and the action will be “actions/checkout@v4”
```bash
steps:
  - name: Checkout Code
  uses: actions/checkout@v4
```
- The next step will be to install AWS CLI
```bash
- name: Install AWS CLI
  run: |
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install --update
```
- Next, we will login to our AWS account using the AWS Access and Secret keys
```bash
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v2
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: us-east-1
```
- Then, we will setup kubectl to enable us to access the cluster
```bash
- name: Set up kubectl
  uses: azure/setup-kubectl@v3
  with:
    version: latest
```
- Next thing to do is to configure our kubeconfig file
```bash
- name: Configure kubeconfig
  run: |
    mkdir -p $HOME/.kube
    echo "${{ secrets.EKS_KUBECONFIG }}" > $HOME/.kube/config
```
- And finally, we will deploy the docker image to EKS
```bash
- name: Deploy to EKS
  run: |
    kubectl apply -f ds.yml
```
- Let us go and re-run the pipeline again
- The Pipeline is successful

### Deleting resources
- Please, do not forget to delete the resources such as EKS cluster to avoid billing.
