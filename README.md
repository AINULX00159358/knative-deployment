# Installing Knative on Kubernets Cluster
 version **1.11.1** _(latest release as of October-2023)_
## Minimum Resource Requirement ##
Kubernetes cluster :
- with Node CPU of 4 core or More
- with Node Available Memery of 8 GB or More

---
## Install Knative Serving
#### Install The Required CRD(s) for Knative Serving ####
```
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.11.1/serving-crds.yaml
```
#### Install Knative Serving Core Components ####
```
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.11.1/serving-core.yaml
```
#### Install Knative Serving Default Domain ####
```
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.11.1/serving-default-domain.yaml
```
#### Install Knative Serving HPA ####
```
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.11.1/serving-hpa.yaml
```
#### Install Service Mesh (Kourier) ####
```
kubectl apply -f https://github.com/knative/net-kourier/releases/download/knative-v1.11.2/kourier.yaml
```
Patch Knative Serving Components with **Kourier** as its networking components. The ConfigMap **config-network** in **knative-serving** namespace will be Patched.
```
kubectl patch configmap/config-network \
  --namespace knative-serving \
  --type merge \
  --patch '{"data":{"ingress-class":"kourier.ingress.networking.knative.dev"}}'
```
#### Verify Installation ####
``` kubectl get pods, svc -n knative-serving ```
```console
NAME                                          READY   STATUS      RESTARTS   AGE
pod/activator-679b47f596-knn62                1/1     Running     0          35h
pod/autoscaler-5c9f86fc54-prv9f               1/1     Running     0          35h
pod/autoscaler-hpa-77579f8877-ntsmm           1/1     Running     0          35h
pod/controller-6fd8d77964-xd7c4               1/1     Running     0          35h
pod/default-domain-ctsb2                      0/1     Completed   0          35h
pod/net-kourier-controller-566bc95c9c-l9l6j   1/1     Running     0          35h
pod/webhook-7654b7cbc8-k8p9k                  1/1     Running     0          35h

NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                   AGE
service/activator-service            ClusterIP   10.96.128.192    <none>        9090/TCP,8008/TCP,80/TCP,81/TCP,443/TCP   35h
service/autoscaler                   ClusterIP   10.98.252.210    <none>        9090/TCP,8008/TCP,8080/TCP                35h
service/autoscaler-bucket-00-of-01   ClusterIP   10.101.232.232   <none>        8080/TCP                                  35h
service/autoscaler-hpa               ClusterIP   10.107.92.127    <none>        9090/TCP,8008/TCP                         35h
service/controller                   ClusterIP   10.103.176.58    <none>        9090/TCP,8008/TCP                         35h
service/default-domain-service       ClusterIP   10.102.38.185    <none>        80/TCP                                    35h
service/net-kourier-controller       ClusterIP   10.97.241.47     <none>        18000/TCP                                 35h
service/webhook                      ClusterIP   10.102.223.160   <none>        9090/TCP,8008/TCP,443/TCP                 35h
```
```kubectl get configmap -n knative-serving```
```console
NAME                     DATA   AGE
config-autoscaler        1      36h
config-defaults          1      36h
config-deployment        2      36h
config-domain            2      36h
config-features          1      36h
config-gc                1      36h
config-kourier           1      36h
config-leader-election   1      36h
config-logging           1      36h
config-network           2      36h
config-observability     1      36h
config-tracing           1      36h
kube-root-ca.crt         1      36h
````
```kubectl get pod,svc -n kourier-system```
```console
NAME                                          READY   STATUS    RESTARTS   AGE
pod/3scale-kourier-gateway-7858c8946f-94qtb   1/1     Running   0          36h

NAME                       TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)                      AGE
service/kourier            LoadBalancer   10.110.213.200   10.110.213.200   80:30980/TCP,443:31831/TCP   36h
service/kourier-internal   ClusterIP      10.102.94.2      <none>           80/TCP,443/TCP               36h
```
_Note: If you are Installing Knative in Minikube, make sure minikube tunnel is running in another shell. This is needed for kourier Loadbalancer to get an External IP_
_In Azure K8S Cluster, Loadbalancer is automatically assigned an External IP Address_

---
## Install Knative Eventing ##

#### Install Knative Eventing CRD(s) ####
``` 
kubectl apply -f https://github.com/knative/eventing/releases/download/knative-v1.11.4/eventing-crds.yaml
```
#### Create Knative Eventing Core ####
```
kubectl apply -f https://github.com/knative/eventing/releases/download/knative-v1.11.4/eventing-core.yaml
```
#### Create InMemory Channel ####
```
kubectl apply -f https://github.com/knative/eventing/releases/download/knative-v1.11.4/in-memory-channel.yaml
```
#### Create MT Channel Broker ####
```
kubectl apply -f https://github.com/knative/eventing/releases/download/knative-v1.11.4/mt-channel-broker.yaml
```
#### Verify ####
``` 
kubectl get pod, svc -n knative-eventing
```
```console
NAME                                        READY   STATUS    RESTARTS   AGE
pod/eventing-controller-cc4cffddd-hc9dw     1/1     Running   0          36h
pod/eventing-webhook-568dfd66dc-jcwlv       1/1     Running   0          36h
pod/imc-controller-7f8996c4f-2bpq7          1/1     Running   0          36h
pod/imc-dispatcher-ff7df45f8-9vxpp          1/1     Running   0          36h
pod/mt-broker-controller-5864c48b6d-q8xk8   1/1     Running   0          36h
pod/mt-broker-filter-7bb976c9-69qxz         1/1     Running   0          36h
pod/mt-broker-ingress-7d94b57997-7d52x      1/1     Running   0          36h

NAME                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                     AGE
service/broker-filter             ClusterIP   10.103.11.28     <none>        80/TCP,443/TCP,9092/TCP                     36h
service/broker-ingress            NodePort    10.103.111.251   <none>        80:30452/TCP,443:30937/TCP,9092:31287/TCP   36h
service/eventing-webhook          ClusterIP   10.109.176.168   <none>        443/TCP                                     36h
service/imc-dispatcher            ClusterIP   10.100.94.185    <none>        80/TCP,443/TCP,9090/TCP                     36h
service/inmemorychannel-webhook   ClusterIP   10.104.227.94    <none>        443/TCP,9090/TCP,8008/TCP                   36h
```
#### (Optional) Exponse Knative Event Broker as NodePort for External Testing ####
Expose Knative Broker as NodePort for Testing, so that test applicaions can post events.
```
kubectl patch svc broker-ingress -n knative-eventing -p '{"spec": {"type": "NodePort"}}'
```

#### (Optional) Create a Knative Event Broker 
Knative Event Broker will also create create a default channel 
```


---

```
