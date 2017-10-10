# Launch An EC2 Instance With LAMP & Setup Environment For Laravel
Dummy Guide To Launch An EC2 Instance

## References
https://aws.amazon.com/marketplace/pp/B00O7WM7QW?ref=cns_srchrow  
https://wiki.centos.org/TipsAndTricks  
https://www.tecmint.com/upgrade-mariadb-5-5-to-10-centos-rhel-debian-ubuntu/  
https://www.liquidweb.com/kb/how-to-upgrade-mariadb-5-5-to-mariadb-10-0-on-centos-7/  
https://blog.lysender.com/2015/07/centos-7-selinux-php-apache-cannot-writeaccess-file-no-matter-what/

Bootstrap an AMI with LAMP Server
=================================

*NB:* Create a free AWS account.

1. Setup Identity and Access Management

Create IAM Role
===============

AWS website describes roles as below:

>An IAM role is similar to a user, in that it is an AWS identity with permission policies that determine what the identity can and cannot do in AWS. However, instead of being uniquely associated with one person, a role is intended to be assumable by anyone who needs it. Also, a role does not have any credentials (password or access keys) associated with it. Instead, if a uer is assigned to a role, access keys are created dynamically and provided to the user.

To create a role, you can use the AWS Management Console, the AWS CLI, the Tools for Windows PowerShell, or the IAM API.If you use the AWS Management Console, a wizard guides you through the steps for creating a role. The wizard has slightly different steps depending on whether you're creating a role for an AWS service, for an AWS account, or for a federated user.

Attach Policy
=============

- S3 Access
- AWSCodeDeployFullAccess
- EC2 FullAccess
- Route53 FullAccess
- DynamoDB FullAccess


Create Usergroup
================

Create User
===========

Add User To Usergroup
=====================

Apply an IAM password policy
============================


*NB:* Download user credentials(Download.csv) immediately after success

2. Services EC2 => Launch an instance(virtual server)

Todo: Define EC2 in your own words

To launch an aws EC2 instance(virtual server), its important we setup a iam role and security group that the instance would make use of. We already created an iam role, next we need to create a security group, to do this:

- Open EC2 service
- On the left nav under network and security section click on security groups
- Create security group
- Add inbound rules
  - SSH (You want to restrict it to your IP for security reasons)
  - HTTP,HTTPS, Add more rule later as need arises
- Name your security group (Open To All)

- Using image(Centos) CentOS Linux 7 x86_64 HVM EBS 1704_01 - ami-0cfcdb69

- Configure Instance

- Name & Create Key pair name

- Download immediately and change permission to 600 

- Check system Log for Instance

```
#!/bin/bash
yum update -y
yum install httpd -y
systemctl start httpd.service
systemctl enable httpd.service
```

- Select General Purpose Storage (There are different types, you can google them)

- Visit System IP, you will be presented with Testing 123..

- ssh into the server

```
ssh -i  {PATH_TO_PEM_FILE} centos@{INSTANCE_IP_ADDR}
```

- Proceed to install php,git,composer,mysql5.7

```
sudo yum install mariadb-server mariadb  
sudo systemctl start mariadb  
sudo systemctl enable mariadb  
sudo mysql_secure_installation  
```

- Install php7

```  
sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm   &&
sudo rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm   &&
sudo yum install -y mod_php71w php71w-cli php71w-common php71w-gd php71w-mbstring php71w-mcrypt php71w-mysqlnd php71w-xml  
```
- Install git & composer

```
yum install -y git  

cd /tmp  

curl -sS https://getcomposer.org/installer | php  

sudo mv composer.phar /usr/local/bin/composer  
```

*NB:* The php.ini file reside at /etc/php.ini, its save to back it up before editing the file


3. Create a vitual host
To begin, we will need to set up the directory that our virtual hosts will be stored in, as well as the directory that tells Apache that a virtual host is ready to serve to visitors. The sites-available directory will keep all of our virtual host files, while the sites-enabled directory will hold symbolic links to virtual hosts that we want to publish. We can make both directories by typing:

`sudo mkdir /etc/httpd/sites-available`  
`sudo mkdir /etc/httpd/sites-enabled`  
Note: This directory layout was introduced by Debian contributors, but we are including it here for added flexibility with managing our virtual hosts (as it's easier to temporarily enable and disable virtual hosts this way).

Next, we should tell Apache to look for virtual hosts in the sites-enabled directory. To accomplish this, we will edit Apache's main configuration file and add a line declaring an optional directory for additional configuration files:

`sudo nano /etc/httpd/conf/httpd.conf`

Run this command to install nano editor `sudo yum install nano.x86_64` (This step is optional, if you are comfortable with vi editor, simply go ahead)  

Add this line to the end of the file:

`IncludeOptional sites-enabled/*.conf`
Save and close the file when you are done adding that line. We are now ready to create our first virtual host file.

`sudo nano /etc/httpd/sites-available/example.com.conf`

##Setup V-host File
```
<VirtualHost *:80>    
    DocumentRoot /var/www/html/your_site/public    
    ServerName your_domain    
    <Directory /var/www/html/your_site/public>   
        Order allow,deny
        Allow from all
        AllowOverride ALL
        DirectoryIndex index.php
    </Directory>  
</VirtualHost>
```


`sudo ln -s /etc/httpd/sites-available/example.com.conf /etc/httpd/sites-enabled/example.com.conf`



