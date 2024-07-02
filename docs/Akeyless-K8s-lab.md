# Overview
Kubernetes Secrets with Akeyless
## The Security Limitations of Kubernetes Secrets
### K8 secrets Misconception: Base64 vs Encryption

Base64 encoding is not **Encryption**! It's just a reversible encoding method. Anyone with access to the encoded data can decode it.

For example:
```bash
kubectl create secret generic my-secret --from-literal=password='s3cr3t'
```
- You can easily decode the Secret as follows:
```bash
kubectl get secret my-secret -o jsonpath="{.data.password}" | base64 -d && echo
# Outputs 's3cr3t'
```
# K8 Authentication method (K8s configuration)
![image](https://github.com/brokedba/Akeyless_demo/assets/29458929/bf9dfb30-ab69-443b-b543-dbd1616a0a9e)

**AUTH METHODS**
- Akeyless supports multiple options to authenticate your K8s cluster with Akeyless platform:
1. [Native Kubernetes (K8s) Auth](https://docs.akeyless.io/docs/kubernetes-auth)
2. Other options Include
   - [Universal Identity](https://docs.akeyless.io/docs/auth-meth-k8s#:~:text=Kubernetes%20(K8s)%20Auth-,Universal%20Identity%20(UID),-Note%3A%20Not%20supported) (UID)
   - [API Key](https://docs.akeyless.io/docs/api-key)
   - Cloud Identity: [Azure AD](https://docs.akeyless.io/docs/azure-ad) | [AWS IAM](https://docs.akeyless.io/docs/aws-iam) | [GCP Auth](https://docs.akeyless.io/docs/gcp-auth-method). 

# Native Kubernetes (K8s) Authentication

Akeyless offers various strategies for Native Kubernetes authentication to securely manage access to secrets. Each strategy uses Kubernetes JWTs for authentication and has its own purposes, pros, and cons. 
Here is a little summary
<table>
  <thead style="background-color: #f2f2f2;">
    <tr>
      <th>Strategy</th>
      <th>Purpose</th>
      <th>Pros</th>
      <th>Cons</th>
      <th>Use Case</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>Akeyless Gateway ServiceAccount<b></td>
      <td>Use the Akeyless Gateway ServiceAccount for authentication</td>
      <td>Simplicity, Centralized Management</td>
      <td>Single Point of Failure, Limited Flexibility</td>
      <td>Prioritize simplicity and centralized management</td>
    </tr>
    <tr>
      <td><b>Dedicated ServiceAccount<b></td>
      <td>Use a dedicated ServiceAccount for specific applications/namespaces</td>
      <td>Fine-Grained Access Control, Isolation</td>
      <td>Management Overhead, Complexity</td>
      <td>Require strict access controls and isolation</td>
    </tr>
    <tr>
      <td><b>Client Certificate<b></td>
      <td>Use client certificates for authentication</td>
      <td>Security (mTLS), Non-Repudiation</td>
      <td>Complex Setup, Maintenance</td>
      <td>Require enhanced security through mTLS and can manage certificates efficiently</td>
    </tr>
  </tbody>
</table>

**CONCLUSION**

- The **Dedicated ServiceAccount** strategy represents a sweet spot for Kubernetes authentication with Akeyless.
- It provides the best balance between high isolation, efficiency, and managable overhead.

## Importance of gateway TLS encryption 
It is strongly recommended to use **Akeyless Gateway with TLS** to ensure all traffic is encrypted at transit. 
Failing to do so will cause errors in any API call requiring GTW-URL. Make sure it is set up before you follow the next steps.
- example:
```bash
docker run -d -p 8000:8000 -p 8200:8200 -p 18888:18888 -p 8080:8080 -p 8081:8081 -p 5696:5696 -e ADMIN_ACCESS_ID="your-access-id" -e ADMIN_ACCESS_KEY="matching-access-key" -e ENABLE_TLS="true" -e ENABLE_TLS_CONFIGURE="true" -e ENABLE_TLS_CURL="true" -e ENABLE_TLS_HVP="true" -e MIN_TLS_VERSION="TLSv1.2" -v $PWD/cert.crt:/home/akeyless/.akeyless/akeyless-api-cert.crt -v $PWD/key.pem:/home/akeyless/.akeyless/akeyless-api-cert.key --name akeyless-gw akeyless/base:latest-akeyless
```
## Using Akeyless with Kubernetes Secrets

Let's explore integrating Akeyless with Kubernetes to manage secrets securely using the Akeyless agent injector. You can find more [details in the docs.](https://docs.akeyless.io/docs/how-to-provision-secret-to-your-k8s)

### Prerequisites

- Kubernetes cluster v1.19 and above
- Helm and kubectl installed
- Akeyless account
- [K8s Authentication Method Configured](https://docs.akeyless.io/docs/dedicated-k8s-auth-service-accounts)

### Step 1: Install Akeyless Secrets Injector

First, install the Akeyless Secrets Injector in your Kubernetes cluster. The injector will automatically inject secrets into your Kubernetes pods at runtime.

```bash
helm repo add akeyless https://akeylesslabs.github.io/helm-charts
helm repo update
helm upgrade --install aks akeyless/akeyless-secrets-injection --namespace akeyless -f values.yaml
```

Notice the `values.yaml` file contains the following environment variables:
```yaml
env:
  AKEYLESS_URL: "https://vault.akeyless.io"
  AKEYLESS_ACCESS_ID: "p-ukwx42wczc5skm" 
  AKEYLESS_ACCESS_TYPE: "k8s"
  AKEYLESS_API_GW_URL: "https://b46b-24-150-170-114.ngrok-free.app"
  AKEYLESS_K8S_AUTH_CONF_NAME: "my-k8s-auth-method"
```

- `AKEYLESS_ACCESS_ID` is the Access ID of the Auth Method with access to the secret.
- `AKEYLESS_ACCESS_TYPE` is k8s since we're using the Kubernetes authentication method.
- `AKEYLESS_K8S_AUTH_CONF_NAME` is set to our Gateway Kubernetes Auth name. Relevant only for Access type of k8s.
- `AKEYLESS_API_GW_URL` is set with the URL of my Akeyless Gateway on port 8080. Since my gateway is running on my local machine, I'm using ngrok with `ngrok http 8080` to forward an external url of `https://b46b-24-150-170-114.ngrok-free.app` to `http://localhost:8080`. This allows communication between my apps running in K8s and the gateway.

#### Modes of Retrieval of Secrets in Akeyless
There are two modes to retrieve secrets with Akeyless in K8s. Environment variables and file injection. The drawback with environment variables mode is you require a restart of the pod if the secret changes. With file injection, the secret is written to a file in the pod. You can use a sidecar to continously retrieve updates to secrets in Akeyless. This is helpful for rotated and dynamic secrets.

### Step 2: Retrieve Secrets into Environment Variables

Let's now retrieve a secret from Akeyless into an environment variable. Run the following command:

```bash
kubectl apply -f app_env.yaml
```

Notice the content of this file:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-environment-variable
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-secrets
  template:
    metadata:
      labels:
        app: hello-secrets
      annotations:
        akeyless/enabled: "true"
    spec:
      containers:
      - name: alpine
        image: alpine
        command:
          - "sh"
          - "-c"
          - "echo $MY_SECRET && echo ...going to sleep... && sleep 10000"
        env:
        - name: MY_SECRET
          value: akeyless:/K8s/my_k8s_secret
```

Notice the following annotation:

- akeyless/enabled: "true": An annotation that enables the K8s Injector plugin.

Now check the logs of the pod:

Output:
```
Defaulted container "alpine" out of: alpine, akeyless-init (init)
myPassword
...going to sleep...
```

It successfully retrieved the password in Akeyless: `myPassword`

### Step 3: Retrieve Secrets into a File

Let's now retrieve a secret from Akeyless into a file. Run the following command:

```bash
kubectl apply -f app_file.yaml
```

Here are the contents of the `app_file.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-file
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-secrets-2
  template:
    metadata:
      labels:
        app: hello-secrets-2
      annotations:
        akeyless/enabled: "true"
        akeyless/inject_file: "/K8s/my_k8s_secret"
    spec:
      containers:
      - name: alpine
        image: alpine
        command:
          - "sh"
          - "-c"
          - "cat /akeyless/secrets/K8s/my_k8s_secret && echo ...going to sleep... && sleep 2000"
```

Notice the following annotations:

- akeyless/enabled: "true": An annotation that enables the K8s Injector plugin
- akeyless/inject_file: "/K8s/my_k8s_secret": An annotation specifying the path to get the secret from in Akeyless. The default location of the Akeyless secrets folder inside your pod file system is `/akeyless/secrets/`. To explicitly set a different location you can override this by adding `|location=<path>` after your secret name within the annotation as we will see in the next example.

Check the logs of this pod:

Output:

```
Defaulted container "alpine" out of: alpine, akeyless-init (init)
myPassword
```

We got the same output as before. However, let's exec into the container and look at the file where the secret was written into. Run this command:

```bash
kubectl exec -it pod/test-file-b9b6d4699-gprjf -c alpine -- sh
cat /akeyless/secrets/K8s/my_k8s_secret 
```

The content of the file will have the same password we saw earlier: `myPassword`.

This is great if our application is intended to retrieve the secret once from Akeyless, so an akeyless-init container retrieving the secret then exiting is appropriate. However, there are times where we need to continuously retrieve secrets from Akeyless. In this case, a sidecar container is needed. Let's see how to do that in the next example.

### Step 4: Retrieve Secrets into a File Continously with a Sidecar

Let's now retrieve a secret from Akeyless into a file continously with a sidecar. Run the following command:

```bash
kubectl apply -f app_file_sidecar.yaml
```

and here are the contents of the `app_file_sidecar.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-file-sidecar
spec:
  replicas: 1
  selector:
    matchLabels:
      app: file-secrets
  template:
    metadata:
      labels:
        app: file-secrets
      annotations:
        akeyless/enabled: "true"
        akeyless/inject_file: "/K8s/my_k8s_secret|location=/secrets/secretsVersion.json" 
        akeyless/side_car_enabled: "true"
        akeyless/side_car_refresh_interval: "5s"
        akeyless/side_car_versions_to_retrieve: "2"
    spec:
      containers:
      - name: alpine
        image: alpine
        command:
          - "sh"
          - "-c"
          - "while true; do [ ! -f /secrets/timestamp ] || [ /secrets/secretsVersion.json -nt /secrets/timestamp ] && touch /secrets/timestamp && cat /secrets/secretsVersion.json && echo ''; sleep 15; done"
```

Notice the following annotations:

- akeyless/enabled: "true" -> An annotation that enables the K8s Injector plugin
- akeyless/inject_file: "/K8s/my_k8s_secret|location=/secrets/secretsVersion.json" -> An annotation specifying the path to get the secret from in Akeyless. Here we're specifying where to write the secret in the pod's file system at `/secrets/secretsVersion.json`.
- akeyless/side_car_enabled: "true" -> This is what enables the sidecar container.
- akeyless/side_car_refresh_interval: "5s" -> This specifies the interval at which the sidecar will check for new versions of the secret.
- akeyless/side_car_versions_to_retrieve: "2" -> This specifies the number of versions of the secret to retrieve.

If you run `kubectl get po`, you will see that we have now two containers running for our pod:

```bash
NAME                                         READY   STATUS    RESTARTS      AGE
test-file-sidecar-759667cccc-6h8xv           2/2     Running   0             43m
```

Now check the logs of the `akeyless-sidecar` container:

```bash
kubectl logs -f po/test-file-sidecar-759667cccc-6h8xv -c akeyless-sidecar
```

Output:

```bash
2024/06/22 14:58:02 [INFO] Secret /K8s/my_k8s_secret was successfully written to: /secrets/secretsVersion.json
```

and check the logs of the `alpine` container:

```bash
kubectl logs -f po/test-file-sidecar-759667cccc-6h8xv -c alpine
```

Output:

```json
{
  "version": 1,
  "secret_value": "myPassword",
  "creation_date": 1719068280
}
```

Now update the secret in the Akeyless UI:

and notice the logs will automatically change to reflect the new secret:
```json
[
 {
  "version": 2,
  "secret_value": "myPassword2",
  "creation_date": 1719071019
 },
 {
  "version": 1,
  "secret_value": "myPassword",
  "creation_date": 1719068280
 }
]
```
Finally, we can exec into the alpine container to see the file where all the secrets live:

```bash
kubectl exec -it pod/test-file-sidecar-759667cccc-6h8xv -c alpine -- sh
cat /secrets/secretsVersion.json 
```

Output:

```json
[
 {
  "version": 2,
  "secret_value": "myPassword2",
  "creation_date": 1719071019
 },
 {
  "version": 1,
  "secret_value": "myPassword",
  "creation_date": 1719068280
 }
]
```

As we just saw, we are now able to actively retrieve new versions of the secret and write them to the file system. Now the application developers need to build some logic into their applications to read the file and use the secret with retry mechanisms in case of failures due to an expired secret. This is the direction organizations should take to reduce long-lived credentials in their environments.
