# Processes

Next in the list of [12factor](https://12factor.net) model, we present **”Processes”** as the sixth best practice. 

Nowadays, with the automated processes and the due intelligence in maintaining applications, it is expected that the application can respond to demand peaks with automatic initialization of new processes without affecting its behavior.

![](images/processos.png)

The best practice says that 12factor application processes are stateless (don’t store state) and share-nothing. Any data that need to persist must be stored in stateful support service, usually used in a database.

The final goal of this practice does not differentiates if the application is executed in the developer’s machine or in production, because in this case what changes is the amount of initialized processes to respond the demands. In the developer’s machine is only one process; in production this number can be higher.

**12factor** points out that the memory space or file system of the server can be used briefly as a single transaction cache. For instance, the download of a big file, working over it and storing the results in the database.

We highlight that a state should never be store between requirements, it doesn’t matter the processing status of the next requirement.

It’s important to emphasize: by following a practice, a application doesn’t assume that any item stored in memory cache or in disk will be available for a future requirement or job - with many different processes running, higher are the chances of a future requirement to be served by a different process, even by a different server. Even when running in a single process, a restart (initiated by a code’s deployment, changes in configuration, or the running environment reallocating the process to a different physical location) usually will end up with the local state (memory and file system, for instance).

Some applications require persistent sessions to store information of the user session and so. Such sessions are used in future requirements from the same visitor. That is, if it’s stored with the processe, it’s clearly violating the best practice. In this case, the advice is to use a support service, such as redis, memcached or similar to this type of job that is external to the process. With that, it’s possible that the next process, no matter where it is, is able to get the update information.

The application we are working on does not keep local data and everything it need is stored on Redis. We don’t need to adequate anything in this code to comply with the best practice, as we can see:

```
from flask import Flask
from redis import Redis
import os
host_redis=os.environ.get('HOST_REDIS', 'redis')
port_redis=os.environ.get('PORT_REDIS', '6379')
app = Flask(__name__)
 redis = Redis(host=host_redis, port=port_redis)
@app.route('/')
def hello():
 redis.incr('hits')
 return 'Hello World! %s times.' % redis.get('hits')
if __name__ == "__main__":
 app.run(host="0.0.0.0", debug=True)
```
To access the code of the practice, go to the [repository](https://github.com/gomex/exemplo-12factor-docker), in the folder **“factor6“**.
