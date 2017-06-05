## Can I run GUI applications?

Absolutely, it’s perfectly possible to run gUI (or X11) applications in containers; that means that all advantages of using Docker apply also to graphic applications. 

In addition, it’s possible to make the application run in multiple systems (Linux, Windows and macOS) just by building it for Linux. 

### How?

First of all… in 99% of the cases it’s necessary to grant access to X: ‘xhost local’ (this access will be available until you shut down/restart the host). 

This is the simplest command, it mount the X11 socket of the host in the container and defines the display (note that we are “evolving” the commands little by little, but you can use only the flags that you find necessary - the only ones that are mandatory are the mounting of the volume ‘/tmp/.X11-unix’ and the environment variable ‘DISPLAY’):

```
docker container run [--rm [-it]|-d] \
-v /tmp/.X11-unix:/tmp/.X11-unix \
-e DISPLAY \
imagem [comando]
```
In some cases, the ‘DISPLAY/ variable has to be’DISPLAY=unix$DISPLAY’ (but, to be honest, I don’t know why, only know that that was the recommended by the person who built the image).


To user the hardware 3D acceleration support:

```
docker container run [--rm [-it]|-d] \
-v /tmp/.X11-unix:/tmp/.X11-unix \
-e DISPLAY \
--device /dev/dri \
imagem [comando]
```

Adding audio:

```
docker container run [--rm [-it]|-d] \
-v /tmp/.X11-unix:/tmp/.X11-unix \
-e DISPLAY \
--device /dev/dri \
--device /dev/snd \
imagem [comando]
```

Adding webcam:

```
docker container run [--rm [-it]|-d] \
-v /tmp/.X11-unix:/tmp/.X11-unix \
-e DISPLAY \
--device /dev/dri \
--device /dev/snd \
--device /dev/video0 \
imagem [comando]
```

Using the same date/hour from the host:

```
docker container run [--rm [-it]|-d] \
-v /tmp/.X11-unix:/tmp/.X11-unix \
-e DISPLAY \
--device /dev/dri \
--device /dev/snd \
--device /dev/video0 \
-v /etc/localtime:/etc/localtime:ro \
imagem [comando]
```
Attention: depending on the distribution, there’s not a /etc/localtime; you have to check how it defines the timezone and “replicate” it into the container.

Keeping the application’s configurations:

```
docker container run [--rm [-it]|-d] \
-v /tmp/.X11-unix:/tmp/.X11-unix \
-e DISPLAY \
--device /dev/dri \
--device /dev/snd \
--device /dev/video0 \
-v /etc/localtime:/etc/localtime:ro \
-v $HOME/.config/app:/root/.config/app \
imagem [comando]
```
Obs.: the path is just an example.

**Bonus:** Videogame joystick (actually, any input device):

```
docker container run [--rm [-it]|-d] \
-v /tmp/.X11-unix:/tmp/.X11-unix \
-e DISPLAY \
--device /dev/dri \
--device /dev/snd \
--device /dev/video0 \
-v /etc/localtime:/etc/localtime:ro \
-v $HOME/.config/app:/root/.config/app \
--device /dev/input \
imagem [comando]
```

### What about docker-compose?
It works normally… Just mount the X11 socket and define the environment variable on docker-compose.yml and will be possible to start multiple application with only one command.

### On Windows and macOS

#### [Mac OS X](https://github.com/docker/docker/issues/8710#issuecomment-71113263)  
Install Docker for Mac

```
brew install socat
brew cask install xquartz
open -a XQuartz

socat TCP-LISTEN:6000,reuseaddr,fork UNIX-CLIENT:\"$DISPLAY\"
docker container run -e DISPLAY=hostip:0 [...] image OU DISPLAY=hostip:0 docker-compose up [-d]
```  
#### [Windows](https://github.com/docker/docker/issues/8710#issuecomment-135109677)  
Install xming  
Install o Docker for Windows

```
xming :0 -ac -clipboard -multiwindow
docker container run -e DISPLAY=hostip:0 [...] image OU DISPLAY=hostip:0 docker-compose up [-d]
```

Obs.: In case you’re using Docker Toolbox, insert VM’s IP (‘docker-machine ip default’ will inform).
