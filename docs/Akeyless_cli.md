# Installation
```
curl -o akeyless https://akeyless-cli.s3.us-east-2.amazonaws.com/cli/latest/production/cli-linux-amd64
chmod +x akeyless
sudo cp akeyless /usr/local/bin
```
# Configuration

```
brokedba@Orpheus:~$ akeyless -h
AKEYLESS-CLI, first use detected
For more info please visit: https://docs.akeyless.io/docs/cli
Enter Akeyless URL (Default: vault.akeyless.io)
Would you like to configure a profile? (Y/n)
Profile Name:  (Default: default)
Access Type (enter for access_key):
  1) access_key
  2) aws_iam
  3) azure_ad
  4) saml
  5) ldap
  6) email/password
  7) oidc
  8) k8s
  9) gcp
  10) certificate
  11) oci
 6
Admin Email:  User@Mymail.com
Admin Password:  *******
The profile: default was successfully configured
Would you like to move 'akeyless' binary to: ~/.akeyless/bin/akeyless? (Y/n)
Please type your answer: n
```
**Note:**

Please refrain from using email/password Authentication as the configuration will store the password in base64 encoding format.
suggestion: Ceate an API Key at least.

- Check the profile configuration
```

brokedbaa@Orpheus:$ cat ~/.akeyless/profiles/default.toml
["default"]
  admin_email = 'User@Mymail.com'
  account_id = ''
  access_type = 'password'
  admin_password = '==' <-------- base64  encoded password
```

echo xxxxxxxx== | base64 -d
> your password in plain text

# Create secret item
1. **Static secret**
```
brokedbaa@Orpheus:$ akeyless create-secret --name mysectest2  --value "StaticIsBad " --type generic
A new secret named mysectest2 was successfully created
```
2. **API Key** 
```
```
