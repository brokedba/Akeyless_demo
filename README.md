# Akeyless_demo


Overview
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
      "id": "cf-p6lta4na60yzkmwrjvtm",
      "value": "naTTMPLA558l934TZCnK6E0jItSaeVbtp+1MtGnCKsLHOrxi0YW1E6K88EUTwWCVMyt4VTDjmj7D/UssLlGCeA==",
      "description": "MyFirstCF",
      "name": "CustomerFragmentDemo"
    }
  ]
}
```
**Create a Gateway**
```
docker run -d -p 8000:8000 -p 8200:8200 -p 18888:18888 -p 8080:8080 -p 5696:5696 -v ./customer_fragments.json:/home/akeyless/.akeyless/customer_fragments.json -e ADMIN_ACCESS_ID="sam.gabrail@tekanaid.com" -e ADMIN_ACCESS_KEY="" --name akeyless-codespaces-gw akeyless/base:latest-akeyless
```
**For K8s Gateway**
Create a base64 encoding of the customer fragment:
```
base64 -w 0 customer_fragments.json
```
and add that to the customer-fragments-secret.yaml file.
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
