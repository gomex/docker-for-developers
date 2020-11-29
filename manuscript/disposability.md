# Disposability

In the ninth position of the list of the [12factor](https://12factor.net) model, we have **“Disposability”**.

When we talk about web applications, the expectancy is that more than one process attends to the whole traffic required to the service. However, as important as the capacity of starting new processes is the ability of a defective process ending up in the same velocity that started, for a process that takes too long to finish can compromise the whole solution, once it can still be responding to requirements defectively.

![](images/descartabilidade1.png)

Summing up, we can say that we applications should be able to quickly remove defective processes.

Aiming to prevent that the service is dependent on instances that serve it, the best practice says that the applications must be disposable; in other words, shutting down one of its instances must not affect the solution as a whole.

Docker gives the option to automatically dispose a container after using it - on **docker container run** use the option **-rm**. It’s important to highlight that this option doesn’t work while in **daemon (-d)** mode, therefore, it only makes sense to use it on **interactive (-i)** mode.

Another important detail of the best practice is to enable the code to shut down “graciously” and restart with no errors. Thus, when hearing a **SIGTERM** the code must finish any requirement in progress and then shut down the process with no problems and quickly, allowing that another process is quickly attended as well.

We consider a “gracious” shut down when an application is capable of self-finishing with no damages to the solution; as it receives the signal to shut down, it immediately refuses new requirements and just finishes up the pendent tasks that are running at that moment. It is implicit in this model: the HTTP requirements are short (no more than a few seconds), and in the case of long connections the client can automatically reconnect if the connection is lost.

The application went through the following alteration to attend the specification:

```
from flask import Flask
from redis import Redis
from multiprocessing import Process
import signal, os

host_redis=os.environ.get('HOST_REDIS', 'redis')
port_redis=os.environ.get('PORT_REDIS', '6379')

app = Flask(__name__)
redis = Redis(host=host_redis, port=port_redis)

@app.route('/')
def hello():
    redis.incr('hits')
    return 'Hello World! %s times.' % redis.get('hits')

if __name__ == "__main__":
    def server_handler(signum, frame):
        print 'Signal handler called with signal', signum
        server.terminate()
        server.join()

    signal.signal(signal.SIGTERM, server_handler)

    def run_server():
        app.run(host="0.0.0.0", debug=True)

    server = Process(target=run_server)
    server.start()
```

In the code above, we added up handling for, when it gets a SIGTERM signal, finishing up the process quickly. Without the handling, the code takes longer to be shut down. Therefore, we conclude that the solution is disposable enough. We can shut down and restart the container in another Docker Host and this change won’t impact the data integrity.

For understanding purposes on what we work here, it’s important to explain: according to Wikipedia a signal is “(…) an asynchronous notification sent to processes aiming to notify the occurrence of an event”. And SIGTERM is “(…) the name of a signal known as a computer process in POSIX operative systems. This is the standard signal sent by kill and killall commands. It causes the process to finish, as in SIGKILL, however it can be interpreted or ignored by the process. Thereby, SIGTERM performs a friendlier shut down, allowing to clear memory and closing files”.

To perform the test of what was presented so far, clone the repository ([https://github.com/gomex/exemplo-12factor-docker](https://github.com/gomex/exemplo-12factor-docker)) and access the folder factor8 (that’s right, number 8; let’s show the difference relating to factor9), executing the command below to initiate the containers:

```
docker-compose up -d
```

Then, execute the command below to finish up the containers:

```
time docker-compose stop
```

You’ll see that the worker finishing up takes about 11 seconds, because of the behavior of Docker-Compose that, in order to finish up first performs a **SIGTERM** and wait 10 seconds so the application shuts down by itself; otherwise, it sends a **SIGKILL** that shuts down the process abruptly. This time out is configurable. In case you wish to change it, just use the parameter **“-t”** or **“–timeout“**. Check an example:

```
docker-compose stop -t 5
```

Obs.: The value informed after the parameter is measured in seconds.

Now, to test the modified code, go to the folder **factor9** and execute the following command:

```
docker-compose up -d
```

Later, request the conclusion:

```
time docker-compose stop
```

Notice that the worker process finished up faster, for it got the SIGTERM signal. The application shut down by itself and didn’t need to receive a SIGKILL signal to be effectively shut down.
