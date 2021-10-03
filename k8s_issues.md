
# Api server cert expired
The symptom of this issue is that no matter what kind of commands are executed with 'kubectl', it always return 
connection refused error. For example, if 'kubectl get pods' is executed, it return:
```
The connection to the server {the host name of api server}:6443 was refused - did you specify the right host or port?
```
Then, please run below command to confirm:
`kubeadm alpha certs check-expiration`
Above command return the certs valid period. If current time is out of the period, it means the cert is expired or not
valid.

After the issue is confirmed, please execute below command to solve the issue:
```
# renew all certs
kubeadm alpha certs renew all


# Copy the new config file to home folder for kubectl to use.
mv ~/.kube/config ~/.kube/config.old
cp -i /etc/kubernetes/admin.conf ~/.kube/config
chown $(id -u):$(id -g) ~/.kube/config
sudo chmod 777 ~/.kube/config

# run kubectl to check whether the connection work
kubectl get nodes
```
