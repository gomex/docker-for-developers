# Tips for using Docker

If you read the first part of the book, you already know the basics on Docker; but now that you intend to start using it more frequently some issues may rise, because as in any tool, Docker has its own set of best practices and tips.

The goal of this article is to present some tips for better use Docker. That doesn’t mean that your way of using is necessarily wrong.

Every tool requires some best practices to make its use more effective and less likely to show future problems.

This chapter is divided in two sections: tips for running (‘docker container run’) e best practices in image building (‘docker build’/‘Dockerfile’).


## Tips for running

Remember: each ‘docker container run’ command creates a new container based on a specific image and starts a process inside it, after a command (‘CMD’ specified on Dockerfile).


### Disposable containers

It’s expected that executed containers could be disposed without any problems. Therefore, it’s important to use truly ephemera containers.

For such, use the arguments '--rm’, that makes all containers, and their data, to bem removed after finishing the execution, preventing from taking unnecessary space in disk.

In general, the command ‘run’ can be used as in the example:

```sh
docker container run --rm -it debian /bin/bash
```

Notice that ‘-it’ means ‘--interactive —tty’. It’s used to fix the command line to the container, thus, after this ‘docker container run’, every command are executed by the ‘bash’ inside the container. To exit, use ‘exit’ or press ‘Control-d’. These parameters are very useful to execute a container in the foreground.

### Check environment variables

Some times, it’s necessary to check which meta-data are defined as environment variables in an image. Use the command ‘env’ to get this information:

```sh
docker container run --rm -it debian env
```

To check the old environment variables of a container:

```sh
docker inspect --format '{{.Config.Env}}' <container>
```

For other meta-data, use variations of command ‘docker inspect’.

### Logs

Docker captures standard output logs (’STDOUT’) and errors output (’STDERR’).

