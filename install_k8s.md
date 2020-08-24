
# How to install a kubernates cluster
This document introduce how to setup kubernates cluster in CentOS 7.8


## OS configuration

Run `cat /etc/redhat-release ` to check the OS version. Make sure the version is 7.8, 7.7 or 7.6

Run `lscpu` to check the number of available CPU core. The minimum number of CPU cores is 2.

Run below commands to change the hostname to make sure all hostnames of nodes in the clusters are unique and not equals to "localhost".

```
# modify hostname
hostnamectl set-hostname your-new-host-name
# check the modified result
hostnamectl status
# update the host file
echo "127.0.0.1   $(hostname)" >> /etc/hosts
```

Run comnmands `ip route show` and `ip address` to check what are the default network adaptor and the ip address. Make sure all nodes in the cluster can access their default IP each other(no firewall, NAT ... blocking the communication).

Shutdown firewall:
```
systemctl stop firewalld
systemctl disable firewalld
```

Shutdown Selinux
```
setenforce 0
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
```

Disable swap
```
swapoff -a
yes | cp /etc/fstab /etc/fstab_bak
cat /etc/fstab_bak |grep -v swap > /etc/fstab
```

Edit `/etc/sysctl.conf`, make sure below configuration in this file:
```
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
net.ipv6.conf.all.forwarding = 1
```



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

If your server is behind GFW, please use docker image registry in Alicloud by adding below statement to `~/.bashrc`
```
export REGISTRY_MIRROR=https://registry.cn-hangzhou.aliyuncs.com
```


Run below command to verify docker installation
```
docker run hello-world
```


