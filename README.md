# Using Solace's SEMPv2 to integrate with CI/CD pipelines

This sample demostrates how you can use Solace's SEMPv2 management API to integrate the automated provisioning of your Solace environment with your CI/CD Pipeline. There are two ways you can get started:

- If your company has Solace message routers deployed, contact your middleware team to obtain the host name or IP address of a Solace message router to test against, a username and password to access it, and a VPN in which you can produce and consume messages.
- If you do not have access to a Solace message router, you will need to go through the “[Set up a VMR](http://docs.solace.com/Solace-VMR-Set-Up/Setting-Up-VMRs.htm)” tutorial to download and install the software.

Ensure that you are running a version of SolOS that supports the SEMPv2 management API. SEMPv2 was introduced in the following releases:
- Virtual Message Router: SolOS 7.2.1
- Message Router Appliance: SolOS 7.2.2

__NOTE:__ This sample has been created to illustrate how SEMPv2 can be used to integrate Solace with a CI/CD pipeline. You may need to modify the samples to align with your custom environment, as well as apply security, hardening and other operational readiness considerations as necessary.

## Contents

This repository contains an ansible playbook which uses the Solace SEMPv2 RESTful administration API to create a new messaging environment. For a nice introduction to the SEMPv2 Management API , check out this [blog](https://solace.com/blog/products-tech/introducing-semp-v2-solace-message-routers-configuration-reinvented), as well as the [SEMP tutorials home page](http://dev.solace.com/get-started/semp-tutorials/)

The sample can be used as-is for automated deployment of your Solace messaging environments for Continuous Integration. Additionally, you can integrate this into your pipeline for Continuous Deployments. This is explained in the subsequent sections.

## Automated provisioning in a CI environment

The ansible playbook uses Solace's SEMPv2 Management API to create a new Solace message-vpn along with associated objects:
- Client Profiles
- ACL Profiles
- Client Usernames
- Queue Endpoints

If the objects already exist, this is indicated in the playbook output. It is not treated as a failure of the ansible task, and the playbook execution continues to to the next task.
 
Object properties can be specified in a configuration file

![CI Flow Diagram](https://github.com/srajgopalan/solace-ci-cd-demo/blob/master/images/CI.jpg "Continuous Integration using Anisble and SEMPv2")

### Pre-requisites

1. Install [Ansible](https://www.ansible.com)
2. The host running the samples should have network connectivity to the management IP of the Solace Router
3. A Management user must be created on the Solace router and given a global-access-level of read-write.

### Checking out and building

To check out the project and build it, do the following:

  1. clone this GitHub repository
  2. `cd solace-ci-cd-demo`
 
### Configuring the Demo:

1. __Edit the ansible inventory__: Edit the inventory.ini to specify the Solace routers on which the messaging environment is to be created. The host running the samples should be able to reach the management IP of the Solace router. You can create one or more host groups against which the Ansible playbook will be run. 

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

__NOTE:__ The current version of this demo does not support the externalization of all the configuration properties. Only some properties are specified in the configuration files, and more properties can be added along with appropriate changes to your ansible playbook, depending on the build of your environment.

### Running the Demo:

In order to run the Ansible playbook, use:

`ansible-playbook create-solace-vpn.yml -i inventory.ini `

## Integrating with a Continuous Delivery System

__NOTE: This section still needs to be written up properly__

The Continuous Integration demo can now be integrated with Jenkins for setting up a Continuous Delivery System. 

![CD Flow Diagram](https://github.com/srajgopalan/solace-ci-cd-demo/blob/master/images/CD.jpg "Continuous Delivery using Git, Jenkins, Anisble and SEMPv2")

*Desribe the pipeline here.
The Jenkins Ansible plugin can be used to create a job in Jenkins, to trigger our Ansible playbook. *

### Pre-requisites

1. Install Jenkins
2. Install Ansible Plugin 
3. Install GIT Integration Plugin
4. ...


### High Level Steps for CD Integration

1. Create a Jenkins Job and configure it to point to the GIThub repo
1. Make some changes to the Github repo and boom!


### Live Demo

Visit the [Solace APAC Jenkins](http://sgdemo.solace.com/jenkins/job/solace-ci-cd-demo/) - A Jenkins Job has been created and configured such that any changes to the Solace environment configuration (such as adding new queues/users) will trigger the job to create the additional Solace configuration. As indicated previously, the Ansible playbook does not attempt to re-create existing objects - they will simply be logged as "ALREADY_EXISTS"

Contact your Solace Account Manager for a Live Demo!

__NOTE:__ If any configuration is removed, the Ansible playbook does not delete the corresponding configurations from the Solace router

## Built With

- [Ansible](https://www.ansible.com) - Simple IT Automation, Configuration Management and Orchestration
- [SEMPv2](https://docs.solace.com/SEMP/SEMP-Home.htm) - Solace's RESTful Administration API 

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
