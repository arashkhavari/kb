#### Multipe config file in Haproxy
##### Create directory fir split config files
```bash
mkdir -p /etc/haproxy/conf.d
```
##### Write script for gather config files
Create **/usr/local/bin/haproxy-multiconf** file and write below script
```bash
#!/bin/bash
for file in /etc/haproxy/conf.d/* /etc/haproxy/*.cfg ; do
test -f $file
CNF="$CNF -f $file"
done
echo "CONF='$CNF'" > /etc/haproxy/haproxy-multiconf.lst
```
Execute file
```bash
chmod +x /usr/local/bin/haproxy-multiconf
```
##### Create haproxy-multiconf service
Create **/etc/systemd/system/haproxy-multiconf.service** file and write below config
```bash
[Unit]
Description=HAProxy Load Balancer Multiconf
After=network.target
Before=haproxy.service
[Service]
Type=oneshot
ExecStart=/usr/local/bin/haproxy-multiconf
[Install]
WantedBy=multi-user.target
```
##### Modify haproxy service
```bash
[Unit]
Description=HAProxy Load Balancer
After=syslog.target network.target
Requires=haproxy-multiconf.service
[Service]
EnvironmentFile=/etc/haproxy/haproxy-multiconf.lst
ExecStart=/usr/sbin/haproxy-systemd-wrapper -p /run/haproxy.pid $OPTIONS $CONF
ExecReload=/usr/sbin/haproxy -c -q $CONF
ExecReload=/bin/kill -USR2 $MAINPID
KillMode=mixed
[Install]
WantedBy=multi-user.target
```
##### Restart service
```bash
systemctl daemon-reload
systemctl restart haproxy
```
##### Config format
All files should this format:
```bash
global
    ...

default
    ...

frontend
    ...

backend
    ...

```
and each file should seperate validate
```bash
haproxy -c -f haproxy-custom.cfg
```
##### Ref
https://elhombrequereventodeinformacion.wordpress.com/2016/12/09/split-haproxy-configuration-in-multiple-files-working-with-systemd/
