# Config

Moving on on the list of [12factor](https://12factor.net/) model, **“Config”** is the third best practice.

When we are creating a software, we apply a given behavior inside the code and usually it’s not parameterizable. For the application behave differently, it will be necessary to change part of the code.

The need of modifying the code to change the application’s behavior makes unfeasable the execution in the machine (development) in the same way it is used to attend users (production). And, with that, we kill the possibility of portability. And with no portability, what is the advantage of using containers, right?

The goal of the best practice is to make feasible the application configuration without the need of changing the code. Since the application behavior varies according to the environment where is executed, the configurations must consider the environment.

Here are some examples:

 - Database configuration that usually are different between environments
 - Credentials for accessing remote services (ex.: Digital Ocean or Twitter)
 - Which DNS name will be used by the application

As we mentioned, when the configuration is statistically explicit in the code, it’s necessary to change manually and do a new binary build at each system reconfiguration.

As showed in the codebase best practice, we use a environments variable to modify the volume we are going to use in redis. In a way, we are already complying with the best practice, but we can go further and change not only the infrastructure behavior, but also something inherent in the code itself.

Here it is the modified application:

```
from flask import Flask
from redis import Redis
import os
host_run=os.environ.get('HOST_RUN', '0.0.0.0')
debug=os.environ.get('DEBUG', 'True')
app = Flask(__name__)
redis = Redis(host='redis', port=6379)
@app.route('/')
def hello():
   redis.incr('hits')
   return 'Hello World! %s times.' % redis.get('hits')
if __name__ == "__main__":
   app.run(host=host_run, debug=debug)
```

Remember! To access the practice code, just clone [this repository](https://github.com/gomex/exemplo-12factor-docker) and go to the **“factor3“** folder.

As we can see, we added some parameters to the configuration of the address used to start the web application that will be parameters based on the **”HOST_RUN”** environment variable value. And the possibility of performing or not the application debug with the **”DEBUG”** environment variable.

It’s worth to say: in this case the environment variable needs to go into the container, it’s not enough to hold the variable on Docker Host. It is necessary to send it into the container using the parameter “-e”, in case you use the command “docker container run” or the instruction “environment” on docker-compose.yml:


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

To execute Docker-Compose, we should do this way:

```
export HOST_RUN="0.0.0.0"; export DEBUG=True ; docker-compose up -d
```

In the command above, we use the environment variables **“HOST_RUN”** and **“DEBUG”** from Docker Host to send the environment variables with the same names into the container that, in turn, is consumed by the Python code. In case there aren’t any parameters, the container assumes the standard values set in the code.

This best practice is followed with the help of Docker, for the code is the same and the configuration is an attachment of the solution that can be parameterized in different ways based on what is configured in the environment variables.

If the application grows, the variables can be carried out in files and paremeterized on docker-compose-yml with the option “env_file”.
