## Useful commands

Here are some useful, simple commands:

-   Remove all inactive containers  
`docker container prune`
-   Stop all containers  
`docker stop $(docker ps -q)`
-   Remove all local images  
`docker image prune`
-   Remove “orphan” volumes  
`docker volume prune`
-   Shows use of containers resources running  
`docker stats $(docker ps --format {{.Names}})`
