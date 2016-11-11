# randomBits
I wanted to spawn VSTS agents in docker containers, which in turn are capable of building docker images for .net core / NodeJS / React apps. This required a bit of tinkering. For this we need VSTS, a docker container capable of building docker images and a host which runs docker for those build agents.

---

## CentOS docker VSTS host
This machine runs docker to spawn 1 or more VSTS agents. We need to install docker but because we are building docker inside docker we need the API version to match. So match the version of this docker install to the latest VSTS agent available.

### Install specific version on docker host
In order to build docker images inside docker the API versions need to match. Here I'm using 1.12.1
```
sudo yum install -y http://yum.dockerproject.org/repo/main/centos/7/Packages/docker-engine-1.12.1-1.el7.centos.x86_64.rpm
sudo usermod -aG docker *theUserRunningDocker*
```
### get vsts agent docker image (note the version is equel to the host 1.12.1)
```
sudo docker pull microsoft/vsts-agent:ubuntu-16.04-docker-1.12.1
```
### start vsts container
Before you're able to start your new container, you need to create a VSTS_TOKEN.
```
sudo docker run -e VSTS_ACCOUNT=*accountname* 
                -e VSTS_TOKEN=*token* 
                -e VSTS_AGENT='$(hostname)-agent' 
                -e VSTS_POOL=ContainerBuilders 
                --privileged 
                -v /var/run/docker.sock:/var/run/docker.sock 
                -d 
                microsoft/vsts-agent:ubuntu-16.04-docker-1.12.1
```
---

## The VSTS agent container itself (ubuntu based)

### start bash in container
```
sudo docker exec -it <containerid> bash
```

### add node and npm in docker client
```
curl -sL https://deb.nodesource.com/setup_6.x | bash -
apt-get install -y nodejs
```

### Something I needed
```
npm install -g webpack
```

### add .net core in docker client
```
sh -c 'echo "deb [arch=amd64] https://apt-mo.trafficmanager.net/repos/dotnet-release/ xenial main" > /etc/apt/sources.list.d/dotnetdev.list'
apt-key adv --keyserver apt-mo.trafficmanager.net --recv-keys 417A0893
apt-get update
apt-get install dotnet-dev-1.0.0-preview2-003131
```

### login to docker hub in docker client (VSTS docker login doesn't set the correct config.json path)
https://stackoverflow.com/questions/36663742/docker-unauthorized-authentication-required-upon-push-with-successful-login
```
docker login
```

---

## Octopus deploy target (tentacle via ssh)
In order to deploy anything from octopus deploy to a CentOS, we need mono.

### install EPEL (needed for docker-compose + mono)
```
sudo yum install epel-release
sudo yum update
```

### install mono
```
sudo yum install yum-utils
sudo rpm --import "http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF"
sudo yum-config-manager --add-repo http://download.mono-project.com/repo/centos/
sudo yum install mono
```

### install docker-compose (make sure pip is up to date)
```
sudo yum install -y python-pip
sudo pip install docker-compose
sudo yum upgrade python*
```
