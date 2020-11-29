# Using Docker in multiple environments

Docker host is the name of the active responsible for managing Docker environments; in this chapter we will demonstrate how is it possible to create them and manage them in distinct infrastructures, such as virtual machines, cloud, and physical machine. 

![](images/machine1.png)

[Docker machine](https://docs.docker.com/machine/) is the tool used for this distributed management, and allow the docker hosts installation and management in an easy and direct way. 

This tool is very much used by users of the “non Linux” operational systems, as we will show, but its function is not limited to that, for is also much used to provide and manage the Docker infrastructure on cloud, such as AWS, Digital Ocean and Openstack. 


![](images/machine2.png)

### How it works

Before we explain how to use the Docker machine, we need to reinforce the knowledge on Docker architecture. 

![](images/machine3.png)

As the picture above shows, the usage of Docker is divided in two services: the one that runs in daemon mode, in background, called **Docker Host**, responsible for viabilization of containers on the kernel Linux; and the client, that we’ll call **Docker client**, responsible for getting commands from the user and translating them into management of **Docker Host**. 

Each **Docker client** is configured to connect itself to a given **Docker host** and, at this moment, **Docker machine** takes the action, for it enables the automatization of access configuration choice of Docker client to distinct **Docker hosts**. 

The **Docker machine** enables the use of several different environments just changing the client configuration to the desired **Docker host**: basically, modify some environment variables. Here’s the example:

```
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/gomex/.docker/machine/machines/default"
export DOCKER_MACHINE_NAME="default"
```
Modifying these four variables, the Docker client will be able to use a different environments rapidly and with no need of restart any service. 

### Creating environment

The Docker machine is good mainly for creating environments that in the future will be managed by it in the automatized exchange of configuration context, through the modification of environment, as we explained previously. 

To create the environment, it is necessary to verify if the infrastructure you wish to create has some driver that supports this process. Here is the [list of available drivers](https://docs.docker.com/machine/drivers/).

#### Virtual machine

For this example, we will use the most common driver, the [virtualbox](https://docs.docker.com/machine/drivers/virtualbox/), that is, we need a [virtualbox](https://www.virtualbox.org/) installed in our station so this driver works properly. 

Before creating the environment, let’s understand how does it work the creation command on Docker machine:

docker-machine create --driver=<nome do driver>  <nome do ambiente>

Regarding the driver **virtualbox**, we have a few parameters that can be used:

|Parameter   | Explanation |
|-----------|------------|
|--virtualbox-memory  | Specifies the amount of RAM memory that the environment can use. The standard value is 1024MB (always in MB).|
|--virtualbox-cpu-count | Specifies the amount of CPU cores that this environment can use. The standard value is 1.|
|--virtualbox-disk-size | Specifies the size of the disk that this environment can use. The standard value is 20000MB (always in MB).|

As a test we run the following command:

```
docker-machine create --driver=virtualbox --virtualbox-disk-size 30000 teste-virtualbox
```

The result of this command is the creation of a virtual machine at the virtual box. The machine will have 30GB of disk space, 1 core and 1GB RAM memory. 

To make sure the process happened as expected, just use the following command:

```
docker-machine ls
```
The command above is responsible for listing all the environments that can be used from the client’s station.

To change clients, just run the command:

```
eval $(docker-machine env teste-virtualbox)
```
Executing the command **ls** makes possible to verify which environment is active:

```
docker-machine ls
```

Initiate a test container to run a test on the new environment:

```
docker container run hello-world
```

If you want to change environments, just type the command below, using the name of the desired environment::

```
eval $(docker-machine env <ambiente>)
```
If you want to turn off the environment, use the command:

```
docker-machine stop teste-virtualbox
```
If you want to initiate the environment, use the command:

```
docker-machine start teste-virtualbox
```
If you want to remove the environment, use the command::

```
docker-machine rm teste-virtualbox
```
Known troubleshooting: in case you’re using Docker machine on MacOS and for some reason the station sleeps when the virtual box environment initiates, it is possible that, when it comes back, the Docker host presents some internet communication problems. We advise to, every time you go through connectivity problems on Docker host with virtual box driver, turn off the environment and reinitiate as a solution.

#### Cloud

For this example we will use the most common cloud driver, [AWS](https://aws.amazon.com/). For such, we need an AWS account to [this driver](https://docs.docker.com/machine/drivers/aws/) works properly.

It is required that your credentials are in the file ~/.aws/credentials as it follows:

```
[default]
aws_access_key_id = AKID1234567890
aws_secret_access_key = MY-SECRET-KEY
```

In case you don’t want to put these information on file, you can specify them via environment variables:

```
export AWS_ACCESS_KEY_ID=AKID1234567890
export AWS_SECRET_ACCESS_KEY=MY-SECRET-KEY
```
You can find more information on AWS credentials in [this article](https://blogs.aws.amazon.com/security/post/Tx3D6U6WSFGOK2H/A-New-and-Standardized-Way-to-Manage-Credentials-in-the-AWS-SDKs).

When we create an environment using the command **docker-machine create**, is is translated into AWS in the creation of a [EC2 instance](https://aws.amazon.com/ec2/) and then it is automatically installed in every required software in the new environment.

The most used parameters in the creation of this environment are:

|Parameter   | Explanation |
|-----------|------------|
|--amazonec2-region | Says what AWS region is used to host your environment. The standard value is us-east-1. |
|--amazonec2-zone | It’s the letter that represents the region used. The standard value is “a”. |
|--amazonec2-subnet-id | Says which sub-network is used in this EC2 instance. Needs to be created previously. |
|--amazonec2-security-group | Says which security group is used in this EC2 instance. Needs to be created previously. |
|--amazonec2-use-private-address | It will be created an interface with private IP, because as default it only specifies an interface with public IP. |
|--amazonec2-vpc-id | Says which VPC ID is desired for this EC2 instance. Needs to be created previously. |

As an example, we will use the following command while creating the environment:

```
docker-machine create --driver amazonec2 --amazonec2-zone a --amazonec2-subnet-id subnet-5d3dc191 --amazonec2-security-group docker-host --amazonec2-use-private-address --amazonec2-vpc-id vpc-c1d33dc7 teste-aws
```
After running the command, just wait it to finish up; it is common to take a while..

To test the success of the action, execute the command below:

```
docker-machine ls
```
Check if the environment called teste-aws exists in the list; if so, use the command below to change the environment:

```
eval $(docker-machine env teste-aws)
```
Create a test container to verify the new environment:

```
docker container run hello-world
```
If you want to turn off the environment, use the command:

```
docker-machine stop teste-aws
```
If you want to start the environment, use the command:

```
docker-machine start teste-aws
```
If you want to remove the environment, use the command:

```
docker-machine rm teste-aws
```
After being removed, the instance EC2, provided on AWS, will be automatically removed.