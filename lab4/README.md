# Creating a Highly Available Environment

What will you learn:
- Inspect a provided VPC
- Create an Application Load Balancer
- Create an Auto Scaling Group
- Test the application for high availability

What's aleady provisioned:
 - An Amazon VPC
 - Public and Private Subnets in 2 AZs
 - An internet gateway associated with the public subnets
 - A NAT gateway in one of the Public subnets
 - An Amazon RDS instance in one of the Private subnets

# Steps
1. Inspect your VPC
2. Create an application load balancer
> Best Practice: launch resources in multiple AZ's since they're physically seperate data centers (or groups of data centers) within the same region which provides greater availability in case of a data center failure. Given that the application is running on multiple application servers, you can distribute traffic amongst those servers by using a load balancer which also performs health checks on instances and only sends requests to healthy instances. 
    - create an Application Load Balancer: name, VPC, AZ1 == public subnet 1, AZ2 == public subnet 2, new security group (name, description, modify existing rule to HTTP/Anywhere and add new rule with HTTPS/Anywhere)
    > the app load balancer automatically performs health checks on all instances to ensure they're responding to requests. The healthy threshold and interval coresponds to the health check being performed every n interval seconds and if the instance responds correctly n healthy threshold in a row, it will be considered healthy.
3. Create an auto scaling group
> Amazon EC2 auto scaling is a service designed to launch/terminate instances automatically based on user-defined policies/schedules/health checks. it also automatically distributes instances across multiple AZ's to make applications highly available.
> Best Practice:  Auto scaling groups that deploy ec2 instances across private subnets, instead users should send requests to the Load Balancer which will forward he requests to ec2 instances in the private subnets
4. Update Security Groups
