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
