# Check log of Envoy
You can run below command to check the system output of the sidecar container:
```
kubectl logs -f {the pod name} -c istio-proxy
```

# Change the envoy log level
Use below command to enter the shell env of envoy container:
```
kubectl exec -it {the pod name} -c istio-proxy -- /bin/sh
```
Then run below command to change the log level to trace for all components:
```
curl -X POST localhost:15000/logging?level=trace
```
The supported levels are (off, info, error, debug, trace). The command above will return a list of log level for
every components. If you want to specify log level for a specified component, you can use below command(assume 
you want to set log level of grpc component to info)
```
curl -X POST localhost:15000/logging?grpc=info
```

Below command can check the current log level for each components:
```
curl -X POST localhost:15000/logging
```

# Customize the log format
You can create a yml to change the log format for the IstioOperator(the meshConfig.accessLogFormat attribute)
```
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
spec:
  meshConfig:
    accessLogFile: /dev/stdout
    accessLogFormat: |
      "[%START_TIME%] \"%REQ(:METHOD)% %REQ(X-ENVOY-ORIGINAL-PATH?:PATH)% %PROTOCOL%\" %RESPONSE_CODE% %RESPONSE_FLAGS% %RESPONSE_CODE_DETAILS% "
``` 
Please refer [log format](https://istio.io/latest/docs/tasks/observability/logs/access-log/) about the detail for log
format.

After the yaml is ready, you can use below command to create the istio cluster with the new log format.
```
istioctl install --set profile={the profile name you want} -f {the name of your yaml file} -y
```

# Envoy admin interface
Use below command to enter the shell env of envoy container:
```
kubectl exec -it {the pod name} -c istio-proxy -- /bin/sh
```
Then, you can send HTTP request to the envoy admin interface(http://localhost:15000). Please refer 
[envoy admin interface](https://www.envoyproxy.io/docs/envoy/latest/operations/admin) for details. 
For example, if you want to check whether the envoy service is ready, you can run `curl -X GET localhost:15000/ready`

# Check the iptable for envoy
Sometimes, read the iptable setting is necessary to understand how IP package is routed inside envoy. To run `iptables`
command, you need to login the container as root. Since kubectl does not support execute command as root inside a 
container, you only can do so by using `docker` command. First of all, please go to the node that your pod is running 
in. Then, run `docker ps` to find the envoy container you need. The name of envoy container usually looks like 
`k8s_istio-xxxx`. If you deploy the sample application from istio installation package and want to check the envoy
container for the productpage pod, the envoy container name may looks like `k8s_istio-proxy_productpage-v1-xxxx`. 
Please run below command to login the container as root:
```
docker exec -u root -it --privileged=true {the name of envoy container} /bin/sh
```

Then you can run `iptables -t nat -L -v` command to check the IP routing rules in envoy container. This command
usually return something like:
```
Chain PREROUTING (policy ACCEPT 3149 packets, 189K bytes)
 pkts bytes target     prot opt in     out     source               destination
 3153  189K ISTIO_INBOUND  tcp  --  any    any     anywhere             anywhere

Chain INPUT (policy ACCEPT 3153 packets, 189K bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 455 packets, 38778 bytes)
 pkts bytes target     prot opt in     out     source               destination
   79  4740 ISTIO_OUTPUT  tcp  --  any    any     anywhere             anywhere

Chain POSTROUTING (policy ACCEPT 469 packets, 39618 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain ISTIO_INBOUND (1 references)
 pkts bytes target     prot opt in     out     source               destination
    0     0 RETURN     tcp  --  any    any     anywhere             anywhere             tcp dpt:15008
    0     0 RETURN     tcp  --  any    any     anywhere             anywhere             tcp dpt:22
    0     0 RETURN     tcp  --  any    any     anywhere             anywhere             tcp dpt:15090
 3148  189K RETURN     tcp  --  any    any     anywhere             anywhere             tcp dpt:15021
    1    60 RETURN     tcp  --  any    any     anywhere             anywhere             tcp dpt:15020
    4   240 ISTIO_IN_REDIRECT  tcp  --  any    any     anywhere             anywhere

Chain ISTIO_IN_REDIRECT (3 references)
 pkts bytes target     prot opt in     out     source               destination
    4   240 REDIRECT   tcp  --  any    any     anywhere             anywhere             redir ports 15006
...
```
The output above means:
* All input IP package will be routed to ISTIO_INBOUND chain first
* In ISTIO_INBOUND chain, if the IP package is sent to port 15008, 22, 15090, 15021 or 15020, they will be sent 
to those ports(no re-direct). For all other IP package they will be routed to ISTIO_IN_REDIRECT chain.
* In ISTIO_IN_REDIRECT chain, all IP packages are redirected to port 15006. This is why 15006 is not IN BOUND port
for envoy.
