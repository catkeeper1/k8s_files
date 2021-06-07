
Download istio from `https://github.com/istio/istio/releases/tag/1.10.0`. Extract the package.

Edit `~/.bash_profile` file, add the `istioctl` command to PATH env variable. For example, 
```
PATH=$PATH:{the extracted istio folder}/bin
export PATH
```

Run command `istioctl install --set profile=demo -y` to install demo profile. Please notes that, if
you have enable docker image registry mirror(such as alicloud), you may need to disable them. Otherwise,
the istiod image cannot be downloaded.

Instruct istiod to inject envoy sidecar proxy automatically when application is deployed to default namespace:
`kubectl label namespace default istio-injection=enabled`

Go to the istio extracted folder. Deploy the Bookinfo sample application:
`kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml`
 
Deploy gateway `kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml`

Run command `kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}'`
to check the node port for external access.

Open browser and use url `http://{node host}:{node port}/productpage` to access the product page.

Run below command to install dashboard
```
kubectl apply -f samples/addons
kubectl rollout status deployment/kiali -n istio-system
```
 



