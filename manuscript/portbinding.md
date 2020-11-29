# Port binding

According to the list of the [12factor](https://12factor.net) model, the seventh best practice is **port binding**. 

It’s usual to find applications executed inside containers of web servers, such as Tomcat or Jboss, for instance. Usually, these applications are deployed into the services so they can be access by user externally.

![](images/vinculos1.png)

The best practice suggests that the given application would be self-contained and depend on a application server, such as Jboss, Tomcat and similar. The software must export a HTTP services and deal with the requirements that come through it. This means that any additional application is unnecessary for the code to be available to the external communication.

Traditionally, the artifact deployments in an application server, such as Tomcat and Jboss, requires the generation of an artifact, that is sent to the given web service. But in the Docker’s container model the ideia is that the artifact of the deployment process would be the container itself.

The old artifact deployment process in an application server usually didn’t have a fast return, overly increasing the process of deploying a service, because each alteration required to send the artifact to the web application service; the later was responsible for importing, reading and executing the new artifact.

By using Docker, the application become self-contained easily. We built a Dockerfile that describes what the application needs:

```
FROM python:2.7
ADD requirements.txt requirements.txt
RUN pip install -r requirements.txt
ADD . /code
WORKDIR /code
CMD python app.py
EXPOSE 5000
```

The dependencies are described in the file requirements.txt and the data that must be persisted are managed by a service (support services) external to the application.

Another point of the best practice: the application must export the service by binding to a single port. As we seed in the example code, the standard Python port (5000) is initiated, but you can choose another one if you think it’s necessary. Here’s the part of the code that approaches the subject:

```
if __name__ == "__main__":
  app.run(host="0.0.0.0", debug=True)
```

The port 5000 can be used to serve data locally in a development environment or through a reverse proxy, when migrating to production with the proper domain name to the given application.

Using the binding port model makes the application update process more fluid, once using an intelligent reverse proxy makes possible to add new knots gradually, with the new version, and remove the old ones as the updated versions are executed in parallel.

Important: even that Docker allows using more than one port per container, the best practice emphasizes that you should only use one binding port per application.
