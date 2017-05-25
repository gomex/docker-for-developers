# How to use Docker without GNU/Linux

This article aims to explain, with details and examples, the use of Docker in MacOS and Windows stations. 

![Docker Toolbox](images/docker_toolbox.png)

This text is for people who already know Docker, but still don’t know how Docker can be used in a “non Linux” station. 

As we said previously, Docker uses specific resources of the host operational system. Today, we have support for Windows and GNU/Linux systems, This means that is not possible to initiate Docker containers in a MacOS station, for instance. 

But don’t worry if you don’t use GNU/Linux, or Windows as an operational system, it is still possible to use this technology without necessarily, executing it in your computer. 

It’s worth to emphasize that Docker images and containers created on Windows won’t work in GNU/Linux because of the dependency of the operations system mentioned earlier. 

It’s possible to user Docker on MacOS and Windows in two ways:

* Toolbox
* Docker For Mac/Windows

Because it’s more complex, therefore demanding a greater context, we will approach in this chapter only the installation and configuration of [Docker Toolbox](https://www.docker.com/products/docker-toolbox). This solution is, in fact, an abstraction for installation of every environment that requires to use Docker from a MacOS or Windows station. 

The installation is simple: both on Windows and on MacOS, just download the corresponding installer in this [site](https://www.docker.com/products/docker-toolbox); and execute it following the steps in the screen.

The softwares installed on the station - MacOS or Windows - from the Docker Toolbox package are: 

* [Virtualbox](https://www.virtualbox.org/)
* [Docker machine](https://docs.docker.com/machine/overview/)
* [Docker client](https://docs.docker.com/)
* [Docker compose](https://docs.docker.com/compose/overview/)
* [Kitematic](https://docs.docker.com/kitematic/userguide/)

Docker Machine is the tool that allows to create and maintain Docker environments in virtual machines, cloud environments and even in physical machines. But in this topic we’ll only talk about the virtual machine with virtual box. 

After installing Docker Toolbox, it’s simple to create a Docker environment with a virtual machine using Docker Machine. 

First, let’s verify if there’s no virtual machines with Docker installed in their environment:

```
docker-machine ls
```
The command above shows only environments created and maintained by their Docker Machine. It’s possible that, after installing Docker Toolbox, you won’t find any machine. In this situation, we use the command below in order to create the machine:

```
docker-machine create --driver virtualbox default
```

![Arquitetura do Docker Toolbox](images/docker_toolbox1.jpg)

The command creates an environment called “default”. In fact, is a virtual machine (“Linux VM” that appears on the image) created on virtual box. With the command is possible to see the machine created:

```
docker-machine ls
```

It should retorne something like this:

![](images/resultado_macos_windows.png)

A virtual machine was created, inside it we have a GNU/Linux operational system with Docker Host installed. This Docker service is listening on the TCP 2376 port from the 192.168.99.100 address. This interface uses a specific network between your computer and the virtual box machines.

To turn off the virtual machine, just execute the command below:

```
docker-machine stop default
```
To start the machine again, just execute the command:

```
docker-machine start default
```
The command “start” is responsible only for starting the machine. It’s necessary that the Docker control applications, installed on the station, will be able to connect to the virtual machine created on virtual box with the command “docker-machine create”.

The control applications (Docker and Docker-compose) use environment variables to configure which Docker Host will be used. The command below facilitates the work of using all the variables correctly:

```
docker-machine env default
```

The result of this command on MacOS is:

![](images/resultado_macos_windows2.png)

As we can see, it shows what can be done to configure all the variables. You can copy the first four lines, that start with “export”, and paste on the terminar, or take just the last line without the “#” in the beginning and execute on the command line:

```
eval $(docker-machine env default)
```

Now, the control applications (Docker and Docker-Compose) are ready to use Docker Host form the connection made in the 192.168.99.100 IP service - machine created with the previously mentioned “docker-machine create” command.

To test, we list the running containers in this Docker Host using the command:

```
docker ps
```
While executed in the MacOS or Windows’ command line, this Docker client connects itself to the virtual machine that here we call “Linux VM”, and requires the list of running containers at the remote Docker Host. 

We initiate a container with the command below:

```
docker container run -itd alpine sh
```
Now we verify again the list of running containers:

```
docker ps
```
We can see that the container created from the image “alpine” is running. It’s important to emphasize that this process is executed on Docker Host, in the machine created inside the virtual box that, in this example, holds the IP 192.168.99.100.

To verify the machine’s IP address, just execute the command below:

```
docker-machine ip
```
If the container exposes any port to the Docker Host, whether via “-p” parameter of the “docker container run -p porta_host:porta_container” command or via “ports” parameter of the docker-compose.yml, it’s good to remember that the IP to access the exposed service is the IP address of the Docker Host; in the example, is “192.168.99.100”.

At this moment, you must be asking yourself: how is it possible to map a folder from the “non Linux” station into a container? Here enters a new Docker artifice to overcome this problem. 

Every machine created with the “virtual box” driver automatically creates a mapping of typ “virtual box shared folders” from the user folder to the Docker Host root. 

To visualize this mapping, we access the virtual machine we’ve just created in the previous steps:

```
docker-machine ssh default
```
In the GNU/Linux machine console, type the following commands:

```
sudo su
mount | grep vboxsf
```

The [vboxsf](https://help.ubuntu.com/community/VirtualBox/SharedFolders) is a file system used by virtual box to put together shared volumes of the station used to instal the virtual box. In other words, using the shared folder resource makes possible to set up the MacOS folder /Users in the folder /Users from the virtual machine of the Docker Host. 

All the content in the MacOS folder /Users/SeuUsuario will be accessible in the folder /Users/SeuUsuario of the GNU/Linux machine that works as a Docker Host in the example. In case you set up the folder /Users/SeuUsuario/MeuCogidog into the container, the data to be set up is the same of the station and nothing needs to be done to replicate this code into the Docher Host. 

Let’s test it. Create a new file inside the user folder:

```
touch teste
```
We start a container and map the current folder into it:

```
docker container run -itd -v "$PWD:/tmp" --name teste alpine sh
```
In the command above, we initiated a container that will be named “test” and will map the current folder (the variable PWD indicates the current address on MacOS) in the folder /tmp, into the container. 

We verify if the file we’ve just created is inside the container:

```
docker container exec teste ls /tmp/teste
```
The line above executed the command “ls /tmp/teste” inside the container named “test”, created in the previous step.

Now, access Docker Host with the command below, and verify if the test file is in the user folder:

```
docker-machine ssh default
```
**Can everything be done automatically? Yes, of course!**

Now that you know how to do it manually, if you need to install Docker Toolbox in a new machine and doesn’t remember the commands to create the new machine or simply how to set the environment for use, just execute the “Docker Quickstart Terminal” software. It will do the job automatically. In case this machine doesn’t exist, it creates one named “default”. In case the machine is already created, it automatically configures its environment variables and sets up for using the remote Docker Host from the control applications (Docker and Docker-Compose).