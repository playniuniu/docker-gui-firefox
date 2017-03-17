# Run GUI apps and Firefox with Docker for Mac 2017

This article introduce how to run GUI apps and Firefox with Docker for Mac, the purpose is just for fun, maybe can be used for build Website test automation with docker, you can check [this](https://github.com/SeleniumHQ/docker-selenium).

All the commands below are tested with Docker 17.03.0-ce, macOS 10.12.3 and XUbuntu 16.04.2.

It include three sections here:

1. How to run GUI for Linux

2. How to run GUI for macOS

3. How to write Firefox Dockerfile

If you don't have so many times, you can just read *Finally commands* on each end of section :)

### How to run GUI for Linux

Let's start with a normal GUI app *xeyes*, here is the Dockerfile based on **Alpine Linux**:

```Dockerfile
FROM alpine:latest
MAINTAINER playniuniu@gmail.com

RUN apk add --no-cache --update xeyes

CMD xeyes
```

You can built your own images with `docker build -t xeyes .`.

To run Docker GUI on Linux, You need 3 steps here:

1. Add docker's X11 authority

2. Set $DISPLAY environment for Docker

2. Loading X11 sockets for Docker

##### Finally commands

1. Build Dockerfile

    ```bash
    docker build -t xeyes .
    ```

2. Add X11 authority

    ```bash
    xhost + local:docker
    ```

3. Run docker GUI apps

    ```bash
    docker run --rm \
               -e DISPLAY=unix$DISPLAY \
               -v /tmp/.X11-unix:/tmp/.X11-unix \
               xeyes
    ```

Now you can see an X11 **xeyes** apps in your screen.

### How to run GUI for macOS

The macOS is a little more complicate for it's lack of X-Window system,

so you need extra two steps to run GUI apps on macOS when you finished build xeyes images.

1. Establish the X11 server

2. Forward linux X11 socket to macOS X11 server ( **Important!** )

**Notes:** Lots of articles online lack of the second step, so cannot successfully run GUI on macOS.

For the two steps above, you need install two softwares here:

1. [XQuartz](https://www.xquartz.org/), you can install it with `brew cask install xquartz`

2. [socat](http://www.dest-unreach.org/socat/), you can install it with `brew install socat`

##### Finally commands

So finally, you need five steps for run GUI apps on macOS:

1.  Forward X11 socket

    ```bash
    socat TCP-LISTEN:6000,reuseaddr,fork UNIX-CLIENT:\"$DISPLAY\"
    ```
    
2. Check your macOS's ip

    ```bash
    export MAC_IP=$(ifconfig en0 | grep inet | awk '$1=="inet" {print $2}')
    ```
    
3. Add X11 authority ( optional )

    ```bash
    xhost + $MAC_IP
    ```

4. Open XQuartz software

5. Run Docker GUI apps

    ```bash
    docker run --rm \
           -e DISPLAY=$MAC_IP:0 \
           xeyes
    ```
    
Now, you can see **xeyes** on your macOS.

### How to write Firefox Dockerfile

For Firefox, all the command steps are same, but you can not just simply replace the package xeyes with Firefox. The Firefox's Dockerfile need some extra steps below to reach the goal:

1. Install dbus-x11 package

2. Install a fonts for Firefox

3. Change to normal user to run Firefox

4. Prepare a profile folder for Firefox or use `firefox --new-instance`

The Dockerfile example is here:

```Dockerfile
FROM alpine:latest
MAINTAINER playniuniu@gmail.com

ENV URL https://www.google.com

RUN apk --no-cache --update add firefox-esr dbus-x11 ttf-ubuntu-font-family \
    && adduser -S normaluser \
    && echo "normaluser:normaluser" | chpasswd

USER normaluser

CMD firefox --new-instance ${URL}
```

You can also check my [GitHub](https://github.com/playniuniu/docker-gui-firefox) from more detail.

##### Finally commands

For Linux Platform, you can run Firefox with docker with:

1. Add X11 authority

    ```bash
    xhost + local:docker
    ```

2. Run Firefox in Docker

    ```bash
    docker run --rm \
               -e DISPLAY=unix:$DISPLAY \
               -v /tmp/.X11-unix:/tmp/.X11-unix \
               --name firefox \
               playniuniu/docker-gui-firefox
    ```

For macOS platform, same five steps as above:

1.  Forward X11 socket

    ```bash
    socat TCP-LISTEN:6000,reuseaddr,fork UNIX-CLIENT:\"$DISPLAY\"
    ```
    
2. Check your macOS's ip

    ```bash
    export MAC_IP=$(ifconfig en0 | grep inet | awk '$1=="inet" {print $2}')
    ```
    
3. Add X11 authority ( optional )

    ```bash
    xhost + $MAC_IP
    ```

4. Open XQuartz software

5. Run Docker GUI apps

    ```bash
    docker run --rm \
           -e DISPLAY=$MAC_IP:0 \
           playniuniu/docker-gui-firefox
    ```

### Reference

1. [How to run a Linux GUI application on OSX using Docker](http://kartoza.com/en/blog/how-to-run-a-linux-gui-application-on-osx-using-docker/)

2. [Docker for Mac and GUI applications](https://fredrikaverpil.github.io/2016/07/31/docker-for-mac-and-gui-applications/)

3. [Running GUI apps with Docker](http://fabiorehm.com/blog/2014/09/11/running-gui-apps-with-docker/)

4. [chrisdaish/docker-firefox](https://github.com/chrisdaish/docker-firefox)

5. [SeleniumHQ/docker-selenium](https://github.com/SeleniumHQ/docker-selenium)