These records can be routed to different systems (‘syslog’, ‘fluentd’, …) that can be specified in the [driver configuration](https://docs.docker.com/engine/admin/logging/overview/) '--log-driver=VALUE’ in the command ‘docker container run’.

When using the standard driver ‘json-file’ (also, ‘journald’), you can use the following command to recover the logs:

```sh
docker logs -f <container_name>
```

Notice yet the argument ‘-f’ to follow up the next log messages interactively. If you want to stop, press ‘Ctrl-c’.

### Backup

Docker container data are exposed and shared via volume arguments used while creating and starting the container. These volumes don’t follow the rules from [Union File System](https://docs.docker.com/engine/reference/glossary/#union-file-system), because the data persist even when the container is removed.

To create a volume in a given container, execute as it follows:

```sh
docker container run --rm -v /usr/share/nginx/html --name nginx_teste nginx
```

By executing this container, we’ll have the Nginx service that uses the volume created to persist its data; the data will persist even after the container is removed.

It a system admin best practice to do periodic backups; to execute this activity (extract data), use the command:

```sh
docker container run --rm -v /tmp:/backup --volumes-from nginx-teste busybox tar -cvf /backup/backup_nginx.tar /usr/share/nginx/html
```

After executing the command, we have a ’backup_nginx.tar’ file inside the folder /tmp of **Docker host**.

In order to restores this backup, use:

```sh
docker container run --rm -v /tmp:/backup --volumes-from nginx-teste busybox tar -xvf /backup/backup.tar /usr/share/nginx/html
```

More information can be found in the [answer](http://stackoverflow.com/a/34776997/1046584), where is possible to find some  *aliases* for these two commands. These  *aliases* are also available below, in the *Aliases* section.

Some other sources:

 * Docker official documentation about [Data backup, restoration or migration](https://docs.docker.com/engine/userguide/containers/dockervolumes/#backup-restore-or-migrate-data-volumes)

 * A backup tool (currently deprecated): [docker-infra/docker-backup](https://github.com/docker-infra/docker-backup)


### Use docker container exec to “enter a container”

Eventually, it’s necessary to enter a running container so you can check any problem, running tests or simply debug.

Never install daemon SSH in a Docker container. Use ‘docker container exec’ to enter a container and run commands:

```sh
docker container exec -it <nome do container em execução> bash
```
The feature is useful in local development and experiments. But avoid using container in production or automating tools around it.

Check the [documentation](https://docs.docker.com/engine/reference/commandline/exec/).

### No space in Docker Host disk

By executing containers and building images several times, the space in disk can become scarce. When it happens, it’s necessary to clean some containers, images and logs.

A fast way of cleaning containers and images is by using the following command:

```sh
docker system prune
```

With this command you will remove:

* All the containers not in use at the moment
* All the volumes not in use by at least one container
* All the networks not in use by at least one container
* Every *dangling* images

Obs.: Let’s not get too deep in Docker’s low level concept and say that *dangling* images are simply images with no tags, therefore unnecessary for conventional use.

Depending on the type of application, logs can occupy some volume too. The management depends a lot on which [driver](https://docs.docker.com/engine/admin/logging/overview/) is used. In the standard driver (‘json-file’), the cleaning can be done by executing the following command inside **Docker Host**:

```sh
echo "" > $(docker inspect --format='{{.LogPath}}' <container_name_or_id>)
```

- The functionality proposal of cleaning the logs history was, actually, rejected. More information on: [https://github.com/docker/compose/issues/1083](https://github.com/docker/compose/issues/1083)

- Consider specifying the ‘max-size’ option to the log _driver_ while executing ‘docker container run’: [https://docs.docker.com/engine/reference/logging/overview/#json-file-options](https://docs.docker.com/engine/reference/logging/overview/#json-file-options)

### Aliases

Alias makes possible to transform big commands into smaller ones. We have some new options to execute more complex tasks.

Use these *aliases* in your ‘.zshrc’ or ‘.bashrc’ to clean imagens and containers, to backup and restore etc.

```sh
# runs docker container exec in the latest container
function docker-exec-last {
  docker container exec -ti $( docker ps -a -q -l) /bin/bash
}

function docker-get-ip {
  # Usage: docker-get-ip (name or sha)
  [ -n "$1" ] && docker inspect --format "{{ .NetworkSettings.IPAddress }}" $1
}

function docker-get-id {
  # Usage: docker-get-id (friendly-name)
  [ -n "$1" ] && docker inspect --format "{{ .ID }}" "$1"
}

function docker-get-image {
  # Usage: docker-get-image (friendly-name)
  [ -n "$1" ] && docker inspect --format "{{ .Image }}" "$1"
}

function docker-get-state {
  # Usage: docker-get-state (friendly-name)
  [ -n "$1" ] && docker inspect --format "{{ .State.Running }}" "$1"
}

function docker-memory {
  for line in `docker ps | awk '{print $1}' | grep -v CONTAINER`; do docker ps | grep $line | awk '{printf $NF" "}' && echo $(( `cat /sys/fs/cgroup/memory/docker/$line*/memory.usage_in_bytes` / 1024 / 1024 ))MB ; done
}
# keeps the commmand history when running a container
function basher() {
    if [[ $1 = 'run' ]]
    then
        shift
        docker container run -e HIST_FILE=/root/.bash_history -v $HOME/.bash_history:/root/.bash_history "$@"
    else
        docker "$@"
    fi
}
# backup files from a docker volume into /tmp/backup.tar.gz
function docker-volume-backup-compressed() {
  docker container run --rm -v /tmp:/backup --volumes-from "$1" debian:jessie tar -czvf /backup/backup.tar.gz "${@:2}"
}
# restore files from /tmp/backup.tar.gz into a docker volume
function docker-volume-restore-compressed() {
  docker container run --rm -v /tmp:/backup --volumes-from "$1" debian:jessie tar -xzvf /backup/backup.tar.gz "${@:2}"
  echo "Double checking files..."
  docker container run --rm -v /tmp:/backup --volumes-from "$1" debian:jessie ls -lh "${@:2}"
}
# backup files from a docker volume into /tmp/backup.tar
function docker-volume-backup() {
  docker container run --rm -v /tmp:/backup --volumes-from "$1" busybox tar -cvf /backup/backup.tar "${@:2}"
}
# restore files from /tmp/backup.tar into a docker volume
function docker-volume-restore() {
  docker container run --rm -v /tmp:/backup --volumes-from "$1" busybox tar -xvf /backup/backup.tar "${@:2}"
  echo "Double checking files..."
  docker container run --rm -v /tmp:/backup --volumes-from "$1" busybox ls -lh "${@:2}"
}
```

Sources:

- [https://zwischenzugs.wordpress.com/2015/06/14/my-favourite-docker-tip/](https://zwischenzugs.wordpress.com/2015/06/14/my-favourite-docker-tip/)
- [https://website-humblec.rhcloud.com/docker-tips-and-tricks/](https://website-humblec.rhcloud.com/docker-tips-and-tricks/)

## Best practices to build images

On Docker, images are traditionally built using a ‘Dockerfile’. There are some good guides on the best practices to build Docker images. Take a look at our recommendations:

- [Documentação oficial](https://docs.docker.com/engine/articles/dockerfile_best-practices/)
- [Guia do projeto Atomic](http://www.projectatomic.io/docs/docker-image-author-guidance/)
- [Melhores práticas do Michael Crosby Parte 1](http://crosbymichael.com/dockerfile-best-practices.html)
- [Melhores práticas do Michael Crosby Parte 2](http://crosbymichael.com/dockerfile-best-practices.html)

### Use a "linter"

*"Linter"* is a tool that provides tips and warning on some source code. There are some simple options for ‘Dockerfile’, but it is, still, a new evolving space.

Many options were discussed [here](https://stackoverflow.com/questions/28182047/is-there-a-way-to-lint-the-dockerfile).

Since January 2016, the most complete “linter” seems to be [hadolint](http://hadolint.lukasmartinelli.ch/), available in two versions: on-line and terminal. The interest thing abou this tool is that it uses the mature [Shell Check](http://www.shellcheck.net/about.html) to validate the shell commands.

### The basics

The container produced by the ‘Dockerfile’ image must be as ephemeral as possible. This means that it should be possible to stop it, destroy it and replace it for a new container built with the minimum effort.

**It’s usual to put other files, such as documentation, in the same directory of ‘Dockerfile’; to improve the building performance, delete files and directories creating a [dockerignore](https://docs.docker.com/engine/reference/builder/) file in the same directory. This file works similarly to ‘.gitignore’. Using it helps to minimize the building context `docker build`.**

Avoid adding packages and unnecessary extra dependencies to the application and minimize complexity, image size, building time and attack surface.

Also minimize the layer amount: whenever possible, group up various commands. However, take in consideration the volatility and maintenance of these layers.

In most cases, run only one process per container. Decoupling applications in several container eases up horizontal scalability, reuse and monitoring of containers.


### Choose COPY over ADD

The ‘ADD’ command exists since the beginning of Docker. It’s versatile, and provides some tricks aside of simply copying files from the building context, and that’s what makes it magical and hard to understand. It allows to download url files and automatically extract files of known formats (tar, gzip, bzip2, etc.).

On the other hand ‘COPY’ is a simpler command to put files and folders of the building path inside the Docker image. Thus, choose ‘COPY’ unless you are absolutely sure that ‘ADD’ is necessary. For more details, check [here](https://labs.ctl.io/dockerfile-add-vs-copy/).

### Run a “checksum” after downloading and before using the file

Instead of using ‘ADD’ to download and add files to the image, prefer using [curl](https://curl.haxx.se/) and verify through a ‘checksum’ after the download. This guarantees that the file is the expected one and will not vary over time. If the file that the URL indicates changes, the ‘checksum’ will change and the image building will fail. This is important, because it favor reproducibility and safety in image building.

A good inspiration is [Jenkins’ official Dockerfile](https://github.com/jenkinsci/docker/blob/83ce6f6070f1670563a00d0f61d04edd62b78f4f/Dockerfile#L36):

```
ENV JENKINS_VERSION 1.625.3
ENV JENKINS_SHA 537d910f541c25a23499b222ccd37ca25e074a0c

RUN curl -fL http://mirrors.jenkins-ci.org/war-stable/$JENKINS_VERSION/jenkins.war -o /usr/share/jenkins/jenkins.war \
  && echo "$JENKINS_SHA /usr/share/jenkins/jenkins.war" | sha1sum -c -
```

### Use a image with smallest base

Whenever possible, use official images as a base for your image. You can use the [‘debian’](https://hub.docker.com/_/debian/) image, for instance, that is very well controlled and kept small (around 150mb). Remember of also using specific *tags*, such as ‘debian:jessie’.

If more tools and dependencies are required, look for images like [‘buildpack-deps’](https://hub.docker.com/_/buildpack-deps/).

However, in case ‘debian’ is still too big, there are minimalist images such as [‘alpine’](https://hub.docker.com/r/gliderlabs/alpine/) or even [‘busybox’](https://hub.docker.com/r/gliderlabs/alpine/). Avoid ‘alpine’ if DNS is required, for there are a [few issues to be solved](https://github.com/gliderlabs/docker-alpine/blob/master/docs/caveats.md). In addition, avoid it for languages that use GCC, such as Ruby, Node, Python, etc.; because ‘alpine’ uses libc MUSL that can produce different binaries.

Avoid gigantic images such as [‘phusion/baseimage’](https://hub.docker.com/r/phusion/baseimage/). This image is too big, it defeats the philosophy of process per container  and much of what makes it up is not essential for Docker containers; [read more here](https://blog.docker.com/2014/06/why-you-dont-need-to-run-sshd-in-docker/) .

[Other sources](http://www.iron.io/microcontainers-tiny-portable-containers/)

### Use the layer building cache

Another useful feature provided by ‘Dockerfile’ is the fast rebuild using the layer cache. In order to take advantage from this resource, add tools and dependencies that change less frequently in the top of ‘Dockerfile’.

For instance, consider to install the code dependencies before adding the code. In the case of NodeJS:

```
COPY package.json /app/
RUN npm install
COPY . /app
```

To read more on this subject, check out this [link](http://bitjudo.com/blog/2014/03/13/building-efficient-dockerfiles-node-dot-js/).

### Clean on the same layer

While using a package manager to install any software, the best practice is to clean the cache generated by the package manager soon after installing the dependencies. For instance, using ‘apt-get’:

```
RUN apt-get update && \
    apt-get install -y curl python-pip && \
    pip install requests && \
    apt-get remove -y python-pip curl && \
    rm -rf /var/lib/apt/lists/*
```

In general, the apt cache (generated by ‘apt-get update’) must be cleaned up by removing ‘/var/lib/apt/lists’. This helps to keep the image size small. In addition, note that ‘pip’ and ‘curl’ also are removed once they’re unnecessary to the production application. Remember that the cleaning must be made in the same layer (command ‘RUN’). Otherwise, data will be persisted on this layer and removing them later will not have the same effect in the final image size.

Note that, according to the [documentation](https://github.com/docker/docker/blob/03e2923e42446dbb830c654d0eec323a0b4ef02a/contrib/mkimage/debootstrap#L82-L105), the official Debian and Ubuntu images run ‘apt-get clean’ automatically. Ergo, the explicit invocation is not necessary.

Avoid to run ‘apt-get upgrade’ or ‘dist-upgrade’, badales several packages of the base image are not going to update inside a container with no privileges. If there’s a specific package to update, just use ‘apt-get install -y foo’ to automatically update it.

To read more on this subject, check out the [link](http://blog.replicated.com/2016/02/05/refactoring-a-dockerfile-for-image-size/) and [this one](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#apt-get).

### Use a “wrapper” script as ENTRYPOINT, sometimes


A *wrapper script* can help to configure the environment and define the application configuration. It can event define the standard configurations when they’re not available.

A great example is provided in [Kelsey Hightower: 12 Fracturated Apps’ article](https://medium.com/@kelseyhightower/12-fractured-apps-1080c73d481c#.xn2cylwnk):

```sh
#!/bin/sh
set -e
datadir=${APP_DATADIR:="/var/lib/data"}
host=${APP_HOST:="127.0.0.1"}
port=${APP_PORT:="3306"}
username=${APP_USERNAME:=""}
password=${APP_PASSWORD:=""}
database=${APP_DATABASE:=""}
cat <<EOF > /etc/config.json
{
  "datadir": "${datadir}",
  "host": "${host}",
  "port": "${port}",
  "username": "${username}",
  "password": "${password}",
  "database": "${database}"
}
EOF
mkdir -p ${APP_DATADIR}
exec "/app"
```

Note: **always** use ‘exec’ in shell scripts regarding the application. Therefore, the application can get Unix signals.

Also, consider using a simple initialization system (e.g. [dumb init](https://github.com/Yelp/dumb-init)), such as ‘CMD’ base, so the Unix signal can be duly treated. Read more [here](http://engineeringblog.yelp.com/2016/01/dumb-init-an-init-for-docker.html).

#### Log for stdout

Applications inside Docker should emit logs for ‘stdout’. However, some applications write logs in files. In these cases, the solution is to create a file *symlink* for ‘stdout’.

Example: Dockerfile of [nginx](https://github.com/nginxinc/docker-nginx/blob/master/Dockerfile):

~~~
# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout /var/log/nginx/access.log
RUN ln -sf /dev/stderr /var/log/nginx/error.log
~~~

To read more, check out this [link](https://serverfault.com/questions/599103/make-a-docker-application-write-to-stdout).

### Be careful while adding data to a volume on Dockerfile

Remember of using the instruction ‘VOLUME’ to expose data from database, configuration or files and folders created by the container. Use for any mutable data and parts served to the user of the service to which the image was created.

Avoid adding a lot of data in a folder and then turn it into a ‘VOLUME’ only when starting the container, because you can slow down the loading. By creating the container, the data will be copied from the image to the set volume. As said before, use ‘VOLUME’ when creating the imagem.

Besides, while still creating the image (‘build’), don’t add data to paths previously declared as ‘VOLUME’. This doesn’t work, the data won’t be persisted, for datas in volumes are not *committed* into images.

Read more at [Jérôme Petazzoni’s explanation](https://jpetazzo.github.io/2015/01/19/dockerfile-and-data-in-volumes/).

### Ports EXPOSE

Docker favors reproducibility and portability. Images should be capable of run in any server, how many times necessary. Therefore, never expose public ports. However, expose the application’ standard ports privately.

```
# public and private mapping, avoid
EXPOSE 80:8080

# only private
EXPOSE 80
```
