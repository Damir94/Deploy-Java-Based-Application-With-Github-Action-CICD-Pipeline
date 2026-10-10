# Deploy a Java-Based Application With GitHub Action CI/CD Pipeline

<img width="674" height="245" alt="Screenshot 2026-10-10 at 10 39 00 AM" src="https://github.com/user-attachments/assets/172c7f64-5ef2-4e3b-a744-8612dc3c59f7" />

## Introduction
- In this document, I will demonstrate how to deploy a Java-based application using GitHub Action Pipeline. In this, we have the concept of slave called runner. There will be some kind of server or virtual machine where the application is going to be built and whatever commands, we are going to be running will get executed.

- In GitHub Action, we have two options for runners. One option is shared runner and the second option is private runner.

- The benefit of shared runner is that it is completely free. It will be provided by GitHub, we do not have to do anything, all we need to do is to provide a name. For example, I want to run my commands on ubuntu. I just have to write “ubuntu-latest”. That means all my commands on a runner that is of configuration ubuntu latest. If I want to run on Windows, I will define “windows-latest”. The advantage of shared runner is that it is completely free and the disadvantage is that you won’t be having access to backend of it.

- In Private runner, we bring our own virtual machine and then add the virtual machine as a runner. The disadvantage here is that you have to pay for it. And the benefit is that it will be complete isolated and private to you only and your company. Secondly, you will be having complete control or complete access over it. So, most companies prefer private runner because they can use their own virtual machine and have complete control over it and make sure that is completely isolated. 
