---
title: kind Kubernetes clusters
excerpt: A tutorial on creating a multi-node 'kind' Kubernetes cluster in local for
  testing & development purposes
category: development
---

For testing and local development, we can't rely on the Kubernetes cluster in the cloud. For one reason, it is costly. One another reason is the need of fast internet connection. So, local Kubernetes cluster comes to the rescue. In this post, lets see how to get started with a local Kubernetes cluster. 

### Pre-requisities
- [Docker](https://docs.docker.com/engine/install/)
- [Kind] [kind installation]
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

### Creating a kind cluster
KinD stands for **K**ubernetes **in** **D**ocker. So kind creates nodes with docker containers.
{: .notice--info}

First and foremost, we need a config file, that will explain `kind` how to create a cluster. This is a sample config file
```yaml
# three node (two workers) cluster config
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
- role: worker
- role: worker
```

I copied this yaml, from the kind website. This yaml, specifies that we need a 3 node cluster, with 1 control plane node and 2 worker nodes. It exposes port 80 & 443  of the control-plane node to localhost. It also add the label `ingress-ready=true` to the control plane node with the `node-labels`. 

Now, lets create the cluster with the below command,

```sh
kind create cluster --config kind-example-config.yaml --name test-kind-k8s
```
The above command, creates a 3 node Kubernetes cluster as specified in the config file(having the above yaml content) with name test-kind-k8s. Once the cluster is up and running, you will see the output like this.

![cluster creation output](/images/kindcluster/cluster-creation.png){: .width-half}

As specified in the output, when I ran the command at the end, I get the below output
```sh 
kubectl cluster-info --context kind-test-kind-k8s
```
```text
Kubernetes master is running at https://127.0.0.1:58975
KubeDNS is running at https://127.0.0.1:58975/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```
### Enabling ingress
Ingress is a Kubernetes resource that specifies the routing rules to reach the services running inside the K8s cluster externally. An ingress controller, takes care of fulfilling the ingress rules defined in the cluster, usually via a LoadBalancer. There are various open source ingress controllers available. We are going to install `nginx` ingress controller for our kind cluster. Lets install the nginx ingress controller by running the below command,

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml
```

If you go to the url specified in the above command, you will see a set of yamls. A closer look at the yaml, says the ingress controller deployment runs in node with the labels `ingress-ready=true`. If you remember, the label is present in our control-plane node. So this implies the controller deployment runs in our control-plane node. Also, the ingress controller listens on port 80 & 443 in the control-plane node. The ports 80 & 443 are already mapped to localhost as mentioned in the config file.
{: .notice--info}

The nginx controller installation is complete, once the deployment is up and running in the `ingress-nginx` namespace.
### Deploy a sample application 
Run these kubectl commands to deploy the hello-world service.
```sh
kubectl create deployment hello-world --image=k8s.gcr.io/echoserver:1.4
kubectl expose deployment hello-world --port=8080
```

Enable ingress to reach the hello-world service by applying the below yaml with kubectl command.

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: kind-ingress
spec:
  rules:
  - http:
      paths:
      - path: /hello
        backend:
          serviceName: hello-world
          servicePort: 8080
```

Now if you go to [http://localhost/hello](http://localhost/hello), you will get the response from the hello-world server.

This way we can add any number of services to the cluster and to reach the service via the ingress, we need to define a ingress rule as in the above yaml.
### Advantages of a kind cluster
- A kind cluster is free. However, the remote i.e cloud based k8s providers such as EKS, will charge on a hourly basis
- Kind cluster saves developer time, by developing & testing his/her changes locally in their laptop. 
- kind cluster provides an isolated environment for a developer and also gives full admin access in their local cluster to experiment things.

[kind installation]: https://kind.sigs.k8s.io/docs/user/quick-start/#installation
