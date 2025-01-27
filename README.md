Omar BaGunaid
## Assignment Implementation Overview
For this assignment I have divided the implementation into 3 sections. First will have to implement the `dockerfiles` images and ensure that they are running correctly. Next will implement infrastructure as a code to deploy the following:

[TODO] Needs reversion...
1. Network
	1. VPC 10.0.0.0/16 <--> IGW
	2. Subnet 10.0.1.0/24 <--> public route <--> VPC_IGW
2. Web-server
	1. EC2 Instance
	2. Security Group
		1. SSH
		2. ports 8081, 8082, 8083
		3. ubuntu OS
		4. Docker
3. Combining everything in a yaml instructions
	1. Will use github actions to:
		1. create private repo in AWS ECR
		2. create an image from a `dockerfile` and push it to the repo
		3. run terraform to deploy the network infrastructure
		4. run terraform to deploy the web-server infrastructure (will have to attach the key.pub in the github repo) This key will be used for the EC2 instance. And will be helpful to connect to it for testing and ensuring everything is working in the cloud.
		5. Will setup the ports in ubuntu vm, will ensure that the required ports are open from there as well.
		6. will ensure these ports are mapped from the containers to the vm (host) as well.

>[!NOTE]
>Will ensure all containers created and mapped the ports required. And all containers are within a specific user-defined network brdige.

### Topology
![](./screenshots/Pasted%20image%2020250127110953.png)