The nginx ingress controller is config base on info from "https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/".

Base on the link above, the config file is checked out from "https://github.com/nginxinc/kubernetes-ingress/" and place at "kubernetes-ingress" folder. If you need to change any setting, please refer file in "kubernetes-ingress" folder.

To make sure we can access the ingress controller through 80 and 443 port, I did not follow the document above that create node port by using "kubectl create -f service/nodeport.yaml"(you can find this command in the document above) but using "nodeport_access.yaml" in this folder
~                                                                                                                                                                    
~                                                                                                                        