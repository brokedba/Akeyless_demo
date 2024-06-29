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
```
- pulling mysql image

```
$ docker pull mysql/mysql-server
$ docker run --name='mysql_db1' -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=welcome1 mysql/mysql-server
```
- Update the bind-address Directive:

```
vi mysqld.cnf
[mysqld]
bind-address = 0.0.0.0
```
- run docker image
```
$ docker run -d --name mysql_db1 -e MYSQL_DATABASE=mysql_db1 \
-e MYSQL_ROOT_PASSWORD=welcome1 \
-e MYSQL_USER=userapp \
-e MYSQL_PASSWORD=welcome1 \
-p 3306:3306 \
-v ./mysqld.cnf:/etc/mysql/conf.d/mysqld.cnf \
mysql:latest

```   
- Connecting to MySQL Server
  ```
   mysql -h 127.0.0.1 -P 3306 -uroot -p
  ```
- Create / ALTER appuser with grants
```
--- CREATE USER 'appuser'@'%' IDENTIFIED with mysql_native_password  BY 'welcome1';
GRANT ALL PRIVILEGES ON *.* TO 'userapp'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
mysql> use mysql;
mysql> select user,host from mysql.user;
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| root             | %         |
| userapp          | %         |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
mysql> SHOW GRANTS FOR 'userapp'@'%';
+---------------------------------------------------------+
| Grants for userapp@%                                    |
+---------------------------------------------------------+
| GRANT USAGE ON *.* TO `userapp`@`%`                     |
| GRANT ALL PRIVILEGES ON `mysql\_db1`.* TO `userapp`@`%` |
+---------------------------------------------------------+
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
- delete target command :
```
akeyless delete-target -n /DBs/MySQLTargetOCI
```
# Create a Dynamic Database Secret from the CLI
- run below command
```
akeyless dynamic-secret create mysql \
  --name /MyVault/DBs/MySQLDynamicSecret \
  --target-name /DBs/MySQLTargetOCI \
  --mysql-statements "CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}' PASSWORD EXPIRE INTERVAL 30 DAY;GRANT SELECT ON *.* TO '{{name}}'@'%';" \
  --mysql-revocation-statements "REVOKE ALL PRIVILEGES, GRANT OPTION FROM '{{name}}'@'%'; DROP USER '{{name}}'@'%';" \
```
**Note:** Gateway is not needed in the cli but mandatory in Akeyless UI
 
 > ```--gateway-url 'https://your-akeyless-gw-url:8000'``` 
 
 ><img src="https://github.com/brokedba/Akeyless_demo/assets/29458929/874b9db4-f28c-4b36-aab9-dc2bf6198d99" width="450" height="150" />  

 

**Fetch a Dynamic Database Secret value from the CLI/Console**

```
akeyless dynamic-secret get-value --name /MyVault/DBs/MySQLDynamicSecret
{
  "id": "tmp_kloudd_ohJDD",
  "password": "TFW3QmEVAT@~2yo8",
  "ttl_in_minutes": "60",
  "user": "tmp_kloudd_ohJDD"
}
```
> <img src="https://github.com/brokedba/Akeyless_demo/assets/29458929/cc33f23d-1bef-4526-908e-9884be1e11d6" width="400" height="300" />
- login using cli and gui client (DBeaver)
```
-- Intsall client locally
sudo apt install mysql-client
mysql --version
--- connect
mysql -h 132.145.108.41 -P 3306 -u tmp_kloudd_ohJDD -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Server version: 8.4.0 MySQL Community Server - GPL

mysql> SELECT CURRENT_USER();
+--------------------+
| CURRENT_USER()     |
+--------------------+
| tmp_kloudd_ohJDD@% |
+--------------------+

mysql> SHOW GRANTS FOR CURRENT_USER();
+-----------------------------------------------+
| Grants for tmp_kloudd_ohJDD@%                 |
+-----------------------------------------------+
| GRANT SELECT ON *.* TO `tmp_kloudd_ohJDD`@`%` |
+-----------------------------------------------+
```
- DBeaver
> <img src="https://github.com/brokedba/Akeyless_demo/assets/29458929/24fcf42d-bf81-4093-81b7-8620f071f8f3" width="500" height="400" />
 

