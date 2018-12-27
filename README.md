High concurrency typical server config
(feel free to comment)
Step by step
Centos 7
nginx
php-fpm
mysql (percona)

#Part One. Installing.

Updating to newest stable kernel
--------------------------------
Check which kernel  OS have with `uname -a`\
Get new one
```bash
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
yum --enablerepo=elrepo-kernel install kernel-ml
```
Change default boot kernel to 0 (latest one installed, first at menuentry at `/boot/grub2/grub.cfg`)\
Edit `/etc/default/grub` and set `GRUB_DEFAULT=0` then `grub2-mkconfig -o /boot/grub2/grub.cfg`

Im disabling SELinux on that step (i know im wrong) you shouldnt do it. Dont like that it cause nasty silent errors that hard to find.
For current session use `setenforce 0`\
To disable SELinux permanently edit `/etc/sysconfig/selinux` and set it to `disabled`. 

Now reboot and check kernel with `uname -a` again.

Getting latest stable Nginx, PHP-FPM and percona mysql
------------------------------------------------------
## Nginx
```sh
yum install epel-release
yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
yum install yum-utils
yum-config-manager --enable remi-nginx
```
Now create `/etc/yum.repos.d/nginx.repo` 
```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
```
And install latest stable nginx `yum install nginx`

## PHP
For php 7 do
```
yum-config-manager --enable remi-php72
```
For php 5.6 edit `/etc/yum.repos.d/remi.repo` 

Find `[remi-php56]` section and change `enabled=1`
Install php 
```yum install php php-fpm php-mysqld php-posix php-xml php-mbstring php-mcrypt php-cli php-gd php-curl php-mysql php-ldap php-zip php-fileinfo```
Get only extensions you really need, as every PHP-FPM child will consume more memory with every extension added.

## MySQL
```
 yum install http://www.percona.com/downloads/percona-release/redhat/0.1-6/percona-release-0.1-6.noarch.rpm
 yum list | grep percona
 yum install Percona-Server-server-57
 service mysql start

```
Now get a temp password from error.log (weird)
```
cat /var/log/mysqld.log |grep password
```
And reset root password
```mysql
mysql -uroot -p
#First lets lower password policy as it too restrictive for me 
SET GLOBAL validate_password_policy=LOW;
#Now change the password and flush privileges then
ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass';
FLUSH PRIVILEGES;
exit;
```

# Part Two. Configuring.
## Kernel tuning
`echo 'fs.file-max=1048576' >> /etc/sysctl.d/01-general.conf`
[Reason](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/tuning_and_optimizing_red_hat_enterprise_linux_for_oracle_9i_and_10g_databases/chap-oracle_9i_and_10g_tuning_guide-setting_file_handles)

## Network tuning
`/etc/sysctl.d/02-netio.conf`


