---
layout: post
title:  "How To Setup Jenkins"
date:   2016-11-14 14:40:56 +0100
permalink: jenkins-install/
---
To install Jenkins onto RHEL7

    sudo yum install wget java
    sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
    sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
    sudo yum install jenkins
    sudo service jenkins start
    sudo cat /var/lib/jenkins/secrets/initialAdminPassword

You should then be able to browse this on port 8080