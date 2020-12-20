---
title: Kubernetes services
category:
- devops
- kubernetes
- concepts
excerpt: A post on how the kubernetes service works and how to use them
---

### Introduction
In Kubernetes, `pods` are the basic building blocks that run the containers. Each pod is given an IP address. So if we want to reach the servers running inside the pods we need a pod IP. Lets say we have 4 pod replicas running for a deployment and we can access the service with the pod IP address of any of the 4 running pods. The pods may be killed and replaced to rollout a new release or new pods can be created to scale for the traffic. The point is, the pods in a deployment may be changing once in a while. So how do we keep track of the up-to-date IP address of the pods in the deployment? `Services` in kubernetes is there to solve this i.e to keep track of the right set of IP address of the pods at any point of time.

### Demo
#### Create a deployment
First create a kubernetes deployment with 3 pod replicas using `kubectl apply` command. Note that the pod has labels `app: hello` and `tier: backend`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  selector:
    matchLabels:
      app: hello
      tier: backend
  replicas: 3
  template:
    metadata:
      labels:
        app: hello
        tier: backend
    spec:
      containers:
        - name: hello
          image: "gcr.io/google-samples/hello-go-gke:1.0"
          ports:
            - name: http
              containerPort: 80
```
Now run, `kubectl get pods -o wide` and see the pods running
```sh
NAME                  READY   STATUS    RESTARTS   AGE   IP            NODE                          NOMINATED NODE   READINESS GATES
hello-d6b44f4-9dwht   1/1     Running   0          43m   10.244.0.22   test-kind-k8s-control-plane   <none>           <none>
hello-d6b44f4-gcmbf   1/1     Running   0          43m   10.244.0.21   test-kind-k8s-control-plane   <none>           <none>
hello-d6b44f4-v24wj   1/1     Running   0          43m   10.244.0.20   test-kind-k8s-control-plane   <none>           <none>
```
#### Create a service for the deployment
Its time to create a service for the deployment to reach the server inside the pod. Note that the pod labels `app: hello` and `tier: backend` are mentioned under the `.spec.selector` field. This tells kubernetes to look for pods with the labels as mentioned in the yaml. Also, the service mentions the port to reach itself via the`port` field and the port to containers via the `targetPort` field.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello
spec:
  selector:
    app: hello
    tier: backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

If the `targetPort` is not mentioned, by default it is considered the same as that of `port` field value.
{: .notice}

Run the command, `kubectl get service` to get the available kubernetes services.
```sh
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
hello        ClusterIP   10.102.56.197   <none>        80/TCP    25m
```
#### Endpoints
The `endpoints` is a kubernetes resource that maintains the IP addressses of the pods for the services to send the traffic to. As soon as we create the above service, Kubernetes creates a new endpoint `hello`. Run the command, `kubectl get endpoints` to get the endpoints.

```sh
NAME         ENDPOINTS                                      AGE
hello        10.244.0.20:80,10.244.0.21:80,10.244.0.22:80   26m
```
You might notice that the ip addresses(10.244.0.20, 10.244.0.21, 10.244.0.22) in the endpoint output are that of the pods created by the above deployment.

Now lets, kill a pod and see what happens.

```sh
> kubectl delete pod hello-d6b44f4-gcmbf
pod "hello-d6b44f4-gcmbf" deleted

> kubectl get pods -o wide                                                                             
NAME                  READY   STATUS    RESTARTS   AGE   IP            NODE                          NOMINATED NODE   READINESS GATES
hello-d6b44f4-9dwht   1/1     Running   0          47m   10.244.0.22   test-kind-k8s-control-plane   <none>           <none>
hello-d6b44f4-smzx7   1/1     Running   0          43s   10.244.0.23   test-kind-k8s-control-plane   <none>           <none>
hello-d6b44f4-v24wj   1/1     Running   0          47m   10.244.0.20   test-kind-k8s-control-plane   <none>           <none>
```

Since the deployment yaml specifies the pod replica count as 3, a new pod is created by Kubernetes. That is why the new pod `hello-d6b44f4-smzx7` is running to compensate the deleted pod. If you see the endpoint object, it will automatically be updated by adding the new pods IP address `10.244.0.23` and removing the deleted pod IP address `10.244.0.21`.

```sh
kubectl get endpoints                                                                                   
NAME         ENDPOINTS                                      AGE
hello        10.244.0.20:80,10.244.0.22:80,10.244.0.23:80   51m
```

### Accessing the service
Create a new pod `test` to see how we can reach our `hello` service.

```sh
> kubectl create deployment test-pod --image=k8s.gcr.io/echoserver:1.4
deployment.apps/test-pod created

> kubectl get pods --selector app=test-pod
NAME                        READY   STATUS    RESTARTS   AGE
test-pod-6f4b5b47d6-dmmh2   1/1     Running   0          2m25s

