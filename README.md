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
-name: Upload JAR artifact
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
