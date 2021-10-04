It is possible to create yml file to customize IstioOperator. For example, you can create a `customized_opt.yaml` as
below:
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
The file above customize the log file path and log format for the sidecar container. Please refer 
[IstioOperator Options](https://istio.io/latest/docs/reference/config/istio.operator.v1alpha1/) for full list of 
options.

You can run below command to create the istio cluster with your customization(assume you use demo profile):
```
istioctl install --set profile=demo -f customized_opt.yaml -y
```

After installation completed, please run below commands to check the configuration for IstioOperator
```
# get the name of IstioOperator
kubectl get IstioOperator -n istio-system
# If the returned name above is 'installed-state', then you can run below command to check the detail configuration
kubectl describe IstioOperator installed-state -n istio-system
```