> kubectl exec -it test-pod-6f4b5b47d6-dmmh2 -- /bin/bash
```

The above commands create a `test-pod` and enters the pod so we can run commands inside it. 

There are multiple ways to reach a service from a pod.
#### 1. DNS 
With DNS enabled in your cluster, a pod can call a service with its name. For the service `hello` in our example, it can be reached via url `http://hello` from the same namespace. If the pod runs in a namespace different from that of the service, it is accessed like `http://hello.<namespace>`. In our example, we defined the service in the `default` namespace. Lets execute the commands within the `test-pod` to reach the `hello` service.
```sh
root@test-pod-6f4b5b47d6-dmmh2:/# curl http://hello
{"message":"Hello"}
root@test-pod-6f4b5b47d6-dmmh2:/# curl http://hello.default
{"message":"Hello"}
```

#### 2. Service IP address
The kubectl get svc command outputs the service IP address. From within the cluster, we can access with the service IP address. For the service `hello`, the IP address is `10.102.56.197`. See that the service is accessible with its IP address when using cURL command.

```sh
curl 10.102.56.197
{"message":"Hello"}
```
#### 3. Environment variables
By default, Kuberenetes exposes the service info with environment variables. Inside the `test-pod`, run the `printenv` command and see the service details populated.

```sh
root@test-pod-6f4b5b47d6-dmmh2:/# printenv | grep HELLO
HELLO_PORT=tcp://10.102.56.197:80
HELLO_PORT_80_TCP=tcp://10.102.56.197:80
HELLO_SERVICE_PORT=80
HELLO_PORT_80_TCP_ADDR=10.102.56.197
HELLO_PORT_80_TCP_PORT=80
HELLO_PORT_80_TCP_PROTO=tcp
HELLO_SERVICE_HOST=10.102.56.197

root@test-pod-6f4b5b47d6-dmmh2:/# curl ${HELLO_SERVICE_HOST}:${HELLO_SERVICE_PORT}
{"message":"Hello"}
```

### Types of service
A Kuberenetes services can be exposed in the following ways. You can specify the service type with the `type` field.
#### 1. ClusterIP
If no type is provided, Kubernetes creates a service with cluster IP address by default. The example used above has only clusterIP. As the name suggests the service is accessible only within the cluster. This is mainly used for services that we don't want to expose outside the cluster or when we are trying to access the service using ingress. When ingress controller is used, it creates a loadbalancer, and the loadbalancer access the service with clusterIP. We can access a ClusterIP service using the above 3 ways i.e DNS, env variable, ClusterIP.
#### 2. NodePort
When the service type is `NodePort`, the service is accessible with the cluster node IP addresses in addition to the above 3 methods. Lets convert the service type from `ClusterIP` to `NodePort` by applying the below yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello
spec:
  type: NodePort
  selector:
    app: hello
    tier: backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: http
    nodePort: 30001
```
The yaml, specifies the type of service as `NodePort` and mentiones the nodePort as `30001`. If the `nodePort` field is not specified, a random port gets alloted. 

```sh
> kubectl get nodes -o wide
NAME                          STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                     KERNEL-VERSION      CONTAINER-RUNTIME
test-kind-k8s-control-plane   Ready    master   2m44s   v1.19.1   172.19.0.4    <none>        Ubuntu Groovy Gorilla (development branch)   4.19.121-linuxkit   containerd://1.4.0
test-kind-k8s-worker          Ready    <none>   2m17s   v1.19.1   172.19.0.2    <none>        Ubuntu Groovy Gorilla (development branch)   4.19.121-linuxkit   containerd://1.4.0
test-kind-k8s-worker2         Ready    <none>   2m17s   v1.19.1   172.19.0.3    <none>        Ubuntu Groovy Gorilla (development branch)   4.19.121-linuxkit   containerd://1.4.0
```

```sh
> kubectl exec -it test-pod-6f4b5b47d6-dmmh2 -- /bin/bash
root@test-pod-6f4b5b47d6-dmmh2:/# curl 172.19.0.2:30001
{"message":"Hello"}
root@test-pod-6f4b5b47d6-dmmh2:/# curl 172.19.0.3:30001
{"message":"Hello"}
root@test-pod-6f4b5b47d6-dmmh2:/# curl 172.19.0.4:30001
{"message":"Hello"}
```

#### 3. LoadBalancer
If the service type is mentioned as `LoadBalancer`, then the cloud provider will create a loadbalancer. This serviceType is not mostly used as a new loadbalancer is created for each service type `LoadBalancer`. This will be costly and also difficult to maintain the multiple load balancers, in case many services are of type loadbalancer.
#### 4. ExternalName
This service type is used when we want to access a website outside the kubernetes cluster. 

```yaml
apiVersion: v1
kind: Service
metadata:
  name: google-svc
spec:
  type: ExternalName
  externalName: google.com
```

As you see, this service is a mapping to google.com website which is accessible within the cluster as follows:
```sh
> kubectl exec -it test-pod-6f4b5b47d6-dmmh2 -- /bin/bash
root@test-pod-6f4b5b47d6-dmmh2:/# curl -k google.com http://github
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```
### References:
- [https://kubernetes.io/docs/concepts/services-networking/service/](https://kubernetes.io/docs/concepts/services-networking/service/)
-
