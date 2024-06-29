# Overview

- Create a mysql container in an OCI VM 

```
ssh  ubuntu@132.145.108.41
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt update
apt-cache policy docker-ce
sudo apt install docker-ce
--- add your username to the docker group:
sudo usermod -aG docker ${USER}
sudo usermod -aG docker ubuntu
--- log out/log in
$ docker pull mysql/mysql-server
$ docker run --name='mysql_db1' -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=welcome1 mysql/mysql-server
--- check root password
docker logs mysql_db1 2>&1 | grep GENERATED
GENERATED ROOT PASSWORD: 96;#M=7QUAv/YvImJ3Rw_?29zu@d9K2+
```
- Connecting to MySQL Server from within the Container
  ```
  docker exec -it mysql_db1 mysql -uroot -p
  password: <enter initial pass>
  mysql> ALTER USER 'root'@'172.17.0.1' IDENTIFIED with mysql_native_password BY 'welcome1';
  mysql> use mysql;
  mysql> select user from user;
  ```
- Create a user for connection
```
CREATE USER 'appuser'@'%' IDENTIFIED with mysql_native_password  BY 'welcome1';
ALTER USER 'appuser'@'172.17.0.1' IDENTIFIED with mysql_native_password  BY 'welcome1'; 
GRANT ALL PRIVILEGES ON *.* TO 'appuser'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;

mysql> select user from user;
+------------------+
| user             |
+------------------+
| appuser          |
| healthchecker    |
| mysql.infoschema |
| mysql.session    |
| mysql.sys        |
| root             |
+------------------+
```
# Create Mysql target
```
akeyless create-db-target --name /DBs/MySQLTargetOCI \
--db-type mysql \
--host 132.145.108.41 \
--port 3306 \
--user-name userapp \
--pwd welcome1 \
--db-name mysql_db1
```

# Create a Dynamic Database Secret from the CLI
- run below command
```
akeyless dynamic-secret create mysql \
  --name /MyVault/DBs/MySQLDynamicSecret \
  --target-name /DBs/MySQLTargetOCI \
  --gateway-url 'https://your-akeyless-gw-url:8000' \
  --mysql-statements "CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}' PASSWORD EXPIRE INTERVAL 30 DAY;GRANT SELECT ON *.* TO '{{name}}'@'%';" \
  --mysql-revocation-statements "REVOKE ALL PRIVILEGES, GRANT OPTION FROM '{{name}}'@'%'; DROP USER '{{name}}'@'%';" \
```
**Fetch a Dynamic Database Secret value from the CLI**

```
akeyless dynamic-secret get-value --name <Path to your dynamic secret>
```


