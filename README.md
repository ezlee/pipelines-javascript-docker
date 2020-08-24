A virtual service will be deployed to the kubernetes cluster default namespace, but the istio gateway service needs to be manually deployed after istio is deployed.
Istio gateway already deloyed on the kubenetes cluster:

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: pipeline-javascript-app-gateway
spec:
  selector:
    istio: ingressgateway # use Istio default gateway implementation
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "pipeline-app.example.local"
```

Validation(with build job number 214:
- Service is using ClusterIP, not LoadBalancer.
```
NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
details                   ClusterIP   10.233.19.56    <none>        9080/TCP   29h
kubernetes                ClusterIP   10.233.0.1      <none>        443/TCP    2d4h
pipeline-nodejs-svc-214   ClusterIP   10.233.38.99    <none>        8080/TCP   3h4m
```
- istio ingress gateway is using LoadBalancer with IP 192.168.99.230.
```
$ kubectl get svc -n istio-system
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                                                                      AGE
grafana                ClusterIP      10.233.22.86    <none>           3000/TCP                                                                     39h
istio-egressgateway    ClusterIP      10.233.26.124   <none>           80/TCP,443/TCP,15443/TCP                                                     40h
istio-ingressgateway   LoadBalancer   10.233.38.57    192.168.99.230   15021:31631/TCP,80:31956/TCP,443:31731/TCP,31400:31820/TCP,15443:31749/TCP   40h 
```
- virtualserver deployed (VirtualServer "bookinfo" uses * for any host, but "pipeline-javascript-app-vs" use a host header 
```
kubectl get vs
NAME                         GATEWAYS                            HOSTS                          AGE
bookinfo                     [bookinfo-gateway]                  [*]                            30h
pipeline-javascript-app-vs   [pipeline-javascript-app-gateway]   [pipeline-app.example.local]   3h13m
```
- validate with host header
```
curl --header 'host:pipeline-app.example.local' 192.168.99.230
Hello world, nodejs running in docker with build number 214!
```
