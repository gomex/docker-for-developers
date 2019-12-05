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
-   List all stopped containers
`docker ps -f "status=exited"`
-   Access a terminal on the container  
`docker exec -it container bash`
-   Save an image  
`docker save -o imagem.docker imagem`
-   Load an image
`docker load -i imagem.docker`