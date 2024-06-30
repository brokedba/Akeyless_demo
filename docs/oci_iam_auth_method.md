# Overview
Instead of authenticating identities itself, in most cases, Akeyless integrates with 3rd party identity providers that provide tokens of authentication.
in this example we will use Cloud IAM based identity through [OCI IAM](https://docs.akeyless.io/docs/oci-iam).

# OCI IAM
This CLoud Identity represents the keyless token for OCI IAM principals like API Key, instances or resources using OCI IAM group or dynamic group components.

# 1. Create instance based dynamic group in OCI 
- Extract the ID of the vm instance in the compartment

![image](https://github.com/brokedba/Akeyless_demo/assets/29458929/b9616cad-1171-4afc-89d5-4095762b1c5d)


```
$ ./check_instances.sh
===============================+========
   Instances INFO within the compartment
========================================
+------------------------+---------+----------------------------------------------------------------------------------------------+
| Name                   | State   | id                                                                                           |
+------------------------+---------+----------------------------------------------------------------------------------------------+
| app-instance-1         | RUNNING | ocid1.instance.oc1.ca-toronto-1.an2g6lxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx | <<<--- our instance
| app-instance-2         | STOPPED | ocid1.instance.oc1.ca-toronto-1.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx |
| web-instance-3         | STOPPED | ocid1.instance.oc1.ca-toronto-1.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx |
+------------------------+---------+----------------------------------------------------------------------------------------------+
$ export instance_ocid=ocid1.instance.oc1.ca-toronto-1.an2g6lxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
- create IAM dynamic group using the ocid
```
DYNAMIC_GROUP_OCID=$(oci iam dynamic-group create \
  --name app_group \
  --description "Dynamic group for app instances" \
  --matching-rule "ALL {instance.id = '${instance_ocid}'}" \
  --query 'data.id' --raw-output)
```
```
$ echo $DYNAMIC_GROUP_OCID
ocid1.dynamicgroup.oc1..xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

tenancy_ocid=$(cat ~/.oci/config |grep -m 1 tenancy | awk -F= '{print $2}')
 
``` 
- Create an OCI IAM Authentication Method from the CLI
```
akeyless create-auth-method-oci --name oci_auth \
--tenant-ocid  $tenancy_ocid --group-ocid $DYNAMIC_GROUP_OCID
--description 'OCI app instances Identity"
```
```
Auth Method oci_auth successfully created
- Access ID: p-2gftz5cc8qwrrm
```
><img src="https://github.com/brokedba/Akeyless_demo/assets/29458929/548d72af-f0bc-4c78-91e7-1d086d5c0075" width="450" height="150" />  

****
# Role Binding to OCI Auth client

1. Create role
in /MyVault/roles
```
akeyless create-role --name /MyVault/roles/roleoci
```
2. add access permission to role (mysqlDB dynamic secret)
`/MyVault/Devops/DBs`
```
akeyless set-role-rule --role-name /MyVault/roles/roleoci --path "/MyVault/DBs/*" \
--capability read --capability list --capability create  --capability update
```

3. associate the role with oci_auth

```
akeyless assoc-role-am --role-name /MyVault/roles/roleoci --am-name oci_auth
```
**Note:**
> At this point, any command run from outside the vm instance will cause below error as it's machine only ID 
```
$ akeyless list-items --profile oci-profile
connect: connection refused
Instance principals authentication can only be used on OCI compute instances. Please confirm this code is running on an OCI compute instance and you have set up the policy properly.
```
# OCI VM instance Akeyless config
- [install Akeyless](https://github.com/brokedba/Akeyless_demo/blob/main/docs/Akeyless_cli.md)
4.  Configure Akeyless CLI profile In the oci instance with OCI IAM auth identity

  # Errors : AKeyless documentation
  The [documetation](https://docs.akeyless.io/docs/oci-iam) says that the type of identity could be machines, users, or resources identified through `--group-ocid` flag .
  > **OCI IAM** authentication method provides an automated flow to retrieve an Akeyless token for OCI IAM principals like **API Key, instances or resources** using OCI **IAM group** or **dynamic group** [components](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/overview.htm#Componen).
  >
**However**, we used a dynamic group representing an instance principal and none of the below authentication worked with Akeyless CLI config.

- first test (Defaut profile + instance auth type)
```
akeyless configure --profile 'default' \
--access-id p-2gftz5cc8qwrrm \
--access-type oci --oci-auth-type instance --oci-group-ocid 'ocid1.dynamicgroup.oc1..xxxx'
```
- check akeyless profile config
  ```
  $ cat ~/.akeyless/profiles/default.toml
   ["default"]
   oci_group_ocids = 'ocid1.dynamicgroup.oc1..xxx'
   access_id = 'p-2gftz5cc8qwrrm'
   access_type = 'oci'
   oci_auth_type = 'instance'
  ```
- action output
```
ubuntu@app-instance-1:~/.akeyless/profiles$ akeyless list-items
can't setup akeyless client: failed to get credentials. error: Error returned by  Service. Http Status Code: 404. Error Code: NotAuthorizedOrNotFound. Opc request id: 68cec1df251020214dc9e0a5bac370a7/FA41F93D98BFD3949ADC37C459EE328D/498EC1969B4841677BBA107C43469F9A. Message: Authorization failed for authenticated request by 'PrincipalImpl{subjectId=ocid1.instance.oc1.ca-toronto-1.xx ...opc-tenant=ocid1.tenancy.oc1..xx, h_(request-target)=post /v1/authentication/authenticateClient, authorization=Signature version="1",headers="date (request-target) host content-length content-type x-content-sha256",keyId="xx..
opc-certtype=instance]}'
Operation Name:
Timestamp: 2024-06-30 06:59:23 +0000 GMT
Client Version: Oracle-GoSDK/65.55.1
Request Endpoint: POST https://auth.ca-toronto-1.oraclecloud.com/v1/authentication/authenticateClient
```
- Second test with apikey auth-type
```
$ akeyless configure --profile 'oci-profile2' --access-id p-2gftz5cc8qwrrm --access-type oci \
--oci-auth-type apikey --oci-group-ocid 'ocid1.dynamicgroup.oc1..xxxxx'
```
- check akeyless profile config
```
$ cat oci-profile2.toml
["oci-profile2"]
  access_id = 'p-2gftz5cc8qwrrm'
  access_type = 'oci'
  oci_auth_type = 'apikey'
  oci_group_ocids = 'ocid1.dynamicgroup.oc1..axxx'
```
- action output
```
akeyless list-items --profile oci-profile2
can't setup akeyless client: failed to get credentials. error: Error returned by  Service. Http Status Code: 401. Error Code: NotAuthenticated. Opc request id: e70ce178ec5851418fcae35a5702b5b7/ABA4901F40A42F9BF2BF6CC125E7A9DA/D27E65A0B00EA0B116052770BFE90832. Message: The required information to complete authentication was not provided or was incorrect.
Operation Name:
Timestamp: 2024-06-30 07:05:20 +0000 GMT
Client Version: Oracle-GoSDK/65.55.1
Request Endpoint: POST https://auth.ca-toronto-1.oraclecloud.com/v1/authentication/authenticateClient
Troubleshooting Tips: See  for more information about resolving this error.
```
- Extract oci-auth-type apikey works on a local machine with email admin auth method
akeyless get-cloud-identity --oci-auth-type instance


> **Note:**
> We also added a user group containing an OCI IAM user to the --group-ocid flag but we still had errors . Akeyless api key is generated but authentication to OCI throws error 401 ( NotAuthenticated)
