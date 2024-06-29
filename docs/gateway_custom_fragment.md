# Overview
 Create a gateway with customer fragment and zero knowledge keys.

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

> You can  add custer_name and cluster URL to respect your production naming convention
```
docker run -d -p 8000:8000 -p 8200:8200 -p 18888:18888 -p 8080:8080 -p 8081:8081 \
-p 5696:5696 -e ADMIN_ACCESS_ID="your-access-id" \
-e ADMIN_ACCESS_KEY="matching-access-key" -e CLUSTER_NAME="meaningful-cluster-name" \
-e INITIAL_DISPLAY_NAME="display-name" -e CLUSTER_URL="https://<GW_URL>" \
--name akeyless-dock-gw akeyless/base:latest-akeyless
```
- Encypt your gateway configuration using `CONFIG_PROTECTION_KEY_NAME` argument
 ```
 docker run -d -p 8000:8000 ... --name akeyless-dock-gw akeyless/base:latest-akeyless -e CONFIG_PROTECTION_KEY_NAME="My-Encryption-Key"
  ```

 **1) Login to the gateway config manager**

 URL: http://localhost:8000
 - Check the Zero knowledge encryption enabled (CF) 
  
> <img src="https://github.com/brokedba/Akeyless_demo/assets/29458929/bb9bb988-bbe9-4527-b342-3f5595b0b90a" width="700" height="300" />


**2) Login to the Gateway Console**


URL: http://localhost:18888

<img src="https://github.com/brokedba/Akeyless_demo/assets/29458929/2275f550-a576-4a80-987f-c909fddc1480" width="500" height="300" /> 

**3) Create a DFC encryption key using customer fragment**

> - Name: DFCEncryptionKeyCF
> - path: /MyVault/Encryption_keys
> - Description: Zero Knowledge based DFC Key (Customer fragment)
 <img src="https://github.com/brokedba/Akeyless_demo/assets/29458929/e02386e1-c9f6-4c71-a5dd-60a8b4bddfed" width="500" height="300" />

- **Complains about Permissions**
> <img src="https://github.com/brokedba/Akeyless_demo/assets/29458929/a0ea0c7f-8d00-406d-bda7-062f6254798a" width="450" height="380" />


**Stop and Remove the Existing Docker Container:**
```
docker stop akeyless-dock-gw
docker rm akeyless-dock-gw
```
- Run the Updated Docker Command including admin permissions to 2 clients (api/email):

```
docker run -d \
  -p 8000:8000 \
  -p 8200:8200 \
  -p 18888:18888 \
  -p 8080:8080 \
  -p 8081:8081 \
  -p 5696:5696 \
  -v ./customer_fragments.json:/home/akeyless/.akeyless/customer_fragments.json \
  -e ADMIN_ACCESS_ID="p-xxxxxx" \
  -e ADMIN_ACCESS_KEY="62Hu...xxx....qlg=" \
  -e ALLOWED_ACCESS_PERMISSIONS='[{"name":"Administrators","access_id":"p-6ax017dqgi6sam","permissions":["admin"]},{"name":"Administrators","access_id":"p-94lrdxzjhg1fem","permissions":["admin"]}]' \
  --name akeyless-dock-gw \
  akeyless/base:latest-akeyless
  ```
- the ZK:DFC is now created 

> <img src="https://github.com/brokedba/Akeyless_demo/assets/29458929/7a9ca7c5-b3cf-40c8-9877-1c06faf935c5" width="250" height="50" />


> <img src="https://github.com/brokedba/Akeyless_demo/assets/29458929/37c4b230-bb7f-4778-89ee-bd19d3b3482c" width="250" height="50" />
 
**Create Zero Knowledg based secret**

- From the Gateway console:
 > <img src="https://github.com/brokedba/Akeyless_demo/assets/29458929/f23d88ff-cb05-4e42-a45d-b1fbae4f9fd1" width="650" height="450" />
- or via CLI
 ```
akeyless create-dfc-key --name MyKeyWithMyCF --alg AES256GCM -f <customer-fragment-id>
 ``` 
<img src="https://github.com/brokedba/Akeyless_demo/assets/29458929/09c00216-59de-48db-8889-8a8c40f3fd50" width="300" height="200" />

# **K8s Gateway**

- Create a base64 encoding of the customer fragment:
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
