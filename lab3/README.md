# Creating a virtual private cloud
> follow the directions to build your own virtual private network, deploy resources and create private peering connections between networks

## Steps
You'll deploy an Amazon VPC with:
    - an internet gateway
    - a public subnet
    - a private subnet
    - an app server to test the vpc
optional challenge: create a vpc peering connection to a shared services vpc. you will use an app & db to test connectivitiy b.w the vpcs. 

1. Create a VPC and configure:
    - name tag, IPvr CIDR block: 10.0.0.0/16
    - click actions after created and select "Edit DNS hostnames" and select "enable", it will assign a friendly DNS name to ec2 instances in the VPC
2. Create subnets and configure:
2a. Create a public subnet:
    - name tag, VPC (select VPC created in step 1), Availability Zone (select the first AZ in the list), IPv4 CIDR block 10.0.0.0/24
    - click actions after created and select "Modify auto-assign IP settings" and select "Auto-assign IPv4" to configure the subnet to automatically assign a public IP for all instances launched within it.
    > the VPC has a CIDR of 10.0.0.0/16 which includes all 10.0.x.x IPs. This subnet has a CIDR of 10.0.0.0/24 which includes all 10.0.0.x IPs, which is smaller than the VPC due to the /24 in the range.
2b. Create a private subnet:
    - name tag, VPC (select VPC created in step 1), Availability Zone (select the first AZ in the list), IPv4 CIDR block 10.0.2.0/23
    > The CIDR block of 10.0.2.0/23 includes all IPs that start with 10.0.2.x and 10.0.3.x, which is twice as large as the public subnet because most resources should be kept private unless they specifically need to be accessible from the internet.
Your VPC now has 2 subnets, but is totally isolated and can't communicate w/ resources outside the VPC. Next we'll configure the public subnet to connect to the internet via internet gateway.
3. Create an internet gateway:
    - name tag
    - once created, from the actions menu select "Attach to VPC" and select VPC created in step 1
    > this will attach the internet gateway to your VPC in step 1. Even though you've created an internet gateway & attached it to your VPC, you must also configure the public subnet route table to use the internet gateway
4. configure route tables 
    - creating a public route table for internet bound-traffic: click on "Create Route Table" and configure name tag, VPC from step 1
    - add a route to the route table to direct internet-bound traffic to the internet gateway: select newly created public route table and in the "Routes" tab, click on "edit routes". Click "add route" and configure destination (0.0.0.0/0), Target(select internet gateway and select Lab IGW from step 3) and click "Save Routes"
    - associate the public subnet with the new route table: Select the public subnet and click on the "subnet associates" tab and click "edit subnet associations". select the row with the public subnet.
> To summarize, you can create a public subnet as follows:
    - create an internet gateway
    - create a route table
    - add a route to the route table that directs 0.0.0.0/0 traffic to the internet gateway
    - associate the route table with a subnet, which therefore becomes a public subnet
5. create a security group for the app server
    > Create a security group that allows users to access your app server via HTTP:
    - Click "Create security group" from the "Security Groups" menu and configure Security group name, description (Allow HTTP traffic), VPC from step 1
    - select created SG and click on "Inbound Rules" tab, click "Edit Rules" and click "add rule" and configure type (HTTP), source (Anywhere), Description (Allow web access)
6. Launch an App Server in the public subnet
    > to test that your VPC is correctly configured, you will now launch an Amazon EC2 instance into the public subnet and confirm that it is accessible from the internet
    - Launch an EC2 instance, AMI (Amazon Linux 2), instance type (t2.micro), network (VPC step 1), subnet (public subnet step 2a), IAM role, configure SG (step 5), user data: 
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
7. Bonus: Configure VPC peering
    - from the VPC menu, "Create Peering Connection" and configure peering connection name tag, VPC Requester (step 1), VPC Accepter (Shared vpc)
    - once created, select and from "Actions", select "Accept request" b/c when a peering connection is created, it must be accepted by the target VPC.
    - confirgure route tables, select public route table and edit routes, add 10.5.0.0/16, target (peering connection). then select shared vpc route table and edit routes , add 10.0.0.0/16 and select target (peering connection)
    > test connection by providing DB info in settings of web app launched in step 6 and see if data can be retrieved
## defs
- A VPC is a virtual network tied to your AWS account, isolated from other virtual networks on AWS. You can launch resources into the VPC and configure the VPC by modifying its IPs range, create subnets, configure route tables, network gateways & security settings.

- A subnet is a sub-range of IPs within a VPC, and AWS resources can be launched into a specified subnet. Public subnets are used for resources that must be connected to the web, where Private subnets are used for resources that should remain isolated from the internet.

- an internet gateway is a horizonally scaled, redundant and highly available VPC component that allows communication b/w instances in a VPC and the internet. It serves two purposes: to provide a target in route tables to connect to the internet and perform network address translation (NAT) for instances that have been assigned public IPv4 addresses

- A route table contains a set of rules called routes that are used to determine where network traffic is directed. Each subnet in a VPC must be associaed with a route table; the table controls the routing for the subnet. A subnet can only be associated with one route table at a time, but you can associate multiple subnets with the same route table. 

- To use an internet gateway, a subnet's route table must contain a route that directs internet-bound traffic to the internet gateway. If a subnet is associated with a route table that has a route to an internet gateway, it is known as a public subnet

- a security group acts as a virtual firewall for instances to control inbound and outbound traffic. SG's operate at the "instance network interface" level, not the subnet level. Therefore, each instance can have is own firewall that controls taffic. if you do not specify a particular SG at launch time, the instance automatically assigned to the default security group for the VPC. 

- Inbound rules determine what traffic is permitted to reach the instance

- A VPC peering connection is a networking connection between 2 VPCs that enables you to route traffic between them privately. Instances in either VPC can communicate with each other as if they are within the same network. You can create a VPC peering connection b/w your own VPCs, within a VPC in another AWS account or with a VPC in a different AWS region