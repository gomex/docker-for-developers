# Basic commands

For using Docker it is necessary to know a few commands and understand directly and clearly what they do, as well as some examples of use. 
We are not approaching the commands for creating image and troubleshooting on Docker, because there are specific chapters on these subjects. 

## Running a container

To create a container is necessary to know from each image it will be executed. In order to list the images on your Docker host, execute the command below:

```
docker image list
```

The images that appear are on your **Docker host** and do not require any download from the [Docker public cloud](hub.docker.com) unless you wish to update it. To update the image, just execute the command below:
```
docker image pull python
```

We use the image named **python** as an example, but in case you wish to update any other image, just replace **python** with your name.

In case you want to inspect the image you just updated, just use the command below:

```
docker image inspect python
```
The command [inspect](https://docs.docker.com/engine/reference/commandline/inspect/) is responsible for informing every image corresponding data. Now that the image is updated and inspected, we can start the container. But before we simply copy and paste the command, let’s see how it really works.

```
docker container run <parâmetros> <imagem> <CMD> <argumentos>
```

The most used parameters in the container’s execution are:
|Parameter   | Explanation                                                                   |
|------------|-------------------------------------------------------------------------------|
|-d          | Running container on background                                               |
|-i          | Interactive mode. Keeps the STDIN open even without console attached          |
|-t          | Allocates a pseudo TTY                                                        |
|--rm        | Automatically removes the container after finishing (**doesn’t work with -d**)|
|--name      | Name the container                                                            |
|-v          | Volume mapping  	                                                             |
|-p          | Port mapping                                                                  |
|-m          | Limit the use of RAM memory                                                   |
|-c          | Balance the use of CPU                                                        |

Here is a simple example on the following command:

```
docker container run -it --rm --name meu_python python bash
```
According to the command above, a container will be created with the name **meu_python**, created from the **python** image, and the process executed in this container will be the **bash**. 
It’s important to remember that, in case the **CMD** is not specified in the **Docker container run** command, it’s going the be used the standard value defined at the image’s **Dockerfile**. In this case is **python** and its standard command executes the **python** binary, that is, if the **bash** is not specified at the end of command on the example above, instead of a shell bash of GNU/Linux it would be shown a **python** shell. 


### Volume mapping

To map the volume, just specify the origin of the data at the host and where it should be set inside the container. 

```
docker container run -it --rm -v "<host>:<container>" python
```
The storage use is better explained in further chapters, that’s why we are not detailing the use of this parameter. 

### Port mapping

To map the ports, you just have to know which port will be mapped on host and which one should get this connection inside the container.

```
docker container run -it --rm -p "<host>:<container>" python
```

An example with the port 80 of the host to a port 8080 inside the container holds the following command:

```
docker container run -it --rm -p 80:8080 python
```

As the command above shows, we have the port **80** accessible on **Docker host**, that passes along all the connections to the port **8080** inside the **container**. In other words, it’s not possible to access the port **8080** at the IP address of **Docker host** because this port is only accessible inside the container, which is isolated at the network level, as said previously.


### Managing resources

While creating the containers it is possible to specify some limits on using resources. We will discuss here the most used – RAM memory and CPU.

To limit the use of RAM memory that can be used by this container, just execute the command below: 

```
docker container run -it --rm -m 512M python
```

With the command above we are limiting this container to use only 512MB of RAM.

To balance the use of CPU by the containers, we specify weights for each container; the lighter the weight, the less priority on use. The weights can oscillate from **1** to **1024**. 

In case the weight of the container is not specified, it will use the heaviest weight possible, in this case, **1024**.

We will use the weight **512** as an example:


```
docker container run -it --rm -c 512 python
```

To better understand, let’s imagine that three containers are running. One of them has the standard weight **1024** and the other two have **512**. In case the three processes require the whole CPU, their time of use will be divided as it follows:
* The process weighting **1024** will use 50% of the processing time. 
* The two processes weighting **512**  will use 25% of the processing time, each.

## Checking the list of containers

To visualize the list of containers of a given **Docker host**, we use [docker ps](https://docs.docker.com/engine/reference/commandline/ps/).

This command is responsible for displaying all containers, even those not running anymore.

```
docker container list <parâmetros>
```

The most used parameters in running a container are:

|Parameter   | Explanation  |  
|-----------|------------|
|-a | Lists all containers, including turned offs|
|-l | Lists the last containers, including turned offs|
|-n | Lists the last N containers, including turned offs|
|-q | Lists only the containers’ ids, great for using on scripts|

## Managing containers

Once the container is created from an image, it is possible to manage the usage with new commands. In case you wish to turn off the container, just use the command [docker stop](https://docs.docker.com/engine/reference/commandline/stop/). It gets as an argument the container **ID** or **name**. Both data can be obtained through **docker ps**, explained in the previous topic. 

An example:

```
docker container stop meu_python
```

In the command above, in case there was a container named **meu_python** running, it would receive a **SIGTERM** signal and, if it was not turned off, it would receive a **SIGKILL** after 10 seconds. 

In case you wish to restart the container that was turned off and not create a new one, just execute the command [docker start](https://docs.docker.com/engine/reference/commandline/start/):

```
docker container start meu_python
```

> It’s important to emphasize that the idea of containers is to be disposable. In case you use the same container for a long period without discarding it, you are probably using Docker wrongly. 
> Docker is not a machine, is a running process. And, as every process, it must be discarded so another one can take its place when reseting. 