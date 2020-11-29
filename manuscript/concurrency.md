# Concurrency

The eighth best practice from the list of [12factor](https://12factor.net) model is **“Concurrency”**.

During the process of developing an application it’s hard to imagine the amount of requirements it will have when in production. On the other hand, a service that bears great usage volumes is expected in modern solutions. Nothing is more frustrating than requiring access to an application and it is not available. It suggests lack of care and professionalism in most cases.

When the application is put into production, it’s usually dimensioned to a given load; however, it’s important that the service is ready to escalate. The solution must be able to initiate new processes of the same application if necessary, without affecting the product. The picture below shows the service scalability graphic.

![](images/concorrencia1.png)

Aiming to avoid any kind of problem in service scalability, the best practice says that the applications must support concurrent executions, and when a process is in execution, instantiating another one in parallel and attend the service, without any loss.

For such, it’s important to distribute tasks correctly. It’s interesting the process of attaining to the goals, in case it’s necessary to execute some activity in backend and later return a page for the browser, it’s salutary that two services respond to two activities, separately. Docker makes this task simpler, because in this model it’s only necessary to specify a container for each function and properly configure the network between them.  

To illustrate this best practice, we’ll use the architecture shown in the picture below:

![](images/concorrencia2.png)

The web service is responsible for receiving the requirements anda balance the workers, which are responsible for processing the requirements, connecting to Redis and return the “Hello World” screen, informing how many times it appeared and which worker name is responding to the requirement (to be sure it’s balancing the load), as we can see in the picture below:

![](images/concorrencia3.png)

The file **docker-compose.yml** exemplifies the best practice:

```
version: "2"
services:
  web:
    container_name: web
    build: web
    networks:
      - backend
    ports:
      - "80:80"

  worker:
    build: worker
    networks:
      backend:
        aliases:
          - apps
    expose:
      - 80
    depends_on:
      - web

  redis:
    image: redis
    networks:
      - backend

networks:
  backend:
      driver: bridge
```

To do the build of the load balancer, we have the web directory containing Dockerfile files (responsible for creating the image used) and nginx.conf files (configuration file from the load balancer used).

Here’s the web Dockerfile content:

```
FROM nginx:1.9

COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

And the content of nginx.conf file:

```
user nginx;
worker_processes 2;

events {
  worker_connections 1024;
}

http {
  access_log /var/log/nginx/access.log;
  error_log /var/log/nginx/error.log;

  resolver 127.0.0.11 valid=1s;

  server {
    listen 80;
    set $alias "apps";

    location / {
      proxy_pass http://$alias;
    }
  }
}
```

In the configuration file above, some innovations were introduced. The first one, **“resolv 127.0.0.11“**, is the Docker’s internal DNS service. By using this approach, it’s possible to balance the load via name, using Docker’s internal resource. For more details on Docker’s internal DNS, check out this documentation ([https://docs.docker.com/config/containers/container-networking/](https://docs.docker.com/config/containers/container-networking/) (only in English).

The second innovation, the set **$alias “apps”** function, is responsible for specifying the name **“apps”** used to configure the reverse proxy, then **“proxy_pass http://$alias;”**. It’s important to emphasize that **“apps”** is the name of the network specified inside the file **docker-compose.yml**. In this case, the balancing is made for the network, and every new container that enters this network is automatically added to the load balancing.

To build the **worker** we have the directory **worker** containing the **Dockerfile** files (responsible for creating the image used), **app.py** (application used in all chapters) and **requirements.txt** (describes the dependencies of app.py).

Below is the content of the file **app.py** that was modified for the practice:

```
from flask import Flask
from redis import Redis
import os
import socket
print(socket.gethostname())
host_redis=os.environ.get('HOST_REDIS', 'redis')
port_redis=os.environ.get('PORT_REDIS', '6379')

app = Flask(__name__)
redis = Redis(host=host_redis, port=port_redis)

@app.route('/')
def hello():
   redis.incr('hits')
   return 'Hello World I am %s! %s times.' % (socket.gethostname(), redis.get('hits'))
if __name__ == "__main__":
   app.run(host="0.0.0.0", debug=True)
```

The content of **requirements.txt**:

```
flask==0.11.1
redis==2.10.5
```

And lastly the **worker** **Dockerfile** holds the following content:

```
FROM python:2.7
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY . /code
WORKDIR /code
CMD python app.py
```

In the **redis** services there’s not building the image, we’ll use the official image to exemplify.

To test what was presented so far, clone the repository ([https://github.com/gomex/exemplo-12factor-docker](https://github.com/gomex/exemplo-12factor-docker)) and access the folder **factor8**, executing the command below in order to initiate the containers:

```
docker-compose up -d
```

Access the containers through the browser at the port 80 from the localhost address. Refresh the page and see that only one name appears.

As a standard, Docker-Compose executes only one instance of each service explicit on **docker-compose.yml**. To increase the amount of **worker** containers from one to two, execute the command below:

```
docker-compose scale worker=2
```

Refresh the page and see that the name of the host alternates between two possibilities, that is, the requirements are being balanced to both containers.

In this new environment proposal, the **web** service is in charge of receiving the HTTP requirements and balance the load. Then, the **worker** is responsible for processing the requirements - basically getting the host name, access **redis** and count how many time the service was required - and then rollback to return it to the **web** service that, in turn, responds to the user. As we can notice, each environment instance has a defined function, thus making easier to escalate it.

We take the opportunity to give the credits to captain [Marcosnils](https://twitter.com/marcosnils), who showed us that is possible to balance the load by the Docker network name.
