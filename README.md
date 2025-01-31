Omar BaGunaid
## Assignment Implementation Overview
The objective is to have an EC2 instance running with a total of four containers 3 of which will be web_applications and 1 of them will be a database using mysql which these running applications will need to use. All of these containers must be running in a custom network bridge mode.
#### Here is a breakdown of what will do:
1. Network
	1. VPC 10.0.0.0/16 <--> IGW
	2. Subnet 10.0.0.0/24 <--> public route <--> VPC_IGW
2. Web-server
	1. EC2 Instance
	2. Security Group
		1. SSH
		2. ports 8081, 8082, 8083
		3. ubuntu OS or Amazon Linux
	3. Docker
		1. Network bridge mode.
		2. Container of mysql-db will use port 3306 will not be mapped to the host.
		3. 3 containers will be mapped to the host with the following ports (8081, 8082, 8083)
		4. Containers will have a python application to use mysql-db.

#### Combining everything in a yaml instructions
1. Will use github actions to:
	1. create private repo in AWS ECR
	2. create an image from a `dockerfile` and push it to the repo
	3. run terraform to deploy the network infrastructure
	4. run terraform to deploy the web-server infrastructure (will have to attach the key.pub in the github repo) This key will be used for the EC2 instance. And will be helpful to connect to it for testing and ensuring everything is working in the cloud.
		-  will ensure these ports are mapped from the containers to the vm (host).

>[!NOTE]
>Will ensure all containers created and mapped the ports required. And all containers are within a specific user-defined network brdige.

Will be running TerraForm to deploy:
1. Deploy Amazon Linux EC2 Instance
2. Within terraform `user_data` Do:
	1. Download the images from AWS ECR
	2. Install Docker
	3. Create environment variables (i.e DBHOST, DBPORT, DATABASE, APP_COLOR1)
	4. Create network bridge mode
	5. Create mysql container
	6. Create 3 web_servers
	7. Ensure all of these containers running within the new network called `application`
### Topology

>[!NOTE]
>Did some modification to the port numbers used for convenience. Will use the names provided for each docker container.

This topology depicting how the EC2 instance will look like, will be creating the infrastructure from scratch. A VPC with IGW attached, within a single public-subnet. An EC2 Instance routed to the internet gateway. A security group to open required ports.

Within that EC2 Instance will have a docker host and running 4 containers.

![](./screenshots/Pasted%20image%2020250130153001.png)
### Testing in local environment

>[!NOTE]
>Assuming docker is installed.

Lets create a new network in docker using bridge mode.

```bash
docker network create --driver bridge application
```

Lets set the environment variables

```bash
export DBHOST=172.17.0.2
export DBPORT=3306
export DBUSER=root
export DATABASE=employees
export DBPWD=pw
export APP_COLOR1=blue
export APP_COLOR2=pink
export APP_COLOR3=lime
```

>[!NOTE]
>This can be added into the `.bashrc` file to skip adding them every time running a new shell.
### Building the images from a docker file

Building mysql database image.

```dockerfile
FROM mysql:8.0

COPY ./mysql.sql /tmp

CMD [ "mysqld", "--init-file=/tmp/mysql.sql" ]
```

```bash
docker build -t my_db -f ./src/dockerfiles/mysql/Dockerfile ./src/dockerfiles/mysql/
```

Building web application image.

```dockerfile
FROM ubuntu:20.04
RUN apt-get update -y
COPY . /app
WORKDIR /app
RUN set -xe \
    && apt-get update -y \
    && apt-get install -y python3-pip \
    && apt-get install -y mysql-client 
RUN pip install --upgrade pip
RUN pip install -r requirements.txt
EXPOSE 8080
ENTRYPOINT [ "python3" ]
CMD [ "app.py" ]
```

```bash
docker build -t web_app -f ./src/dockerfiles/web_server/Dockerfile ./src/dockerfiles/web_server/
```
### Running Containers

Running the database container first.

```bash
docker run -d -e MYSQL_ROOT_PASSWORD=pw  my_db
```

Then will run the other containers all the 3 webapps.

```bash
docker run -p 8081:8080  -e DBHOST=$DBHOST -e DBPORT=$DBPORT -e  DBUSER=$DBUSER -e DBPWD=$DBPWD -e APP_COLOR=$APP_COLOR1 -e DATABASE=$DATABASE my_app

docker run -p 8082:8080  -e DBHOST=$DBHOST -e DBPORT=$DBPORT -e  DBUSER=$DBUSER -e DBPWD=$DBPWD -e APP_COLOR=$APP_COLOR2 -e DATABASE=$DATABASE my_app

docker run -p 8083:8080  -e DBHOST=$DBHOST -e DBPORT=$DBPORT -e  DBUSER=$DBUSER -e DBPWD=$DBPWD -e APP_COLOR=$APP_COLOR3 -e DATABASE=$DATABASE my_app
```
