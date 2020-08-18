
# How to install a kubernates cluster
This document introduce how to setup kubernates cluster in CentOS.

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

$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

```
yum install docker-ce docker-ce-cli containerd.io
```
