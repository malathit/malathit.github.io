---
title: Godaddy external secrets with localstack
excerpt: A tutorial on fetching the secrets stored in the AWS secrets manager using
  the GoDaddy external secrets and localstack.
category:
- devops
- kubernetes
tags:
- Kubernetes secrets
- Godaddy's external secret
- localstack
---

Kubernetes secrets use `Base64` encoding to encode the secrets. It is simple to decode a base64 encoded, but from a security point of view, storing the secret in the git repository in the base64 encoded format is not good enough. In this tutorial, you will see about storing the secrets in AWS secrets manager and use external secrets by GoDaddy to create that as a Kubernetes secret.

### Pre-requisites
- [Localstack](https://github.com/localstack/localstack)(to emulate AWS services secrets manager and STS in local)
- A [KinD](https://kind.sigs.k8s.io/) cluster
- [Helm](https://helm.sh/) (installed in the KinD cluster)

### 1. Create kind cluster
To create a kind cluster, follow this tutorial [here]({% post_url 2020-12-07-kind-kubernetes-clusters %})
### 2. Install helm
You can install Helm in your laptop with 2 different methods
- Through binaries created for your OS using [this][helm-binary-install] link.
- Through the install script as mentioned [here][helm-script-install]

### 3. Install localstack
For the purpose of this tutorial, we will be using `localsatck`, a mock implementation of many AWS services. Lets install [localstack][localstack-helm-install] in the cluster using helm by running the below commands.
```sh
> helm repo add localstack-repo http://helm.localstack.cloud
> helm upgrade --install localstack localstack-repo/localstack

> kubectl get pods
NAME                                                            READY   STATUS    RESTARTS   AGE
localstack-859d64b555-hgzxh                                     1/1     Running   0          4m48s
```
If you want to know more about localstack check [here](https://github.com/localstack/localstack).
### 4. Install external secrets
Use helm to install GoDaddy's external secrets
```sh
> helm repo add external-secrets https://external-secrets.github.io/kubernetes-external-secrets/
> helm install external-secrets external-secrets/kubernetes-external-secrets

> kubectl get pods
NAME                                                            READY   STATUS    RESTARTS   AGE
external-secrets-kubernetes-external-secrets-7c565576fb-5x6tz   1/1     Running   0          3m22s
```

Now, edit the external secrets deployment to use localstack instead of AWS
```sh
kubectl edit deployment/external-secrets-kubernetes-external-secrets
```

In the deployment yaml, add the below environmental variables,
```yaml
 - name: LOCALSTACK
   value: "1"
 - name: LOCALSTACK_STS_URL
   value: http://localstack:4566
 - name: LOCALSTACK_SM_URL
   value: http://localstack:4566
 - name: AWS_ACCESS_KEY_ID
   value: some-access-key
 - name: AWS_SECRET_ACCESS_KEY
   value: some-secret-key
```
The above variables instruct external secrets to refer to localstack services(sts and Secrets manager) running in our cluster instead of aws services. You can refer to the external secrets AWS config [here](https://github.com/external-secrets/kubernetes-external-secrets/blob/master/config/aws-config.js) to understand why we are setting these variables.

The above environment variables also includes some random access key and secret key to connect to localstack. This is needed by the external secrets pod to connect to localstack. Instead of the access & secret keys, in the AWS environment, you can use a node IAM role or service accounts as mentioned [here](https://github.com/external-secrets/kubernetes-external-secrets#aws-based-backends).
### 5. Create secret in localstack secret manager
Now lets create a secret in secrets manager and create an IAM role to access the secret. The IAM role json should be like this,
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetResourcePolicy",
        "secretsmanager:GetSecretValue",
        "secretsmanager:DescribeSecret",
        "secretsmanager:ListSecretVersionIds"
      ],
      "Resource": [
        *
      ]
    }
  ]
}
```
The above json specifies the permissions to read all the secrets in secrets manager. Store this in a file `iam-role.json` inside the container.

```sh
> kubectl exec -it localstack-859d64b555-hgzxh -- bash

bash-5.0# awslocal secretsmanager create-secret --name test-secret --secret-string "hello-1234" --region us-east-1
{
    "ARN": "arn:aws:secretsmanager:us-east-1:000000000000:secret:test-secret-Walhb",
    "Name": "test-secret",
    "VersionId": "81c8df61-dfa1-4518-93b1-8d90c23bd0f2"
}

bash-5.0# awslocal iam create-role --role-name Test-Secret-Role --assume-role-policy-document file://iam-role.json
{
    "Role": {
        "Path": "/",
        "RoleName": "Test-Secret-Role",
        "RoleId": "kvwu8ik8ivxgd6lzxnk9",
        "Arn": "arn:aws:iam::000000000000:role/Test-Secret-Role",
        "CreateDate": "2020-12-29T13:28:11.625Z",
        "AssumeRolePolicyDocument": "<role json>",
        "MaxSessionDuration": 3600
    }
}
```
### 6. Create external secret
```yaml
apiVersion: 'kubernetes-client.io/v1'
kind: ExternalSecret
metadata:
  name: test-secret
spec:
  backendType: secretsManager
  roleArn: arn:aws:iam::000000000000:role/Test-Secret-Role
  region: us-east-1
  data:
    - key: test-secret
      name: password
```

The yaml specifies the backend type as secrets manager. The region we created the secret `us-east-1` is mentioned. The role ARN of the created IAM role is mentioned in step 5. Then the secret is described under the `.spec.data` key. The `key` field specifies the name of the secret we created in localstack in step 5. The field `name` instructs external secrets to create a kubernetes secrets with the key `password` .

Now, apply the external secrets yaml and see a new secrets created.

```sh
> kubectl get externalsecrets                                                
NAME          LAST SYNC   STATUS    AGE
test-secret   0s          SUCCESS   21s

> kubectl get secrets test-secret -o yaml                                                      
apiVersion: v1
data:
  password: aGVsbG8tMTIzNA==
kind: Secret
metadata:
  creationTimestamp: "2020-12-30T01:39:36Z"
  name: test-secret
  namespace: default
  ownerReferences:
  - apiVersion: kubernetes-client.io/v1
    controller: true
    kind: ExternalSecret
    name: test-secret
    uid: 6bbc0343-75be-4aae-b6c4-d63d4d790fb7
  resourceVersion: "30419"
  selfLink: /api/v1/namespaces/default/secrets/test-secret
  uid: 6d032e09-1375-4ef2-9ae5-8545b4b5af76
type: Opaque
```

Note down the base64 encoded value of the secret `password`. Lets verify that the secret value is the same we created the secret in secrets manager.

```sh
echo aGVsbG8tMTIzNA== | base64 --decode
hello-1234
```

That's it. Lets test external secrets in your local kind cluster without having to use AWS services.

[helm-binary-install]: https://helm.sh/docs/intro/install/#from-the-binary-releases
[helm-scrip-install]: https://helm.sh/docs/intro/install/#from-script
[localstack-helm-install]: https://github.com/localstack/localstack#using-helm
