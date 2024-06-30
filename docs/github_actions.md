# Overview
# GitHub Actions Plugin
[Akeyless Official GitHub Actions ](https://github.com/marketplace/actions/akeyless-authentication-and-fetching-secrets), plugin enables you to automate workflows for your GitHub-hosted repositories. 

With the GitHub Actions plugin, you can fetch secrets directly from Akeyless into your workflows. This guide describes how to use our various Authentication Methods to fetch Static, Dynamic, and Rotated secrets, as well as SSH and PKI certificates, from Akeyless.

**Prerequisites**
Job permissions requirement: (Relevant for OAuth 2.0 / JWT Authentication only)
The default usage relies on using the GitHub JWT (JSON Web Token) to authenticate to Akeyless. To make this available, you must configure it inside your job workflow.

```YAML

jobs:
  my_job:
    #---------Required---------#
    permissions: 
      id-token: write
      contents: read
    #--------------------------#
```
