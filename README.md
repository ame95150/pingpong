# pingpong : Ping Pong Aeron with Earthly and Docker

The purpose of this project is to set up an Aeron Ping Pong project with Earthly and Docker, by performing a ping pong between two applications under Ubuntu server.

## Table of Contents

- [Configuration](#Configuration)
- [Prerequisites](#Prerequisites)
- [Deployement](#Deployement)


## Configuration

To run this project in an Ubuntu server, should be installed :
- Git  
    ```bash
       sudo apt-get update
       sudo apt-get -y install git
       git --version
    ```    
- Docker
    1. uninstall all conflicting packages

   ```bash
    for pkg in docker.io docker-doc docker-compose  docker-compose-v2 podman-docker containerd runc; 
    do sudo apt-get remove $pkg; 
    done
    ```
    
    2. Set up Docker's apt repository
    ```bash
    # Add Docker's official GPG key:
    sudo apt-get update
    sudo apt-get install ca-certificates curl gnupg
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg
    
    # Add the repository to Apt sources:
    echo \
    "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    ```
    3. Install the Docker packages
    ```bash
    # Install the last version of docker
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```
    4. Create the docker group and add your user
    ```bash
    sudo groupadd docker
    sudo usermod -aG docker $USER
    ```
- Earthly CLI 
    ```bash
    sudo /bin/sh -c 'wget https://github.com/earthly/earthly/releases/latest/download/earthly-linux-amd64 -O /usr/local/bin/earthly && chmod +x /usr/local/bin/earthly && /usr/local/bin/earthly bootstrap --with-autocomplete'
    ```
## Prerequisites
1. JDK JAVA 11  (openjdk:11-ea-jdk-slim)
2. Aeron all in one JAR (https://jar-download.com/artifacts/io.aeron/aeron-all)

Aeron all in one Jar can be downloaded, and can also be builded with gradle.

## Deployement 
In the initial release, Earthly is used for this project, but a docker compose deployment is implememented to facilitate the handling of the Ping and Pong features of the Aeron project.

### Earthly deployment (draft)

The Earthfile is composed of :
- openjdk:11-ea-jdk-slim container as base, /aeron as work directory and the aeron-all-1.42.1.jar copied in /aeron
- ping process is run inside the buil kit container (base)
- pong process is run inside a dind Earhtly container earthly/dind:latest (+openjdk11, +aeron-all-1.42.1.jar)

**This section is currently under development.**

### Docker compose deployment

The stack is in [the docker-compose directory](docker-compose) at the root of the the git project.
The [docker-compose.yml](docker-compose/docker-compose.yml) is composed of 2 services and one network : 
- Aeron Ping and Pong services 
```
- openjdk:11-ea-jdk-slim as docker image base
- java command to run class with an embeded media driver
- shared memory parameter
- docker volumes to share Aeron jar and get log files
- pingpong network
```
- Aeron Ping and Pong network

To test the stack simply run the command that creates the containers, the networks and retrieves the logs in the volume directory
```
cd docker-compose && docker-compose up -d
```

To check the running containers 
```
docker ps -a

CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS         PORTS     NAMES
24ad0d6fc56b   openjdk:11-ea-jdk-slim   "bash -c 'java -cp /…"   10 seconds ago   Up 8 seconds             ping
b17cb0a7430d   openjdk:11-ea-jdk-slim   "bash -c 'java -cp /…"   10 seconds ago   Up 8 seconds             pong
```

To check containers logs
```
docker logs ping --follow

WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by org.agrona.nio.TransportPoller (file:/aeron/aeron-all-1.42.1.jar) to field sun.nio.ch.SelectorImpl.selectedKeys
WARNING: Please consider reporting this to the maintainers of org.agrona.nio.TransportPoller
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
```

At the end of the test, log files are created [ping](docker-compose/volumes/ping.log) and [pong](docker-compose/volumes/pong.log) 
