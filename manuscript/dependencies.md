# Dependencies

Moving in the list of the [12factor](https://12factor.net/) model, right after we approached the code base in this [article](https://imasters.com.br/desenvolvimento/dockerizando-aplicacoes-base-de-codigo), we have **”Dependency”** as the second best practice.

![](images/dependencia.png)

This best practice suggests the declaration of all required dependencies to execute the code. You must not assume that some component is previously installed in the active responsible for hosting the application.

To make feasible the portability “dream”, we need to manage correctly the given application dependencies; this indicates that we should also avoid the need of manual work while preparing the infrastructure that supports the application.

Automating the dependency installation process is the big secret of success to attend this best practice. In case the infrastructure is not automated enough to provide initialization without errors, the attendance to this best practice is compromised.

These automated procedures help maintaining the integrity of the process, for the name of dependency packages and their respective versions are specified in the file located in the same repository of the code that, in turn, is traced in a control version system. Thus, we can conclude that nothing is modified without the due record.

Docker fits perfectly in the best practice. It’s possible to deliver a minimum infrastructure profile for the application. In turn, it’s necessary the explicit declaration of dependencies, so the application runs in the environment.

The example application, written in Python, as we saw a little in the code below, needs two libraries in order to work correctly:

```
from flask import Flask
from redis import Redis
```

These two dependencies are specified in the file requirements.txt and this file is used a PIP parameter.

“PIP is a package management system used to install and manage software packages written in Python”. (Wikipedia)

The PIP command is used with the file requirements.txt in the creation of image, as shown at the Dockerfile of previous best practice (codebase):

```
FROM python:2.7
ADD requirements.txt requirements.txt
RUN pip install -r requirements.txt
ADD . /code
WORKDIR /code
CMD python app.py
```

Notice that one of the steps of Dockerfile is to install the dependencies written in the file requirements.txt with the Python’s PIP package manager. Check out the content of requirements.txt file:

```
flask==0.11.1
redis==2.10.5
```

It’s important to emphasize the need of specifying the versions of each dependency, because, as in the container model, the images can be built at any time. It’s important to know which specific version the application requires. Otherwise, we can find some compatibility problems if one of the dependencies updates and doesn’t stay compatible with the complete set of other dependencies and the application that uses it.

To access the code written here, download the [repository](https://github.com/gomex/exemplo-12factor-docker) and go to the  **“factor2“** folder.

Another positive outcome of using the best practices is the simplification if another developer uses the code. A new developer can verify in the dependencies file which are prerequisites for the application to run, as well as executing the environment without the need of following the extensive documentation that is rarely updated.

By using Docker it is possible to configure automatically the necessary to run the application code, following perfectly the best practice.
