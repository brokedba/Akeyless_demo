# Overview
Instead of authenticating identities itself, in most cases, Akeyless integrates with 3rd party identity providers that provide tokens of authentication.
in this example we will use Cloud IAM based identity through [OCI IAM](https://docs.akeyless.io/docs/oci-iam).

# OCI IAM
This CLoud Identity represents the keyless token for OCI IAM principals like API Key, instances or resources using OCI IAM group or dynamic group components.

# 1. Create dynamic group in OCI based on all instances from a compartment
- Extract the ID
> 
- Create an OCI IAM Authentication Method from the CLI
```
akeyless create-auth-method-oci \
--name oci_auth \
--tenant-ocid  \
--group-ocid <Oracle Group Id> 
```
Configure Akeyless CLI with the OCI IAM authentication method
akeyless configure --profile 'oci-profile' --access-id <Your OCI IAM Auth AccessID> \
--access-type oci --oci-auth-type apikey 
Extract apikey
akeyless get-cloud-identity --oci-auth-type apikey 
