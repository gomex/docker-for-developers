# Build, release, run

The next item of the list of [12factor](https://12factor.net) model, “Build, launch, run”, is the fifth best practice.

In the process of automating the software deployment infrastructure, we need to be careful so the process behavior is within the expectations and so human errors have low impact in the whole development process, from release to production.

![](images/release.png)

Aiming to organize, divide duties and make the process clearer, 12factor indicates that the base code, to be put into production, needs to go through three phases:

- **Build** - convert the repository code into executable package. In this process we obtain the dependencies, compile the code’s binaries and actives.
- **Release** - the package produced in the **build** phase is combined with the configuration. The result is the whole environment, configured and ready to **run**.
- **Run** (also known as “runtime”) - begins **running** the **release** (application + configuration of that environment), based on the specific configurations of the required environment.

The best practice points out that the application has explicit separations at the  
**Build**, **Release** and **Run** stages. Thus, every change in the application code is build only once in the **Build** stage. Changes in configuration don’t need a new **build**, so it’s only necessary the **release** and **run** stages.

In such a way, it’s possible to create clear controls and processes in each stage. In case something happens in the code **build**, a measure can be taken or even the release can be canceled, so the code in production wouldn’t be compromised due to a possible error.

The separation of duties makes possible to know in which stage the problem happened, and fix it manually, if needed.

The artefacts produced must have a single **release** ID. It can be the timestamp (like 2011-04-06-20:32:17) or an incremental number (like v100). With the single artefact, it’s possible to guarantee the use of the old version, whether for a rollback or even to compare behaviors after changing the code.

In order to follow the best practice, we need to build the Docker image with the application inside of it. It will be our artifact.

We will have a new script, here called build.sh, with the following content:

```
#!/bin/bash

USER="gomex"
TIMESTAMP=$(date "+%Y.%m.%d-%H.%M")

echo "Construindo a imagem ${USER}/app:${TIMESTAMP}"
docker build -t ${USER}/app:${TIMESTAMP} .

echo "Marcando a tag latest também"
docker tag ${USER}/app:${TIMESTAMP} ${USER}/app:latest

echo "Enviando a imagem para nuvem docker"
docker push ${USER}/app:${TIMESTAMP}
docker push ${USER}/app:latest
```

Aside of building the image, it sends it to the Docker’s image [repository](https://hub.docker.com/).

Remember: this code and others from the best practice are in the [repository](https://github.com/gomex/exemplo-12factor-docker), in the folder **“factor5“**.

Sending the image to the repository is an important part of this best practice, for it isolates the process. In case the image is not sent to the repository, it remains only in the server that executed the **build** process; therefore, the next stage needs to be executed in the same server, because such stage needs the image to be available.

In the proposed model, the image in the central repository is available to be downloaded in the server. In case you are using a pipeline tool, it’s important to use the product variables (instead of using the date) to uniquely identify the artifact, and guarantee that the image that is going to be used in the Run stage is the same built in the Release stage. Exemple on GoCD: the variables **GO_PIPELINE_NAME** and **GO_PIPELINE_COUNTER** can be used together to identify the artifact.

With the image creation, we can guarantee that the **Build** stage was fulfilled perfectly, because now we have an artifact built and ready to be put together with the configuration.

Com a geração da imagem podemos garantir que a etapa **Construir** foi atendida perfeitamente, pois, agora temos um artefato construído e pronto para ser reunido à configuração.

The **Release** stage is the file docker-compose.yml itself, because it gets the due configurations for the environments in which you wish to put the application. Therefore, the file docker-compose.yml changes a little and stops making the image **build**, since now it will be used only for **Release** and **Run** (later):

```
version: "2"
services:
  web:
    image: gomex/app:latest
    ports:
     - "5000:5000"
    volumes:
     - .:/code
    labels:
     - 'app.environment=${ENV_APP}'
    environment:
     - HOST_RUN=${HOST_RUN}
     - DEBUG=${DEBUG}
     - PORT_REDIS=6379
     - HOST_REDIS=redis
  redis:
    image: redis:3.2.1
    volumes:
     - dados:/data
    labels:
     - 'app.environment=${ENV_APP}'
volumes:
  dados:
    external: false
```

In the **docker-compose.yml** example above, we used the tag latest to guarantee it always searches for the last **built** image in the process. But as we have already mentioned, in case you are using some continuous delivery tool (such as GoCD, for instance), use the variables to guarantee the image created in the specific pipeline execution.

Therefore, **release** and **run** will use the same artifact: the Docker image, built in the build stage.

The **run** stage, basically, executes the Docker-Compose with the command below:

```
docker-compose up -d
```
