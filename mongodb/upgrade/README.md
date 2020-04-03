#### Upgrade a standalone mongod from version 3.2 to 4.2
  
##### Backup data from last MongoDB
  
```bash
mongodump --host <host_ip> -u<username> -p<password> --db <db_name> --out=<output_dump> --authenticationDatabase <auth_db>
```

##### Install MongoDB CE version 3.2 with package manager

1- Add repo

```bash
cat > /etc/yum.repos.d/mongodb-org-3.2.repo << EOF
[mongodb-org-3.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.2/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.2.asc
EOF
```
2- Install MongoDB and start service

```bash
yum install -y monogdb-org
systemctl start mongod
```
3- Add username and password for MongoDB

```bash
mongo
```
```json
use admin
db.createUser(
{	user: "admin",
	pwd: "password",
	roles:[{role: "root" , db:"admin"}]
    }
)
```
** Uncommnet ```secuirty``` in /etc/mongod.conf and add ``` authorization: "enabled"``` in security option

Restart mongod

```bash
sytemctl restart mongod
```
Then login to mongod
```bash
mongo -uadmin -psamir123 --authenticationDatabase admin
```

##### Restore data to new MongoDB
```bash
mongorestore --host <host_ip> --username <username> --password <passowrd> --authenticationDatabase <auth_db> <dump_path>
```
##### Upgrade to 3.4
1- Add repo
```bash
cat > /etc/yum.repos.d/mongodb-org-3.4.repo << EOF
[mongodb-org-3.4]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/7/mongodb-org/3.4/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc
EOF
```
2- Install new version MongoDB
```bash
yum install -y mongodb-org
```
3- Restart mongod
```bash
systemctl daemon-reload
systemctl stop mongod
systemctl start mongod
```
4- Check feature compatility with this version
```bash
mongo -uadmin -ppassword --authenticationDatabase admin
```
```json
db.adminCommand( { setFeatureCompatibilityVersion: "3.4" } )
```
response:

```{ "ok" : 1 }```

##### Upgrade to 3.6
1- Add repo
```bash
cat > /etc/yum.repos.d/mongodb-org-3.6.repo << EOF
[mongodb-org-3.6]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/7/mongodb-org/3.6/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.6.asc
EOF
```
2- Install new version MongoDB
```bash
yum install -y mongodb-org
```
3- Restart mongod
```bash
systemctl daemon-reload
systemctl stop mongod
systemctl start mongod
```
4- Check feature compatility with this version
```bash
mongo -uadmin -ppassword --authenticationDatabase admin
```
```json
db.adminCommand( { setFeatureCompatibilityVersion: "3.6" } )
```
response:

```{ "ok" : 1 }```

##### Upgrade to 4.0
1- Add repo
```bash
cat > /etc/yum.repos.d/mongodb-org-4.0.repo << EOF
[mongodb-org-4.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/7/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
EOF
```
2- Install new version MongoDB
```bash
yum install -y mongodb-org
```
3- Restart mongod
```bash
systemctl daemon-reload
systemctl stop mongod
systemctl start mongod
```
4- Check feature compatility with this version
```bash
mongo -uadmin -ppassword --authenticationDatabase admin
```
```json
db.adminCommand( { setFeatureCompatibilityVersion: "4.0" } )
```
response:

```{ "ok" : 1 }```

##### Upgrade to 4.2
1- Add repo
```bash
cat > /etc/yum.repos.d/mongodb-org-4.2.repo << EOF
[mongodb-org-4.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/7/mongodb-org/4.2/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.2.asc
EOF
```
2- Install new version MongoDB
```bash
yum install -y mongodb-org
```
3- Restart mongod
```bash
systemctl daemon-reload
systemctl stop mongod
systemctl start mongod
```
4- Check feature compatility with this version
```bash
mongo -uadmin -ppassword --authenticationDatabase admin
```
```json
db.adminCommand( { setFeatureCompatibilityVersion: "4.2" } )
```
response:

```{ "ok" : 1 }```



##### Ref

https://docs.mongodb.com/manual/release-notes/4.2-upgrade-standalone/

https://docs.mongodb.com/manual/release-notes/4.0-upgrade-standalone/

https://docs.mongodb.com/manual/release-notes/3.6-upgrade-standalone/

https://docs.mongodb.com/manual/release-notes/3.4-upgrade-standalone/
