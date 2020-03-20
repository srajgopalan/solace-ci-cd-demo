# Using Solace's SEMPv2 to integrate with CI/CD pipelines using Ansible and Jenkins

This sample demostrates how you can use Solace's SEMPv2 management API to perform automated provisioning of Solace configuration, as well as integrate the automated provisioning of your Solace environment with your CI/CD Pipeline. 

There are two ways you can get started:

- If your company has Solace message routers deployed, contact your middleware team to obtain the host name or IP address of a Solace message router to test against, a username and password to access it, and a VPN in which you can produce and consume messages.
- If you do not have access to a Solace message router, you will need to go through the “[Set up a VMR](http://docs.solace.com/Solace-VMR-Set-Up/Setting-Up-VMRs.htm)” tutorial to download and install the software.

Ensure that you are running a version of SolOS that supports the SEMPv2 management API. SEMPv2 was introduced in the following releases:
- Virtual Message Router: SolOS 7.2.1
- Message Router Appliance: SolOS 7.2.2

__NOTE: The sample has been created to illustrate how SEMPv2 can be used to integrate Solace with a CI/CD pipeline. You may need to modify the samples to align with your custom environment, as well as apply security, hardening and other production readiness considerations as necessary.__

## Contents

[Ansible](https://www.ansible.com/) is an open source tool for automating tasks such as application deployment, IT service orchestration, cloud provisioning and many more. Ansible is easy to deploy, does not use any agents, and has a simple learning curve. Ansible uses "playbooks" to describe its automation jobs, known as "tasks" and the playbooks are described using YAML.

This repository contains an ansible playbook which uses the Solace SEMPv2 RESTful administration API to create a new messaging environment on an existing Solace message router. Ansible's [URI](http://docs.ansible.com/ansible/latest/uri_module.html) module is used to interact with the SEMPv2 API. 

For a nice introduction to the SEMPv2 Management API , check out this [blog](https://solace.com/blog/products-tech/introducing-semp-v2-solace-message-routers-configuration-reinvented), as well as the [SEMP tutorials home page](http://dev.solace.com/get-started/semp-tutorials/)

The sample can be used for automated deployment of your Solace messaging environments for Continuous Integration. Additionally, you can integrate this into your pipeline for Continuous Deployments. This is explained in the subsequent sections.

## Automated provisioning in a CI environment using Ansible

Using Ansible, you can automatically provision configuration on the Solace router. The Ansible playbook consists of a number of tasks, and each task uses Solace's SEMPv2 Management API to create a new Solace message-vpn on an existing Solace message router along with associated objects:

- Client Profiles
- ACL Profiles
- Client Usernames
- Queue Endpoints

If the objects already exist, this is indicated in the playbook run's output. It is not treated as a failure of the Ansible task, and the playbook execution continues to to the next task. Object properties can be specified in a configuration files.

![CI Flow Diagram](https://github.com/srajgopalan/solace-ci-cd-demo/blob/master/images/CI.jpg "Continuous Integration using Anisble and SEMPv2")

### Pre-requisites

1. Install [Ansible](https://www.ansible.com)
2. The host running the samples should have network connectivity to the management IP of the Solace Router
3. A Management user must be created on the Solace router and given a global-access-level of read-write.

### Checking out and building

To check out the project and build it, do the following:

  1. clone this GitHub repository
  2. `cd solace-ci-cd-demo`
 
### Configuring the CI Demo:

1. __Edit the Ansible inventory__: Edit the inventory.ini to specify the Solace routers on which the messaging environment is to be created. You can create one or more host groups against which the Ansible playbook will be run. 

For example:

```
[solace]
sgdemo mgmt_host=192.168.42.11 mgmt_port=80 semp_username=ansible semp_password=ansible
```

__NOTE:__ Do not edit the `solace-vars` group or its contents

2. __Edit the configuration files__: The configuration for Solace environments to be created is specified in `vars/solace-env.yml`. Edit the configuration objects as necessary. The current version of the sample supports the creation of:

- A single message-vpn
- One or more client profiles within the message-vpn
- One or more ACL profiles within the message-vpn
- One or more Client Usernames within the message-vpn
- One or more Queues within the message-vpn, with topic subscriptions on these queues

__NOTE:__ 

- The current version of this sample does not support the externalization of all the configuration properties for the Solace environment. Only some properties are specified in the configuration files, and more properties can be added along with appropriate changes to your ansible playbook, depending on your environment.
- The sample currently does not have the feature to remove any message-vpns when they are removed from the configuration file.This operation will have to be performed manually.

### Running the CI Demo:

In order to run the Ansible playbook, use:

`ansible-playbook create-solace-vpn.yml -i inventory.ini `

## Integrating with a Continuous Delivery System

The Continuous Integration demo can now be integrated with Jenkins and Git for setting up a Continuous Delivery System. The below figure shows a sample CD pipeline, using Git, Jenkins and Ansible - I've chosen them as they are the common DevOps tools for CD, but you can replace parts or all of these with whatever tooling you use. 

![CD Flow Diagram](https://github.com/srajgopalan/solace-ci-cd-demo/blob/master/images/CD.jpg "Continuous Delivery using Git, Jenkins, Anisble and SEMPv2")

I've set up a job in Jenkins for this project, which will be triggered every time code is checked in to the Git repository, using a Githuub Webhook. The Jenkins job runs the Ansible playbook, which uses SEMPv2 to create the messaging environment as specified in the config files.

Finally, after the creation of the messaging environment, Jenkins triggers another job to pull the Solace Javascript Samples and deploys them to our web server. You can then run the Solace Javascript samples against your newly provisioned messaging environment! 

In summary, when I do a Git push, this will trigger Jenkins to create my messaging environment on Solace, and then deploy my app (Solace Javascript samples, in my case) - As Jürgen Klopp says, BOOM!

### Live Demo

Visit [Solace SGDemo Jenkins](http://sgdemo.solace.com/jenkins/job/solace-ci-cd-demo/) - A Jenkins job ('solace-ci-cd-demo') has been created and configured such that any changes to the configuration in GitHub (such as adding new queues/users) will trigger the job to create the corresponding Solace configuration. As indicated previously, the Ansible playbook does not attempt to re-create existing objects - they will simply be logged as "ALREADY_EXISTS"

Once the Solace environment has been created, this triggers a second job which will pull the solclient JS samples from Github and deploy them to the sgdemo web server. Click [here](http://sgdemo.solace.com/solclientjs) to try out the newly deployed Solclient samples.

Contact your Solace Account Manager for a Live Demo!

__NOTE:__ If any configuration is removed, the Ansible playbook does not delete the corresponding configurations from the Solace router

## Reproducing the CD Demo:

As the material for setting up the CD demo is publicly available, here are some references:

1. Trigger a Jenkins Job using a GIT push: [GIT Integration plugin](https://wiki.jenkins.io/display/JENKINS/GitHub+Integration+Plugin)
2. Run Ansible playbooks in Jenkins: [Ansible Plugin for Jenkins](https://wiki.jenkins.io/display/JENKINS/Ansible+Plugin)

## Built With

- [Ansible](https://www.ansible.com) - Simple IT Automation, Configuration Management and Orchestration
- [SEMPv2](https://docs.solace.com/SEMP/SEMP-Home.htm) - Solace's RESTful Administration API 
- [Jenkins](https://jenkins.io/) - Open Source Automation Server

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.

## Authors

- Shrikanth Rajgopalan

See also the list of [contributors](https://github.com/srajgopalan/solace-ci-cd-demo/contributors) who participated in this project.

## License

This project is licensed under the Apache License, Version 2.0. - See the [LICENSE](LICENSE) file for details.

## Resources

For more information try these resources:

- The Solace Developer Portal website at: http://dev.solace.com
- Get a better understanding of [Solace technology](http://dev.solace.com/tech/).
- Get your hands dirty with [SEMP Tutorials](http://dev.solace.com/get-started/semp-tutorials/)
- Check out the [Solace blog](http://dev.solace.com/blog/) for other interesting discussions around Solace technology
- Ask the [Solace community.](http://dev.solace.com/community/)
