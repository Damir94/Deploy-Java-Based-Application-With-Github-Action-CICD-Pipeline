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
