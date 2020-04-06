Renewal license
Download from https://mmonit.com/download
Extract here
Go to ./conf/server.xml in license section get license key
Replace in product mmonit and restart mmonit service

Change dashboard admin username's password
Genrate MD5 password:

echo -n newpassword | md5sum
Stop mmonit service:

systemctl stop mmonit
Open MySQl database:

mysql -u Username -pPassword
use mmonit
update users set password = "newpasswordmd5" where uname = "admin" ;
And run mmonit:

systemctl start mmonit
