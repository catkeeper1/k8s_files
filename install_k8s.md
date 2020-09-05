
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
Execute command `sysctl -p` to make config above effective


## Install docker

Run below script to remove existing docker installation:
```
yum remove -y docker \
    docker-client \
    docker-client-latest \
    docker-ce-cli \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-selinux \
    docker-engine-selinux \
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
yum install -y docker-ce-19.03.8 docker-ce-cli-19.03.8 containerd.io
```

Run below command to enable docker service
```
systemctl enable docker
systemctl start docker
```

If your server is behind GFW, please use docker image registry in Alicloud. Open `/etc/docker/daemon.json` and add 
below content:
```
{
  "registry-mirrors": ["https://registry.cn-hangzhou.aliyuncs.com"]
}
```

Change the cgroup driver from the default value to systemd. If this is not changed, when a worker node is added
to the cluster, warning message will be prompted `detected "cgroupfs" as the Docker cgroup driver. 
The recommended driver is "systemd".Please follow the guide at https://kubernetes.io/docs/setup/cri/`.
Open `/etc/docker/daemon.json` and add below content:
```
{
  ... 
  ,
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```

Run below commands to reload above setting
```
systemctl daemon-reload
systemctl restart docker
```


Run below command to verify docker installation
```
docker run hello-world
```



## Install kubeadm
Install kubernetes repository:
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

```

Remove existing installation:
```
yum remove -y kubelet kubeadm kubectl
```

Install kubeadm:
```
yum install -y kubelet-1.18.8 kubeadm-1.18.8 kubectl-1.18.8

systemctl enable kubelet 
systemctl start kubelet
```

Open `/etc/hosts` add the DNS name and IP of the master node in it. If the master node already has a DNS name,
please ignore this step. 

Run below command to create a kubeadm config file. Please replace "k8s.apiserver" with your master node DNS name. 
Please make sure the serviceSubnet and podSubnet is different from the subnet of your nodes. 

## Init master node

```
cat <<EOF > ./kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.18.8
imageRepository: registry.aliyuncs.com/k8sxio
controlPlaneEndpoint: "k8s.apiserver:6443"
networking:
  serviceSubnet: "10.96.0.0/16"
  podSubnet: "10.100.0.1/16"
  dnsDomain: "cluster.local"
EOF
```
Please refer `https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2` for the completed reference
about kubeadm config file.

Please run `kubeadm init --config=kubeadm-config.yaml --upload-certs` to init kubeadm.

Run below command to setup the config for kubectl:
```
mkdir /root/.kube/
cp -i /etc/kubernetes/admin.conf /root/.kube/config
```

Install calico CNI. Please run `curl https://docs.projectcalico.org/manifests/calico.yaml -O` to download 
the installation file. If you cannot download it because of network issue, please use the `calico3.15.2.yaml` that
just besides this file(rename it to `calico3.15.2.yaml`). Then run command `kubectl apply -f calico.yaml` to install
calico CNI.

Run command `watch kubectl get pod -n kube-system -o wide` to check the status of pods in kube-system namespace.
Wait until all of them are in "running" status. Run `kubectl get node` to check the node status. 

If you want pod scheduled to master node as well, please run command 
`kubectl taint nodes --all node-role.kubernetes.io/master-`.

## Init worker node
In master node run `kubeadm token create --print-join-command` to get the join command. 
The command may looks like 
```
kubeadm join k8s.apiserver:6443 --token 7dugxy.hb7voxef53yrvc38     --discovery-token-ca-cert-hash sha256:29b1eb1777a486e6030898f12163ab3e4a2b6584897593d833e38ddf1c032cce

```
Then, run the command to join the cluster. 

## Test with a deployment
Create a file `webtest.yaml`, place below sources in it:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp1
  template:
    metadata:
      labels:
        app: webapp1
    spec:
      containers:
      - name: webapp1
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80

##
apiVersion: v1
kind: Service
metadata:
  name: webapp1-svc
  labels:
    app: webapp1
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30080
  selector:
    app: webapp1

```

Run command `kubectl get pod`. This commands should return 2 pods with prefix `webapp1`. Run command `kubectl get svc`
. This command should return 1 service with prefix `webapp1`

Run command ` curl localhost:30080`, it should return somethng like below:
```
<h1>This request was processed by host: webapp1-7c456784b7-xb2cm</h1>
```


