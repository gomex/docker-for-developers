#  Codebase

Aiming to facilitate controlling the code changes, by enabling the traceability of alterations, this best practice indicates that each application must have only one code base and, from it, must be deployed in different environments. It’s important to emphasize that this practice is also part of the Continuous Integration ([CI](https://www.thoughtworks.com/continuous-integration)) practices. Traditionally, most part of continuous integration systems have, as a starting point, a code base that is built and, later, deployed to development, test and production.

For this explanation, we use the version control system Git and the hosting service Github. We create and provide an example [repository](https://github.com/gomex/exemplo-12factor-docker.git).

See, every code is inside the repository, arranged by practice in each folder, to facilitate the reproduction. Remember of entering the corresponding folder at each best practice presented.

Docker holds the possibility of using the environment variable to parameterize the infrastructure. Therefore, the same application will behave differently based on the value of environment variables.

Here we use Docker Compose to compose different relevant services for the application in time to execute. Thus, we must define the configuration of these distinct services and the way they communicate.

![](images/basecode.png)

Later, more precisely in the third best practice (called **Config**) of this suggestion compendium we will approach application parameterization in details. For now, we just use options via environment variable in architecture, instead of using it internally in the application code.

To configure the development environment for the example presented, we created the file docker-compose.yml:

```
version: '2'
services:
  web:
    build: .
    ports:
     - "5000:5000"
    volumes:
     - .:/code
    labels:
     - 'app.environment=${ENV_APP}'
  redis:
    image: redis
    volumes:
     - dados_${ENV_APP}:/data
    labels:
     - 'app.environment=${ENV_APP}'
```

We can notice that the “redis” service is used from the official “redis” image, with no modification. And the web service is generated from the building of a Docker image.

In order to build the Docker image of the web service, we create the following Dockerfile, using the official Python 2.7 image as a base:

```
FROM python:2.7
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
ADD . /code
WORKDIR /code
CMD python app.py
```

After putting all files in the same folder, we start the environment with the following command:

```
export ENV_APP=devel ; docker-compose -p $ENV_APP up -d
```

As we can notice in the example of this chapter, the environment variable ‘ENV_APP’ defines which volume is use for persisting the data that is going to be send by the web application. In other words, based on the change of this option, we have the service running with a different behavior, but always from the same code. There you have the concept of the first best practice.
