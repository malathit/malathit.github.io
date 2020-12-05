---
title:  "Hello kube"
date:   2019-05-10 10:49:41 +0530
description: Getting started with kubernetes to deploy an app
tags: [ Kubernetes, k8s, Kubernetes example ]
category: devops
excerpt: "Kubernetes 101 tutorial with an example"
---
Kubernetes is one of the highly demanded skill in the software deployment process right now. Let us understand Kubernetes, mystery resolved with a simple demo.

The complete code used here is available in the github repo. Clone the repo using the command:

> git clone [https://github.com/malathit90/hello-kube.git](https://github.com/malathit90/hello-kube.git)

Disclaimer : This article assumes you have a basic knowledge of kubernetes concepts like pods, deployments, service, configMaps, secrets, volumes. Please refer to [Kubernetes](http://kubernetes.io) website if you are not familiar.

### About the project

The github repository mentioned above basically deploys a [nodejs rest service](https://github.com/malathit90/hello-world-node) supported by mariadb service.

----------

### 1. DB Secret

The yaml file [config.yaml](https://github.com/malathit90/hello-kube/blob/master/config.yaml) has the secret data such as database credentials.

{% gist 924737d44d93c2f6250caaf9340d9aeb %}

In the above snippet as you see, all the value of the secrets username, password, database are _base64_ encoded. They are decoded by kubernetes when other deployments use the secrets.

Create the secret using the below command:

> kubectl apply -f config.yaml

### 2. Init sql as ConfigMap

There is a _scripts_ directory under db folder. It has the _init.sql_ file that creates 2 tables and populate the tables with values from data files _states.csv, districts.csv_. Create a configMap using these files as follows:

> kubectl create configmap maria-init-config — from-file=db/scripts

### 3. Maria db service

{% gist 687d4f867e2e01417ff199032184334c %}

The above snippet creates a service named maria-db by selecting pods with label **_app:maria_** and exposes the port 3306 and redirects the traffic to the pods listening at the target port 3306.

{% gist 0aff6c13cce070c1d8d39547257817e2 %}

There are many things done in the database deployment snippet above. Let us see them one by one.

**a)** The image mariadb:10 is used to bring up the database container.

**b)** The value of the environmental variables such as MYSQL_USERNAME, MYSQL_PASSWORD and MYSQL_DATABASE are referenced from the secret **_db-secret_** created already.

**c)** The init.sql script is run during the initialisation of the container by hosting the configMap ‘**maria-init-config**’ as a volume mount ‘/docker-entrypoint-initdb.d/’. The 3 files init.sql, states.csv, districts.csv are available under the directory /docker-entrypoint-initdb.d/ inside the maria pod.

**d)** The pod has the label **_app:maria_** to mark the connection with maria-db service.

**e)** The pod hosts the service in port 3306.

Run the below command to create the maria db service and deployment:

> kubectl apply -f db/db.yaml

### 4. Rest service
{% gist 21c2da5746dbf618569151b6a8b16538 %}

The above snippet creates a service named rest-service similar to maria-db service above.

{% gist 6642bfcd0dfbc56b045f3e56bcc0dfd7 %}

Note the below set of things while creating the rest-deployment:

**a)** The above snippet creates a Kubernetes deployment with container malathit90/ind-geo:v2 and as the nodejs app runs in port 3000, containerPort is specified as 3000.

**b)** The nodejs app is given the database connection config using the environment variables.

**c)** See that the env variable DB_HOST specifies the **mariadb** service that is created already. The kubeDNS provides way to access services using their name within the cluster.

**d)** Do note that the container is given the label **_app:node-rest_** which maps the service with the deployment, which in turn maps with the pods.

Run the command to create the rest service and deployment:

> kubectl apply -f api/app.yaml

### 5. Expose the rest service with port forwarding

This step is not required if you are not running minikube. In minikube, the services are not accessible via localhost without port forwarding.

Run,

> kubectl get pods

You will get a list of pods. Select the name of the pod that starts with node-rest and then port forward 3000 to access the rest service using localhost. The below command will do the same.

> kubectl port-forward <node-rest-xxxxxxxx> 3000:3000

Use your pod name in the above commad within <> braces. Open the browser and type,

> http://localhost:3000/state

You should see the list of states available in India in json format.

Happy Kubing!!!