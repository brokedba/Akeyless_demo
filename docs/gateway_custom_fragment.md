# Overview
 A 10-minute demo to introduce Akeyless.

- Generate a Customer Fragment

```
akeyless gen-customer-fragment --name CustomerFragmentDemo --description MyFirstCF --json > customer_fragments.json
```
- You'll get the following output:
```
{
  "customer_fragments": [
    {
      "id": "cf-tcoknf7lh65hrxpr0q5q",
      "value": "9svDNiQz3f3pE2zCzd1V/yIAyhIE3NqZuJs+eMYttljHtJeYn6kGUdbmlVTo1kmtEIqIBrNL96GBo6+HjFXsYg==",
      "description": "Customer Fragment demo",
      "name": "CustomFragmDemo",
      "fragment_type": "standard"
    }
  ]
}
```
**Create a Gateway**
The following ports need to be open on the server only for internal network access:

| Service                      | Port |
|------------------------------|------|
| Gateway Configuration Manager| 8000 |
| Akeyless Gateway Console     | 18888|
| HVP (Hashi Vault Proxy)      | 8200 |
| Akeyless V1 REST API         | 8080 |
| Akeyless V2 REST API         | 8081 |
| KMIP Server                  | 5696 |

**Docker**
- Auth methods

**1. Email**
```
docker run -d -p 8000:8000 -p 8200:8200 -p 18888:18888 -p 8080:8080 -p 5696:5696 -v ./customer_fragments.json:/home/akeyless/.akeyless/customer_fragments.json -e ADMIN_ACCESS_ID="klouddude@gmail.com" -e ADMIN_ACCESS_KEY="" --name akeyless-demo-gw akeyless/base:latest-akeyless
```
**Note**: 
> Using your default account credentials is not recommended for production environments and can not work with MFA.

**2. API Key**
```
docker run -d -p 8000:8000 -p 8200:8200 -p 18888:18888 -p 8080:8080 -p 8081:8081 -p 5696:5696 -v ./customer_fragments.json:/home/akeyless/.akeyless/customer_fragments.json -e ADMIN_ACCESS_ID="p-xxxxxx" -e ADMIN_ACCESS_KEY="62Hu...xxx....qlg=" --name akeyless-gw akeyless/base:latest-akeyless
```

- Check the logs :
```
[root@localhost ~]# docker logs  akeyless-dock-gw
Starting up
Network connectivity check successful!   <------ ALL good
Checking for latest artifacts
download: s3://akeylessservices/environment/akeyless-api-proxy to ./akeyless-api-proxy_akeyless-api-proxy_latest
```
**Additional attributes**
You can  add custer_name and cluster URL to respect your production naming convention
```
docker run -d -p 8000:8000 -p 8200:8200 -p 18888:18888 -p 8080:8080 -p 8081:8081 \
-p 5696:5696 -e ADMIN_ACCESS_ID="your-access-id" \
-e ADMIN_ACCESS_KEY="matching-access-key" -e CLUSTER_NAME="meaningful-cluster-name" \
-e INITIAL_DISPLAY_NAME="display-name" -e CLUSTER_URL="https://<GW_URL>" \
--name akeyless-gw akeyless/base:latest-akeyless
```
- Encypt your gateway configuration
 ```
 docker run -d -p 8000:8000 ... --name akeyless-gw akeyless/base:latest-akeyless -e CONFIG_PROTECTION_KEY_NAME="My-Encryption-Key"
  ```
  

**Login to the gateway config manager**
http://localhost:8000
 gateway config mgr URL http://localhost:8000

![image](https://github.com/brokedba/Akeyless_demo/assets/29458929/bb9bb988-bbe9-4527-b342-3f5595b0b90a)

**K8s Gateway**
Create a base64 encoding of the customer fragment:
```
base64 -w 0 customer_fragments.json
```
and add that to the `customer-fragments-secret.yaml` file.
Now apply this secret to our namespace:

```
kubectl apply -f customer-fragments-secret.yaml
```

Update the values.yaml file to reference this secret:
```
customerFragmentsExistingSecret: customer-fragments-secret
```
- Install with Helm:

```
helm repo add akeyless https://akeylesslabs.github.io/helm-charts
helm repo update
helm upgrade akeyless-gw akeyless/akeyless-api-gateway -f values.yaml
``` 
