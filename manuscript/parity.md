# Development/production parity

Next on the [12factor](https://12factor.net) model list, we have **“Development/production parity”** as the tenth best practice. 

![](images/paridade1.png)

Unfortunately, in most software work environments there is a deep abyss between development and production. It’s not just fortuity or lack of luck, it’s because of the differences between the development and infrastructure teams. According to 12factor, these differences appear within the following:

 * **Time**: developers can work in a code for days, weeks or even months to go into production.
 * **Personnel**: developers write code, operation engineers deploy the code.
 * **Tools**: devedores can use sets such as Nginx, SQLite and OS X, while the app in production uses Apache, MySQL and Linux.

12factor intends to collaborate in order to reduce this abyss between teams and equalize environments. Regarding the differences presente, here are the respective proposals:

 * **Time**: developers can write code and see the deployment finished hours or even minutes later.
 * **Personnel**: developers who write code are closely involved in deployment and following its behavior in production.
 * **Tools**: keep development and production as similar as possible.

One of container’s making goals is to collaborate with portability between development and production environments. The ideia is that the image is built and only its status will be modified to be put into production. The current code is ready to this behavior, thus there’s not much to be modified to guarantee the best practice. It’s like a bonus for adopting Docker and following the other 12factor best practices.
