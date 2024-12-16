# 3tier
3-tier architecture report.

I’ll explain the 3-tier architecture using the Terraform file (.tf file) in the attached VX code script. First, define the cloud provider in the provider block. Terraform requires Linux knowledge for IAC.

For simplicity, I’ll use bullet points for each Terraform script and attach pictures when necessary.

1.	Provider block: The provider.tf file defines the cloud provider (AWS in this case).
2.	VPC block: The vpc.tf file creates the virtual private cloud (VPC), which is like your private space in the cloud. All VPC attributes are explained in the script.
3.	Subnets: In the VPC (house), subnets are like rooms and partitions. We have two kinds of subnets: private and public. The web server (bastion host, websubnet.tf) will be on the public subnet, while the database (dbsubnet.tf) and application (appsubnet.tf) servers will be on the private subnet.
4.	To enable internet communication, we set up an internet gateway (internetgw.tf) and a network address translation (Nat) gateway (natgw.tf). Before using a Nat gateway, we need an elastic IP address, which is also defined in the natgw.tf file. To save costs, we attach a single Nat gateway to all jump servers. Ideally, we should have three.
5. To coordinate traffic, we use the route table (routetable.tf) and the route association script (routeassociation.tf). The route is attached to a subnet, with the public subnet route directed to the internet gateway and the private subnet route to the Nat gateway.

6. With all our routes and subnets configured, we’ve completed the foundation of our infrastructure. The next step is to build upon it. The jumpserver.tf file contains the configuration for a jump server instance. This instance includes a security group that allows access to the application server’s server. This security group enables SSH access to the application server for enhanced security. We’ve deployed three jump servers across three subnets to ensure high availability for our application.

7. Next, we created and installed our application on the application data file (php_instance.tf). To connect to the server via the jump server using SSH, we need a key. The script (jump_server.tf) contains a block to generate the key. Subsequently, we created a security group to allow access to the web server’s security group, further enhancing security. Additionally, the provisioner block (provisioner.tf) installs PHP, my admin, and the application on the server. Another block (remote_access.tf) remotely accesses the instance to ensure proper installation.
 
Figure 1: Showing all the EC2 instances created by the terraform script 



8. Since our application resides in a different subnet, we need a mechanism to distribute traffic evenly among the instances. This is where the application load balancer (alb.tf) comes into play.


 
Figure1.1: Showing the application load balancer for the infrastructure 

9. Finally, we’ve installed the database layer in the db_ins.tf file. This file contains the script for creating an RDS instance. It includes all the necessary configurations to launch the instance in multi-AZ and has already established an endpoint connection with the application server.

 
Figure1.2: Showing the RDS database connected to the infrastructure 


   
Figure1.3: Showing the landing page using the ALB IP address. 

This concludes all the steps to deploy the three-tier architecture. As I mentioned earlier, Linux plays a crucial role in this process. You’ll initialize Terraform by running Terraform init, which will plan the deployment and then apply the changes to the infrastructure. For this project, the state file is saved in the cloud on an S3 bucket, and the Terraform script that handles this is statefile-backend.tf. 
