#### Tune host for MongoDB case  
##### Linux ulimit  
Create mongod.conf in ulimits daemon  
```bash
vim /etc/security/limits.d/mongod.conf
```
Add these form in mongod.conf  
```bash
mongod   soft    nproc    64000
mongod   hard    nproc    64000
mongod   soft    nofile   64000
mongod   hard    nofile   64000
```
##### Sysctl kernel  
Modify ``` /etc/sysctl.conf ``` file.  
```bash
vm.dirty_ratio = 15
vm.dirty_background_ratio = 5
vm.swappiness = 0
net.core.somaxconn = 4096
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_intvl = 30
net.ipv4.tcp_keepalive_time = 120
net.ipv4.tcp_max_syn_backlog = 4096
```
##### Transparent HugePage  
```bash
echo 'never' > /sys/kernel/mm/transparent_hugepage/enabled
echo 'never' > /sys/kernel/mm/transparent_hugepage/defrag
```
##### Disable NUMA arch  
Modify ``` /etc/default/grub ``` and add ``` numa=off ``` in GRUB_CMDLINE_LINUX section.  
```bash
GRUB_CMDLINE_LINUX="rd.lvm.lv=rhel_vm-210/root rd.lvm.lv=rhel_vm-210/swap vconsole.font=latarcyrheb-sun16 crashkernel=auto  vconsole.keymap=us rhgb quiet numa=off"
```
Rebuild the changes.  
```bash
grub2-mkconfig -o /etc/grub2.cfg
```
##### Change Read-ahead size  
Create ``` /etc/udev/rules.d/60-sda.rules ``` file and add below data.  
```bash
ACTION=="add|change",KERNEL=="sda",ATTR{queue/scheduler}="deadline",ATTR{bdi/read_ahead_kb}="16"
To check the IO scheduler was applied ([square-brackets] = enabled scheduler):
$cat/sys/block/sda/queue/scheduler
noop[deadline]cfq
To check the current read-ahead setting:
$sudo blockdev--getra/dev/sda
```
##### Ref  
https://dzone.com/articles/tuning-linux-for-mongodb
