# Why using Docker?

Docker has been a very commented subject lately, many articles have been written, usually talking about how to use it, auxiliary tools, integrations and the like, but many people still ask the most basic question when it’s about the possibility of using any new technology: “Why should I use this?” Or would it be: “What this has to offer me that is different from what I have today?”

![](images/docker_porque.jpg)

It is natural that people still doubt the Docker’s potential, some even think that it’s about some hype. But in this chapter we intend to show some good reasons to use Docker. 

It’s important to highlight that Docker is not a “silver bullet” – it is not intended to solve all the problemas, much less being the only solution to several situations. 

Here are some good reasons for using Docker:

### 1 – Similar environments

It’s worth to highlight that Docker is not a “silver bullet” – it is not intended to solve all the problemas, much less being the only solution to several situations. Once your application is turned into a Docker imagem, it can be instantiated as a container in any environment you wish. That means you can use the application at the developer’s notebook as well as it would run at the production server.

The Docker image accepts parameters at the start of container, thus indicating that the same image can behave differently in distinct environments. This container can connect to its loca database for testing, using the credentials and the testing database. But when the container created from the image receives parameters from the production environment, it will access the database in a more robust infrastructure, with its respective production credentials and database, for instance. 

The Docker images can be considered atomic implantations – which provides more predictability compared to other tools such as Puppet, Chef, Ansible etc. – impacting positively on errors analysis, as well as on the reliability of the [continuous delivery process](https://www.thoughtworks.com/continuous-delivery), which is strongly based on the creation of a single artefact that migrates between environments. In the case of Docker, the artefact would be the image itself with all the dependencies required to execute its code, whether compiled or dynamic. 

### 2 – Application as a whole package
 
Using the Docker images makes possible to package all of your application and dependencies, making the distribution easy because it won’t be necessary to send an extent documentation explaining how to configure the required infrastructure to allow the execution, just make the image available in a repository and grant the access to the user, so the user can download the build that will be executed with no problems.

Updating is also positively affected, because the [layer structure](https://imasters.com.br/devsecops/entendendo-o-armazenamento-de-dados-docker) of Docker allows that, in case of change, only the altered part is transferred, so the environment can be changed faster and simpler. The user only needs to execute one command to update the application image, that will be reflected on the container running on the desired moment. The Docker images can hold tags, thus making possible to store multiple versions of the same application. That means that, if there’s a problem on the update, the backup plan will be basically to use the image along with the previous tag. 

### 3 – Standardization and replication 

As the Docker images are built with [definition files](https://docs.docker.com/engine/reference/builder/), it is possible to guarantee that a given pattern will be followed, increasing the confidence on replication. You just the need the images to follow the [best practices] (https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) of building so it is viable to [escalate](https://en.wikipedia.org/wiki/Scalability) the structure quickly. 

In case a new member enters the team to work on development, he/she can get the work environment with a few commands. This process will take the time of downloading the images that are going to be used, as well as the definition files to manage them. This helps the introduction of a new member in the process of developing an application, who will be able to quickly reproduce the environment in his/her station, thus writing codes according to the team’ standard. 

If there’s the need of testing a new version of a certain part of the solution, using Docker images, usually it’s only necessary to change one or more parameters of the definition file in order to start a modified environment with the requested version to evaluation. That is: creating and modifying the infrastructure got easier and faster. 

### 4 – Common language between infrastructure and development 

The syntax used to parameterize the Docker images and environments can be considered as a common language between areas that usually don’t dialogue well. Now it’s possible to both sectors to make proposals and counter-proposals based on a common document. 

The required infrastructure is going to be present in the code of the developer and the infrastructure area will be able to analyse the document, suggesting changes to get in sync with the standards of the sector or not. All that through comments and acceptance of *merge* or *pull request* from the code version control system. 

### 5 – Community 

As it is possible to access [github](https://github.com/) or [gitlab](https://about.gitlab.com/) to search code samples, using the [image repository](http://hub.docker.com/) of Docker makes possible to get good models of application infrastructure or services for complex integrations. 

For example: [nginx](https://hub.docker.com/_/nginx/) as a reverse proxy, and [mysql](https://hub.docker.com/_/mysql/) as a database. In case the application needs these two resources, you don’t have to need to waste time installing and setting up theses services. Just use the images from the repository, setting up minimum parameters for suitability to the environment. Usually the official images follow the good practices for using the services offered. 

Using these images doesn’t mean to “be held hostage” of their configuration, because it is possible to send you own configuration to the environments and just prevent the basic installation. 

### Questions

Some people sent some questions regarding the advantages we presented in this text. Thus, instead of answering them one by one, we decided to publish the questions and their answers here. 

#### What is the difference between Docker image and definitions created by an infrastructure [automation tool](https://www.ibm.com/developerworks/br/library/a-devops2/)?

As an example of infrastructure automation tools we have [Puppet](https://puppetlabs.com/), [Ansible](https://www.ansible.com/), and [Chef](https://www.chef.io/chef/). They can guarantee similar environments, once their job is to keep a given configuration on the desired asset. 

The difference between the Docker solution and the configuration management may seem very thin, for both can support the necessary configuration of every infrastructure that an application demands to be implanted, but we think that one of the most relevant differences is in the following fact: the image is a complete abstraction and doesn’t require any treatment to deal with most varied GNU/Linux distributions that exit, since the Docker image comes along with a full file copy of a lean distribution.

To carry within the copy of a GNU/Linux distribution is usually not a problem for Docker, because using the layer model saves a lot of resources by reusing the base layers. Read [this article](https://imasters.com.br/devsecops/entendendo-o-armazenamento-de-dados-docker) to know more about Docker storage.

Another advantage of image in relation to the configuration management is that, when using the image, it is possible do make available the complete application package in a repository, and this “final product” be easily used without needing a complete configuration. Just one configuration file and one command can be enough to start an application build as a Docker image. 

Still on the process of the Docker image as a product in the repository: it can also be used in the process of updating the app, as we previously explained in this chapter.

#### The use of the base image on Docker of a given distribution is not the same of creating a definition of a configuration management for a distribution?

No! The difference is in the host perspective. On Docker, it doesn’t matter which GNU/Linux distribution is used on the host, for there is a part of the image that carries all the files from a mini-distribution that will be sufficient to support everything the app needs. In case your Docker host is Fedora and the app needs files from Debian, don’t worry because this given image will bring up Debian files to support the environment. As said previously, this usually doesn’t affect negatively in disk space consumption.

#### Does it mean that I, as a developer, have to worry about everything on Infrastructure?

No! When we say that it is possible to the developer to specify the infrastructure, we are talking about the closest layer of the application and not all the required architecture (Basic operational system, firewall rules, network rules etc.).

The ideia on Docker is that relevant subjects directly connected to the application can be configured by the developer. This does not obligate him/she to perform this activity. This is a possibility that pleases many developer, but in case it is not your situation you can relax, another team will deal with this part. The deploy process will get a little slower. 
 
#### Many people refer to Docker for be used with [microservices](https://www.thoughtworks.com/pt/insights/blog/microservices-nutshell). Is it possible to use Docker to monolithic applications?

Yes! However, in some cases minor changes in the applications are required so it can enjoy the facilities of Docker. A common example is the log that usually the application sends to a given file, that is, in the Docker model the applications in the containers should not try to write or generate log files. Au contraire, each process in execution writes its own event flow, no buffer, to [stdout](https://en.wikipedia.org/wiki/Standard_streams), because Docker holds specific driver to treat the log sent this way. The subject on best practices of log manager will be approached in the next chapters.  

At some point you will realize that using Docker to your application demands lots of effort. In this cases, usually the problem relies in how the application works and not on the Docker configuration. Be aware of that. 

Do you have more questions and/or good reasons for using Docker? Leave your comment [here](https://github.com/gomex/docker-for-developers/issues).
