# Logs

The eleventh best practice in the [12factor](http://12factor.net) model list is **“Logs”**.

![](images/logs1.png)

While developing codes, generating data for logs is something very much consolidated. We don’t believe that there’s softwares in development without this concern. However, the correct use of log goes beyond of just generating data.

For context effect, according to 12factor, log is: “(…) the stream of aggregated, time-ordered events collected from the output streams of all running processes and backing services”.

Usually, logs are stores in files, with events per line (backtraces from exception may span multiple lines). But this practice is not recommended, at least not from an application’s perspective. This means that the application should not worry in which file it will store the logs.

To specify files implies on informing the correct directory of this file, that, in turn, results in previous environment configuration. This impacts negatively in application’s portability, because is necessary that the environment that will get the solution follows a series of technical requirements to back the application, burying the possibility of “Build it once, run it anywhere”.

The best practice says that the applications should not manage or route log files, but should be deposited without any buffer to the standard output (STDOUT). Thus, an infrastructure external to the application - platform - must manage, collect and format the logs output to future reading. This is really important when the application is running in several instances.

With Docker, such task becomes easy for Docker already collects standard output logs and send them to some of the several log drivers. The driver can be configured in the container initialization in order to group the logs in the log remote service, such as syslog.

The example code in the repository([https://github.com/gomex/exemplo-12factor-docker](https://github.com/gomex/exemplo-12factor-docker)), in the folder factor11, is ready to test the best practice, for it sends all data outputs to STDOUT and you can check it by initiating the service with the command below:

```
docker-compose up
```

After initiating, access the browser and verify the application requirements that appear on the Docker-Compose console.
