# Backing services

Moving on on the list of [12factor](http://12factor.net/pt_br) model, we find **”Backing services”** as the fourth best practice.

To bring some context, “support service” is any application your code consumes in order to function correctly (ex.: database, message service etc.).

![](images/servicoapoio.png)

Aiming to prevent that the code is overly dependent of a given infrastructure, the best practice says that you, while writing the software, don’t differentiate the internal and external service. That is, the application must be ready to get parameters that configure the service correctly, thus making possible the consume of applications necessary to the solution.

The application in the example was modified to support the best practice:

```
from flask import Flask
from redis import Redis
import os
host_run=os.environ.get('HOST_RUN', '0.0.0.0')
debug=os.environ.get('DEBUG', 'True')
host_redis=os.environ.get('HOST_REDIS', 'redis')
port_redis=os.environ.get('PORT_REDIS', '6379')
app = Flask(__name__)
redis = Redis(host=host_redis, port=port_redis)
@app.route('/')
def hello():
   redis.incr('hits')
   return 'Hello World! %s times.' % redis.get('hits')
if __name__ == "__main__":
   app.run(host=host_run, debug=True)
```

As you can see in the code above, the application now gets environment variables to configure the host name and Redis service port. In this case, it’s possible to configure the host and Redis port you wish to connect. And this can and must be specified in the docker-compose.yml that has also being through a change to suit this new best practice:

```
version: "2"
services:
  web:
    build: .
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

As we could see in the mentioned codes, the advantage of the best practice goes through the possibility of changing behavior without changing the code. Once more it’s possible to enable that the same code build in a certain moment can be reused in a similar way, both in the developer’s notebook and in the production server.  

Pay attention to storing secrets inside the docker-compose.yml, because this file is sent to the version control repository and it’s important to think about another strategy for keeping secrets.

A possible strategy is to maintain environment variables in Docker Host. Therefore, you need to use variables like **${variable}** inside docker-compose.yml to repass the configuration or use another more advanced secret management resource.
