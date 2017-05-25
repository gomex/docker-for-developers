# Managing multiple Docker containers with Docker Compose

This article aims to explain in details, and with examples, how does it work the process of managing multiple Docker containers, because as your confidence in using Docker grows in you, your need of using a bigger number of containers increases in the same proportion, and following the good practice of keeping only one service per container commonly results in some extra-request.

![](images/compose.png)

Usually, with the increase of the number of running containers, it becomes evident the need of a better communication management, for it is ideal that the services can exchange data between containers when required, that is, you need to deal with the **network** of this new environment. 

Think about the work you would have executing dozens of containers manually in the command line, one by one and every required parameter, the network configurations between containers, volumes, etc. Well, you don’t have to think about it anymore, for that won’t be necessary. To meet this demand of management of multiple containers, the solution is [Docker Compose](https://docs.docker.com/compose/overview/).

**Docker compose** is a tool for define and execute multiple Docker containers. It makes possible to configure all required parameters to execute each container from a **definition file**. Inside this file we define each container as a **service**, that is, from now on, every time this text mentions **service**, imagine that that is the definition that is going to be used to initiate a **container**, such as exposed ports, environment variables, etc. 

With Docker Compose we can also specify which **volumes** and **network** will be created to be used in the **services** parameters; in other words, that means  that you don’t have to create them manually so the **services** use additional **network** and **volume** resources.

Docker Compose’s **definition file** is the place where all the environment is specified (**network**, **volume** and **services**); it’s written according to the [YAML](https://en.wikipedia.org/wiki/YAML) format. As a standard, this file is named [docker-compose.yml](https://docs.docker.com/compose/compose-file/).

### Anatomy of docker-compose.yml

The YAML standard uses the indentation to separate the code block from the definitions; because of this the use of indentation is very important, that is, if you don’t use it correctly, the docker-compose will fail to execute. 

Each line of this file can be defined as a value key or a list. Let’s see the examples to make the explanation clearer:

```
version: '2'
services:
  web:
    build: .
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        versao: 1
    ports:
      - "5000:5000"
  redis:
    image: redis
```
In the file above we have the first line that define the version of **docker-compose.yml**; in this case we will use the latest version. If you want to compare the difference amongst versions, check out this [link](https://docs.docker.com/compose/compose-file/#versioning).

```
version: '2'
```
On the same indentation level we have **services**, that define the beginning of the **services** block that will be define right below.

```
version: '2'
services:
```
On the second indentation level (here it’s done with two spaces) we have the name of the first **service** of this file, that gets the name **web**. It opens the **service** definitions block, that is, from the next level of indentation everything that is defined is going to be part of this service. 

```
version: '2'
services:
  web:
```
On the next level of indentation (done again with two more spaces) we have the first definition of the **web service** that, in this case, is the [build](https://docs.docker.com/compose/reference/build/) that informs that this service will be created not from an existent image, but it will be necessary to build your image before executing it. It also opens a new block of code to parameterize the operation of this image build. 

```
version: '2'
services:
  web:
    build: .
```
On the next level of indentation (done again with two more spaces) we have a **build** parameter that, in this case, is the **context**. It is responsible for informing which file context will be used to build the given image; in other words, only files that exist inside this folder will be able to be used in the image building. The context chosen was the **”./dir”**, that is, this indicates that a folder named **dir**, that is in the same file system level of **docker-compose.vml** or of the place where this command will be executed, will be used as a context in the creation of this image. When, soon after the key, a value is provided, this indicates that no block of code will be opened. 

```
    build: .
      context: ./dir
```
On the same level of indentation of the **context** definition, that is, still inside the **build** definition block, we have the **dockerfile**, that indicates the name of the file that will be used to build the given image. It would be the equivalent to the parameter [“-f”](https://docs.docker.com/engine/reference/commandline/build/#specify-dockerfile-f) of the **docker build** command. If this definition didn’t exist, the **docker-compose**, as a standard, would look for a file called **Dockerfile** inside the folder informed in the **context**.  

```
    build: .
      context: ./dir
      dockerfile: Dockerfile-alternate
```
On the same level of indentation of the **dockerfile** definition, that is, still inside the **build** definition block, we have the **args**, that defines the arguments that will be used by **Dockerfile**; it’s similar to the parameter [“–build-args”](https://docs.docker.com/engine/reference/commandline/build/#set-build-time-variables-build-arg) of the **docker build** command. As its value is not informed in the same line, it’s evident that it opens a new block of code. 

On the next level of indentation (done again with two more spaces) we have the key **”version”** and the value **”1”**, that is, as this definition is part of the
code block **args**, this value key is the only argument sent to **Dockerfile**; in other words, the given **Dockerfile** file must be prepared to receive this argument or it will be lost in the image building. 

```
    build: .
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        versao: 1
```
Going back two indentation levels (four spaces less in relation to the previous line), we have the **ports** definition, that would be similar to the parameter [“-p”](https://docs.docker.com/engine/reference/commandline/run/#publish-or-expose-port-p-expose) of the Docker container run command. It defines which container port will be exposed at the **Docker host**. In our example, is going to be the container port *5000, with the **Docker host** port 5000.

```
  web:
    build: .
    ...
    ports:
      - "5000:5000"
```
Going back one indentation level (two spaces less in relation to the previous line), we leave the block of code of the **web** service; this indicates that no definition informed on this line will be applied to this service, that is, we need to start a block of code of a new service, that in our example will be named **redis**. 

```
  redis:
    image: redis
```

On the next indentation level (done again with two more spaces), we have the first definition of the **redis** service, that in this case is the **image**, that is responsible for showing which image will be used to initiate this container. This image will be found in the repository configured on **Docker host**, that is [hub.docker.com](https://hub.docker.com/) by default.

### Executing Docker Compose

After understanding and creation your own **definition file**, we need to manage it by using the docker-compose binary; the most common usage options are the following:

 * **build** : Used to build all the **services** images that are described with the definition **build** in their block of code.
 * **up** : Initiates all the **services** are in the **docker-compose.yml** file.
 * **stop** : Stops all the **services** that are in the **docker-compose.yml** file. 
 * **ps** :  Lists all the **services** that were initiated from the **docker-compose.yml** file. 

To check other options, see the [documentation](https://docs.docker.com/compose/reference/).
