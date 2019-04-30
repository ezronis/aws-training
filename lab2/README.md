#  Deploying a Web Application on AWS

## Steps
- launch a database using Amazon RDS
- launch an application server using Amazon EC2
- automatically install an application

1. Create your security groups (a security group acts as a virtual firewall that controls the traffic for one or more instances):
    - From the ec2 menu, create an app security group (name app-sg, rule http anywhere)
    - From the ec2 menu, create a DB security group (name DB-sg, copy ID of app sg you just created and add to DB-sg as a rule. the config now indicates that the DB-sg permits inbound access from the app-sg. Any instances associated to the app-sg will have permission to communicate with the db-sg)
2. Create a mysql rds:
    - dev/test mysql, db instance class: t2.micro, db instance identifier: [name], master username: [master], master password:[some password] and select existing security group db-sg created in step 1
3. launch an ec2 instance:
    - amazon linux 2 ami, t2.micro, add any tags, add app-sg, and pass the following to user data:
        ```
        #!/bin/bash
        # Install Apache Web Server and PHP
        yum install -y httpd mysql
        amazon-linux-extras install -y php7.2
        # Download Lab files
        wget https://us-west-2-tcprod.s3.amazonaws.com/courses/ILT-TF-100-ARCHIT/v6.4.0/lab-2-webapp/scripts/inventory-app.zip
        unzip inventory-app.zip -d /var/www/html/
        # Download and install the AWS SDK for PHP
        wget https://github.com/aws/aws-sdk-php/releases/download/3.62.3/aws.zip
        unzip aws -d /var/www/html
        # Turn on web server
        chkconfig httpd on
        service httpd start
        ```
4. Access the application by navigating to the IPv4 public IP address availaible in the EC2 console, copy the endpoint to the database created in step 2 from the connectivity section and paste info requsted related to the DB in the gui. 
5. Try adding some inventory!