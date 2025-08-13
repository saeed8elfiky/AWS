# AWS Cloud Architecture 3 - Capstone Project


## Table of Contents

1. [Scenario Overview](#Scenario)  
2. [Solution Summary](#Solution)  
3. [Implementation Steps](#implementation-steps)  
---

### Scenario 
Example Social Research Organization is a (fictitious) nonprofit organization that provides a website for social science researchers to obtain global development statistics. For example, visitors to the site can look up various data points, such as the life expectancy for any country in the world over the past 10 years.

Shirley Rodriguez, a researcher at the nonprofit organization, developed the website. She thought it would be valuable to share the data that she had gathered with other researchers. Shirley stores the data in a MySQL database, and the data is available through a PHP website that she built. She initially published the site through a commercial hosting company that provides limited support for technical issues and security.

Over the past year, Shirley’s website has grown in popularity. As a result of increased traffic, she started receiving complaints that the site is not as responsive as it used to be. She also experienced an attempted ransomware security breach. The security breach was unsuccessful, but her supervisor, Mateo Jackson, suggested that Shirley investigate new ways to host the website.

Shirley heard about AWS and initially moved her website and database to an EC2 instance that runs in a public subnet. She also runs an instance of MySQL on the same EC2 instance.

Shirley approached your team to make sure that her current design follows architectural best practices. She wants to make sure that she has a robust and secure website. One of your colleagues started the process of migrating the site to a more secure implementation, but they were reassigned to another project. Your tasks are to complete the implementation, make sure that the website is secure, and confirm that the website returns data from the query page.

### Solution: 
- Provide secure hosting of the MySQL database.
    
- Provide secure access to the database.
    
- Provide anonymous access to web users.
    
- Run the website on a t2.micro EC2 instance running in private subnets and provide Secure Shell (SSH) access to administrators.
    
- Provide high availability to the website through a load balancer.
    
- Provide automatic scaling that uses a launch template.
---
### Implementation-Steps
1. Creating-RDS
	- **RDS (MySQL)** >
	- Keep the default settings
	- Templates `SandBox`
	- Availability and Durability `Single-AZ`
	- username `admin`
	- Credentials management `Managed in AWS Secrets Manager - most secure`
	- Instance configuration `Burstable classes (includes t classes)`
	- Storage `20 giga`, `Enable storage autoscaling`
	- Connectivity `Don't connect to an EC2`, VPC Choose `Project VPC`
	- VPC security group `Choose existin` > `ExampleDB-SG`
	- Database authentication `Password authentication`
	- Monitoring <font color="yellowgreen">Uncheck</font> `Enable Enhanced monitoring`
	- Additional configuration: name `countries` 

2.  **Creating an Application Load Balancer**
	- name it `AppELB`
	- scheme `internet-facing`
	- VPC `Project VPC`
		- Availability zones: **us-east-1a (use1-az1)** `Public 1`
		- Availability zones: **us-east-1b (use1-az1)** `Public 2`
	- Security groups `ALBSG`
	- Listeners and routing `create target group` link
		- Basic configuration `instances`
		- target group name `ELB-TG`
		- Next
	- Open new tap 
		- Search for EC2
		- From left sidebar > `Launch Templates`
		- Check `Project-LT`
		- Action > `Launch instance from template`
		- Key pair `vokey`
		- network > `Private Subnet 1`
		- Launch instance
	- Back to EC2/Target groups >
		- select `ExampleAPP` and click ***include as pending below***
		- Create target group
	- Back to the ELB
		- in the *Listeners and routing* select `ELB-TG`
		- Create load balancer

3. **Creating the application server in an Auto Scaling Group**
	- Search for EC2 
	- In the left side bar choose - *Auto Scaling Groups*
	- name `autoscaling-g`
	- in launch template `Project-LT`
	- Next
	- VPC `Projectt VPC`
	- Availability Zones and subnets `Private DB Subnet 1`
	- Next
	- Attach to an existing load balancer
		- *Choose from your load balancer target groups*
		- Existing load balancer target groups `ELB-TG | HTTP`
	- next
	- Desired capacity *Note: The desired capacity is up to you*
		- Min = 1
		- max = 2
	- Next > create auto scaling group

4. **Importing data into an RDS MySQL database**
	- Copy the download link of the data dump file 
	- Go to the EC2 > Instances > `ExampleAPP`
	- Connect (it should open the terminal)
	- write 
		- `sudo su ` to get the root privilege
		- `cd /home`
		- `wget <data_dump_url>`
		- 
	- Now if you back to the instances you should see there is more than one instance running
	- Now go to the AWS Secret Manger
		- secret
		- choose the database secret 
		- in secret value choose `Retrieve secret value`
		- copy the password: *it will seem like  zW9i(5B7BT4qx4.[TF.L)Ml4hR>8*
	- Go to RDS and get the End-point-url
	- back to the terminal 
		- `mysql -u admin -p'thesecrte_password' -h <endpoint>`
		- `show databases;`
		- `use countries;`
		- `show tables;`
		- `exit`
		- `mysql -u admin -p countries -h <endpoint> < Countrydatadump.sql`
		- `mysql -u admin -p'thesecrte_password' -h <endpoint>`
		- `use countries`
		- `show tables;`

5. **Testing the website**
	- Go to Load balancer
	- copy the DNS url
	- open the link in a new tap 
	- Choose query > choose any of these category 
	- Submit

