# Creating your own image on Docker 


Before we explain how to create your image, it’s important to bring up a question that usually confuses Docker beginners: “Image or container?”
### What’s the difference between Image and Container?

Making a parallel with the concept of [object orientation](https://pt.wikipedia.org/wiki/Orienta%C3%A7%C3%A3o_a_objetos), **image** is the class and **container** is the object. The image is the infrastructure abstraction with reading only status, from where the container is going to be instantiated. 
Every container is created from an image; thus we can conclude that we will never have an image running. A container can only be created from a single image. In case you require a different behavior, it will be necessary to customize the image. 

### Anatomy of an image

Images can be official and non-official.

#### Official and non-official images 

The official Docker images are those with no users in their names. The image **“Ubuntu:16.04”** is official; on the other hand, the image [“nuagebec/ubuntu”](https://hub.docker.com/r/nuagebec/ubuntu/) is non-official. This second image belongs to user [nuagebec](https://hub.docker.com/u/nuagebec/), who keeps other non-official images. Official images are maintained by Docker and are [available](https://hub.docker.com/explore/) at the Docker cloud.
 The goal of the official images is to promote a basic environment (i.e. debian, alpine, ruby, python), a starting point for users to create images, as we will explain further ahead in this chapter. 
The non-official images are kept by users who created them. We will talk about sending images to the Docker cloud on another topic.


**Name of the image**

The name of an official image is formed by two parts. The first one is called **“repository”** according to the [documentation](https://docs.docker.com/engine/userguide/containers/dockerimages/), and the second one is called **“tag”**. In the example of image **“ubuntu:14.04”**, **ubuntu** is the repository and **14.04** is the tag. For Docker, the **“repository”** is an abstraction of the image set. Don’t mistake it by the image storage, that we will approach later. The **“tag”** is an abstraction to create unity inside the image set determined in the **“repository”**. 
A **“repository”** can contain more than one **“tag”** and each set repository:tag represents a different image. 

Execute the [command](https://docs.docker.com/engine/reference/commandline/images/) below to visualize all images that are found locally in your station at this exact moment:

```
docker image list
```

### How to create images 

There are two ways of creating customized images: using **commit** and using **Dockerfile**. 

#### Creating images with commit

It is possible to create images executing the [commit](https://docs.docker.com/engine/reference/commandline/commit/) command, related to a container. This command uses the current status of the chosen container and creates the image based on it. Let’s see the example. First, we create a container:

```
docker container run -it --name containercriado ubuntu:16.04 bash
```

Now that we are at the container bash, we install the nginx:

```
apt-get update
apt-get install nginx -y
exit
```

We stop the container using the command below:

```
docker container stop containercriado
```

Now, we **commit** this **container** into an **image**:
```
docker container commit containercriado meuubuntu:nginx
```

In the previous example, **containercriado** is the name of the container created and altered in the previous steps; the name **meuubuntu:nginx** is the **commit’s** resulting image; the status of **containercriado** is stored in an image called **meuubuntu:nginx** which, in this case, holds the only alteration we have in the official image of ubuntu on version 16.04 – the **nginx** package installed. 
To see the image list and find the one you just created, execute the command below again:

```
docker image list
```

To run a test in you new image, let’s creat a container from it and check if the nginx is installed:

```
docker container run -it --rm meuubuntu:nginx dpkg -l nginx
```

If you want validation, run the same command on the official image of ubuntu: 

```
docker container run -it --rm ubuntu:16.04 dpkg -l nginx
```

> It’s worth to emphasize that the **commit** method is not the best option to create images, because, as we saw, the process of altering the image is completely manual and presents a little difficulty to track the alterations made, once the manual changes are not registered automatically in the Docker structure. 

#### Creating images with Dockerfile 

When you use Dockerfile to create an image, basically you are presented a list of instructions that will be applied in a given image to the other one is created based on alterations. 

![Dockerfile](images/dockerfile.png)

We can summarize this by saying that the Dockerfile file, in fact, represents the exact difference between a given image, that we call **base**, and the image you want to create. In this model, we hold total traceability on what is going to be modified in the new image. Let’s go back to the example on the nginx installation on ubuntu 16.04. 

First, create a file for a future test:

```
touch arquivo_teste
```

Create a file called **Dockerfile** and insert the following content on it:

```
FROM ubuntu:16.04
RUN apt-get update && apt-get install nginx -y
COPY arquivo_teste /tmp/arquivo_teste
CMD bash
```

In the file above, we used four [instructions](https://docs.docker.com/engine/reference/builder/):

**FROM** to inform which image we will use as a base; in this case, **ubuntu:16.04**.

**RUN** to inform which commands are going to be executed in this environment to make the necessary changes in the system’s infrastructure. They’re like commands executed at the environment’ shell, just like the commit model, but in this case done automatically and it is completely traceable, since this Dockerfile will be store in the system at the version control.

**COPY** is used to copy files from the station where the building is being executed into the image. We use a test file just to exemplify this possibility, but this instruction is much used to send environment configuration files and codes to be executed in application services. 

**CMD** to inform which command will be executed as a standard, in case none is informed by the container initialization from this image. In the example, we put the command bash; if this image is being used to initiate a container and we don’t inform the command, it will execute the bash.

After building your Dockerfile, just execute the [command](https://docs.docker.com/engine/reference/commandline/build/) below:

```
docker image build -t meuubuntu:nginx_auto .
```

Such command has the option **“-t”**, that also works to inform the name of the image that is going to be created. In this case, is going to be **meuubuntu:nginx_auto** and the **“.”** at the end, informing which context must be used in this image building. All files from the current folder will be send to the Docker service and only they can be used to manipulations on Dockerfile (example of using COPY).


#### The sequence matters

It is important to notice that the **Dockerfile** file is a sequence of instructions read from the top to the base, and each line is executed at a time. If any instructions depend on another one, this dependency must be described earlier in the document. 
The result of each file instruction is stored in the local cache. If the **Dockerfile** is not modified in the next image creation (**build**), the process won’t take long, for everything will be in the cache. If there’s any alterations, just the modified instruction and the next will be executed again. 

The suggestion to better use the **Dockerfile** cache is to keep instructions often altered next to the base of the document. It’s important to remember to attend the dependencies between instructions as well. 


An example to make it clear:
```
FROM ubuntu:16.04
RUN apt-get update
RUN apt-get install nginx
RUN apt-get install php5
COPY arquivo_teste /tmp/arquivo_teste
CMD bash
```

If we modify the third line of the file and, instead of installing nginx, we change it to apache2, the instruction that updates on apt will not be executed again, but rather the installation of apache2, because it just entered the file, as well as the php5 and the file copy, because all of them are subsequent to the modified line. 
As we noticed, holding the **Dockerfile** file enables us to have the exact notion of which changes were made in the image, thus recording the modifications in the version control system.

### Sending your image to the cloud
