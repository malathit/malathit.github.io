---
title: Kubernetes - Tips & Tricks
date: '2020-11-14 21:13:04'
tags:
- Kubernetes
- k8s
- devops
category: devops
excerpt: "Things to note and remember when working with Kubernetes - set of beginner friendly tips"
---
Kubernetes is the orchestration framework for containers. It has greatly simplified and automated the DevOps world. There are many advancements and new tools created every day. In this post, I am going to explain the tips and tricks, I came across, that can be used to get the maximum benefit out of k8s(Kubernetes).

### Secrets
We need to have store the secrets somewhere, either in the cloud or in your GitHub repo or in the deployment machine(this may be less secure). With Kubernetes, secrets are encoded using `base64`. However, it is not always a good idea to store the secret yamls in your git repo with base64 encoding. This is because it is very easy to decode base64 encoding. To solve this, there are open source alternatives such as Shopify's [ejson](https://github.com/Shopify/ejson), GoDaddy's [external secrets](https://github.com/godaddy/kubernetes-external-secrets). I found the external secrets by GoDaddy useful, as at work, we used cloud providers for storing the secrets. Ejson is particularly useful when we want to store the secrets in a git repo and it encodes and decodes secrets using public/private key pair.

### Namespaces
In Kubernetes, we have namespaces for separation of concerns, just as it does in the Linux world. We can use namespace to provide access for different types of users or to run the different environments of the software, say dev or staging without the need to have different k8s clusters. We can set memory or CPU limits on the namespace using [`resource quotas`](https://kubernetes.io/docs/concepts/policy/resource-quotas/).  If you are not using namespaces, it is a must to utilize the benefits of Kubernetes.

### UI to interact with K8s
- Kubernetes come in-built with its web dashboard. It has all the necessary features to edit yamls, view pod logs, and see what is going on with your cluster. But exposing the Kubernetes dashboard may be a security risk. 

- [K9s](https://k9scli.io/) is a terminal-based UI to interact with k8s clusters. It simplifies so many things we try to do with kubectl command. All that we need to do is press a single key to switch namespaces, type a word to view pods, services, hpa, etc. Once you get to know the comfort of K9s, you won't look forward to typing your commands with kubectl.  
- There are other UIs such as [lens](https://k8slens.dev/), which I haven't used so far. I will try and update this post if possible.

### Kubernetes context 
Many of us might have worked with more than one Kubernetes cluster. Let's say, I have a dev and staging cluster and I work on both the clusters. `kubeconfig` file comes to the rescue. The kubeconfig file has the information necessary for kubectl to connect to a cluster. All that I needed was to maintain 2 kubeconfig files, one for the dev cluster and another for staging cluster. More about Kubernetes context can be found [here](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/). To get the available contexts in a machine we can run
```
kubectl get contexts
```
[`Kubectx`](https://github.com/ahmetb/kubectx) is a tool that has inbuilt support for switching contexts. With `kubens`, a tool bundled with kubectx, it is easy to select the namespace and there is no need to add the `--namespace <namespace-name>` on kubectl commands.

### Kubectl in debug mode
The Kubernetes CLI `kubectl` can be run in debug mode as mentioned [here](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#kubectl-output-verbosity-and-debugging). The debug logs can help figure out what exactly is kubectl trying to execute the command. This is particularly useful when we are trying to figure out the calls to the API server behind the scenes.
```sh
kubectl get pods --v=8
```

### kustomize vs Helm
You might have wondered, how you can manage the yamls of your software, for your dev, test, prod environments. The point is, there will be subtle differences across the environments and you can't afford to maintain different copies of YAML for each environment. [`kustomize`](https://github.com/kubernetes-sigs/kustomize) or [`helm`](https://helm.sh/), both helps to maintain Kubernetes yamls for different environments. Helm is more like a templating tool, you maintain different config files containing the template variable values, for each of the environment. Kustomize will maintain a base copy and then have overlays that contain the difference mentioned as YAML. Both are used fairly by users.
