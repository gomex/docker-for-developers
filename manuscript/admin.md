# Admin processes

The twelfth and last best practice from the [12factor](https://12factor.net) model list: **“Admin processes”**.

![](images/admin1.png)

Every application requires administration. That means that, once deployed, it’s possible that the application need to receive some commands to correct possible issues or simply change behavior. As examples we have database migrations, running several scripts as backup and also running a console to inspect the service.

The best practice recommends admin processes executed in environments similar to the ones used in the running code, following all of the practices presented so far.

Using Docker makes possible to run the processes using the same base image in the running environment you wish. Thereby, with can benefit from the communication between containers and the use of volumes required and similar.

To exemplify the best practice we create the file **reset.py**:

```
from redis import Redis
import signal, os

host_redis=os.environ.get('HOST_REDIS', 'redis')
port_redis=os.environ.get('PORT_REDIS', '6379')

redis = Redis(host=host_redis, port=port_redis)

redis.set('hits', 0)
```

The command is given by using a different container from the same Docker image, and it’s responsible for reinitiating the Redis’ visit counter. First, we start the environment, download the repository, and access the folder factor12 and execute the command:

```
docker-compose up
```

Access the application in the browser. In case you are using GNU/Linux or Docker For Mac and Windows, access the address 127.0.0.1. You’ll see the following sentence:

```
“Hello World! 1 times.”
```

Access the application a couple more times so the counter goes up.

Then, execute the admin command from the worker service:

```
docker-compose exec worker python reset.py
```

The command **“python reset.py”** will be executed inside a new container, but using the same image of a regular worker.

Access the application again and check if the counter started from 1 again.
