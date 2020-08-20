
# How to install a kubernates cluster
This document introduce how to setup kubernates cluster in CentOS 7.

## Install docker

Run below script to remove existing docker installation:
```
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

```

Install docker repository:
```
yum install -y yum-utils

yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

```
yum install docker-ce docker-ce-cli containerd.io
```

Run below command to enable docker service
```
systemctl enable docker
systemctl start docker

```

Run below command to verify docker installation
```
docker run hello-world
```

