# Gateway staling issue
# Root cause
1. VM Maxed out CPU at  90%
   ![image](https://github.com/brokedba/Akeyless_demo/assets/29458929/bbc69270-2505-429a-9308-1148ea8e5eb6)
2. laptop processor power management  max processor sate reduced to 85%
- I always do that before my podcats recording to avoid noisy fan cooling

> <img src="https://github.com/brokedba/Akeyless_demo/assets/29458929/3bb652c3-80b3-442a-a25a-71cf882b3759" width="400" height="500" />


# Fix 
- Increase the max processor state back to 100%
- Optional : add a little bit of juice to the VM with 8G /4 vcpu 
```yaml
# Variables
var_box            = 'bento/oracle-7'
var_vm_name        = 'devops_vagrant_vm'
var_mem_size       = 8192   # #4092  # More would be better.
var_cpus           = 4 #2
```

- **Restart the vm**
# Result 
![image](https://github.com/brokedba/Akeyless_demo/assets/29458929/d9096103-535e-4f88-8407-b23fc4cb2979)

- **Confirm the docker container is running**
```bash

[root@localhost ~]# docker ps | grep akeyless
1e891d7cb994   akeyless/base:latest-akeyless   "/bin/sh -c '/bin/ba…"   7 days ago    Up 6 minutes    0.0.0.0:5696->5696/tcp, :::5696->5696/tcp, 0.0.0.0:8000->8000/tcp, :::8000->8000/tcp, 0.0.0.0:8080-8081->8080-8081/tcp, :::8080-8081->8080-8081/tcp, 0.0.0.0:8200->8200/tcp, :::8200->8200/tcp, 0.0.0.0:18888->18888/tcp, :::18888->18888/tcp   akeyless-dock-gw

```
- **Docker top**
![image](https://github.com/brokedba/Akeyless_demo/assets/29458929/60247124-7422-4284-887d-d40342706042)

- Verify the endpoint is responsive using curl
![image](https://github.com/brokedba/Akeyless_demo/assets/29458929/34413b0f-a490-42bf-acf5-81fc62e710f9)


# test login to the Gateway 


https://github.com/brokedba/Akeyless_demo/assets/29458929/59f8c5d6-f276-46de-8a99-52702b2db12f


