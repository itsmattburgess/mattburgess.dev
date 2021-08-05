---
title: "Installing Jenkins on Amazon Linux"
date: 2017-11-26T23:45:52+01:00
draft: false
thumbnail: /images/jenkins-amzn.png
ogimage: /images/jenkins-amzn.png
tags:
- AWS
- Jenkins
---
If you’re looking to install Jenkins on an instance of Amazon Linux, there are a few more steps you’ll need to perform compared to installing Jenkins on a ubuntu instance.

This guide assumes that you have launched an Amazon Linux instance with a public DNS and have configured a security group allowing SSH access and all requests to port 8080 for this instance.

This guide covers only installing Jenkins. After installation, You’ll need to make some configuration changes to enable some security checks. The Jenkins documentation covers these steps here: https://jenkins.io/doc/book/system-administration/security/

## Automating the process

I have created Packer templates which will build an Amazon Machine Image (AMI) for you, so that you’re able to launch an instance with all of the below configuration already in place. You can find these templates in my GitHub repository: https://github.com/itsmattburgess/jenkins-packer

### Installing Jenkins

First things first, SSH into your instance.
```
ssh ec2-user@{ec2-public-dns}
```

Before we do anything, we’re going to check that our operating system is up to date and install any outdated packages.
```
sudo yum -y update
```

Jenkins requires Java 1.8. However, Amazon Linux comes preinstalled with Java 1.7. Let’s replace 1.7 with 1.8. We’re going to install Java 1.8 first to prevent any of our AWS CLI tools from being uninstalled due to the Java dependancy breaking.
```
sudo yum install java-1.8.0
sudo yum remove java-1.7.0-openjdk
```

We need to add the Jenkins repository so that yum knows where to install Jenkins from.
```
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
```

Next, we’re adding the Jenkins GPG key to our trusted keys so that we’re able to install Jenkins, verifying that the files are being sourced from a trusted location.
```
sudo rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key
```

Great! We’ve prepared our environment with the required dependancies so we can now install Jenkins.
```
sudo yum install jenkins
```

Let’s start the Jenkins service:
```
sudo service jenkins start
```

If you want Jenkins to automatically start when your instance is started, we can use chkconfig to add Jenkins to our startup services.
```
sudo chkconfig --add jenkins
```

All done! You can now access your Jenkins server using the public DNS on port 8080.
```
http://{ec2-public-dns}:8080
```

Don’t forget to secure your Jenkins installation by following the recommended steps over on the Jenkins website: https://jenkins.io/doc/book/system-administration/security/