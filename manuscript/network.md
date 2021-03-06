# Understanding the network on Docker

What Docker calls network, in fact, is an abstraction created to ease the data communication management between containers and untie the external knots of the Docker environment. 

Don’t mistake the Docker network with the already known network used to group the IP addresses (ex.: 192.168.10.0/24). Therefore, every time we mention this second type of network, we’ll use “IP network”. 

### Standard networks on Docker

Docker has three standard networks. These networks offer specific configurations to manage data usage. To visualize these interfaces, just use the command below:

docker network ls

The return is going to be:

![](images/resultado_rede.png)

#### Bridge

Each container created on Docker is associated to an specific network. This is the standard network to any container, unless we associate, explicitly, another network to it. The network gives to the container an interface that makes a bridge with the docker0 interface of the Docker host. This interface receives, automatically, the next address available in the IP network 172.17.0.0/16. 

All containers in this network can communicate via TCP/IP protocol. If you know which is the IP address of the container you wish to connect, it is possible to send data to it. After all, they are all in the same IP network (172.17.0.0/16).

A detail worth noticing: as the IPs are assigned automatically, it’n not a trivial task to discover which is the IP of the destination container. To help on this location, Docker provides, at the moment of creating a container, the option “-link”. 

> It’s important to emphasize that “-link” is an outdated option and its use is discouraged. We’ll explain this feature only for understanding the legacy. This function was replace with a built-in DNS on Docker and it doesn’t work for Docker standard networks, only for networks created by the user. 

The option “-link” is responsible for associating the destination container IP to its name. In case you create a container from the Docker image of the mysql with the name “bd”, then create another with the name “app” from the image tutum/apache-php, you wish that the later container can connect on mysql using the name of the container “bd”, just create both containers the same way:

```
docker container run -d --name bd -e MYSQL_ROOT_PASSWORD=minhasenha mysql

docker container run -d -p 80:80 --name app --link db tutum/apache-php
```

After executing the commands, the container named “app” will be able to connect to the mysql container using the name “bd”, that is, every time it try to access the name “bd”, it will be automatically resolved to the IP of the IP network 172.17.0.0/16 that the mysql container got on its creation. 

To test it, we’ll use the function exec to run the command inside an existent container. For this, we will use the name of the container as a parameter of the command below: 

```
docker container exec -it app ping db
```
The action will be responsible for executing the command “ping db” inside the container “app”, that is, the container “app” will send icmp packages, usually used to test the connectivity between two hosts, to the “db” address. The name “db” is translated to the IP the container - created from the image mysql - got when created. 

**Example:** The “db” container created firstly and got the IP 172.17.0.2. The “app” container started next and got the IP 172.17.0.3. When the “app” container executes the command “ping db”, in fact, it will send icmp packages to the 172.17.0.2 address.

> Caution: The name of the option “-link” causes some confusion because it doesn’t create any IP network link between containers, once the communication between them is already possible, even if the option link is not configured. As we cleared up in the previous paragraph, it just facilitates the translation of names to the dynamic IP received in initialization.

The containers configures to this network will have the opportunity of external traffic using the routes of the IP networks defined on Docker host. In case Docker host has internet access, automatically, the containers will also have. 

In this network is possible to display containers ports to all the actives that have access to the Docker host. 

#### None

This network aims to isolate the container regarding external communications. The network doesn’t get any interface to external communication. The only interface of the IP network will be the localhost. 

This network, usually, is used for containers that only manipulate files, with no need of sending them to another place using the network (Ex.: backup container uses the volumes of the database container to do the dump and will be used in the data retention process. 

![Example of none network](images/rede_none.png)

If you have any questions on using volumes on Docker, check out [this article](https://imasters.com.br/devsecops/entendendo-o-armazenamento-de-dados-docker) and see more on Docker storage.

#### Host

This network has the objective of delivering into the container all the interfaces existent on Docker host. In a way, it can speed up the package delivery, once there’s no bridge on the way of the messages. But usually this overhead is minimum and the use of a bridge can be important to security and management of traffic. 

### Networks set by user

Docker allows the user to create networks. These networks are associated to the element the Docker calls network driver. 

Each network created per user must be linked to a given driver. And in case you didn’t create your own driver, you must choose amongst the drivers provided by Docker:

#### Bridge

This is the network driver more simple to use, for requires little configuration. The network created by the user using the bridge driver is similar to the Docker standard network named “bridge”. 

> One more point that deserves attention: Docker has a standard network called “bridge” that uses a driver also called “bridge”. Maybe, because of this, the confusion only gets bigger. But it is important to make clear that they are distinct. 

The networks created by the user with the bridge driver have all features described in the standard network, called bridge. However, it has additional features. 

Amongst one of the features: the network created by the user doesn’t need to user the old “-link” option. Because every network created by the user with the bridge driver will be able to user the Docker internal DNS that automatically associates every container names of this network to its respective IPs from the corresponding IP network. 

To make it clearer: the containers using the standard bridge network will not be able to enjoy the Docker internal DNS feature. In case you are using this network, it is necessary to specify the “-link” legacy for translating the names in IP addresses dynamically allocated on Docker. 

To exemplify the usage of the network created by user, let’s create the network called isolated_nw with the bridge driver:

```
docker network create --driver bridge isolated_nw
```
Now, we verify the network:

```
docker network list
```
The result must be:

![](images/resultado_rede2.png)

Now, we create a container at the isolated_nw network:

```
docker container run -itd --net isolated_nw alpine sh
```

![Isolated network](images/bridge_network.png)

It’s important to emphasize: a container that is in a given network doesn’t access another container that is in another network. Even if you know the destination IP. For one container to access another in another network, it’s necessary that the origin is present in both network that you wish to reach. 

The containers in the isolated_nw network can expose their ports on Docker host and these ports can be accessed both by containers outside the network - the isolated_nw - and external machines with access to Docker host. 

![Isolated network published port](images/network_access.png)

To find the container associated with a given network, use the below command:

```
docker network inspect isolated_nw
```

The result must be:

![](images/resultado_rede3.png)

In the section “Containers” it is possible to check which containers are part of this network. All containers that are in the same network will be able to communicate using only their respective names. As we can see in the example above, if a new container access the isolated_nw network, it will be able to access the amazing_noyce container using only its name.

#### Overlay

The overlay driver allows the communication between Dockers hosts; by using it, the containers of a given Docker host will be able to access, natively, containers from another Docker environment. 

This driver require a more complex configuration, therefore, we’ll approach the details in some other opportunity.

### Using networks on Docker Compose

The subject deserves a whole paper for itself. So, we’ll just show [an interesting link](https://docs.docker.com/compose/networking/) for furthers references on the subject. 

### Concluding

We realize that the use of networks defined by the user make the option “-link” obsolete, as well as provides a new internal Docker DNS service, making it easy for those who want to keep a big and complex Docker infrastructure, as well as provide the network isolation of its services.   

To know and to use well the new technologies is a good practice that avoids future problems and facilitates building and maintaining big and complex projects. 
