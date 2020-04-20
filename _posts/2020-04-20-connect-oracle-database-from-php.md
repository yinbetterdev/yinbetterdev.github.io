---
layout: post
title: "Connect oracle database from php in Redhat/CentOS/Fedora"
description: "The post will describe how to connect Oracle database from PHP in Redhat/CentOS in step by step. The easiest way to configure PHP to access a remote Oracle Database is to use Oracle Instant Client libraries"
comments: true
keywords: "centos, php, oracle"
---

### Connect oracle database from php in Redhat/CentOS/Fedora

The post will describe how to connect Oracle database from PHP in Redhat/CentOS in step by step. The easiest way to configure PHP to access a remote Oracle Database is to use Oracle Instant Client libraries. This note describes how to install PHP with the OCI8 Extension and Oracle Instant Client on Redhat/CentOS. OCI8 is the PHP extension for connecting to Oracle Database. You need to install two packages: oracle-instant-client and php oci8 extension. It is assumed that PHP with pecl package is installed in your server

1. Check your server architecture and OS version
```
# uname -a
Linux localhost 2.6.32-220.el6.x86_64 #1 SMP Wed Nov 9 08:03:13 EST
2011 x86_64 x86_64 x86_64 GNU/Linux
```
el6 means it is enterprise linux 6 and x86_64 means it is 64 bit machine. For 32 bit machine it would be i386

2. Download two RPM packages: oracle-instantclient basic and oracle-instantclient devel from [instant client downloads page](http://www.oracle.com/technetwork/database/features/instant-client/index-097480.html). 

Here you need to choose “Instant Client for Linux x86_64”. For 32 bit machine you should select “Instant Client for Linux x86”.

Download red tick instant client basic RPM package.
![instantclient_basic](https://i1.wp.com/www.techinfobest.com/wp-content/uploads/2014/02/instantclient_basic.png)
Scroll down and download red tick instant client devel rpm package.
![instantclient_devel](https://i0.wp.com/www.techinfobest.com/wp-content/uploads/2014/02/instantclient_devel.png)

3. Install the RPMs as the root user
```
# rpm -Uvh oracle-instantclient12.1-basic-12.1.0.1.0-1.x86_64.rpm
Preparing...                ########################################### [100%]
   1:oracle-instantclient12.########################################### [100%]
# rpm -Uvh oracle-instantclient12.1-devel-12.1.0.1.0-1.x86_64.rpm
Preparing...                ########################################### [100%]
   1:oracle-instantclient12.########################################### [100%]
```
The instantclient library and executable files are generally installed on /usr/lib/oracle/12.1/client64/ for 64 bit machine and on /usr/lib/oracle/12.1/client/ for 32 machine

4. Set environment variables ORACLE_HOME and LD_LIBRARY_PATH

```
# ORACLE_HOME=/usr/lib/oracle/12.1/client64; export ORACLE_HOME
# LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib; export LD_LIBRARY_PATH
You should also add the above two lines in ~/.bash_profile file. Edit the file by vi editor and add the lines above PATH=$PATH:$HOME/bin

#vim  ~/.bash_profile
```
After edited the file would be

```
# .bash_profile
# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

ORACLE_HOME=/usr/lib/oracle/12.1/client64; export ORACLE_HOME
LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib; export LD_LIBRARY_PATH
PATH=$PATH:$HOME/bin

export PATH
```

5.Check you have php-pear and php-devel packages installed by this command. You must have these packages installed 
```
# rpm -qa | grep -i php
php-5.3.3-3.el6_1.3.x86_64
php-cli-5.3.3-3.el6_1.3.x86_64
php-devel-5.3.3-3.el6_1.3.x86_64
php-common-5.3.3-3.el6_1.3.x86_64
php-mysql-5.3.3-3.el6_1.3.x86_64
php-soap-5.3.3-3.el6_1.3.x86_64
php-pear-1.9.4-4.el6.noarch
php-gd-5.3.3-3.el6_1.3.x86_64
```
6.Install oci8 extension
```
# pecl install oci8
```
You will be asked to provide instantclient library path and so provide the instant cleint library path just you installed( put instantclient then coma then library path)
```
Please provide the path to the ORACLE_HOME directory. Use 'instantclient,/path/to/instant/client/lib' if you're compiling with Oracle Instant Client [autodetect] :instantclient,/usr/lib/oracle/12.1/client64/lib
```
After pressing enter ,at the end portion you will see the line “Build process completed successfully”
```
Build process completed successfully
Installing '/usr/lib64/php/modules/oci8.so'
install ok: channel://pecl.php.net/oci8-2.0.6
configuration option "php_ini" is not set to php.ini location
You should add "extension=oci8.so" to php.ini
```
7.Add “extension=oci8.so” to php.ini file just below the [PHP] line and it would like
```
[PHP]
extension=oci8.so
;;;;;;;;;;;;;;;;;;;
; About php.ini   ;
;;;;;;;;;;;;;;;;;;;
; PHP's initialization file, generally called php.ini, is responsible for
; configuring many of the aspects of PHP's behavior.
```
8.Restart Web Server(Apache or Nginx)
```
# /etc/init.d/httpd restart
```
9.Now check oci8 extension is installed and OCI8 Support is enabled
```
# echo "<?php print phpinfo();?>" | php | grep -i oci
oci8
OCI8 Support => enabled
OCI8 DTrace Support => disabled
OCI8 Version => 2.0.6
oci8.connection_class => no value => no value
oci8.default_prefetch => 100 => 100
oci8.events => Off => Off
oci8.max_persistent => -1 => -1
oci8.old_oci_close_semantics => Off => Off
oci8.persistent_timeout => -1 => -1
oci8.ping_interval => 60 => 60
oci8.privileged_connect => Off => Off
oci8.statement_cache_size => 20 => 20
```

Now you have OCI8 installed and you can write code for connecting to Oracle Database. Here is the sample PHP Code
```php
<?php

$conn = oci_connect('test_user', 'test_password', 'mymachine.mydomain/orcl');

$stid = oci_parse($conn, 'select table_name from user_tables');
oci_execute($stid);

echo "<table>\n";
while (($row = oci_fetch_array($stid, OCI_ASSOC+OCI_RETURN_NULLS)) != false) {
    echo "<tr>\n";
    foreach ($row as $item) {
        echo "  <td>".($item !== null ? htmlentities($item, ENT_QUOTES) : "&nbsp;")."</td>\n";
    }
    echo "</tr>\n";
}
echo "</table>\n";

?>
```
```
test_user is user of the  Database
test_password is the password of the database
mymachine.mydomain is the hostname.domain of the Database. You can put IP directly here.
orcl is the SID of the Database.
```
The above procedure regarding “Connect Oracle Database from PHP in Redhat/CentOS” should work for major version of Linux distribution