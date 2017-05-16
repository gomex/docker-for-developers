# Understanding storage on Docker

To understand how Docker manages its volumes, first we need to explain how does it work at least a Docker storage [backend](http://searchdatacenter.techtarget.com/definition/back-end). We will do this here with the AUFS, that was the first one and still is the standard in a good part of Docker installations. 
![](images/aufs_layers.jpg)

### How does it work a Docker backend (ex.: AUFS)

Storage backend is the part of Docker that takes care of data management. On Docker there are several possibilities of storage backend, but here we’ll only talk about the one that deploys the AUFS. 
[AUFS](https://en.wikipedia.org/wiki/Aufs) is a unification file system. It is responsible for managing multiple directories, stacking them up, and provide a single and unified view, as if they all together were only one directory. 
This single directory is used to present the container and works as if it was a single common file system. Each directory used in the stack corresponds to one layer. And that’s how Docker unifies them and provides reutilization amongst containers. Because the same directory that corresponds to the image can be set up in several stacks of several containers. Aside the folder (layer) that corresponds to the container, every other one is set up with read only permission; otherwise, the changes in a container could interfere on another one. And this really goes totally against the principles of Linux Container. 
In case it is necessary to modify a file on the layers (folders) referring to the images, the technology [Copy-on-write](https://en.wikipedia.org/wiki/Copy-on-write) (CoW) is used, and it’s responsible for copying the file to the folder (layer) of the container and to do all the modifications on this level. Thus, the original file of the inferior layer is superimposed in this stack, that is, the given container will always see only the files of the highest layers.

![Removing a file](images/aufs_delete.jpg)

In case of removal, the file of the superior layer is marked as whiteout file, enabling the visualization of the file of lower layers. 
### Performance issues

Docker takes advantage of the AUFS’s Copy-on-write (CoW) technology to allow image sharing and the use of disk space. AUFS works at file level. This means that all AUFS CoW operations will copy whole files even if only a little part of the file is being altered. This behavior can significantly impact the container’s performance, especially if the copied files are big and are located below several image layers. In this case, the procedure copy-on-write will spend a lot of time to make an internal copy.

### Volume as a performance solution

By using volumes, Docker sets up this folder (layers) in the level immediately below the container, allowing fast access of all stored data in this layer (folder), solving the performance problem. 
The volume also solves matters of data endurance, for the information stored in the container layers (folder) are lost while removing the container, that is, by using volumes we have a bigger guarantee in storing these data.

### Using volumes

#### Mapping of a specific host folder


In this model the user choses a specific folder of the host (ex.: /var/lib/container1) and maps it into a container’s inside folder (ex.: /var). What is written in the folder /var of the container is also written in the folder /var/lib/container1 of the host. 

Here’s the sample of the command used to this mapping model:

```
docker container run -v /var/lib/container1:/var ubuntu
```

This model is not portable. It needs the host to have a specific folder so the container works properly. 

#### Mapping via data container

In this model, a container is created and, inside it, a volume to be consumed by other containers is named. Therefore, it’s not necessary to create a specific folder on the host to persist data. This folder is automatically created inside the root folder of Docker daemon. However, you don’t need to worry with it, since all the reference is going to made for the container that holds the volume and not for the folder.
 Here’s an example of using the mapping model::

```
docker create -v /dbdata --name dbdata postgres /bin/true
```
In the command above, we created a data container, in which the folder /dbdata can be used by other containers, that is, the content of the folder /dbdata can be visualized and/or edited by other containers. 
To consume this container volume, just use the command:
```
docker container run -d --volumes-from dbdata --name db2 postgres
```
Now the container db2 has a folder /dbdata that is the same as the one from the container dbdata, making this model completely portable. 

A disadvantage is the need of keeping a container just to this, because in some environment containers are removed with some frequency, making it necessary to take special care with special containers. In a certain way, it’s an additional management problem.

#### Mapping volumes

In Docker’s 1.9 version, the possibility of creating containers’ isolated volumes was added. Now it’s possible to create a portable volume, with no need of associating it to a special container. 
Here’s an example of using the mapping model:

```
docker volume create --name dbdata
```
In the command above, Docker created a volume that can be consumed by any container.
 The association of the volume to the container happens in similar way to the one practiced on mapping the host folder, because in this case you need to associate the volume to a folder inside the container, as we can see below:

```
docker container run -d -v dbdata:/var/lib/data postgres
```
This model is the most indicated since the release, because it gives you portability. It is not removed easily when this container is deleted and, still, is very easy to manage.