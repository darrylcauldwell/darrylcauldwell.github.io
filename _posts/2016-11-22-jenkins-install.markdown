
sudo yum install wget java
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
sudo yum install jenkins
sudo service jenkins start
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

http://54.154.139.46:8080/

