<!--  -->
###use the docker file
Open up a terminal window.

- Create a bridge network in Docker using the following docker network create command:
```sh
docker network create jenkins
```

- write docker file
```sh
FROM jenkins/jenkins:2.387.1
USER root
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean docker-workflow"
```

- build the image . This incorporates the blueocean plugin

```sh
docker build -t myjenkins-blueocean:2.387.1-1 .
```

Run your own myjenkins-blueocean:2.387.1-1 image as a container in Docker using the following docker run command:



```sh
docker run \
  --name jenkins-blueocean \
  --restart=on-failure \
  --detach \
  --network jenkins \
  --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client \
  --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 \
  --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:2.387.1-1 
```

- go to the location cause it is runing locally
```sh
http://localhost:8080/
http://127.0.0.1:8080/
```

- you could access your docker container (through a separate terminal/command prompt window) with a docker exec command like:

```sh
docker exec -it jenkins-blueocean bash
```
- then go to 

```sh
cat /var/jenkins_home/secrets/initialAdminPassword
```

- alternatively as you are running Jenkins in Docker using the official jenkins/jenkins image you can use 

```sh
sudo docker exec ${CONTAINER_ID or CONTAINER_NAME} cat /var/jenkins_home/secrets/initialAdminPassword
```
to print the password in the console without having to exec into the container.

## alpine/socat container to forward traffic from Jenkins to Docker Desktop on Host Machine

https://stackoverflow.com/questions/47709208/how-to-find-docker-host-uri-to-be-used-in-jenkins-docker-plugin
```sh
docker run \
  --name alpine-socat-docker \
  -d \
  --restart=always \
  -p 127.0.0.1:2376:2375 \
  --network jenkins \
  -v /var/run/docker.sock:/var/run/docker.sock alpine/socat \
  tcp-listen:2375,fork,reuseaddr \
  unix-connect:/var/run/docker.sock
```
get the container id of alpine
```sh
docker inspect <container_id> | grep IPAddress
```

> Note: in this case you can inspect it by `alpine-socat-docker` instade of `<container_id>` name


```sh
docker inspect alpine-socat-docker | grep IPAddress
```
> If system restarts then ip of exposed in this container may change.

### To configure docker as cloud host 
- go `Dashboard > Manage Jenkins > Configure Clouds`

- get the ip address from the alpine socat container.
- put in ththat with proper socket `Docker Cloud details > 
Docker Host URI` field 
  
  > Note: This procedure is running a local docker as remote service. you can configure same sittings as true remote service
  ```sh
  tcp://172.18.0.3:2375 # example
  ```
  Then test the connection 

- setup `Docker Agent templates`
####Note if docker is running in `RHEL` flavour there may some firewall issue 

issue can ve solved from the stackoverflow link

https://stackoverflow.com/questions/40214617/docker-no-route-to-host

```
We hit this issue on a RHEL box which was running firewalld. The firewall was preventing container to host access (other than icmp traffic).

We needed to configure the firewall to allow traffic from the docker containers through to the host. In our case, the containers were in a bridge network on subnet 172.27.0.0/16 (determined via docker network ls and docker inspect <network-name>). Firewall rules for firewalld can be updated via:

firewall-cmd --permanent --zone=public --add-rich-rule='rule family=ipv4 source address=172.27.0.0/16 accept'
firewall-cmd --reload

This was a useful reference in resolving the issue.
```

```
If anyone is still stuck with this problem on CentOS 8 or any system using firewalld

try the following settings for firewalld

# Allows container to container communication, the solution to the problem
firewall-cmd --zone=public --add-masquerade --permanent

# standard http & https stuff
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --add-port=443/tcp --permanent
# + any other port you may need

# reload the firewall
firewall-cmd --reload
you may also need to restart the docker service if it does not work immediately, 
there's no need to add the docker0 interface onto the trusted zone as many of the guides I've gone through stated

I was struggling with setting up a Traefik reverse proxy for my docker containers, 
I only got 502 responses with a no route error to my container from Traefik logs. 
At first I thought it was my Traefik setup but it turned out it was the firewall restrictions as @al. mentioned. 
It pointed me in the right direction and I got my answer from 
https://serverfault.com/questions/987686/no-network-connectivity-to-from-docker-ce-container-on-centos-8

```
#### Also you can see everything by yourself from here
https://www.youtube.com/watch?v=6YZvp2GwT0A