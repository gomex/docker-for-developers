# Turning your application into a container

We are continually evolving to deliver ever better applications, in less time, replicable and scalable. However, the efforts and learnings to reach this level of maturity, many times, are not so simple to achieve. 

Currently, we observe the rising of several platforms to facilitate the deployment, configuration and scalability of the applications we develop. However, to increase our maturity level we can not just depend on the platform, we need to build our application following the best practices. 

Aiming to define a series of best practices common to modern web applications, some developers from [Heroku](https://www.heroku.com/) wrote the [12Factor app](https://12factor.net) manifesto, counting on a wide experience in developing web applications.  

![](images/12factor.gif)

"The Twelve-Factor app" (12factor) is a manifesto with a series of best practices for building software using automation declarative formats, maximizing the portability and minimizing divergencies amongst execution environments, allowing the deployment in modern cloud platforms and facilitating scalability. Thus, applications are build stateless and connected to any infrastructure services combination to data retention (database, queue, cache memory and similar). 

In this chapter we’ll talk about creating applications with Docker images based on 12factor app. The idea is to show the best practices to create an infrastructure to support, pack and make your application available with a high level of maturity and agility. 

The use of 12factor practices fits Docker very well, because many of its resources are better used if your application is thought within this purpose. Therefore, we will give you an idea of how to take advantage of your application’s whole potential. 

We will show a HTTP service as an example. It’s written in Python, that displays how many times it was accessed, and this information is stored in a conter in a Redis instance. 

Now, let’s hit these best practices!
