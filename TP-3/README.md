# Service SSH

## Analyse dy service

```powershell
[root@localhost ~]# systemctl status sshd
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)
     Active: active (running) since Mon 2024-01-29 10:31:46 CET; 5min ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 716 (sshd)
      Tasks: 1 (limit: 11025)
     Memory: 5.0M
        CPU: 173ms
     CGroup: /system.slice/sshd.service
             └─716 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Jan 29 10:31:46 localhost.localdomain systemd[1]: Starting OpenSSH server daemon...
Jan 29 10:31:46 localhost.localdomain sshd[716]: Server listening on 0.0.0.0 port 22.
Jan 29 10:31:46 localhost.localdomain sshd[716]: Server listening on :: port 22.
Jan 29 10:31:46 localhost.localdomain systemd[1]: Started OpenSSH server daemon.
Jan 29 10:37:23 localhost.localdomain sshd[1279]: Accepted password for root from 10.1.1.1 port 59855 ssh2
Jan 29 10:37:23 localhost.localdomain sshd[1279]: pam_unix(sshd:session): session opened for user root(uid=0) by (uid=0)
```

```powershell
[root@localhost ~]# ps -ef | grep sshd
root         716       1  0 10:31 ?        00:00:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
root        1279     716  0 10:37 ?        00:00:00 sshd: root [priv]
root        1283    1279  0 10:37 ?        00:00:00 sshd: root@pts/0
root        1331    1284  0 10:46 pts/0    00:00:00 grep --color=auto sshd
```

```powershell
[root@localhost ~]# sudo ss -alnpt | grep sshd
LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=716,fd=3))
LISTEN 0      128             [::]:22           [::]:*    users:(("sshd",pid=716,fd=4))
```

```powershell
[root@localhost ~]# journalctl | grep sshd
Jan 29 10:31:40 localhost systemd[1]: Created slice Slice /system/sshd-keygen.
Jan 29 10:31:44 localhost systemd[1]: Reached target sshd-keygen.target.
Jan 29 10:31:46 localhost.localdomain sshd[716]: Server listening on 0.0.0.0 port 22.
Jan 29 10:31:46 localhost.localdomain sshd[716]: Server listening on :: port 22.
Jan 29 10:37:23 localhost.localdomain sshd[1279]: Accepted password for root from 10.1.1.1 port 59855 ssh2
Jan 29 10:37:23 localhost.localdomain sshd[1279]: pam_unix(sshd:session): session opened for user root(uid=0) by (uid=0)
```

## 2. Modification du service

```powershell
[root@localhost ~]# ls -al /etc/ssh
total 616
drwxr-xr-x.  4 root root       4096 Dec 18 12:01 .
drwxr-xr-x. 78 root root       8192 Jan 29 10:31 ..
-rw-r--r--.  1 root root     578094 Nov  2 20:33 moduli
-rw-r--r--.  1 root root       1921 Nov  2 20:33 ssh_config
drwxr-xr-x.  2 root root         28 Dec 18 11:57 ssh_config.d
-rw-------.  1 root root       3667 Nov  2 20:33 sshd_config
drwx------.  2 root root         59 Dec 18 11:59 sshd_config.d
-rw-r-----.  1 root ssh_keys    480 Dec 18 12:01 ssh_host_ecdsa_key
-rw-r--r--.  1 root root        162 Dec 18 12:01 ssh_host_ecdsa_key.pub
-rw-r-----.  1 root ssh_keys    387 Dec 18 12:01 ssh_host_ed25519_key
-rw-r--r--.  1 root root         82 Dec 18 12:01 ssh_host_ed25519_key.pub
-rw-r-----.  1 root ssh_keys   2578 Dec 18 12:01 ssh_host_rsa_key
-rw-r--r--.  1 root root        554 Dec 18 12:01 ssh_host_rsa_key.pub

# Le fichier est sshd_config
```

```powershell
[root@localhost ~]# echo $RANDOM
25346
[root@localhost ~]# sudo nano /etc/ssh/sshd_config
[root@localhost ~]# sudo nano /etc/ssh/sshd_config
[root@localhost ~]# sudo cat /etc/ssh/sshd_config | grep Port
Port 25346
#GatewayPorts no
```

```powershell
[root@node1 ~]# sudo firewall-cmd --add-port=25346/tcp --permanent
success
[root@node1 ~]# sudo firewall-cmd --zone=public --permanent --remove-port 22/tcp
success
[root@node1 ~]# sudo firewall-cmd --reload
success

[root@node1 ~]# sudo systemctl restart firewalld

[root@node1 ~]# sudo systemctl restart firewalld
[root@node1 ~]# firewall-cmd --list-all | grep ports
  ports: 25346/tcp
  forward-ports:
  source-ports:
```

```powershell
PS C:\Users\guill> ssh -p25346 root@10.1.1.44
root@10.1.1.44's password:
Last login: Mon Jan 29 11:41:42 2024 from 10.1.1.1
```
---

# Sevice HTTP

## 1. Mise en place

```powershell
[root@node1 ~]# dnf search NGINX
============================================================== Name & Summary Matched: NGINX ===============================================================
nginx-all-modules.noarch : A meta package that installs all available Nginx modules
nginx-core.x86_64 : nginx minimal core
nginx-filesystem.noarch : The basic directory layout for the Nginx server
nginx-mod-http-image-filter.x86_64 : Nginx HTTP image filter module
nginx-mod-http-perl.x86_64 : Nginx HTTP perl module
nginx-mod-http-xslt-filter.x86_64 : Nginx XSLT module
nginx-mod-mail.x86_64 : Nginx mail modules
nginx-mod-stream.x86_64 : Nginx stream modules
pcp-pmda-nginx.x86_64 : Performance Co-Pilot (PCP) metrics for the Nginx Webserver
=================================================================== Name Matched: NGINX ====================================================================
nginx.x86_64 : A high performance web server and reverse proxy server
```

```powershell
[root@node1 ~]# sudo dnf install nginx
Last metadata expiration check: 0:07:53 ago on Mon 29 Jan 2024 11:44:20 AM CET.
Dependencies resolved.
============================================================================================================================================================
 Package                                  Architecture                  Version                                      Repository                        Size
============================================================================================================================================================
Installing:
 nginx                                    x86_64                        1:1.20.1-14.el9_2.1                          appstream                         36 k
Installing dependencies:
 nginx-core                               x86_64                        1:1.20.1-14.el9_2.1                          appstream                        565 k
 nginx-filesystem                         noarch                        1:1.20.1-14.el9_2.1                          appstream                        8.5 k
 rocky-logos-httpd                        noarch                        90.14-2.el9                                  appstream                         24 k

Transaction Summary
============================================================================================================================================================
Install  4 Packages

Total download size: 634 k
Installed size: 1.8 M
Is this ok [y/N]: y
Downloading Packages:
(1/4): nginx-filesystem-1.20.1-14.el9_2.1.noarch.rpm                                                                        1.7 kB/s | 8.5 kB     00:05
(2/4): rocky-logos-httpd-90.14-2.el9.noarch.rpm                                                                             4.8 kB/s |  24 kB     00:05
(3/4): nginx-1.20.1-14.el9_2.1.x86_64.rpm                                                                                   7.0 kB/s |  36 kB     00:05
(4/4): nginx-core-1.20.1-14.el9_2.1.x86_64.rpm                                                                              4.3 MB/s | 565 kB     00:00
------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                        60 kB/s | 634 kB     00:10
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                    1/1
  Running scriptlet: nginx-filesystem-1:1.20.1-14.el9_2.1.noarch                                                                                        1/4
  Installing       : nginx-filesystem-1:1.20.1-14.el9_2.1.noarch                                                                                        1/4
  Installing       : nginx-core-1:1.20.1-14.el9_2.1.x86_64                                                                                              2/4
  Installing       : rocky-logos-httpd-90.14-2.el9.noarch                                                                                               3/4
  Installing       : nginx-1:1.20.1-14.el9_2.1.x86_64                                                                                                   4/4
  Running scriptlet: nginx-1:1.20.1-14.el9_2.1.x86_64                                                                                                   4/4
  Verifying        : rocky-logos-httpd-90.14-2.el9.noarch                                                                                               1/4
  Verifying        : nginx-filesystem-1:1.20.1-14.el9_2.1.noarch                                                                                        2/4
  Verifying        : nginx-1:1.20.1-14.el9_2.1.x86_64                                                                                                   3/4
  Verifying        : nginx-core-1:1.20.1-14.el9_2.1.x86_64                                                                                              4/4

Installed:
  nginx-1:1.20.1-14.el9_2.1.x86_64 nginx-core-1:1.20.1-14.el9_2.1.x86_64 nginx-filesystem-1:1.20.1-14.el9_2.1.noarch rocky-logos-httpd-90.14-2.el9.noarch

Complete!

[root@node1 ~]# sudo systemctl start nginx
```

```powershell
[root@node1 ~]# sudo ss -alnpt | grep nginx
LISTEN 0      511          0.0.0.0:80         0.0.0.0:*    users:(("nginx",pid=41866,fd=6),("nginx",pid=41865,fd=6))
LISTEN 0      511             [::]:80            [::]:*    users:(("nginx",pid=41866,fd=7),("nginx",pid=41865,fd=7))

[root@node1 ~]# sudo firewall-cmd --add-port=80/tcp --permanent
success

[root@node1 ~]# sudo firewall-cmd --reload
success
```

```powershell
[root@node1 ~]# ps -ef | grep nginx
root       41865       1  0 17:30 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx      41866   41865  0 17:30 ?        00:00:00 nginx: worker process
root       41907   41767  0 17:49 pts/0    00:00:00 grep --color=auto nginx

[root@node1 ~]# cat /etc/passwd | grep nginx
nginx:x:991:991:Nginx web server:/var/lib/nginx:/sbin/nologin
```

```powershell
[root@node1 ~]# curl http://10.1.1.44:80 | head -n 7
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
100  7620  100  7620    0     0   930k      0 --:--:-- --:--:-- --:--:-- 1063k
curl: (23) Failed writing body
```


## 2. Analyser la conf de NGINX

```powershell
[root@node1 ~]# sudo ls -al /etc/nginx
total 84
drwxr-xr-x.  4 root root 4096 Jan 29 11:52 .
drwxr-xr-x. 79 root root 8192 Jan 29 16:25 ..
drwxr-xr-x.  2 root root    6 Oct 16 20:00 conf.d
drwxr-xr-x.  2 root root    6 Oct 16 20:00 default.d
-rw-r--r--.  1 root root 1077 Oct 16 20:00 fastcgi.conf
-rw-r--r--.  1 root root 1077 Oct 16 20:00 fastcgi.conf.default
-rw-r--r--.  1 root root 1007 Oct 16 20:00 fastcgi_params
-rw-r--r--.  1 root root 1007 Oct 16 20:00 fastcgi_params.default
-rw-r--r--.  1 root root 2837 Oct 16 20:00 koi-utf
-rw-r--r--.  1 root root 2223 Oct 16 20:00 koi-win
-rw-r--r--.  1 root root 5231 Oct 16 20:00 mime.types
-rw-r--r--.  1 root root 5231 Oct 16 20:00 mime.types.default
-rw-r--r--.  1 root root 2334 Oct 16 20:00 nginx.conf
-rw-r--r--.  1 root root 2656 Oct 16 20:00 nginx.conf.default
-rw-r--r--.  1 root root  636 Oct 16 20:00 scgi_params
-rw-r--r--.  1 root root  636 Oct 16 20:00 scgi_params.default
-rw-r--r--.  1 root root  664 Oct 16 20:00 uwsgi_params
-rw-r--r--.  1 root root  664 Oct 16 20:00 uwsgi_params.default
-rw-r--r--.  1 root root 3610 Oct 16 20:00 win-utf
```

```powershell
[root@node1 ~]# cat /etc/nginx/nginx.conf | grep "server {" -A 30
    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }

# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2;
#        listen       [::]:443 ssl http2;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers PROFILE=SYSTEM;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }
```

```powershell
[root@node1 ~]# cat /etc/nginx/nginx.conf | grep "include"
include /usr/share/nginx/modules/*.conf;
    include             /etc/nginx/mime.types;
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/default.d/*.conf;
#        include /etc/nginx/default.d/*.conf;
```

## 3. Deployer un nouveau site web

```powershell
[root@node1 ~]# ls -al /var
total 20
drwxr-xr-x. 20 root root 4096 Jan 29 18:32 .
dr-xr-xr-x. 18 root root  235 Dec 18 11:57 ..
drwxr-xr-x.  2 root root    6 May 16  2022 adm
drwxr-xr-x.  8 root root   88 Dec 18 11:57 cache
drwxr-xr-x.  2 root root    6 Nov  1 06:39 crash
drwxr-xr-x.  3 root root   18 Dec 18 11:57 db
drwxr-xr-x.  2 root root    6 May 16  2022 empty
drwxr-xr-x.  2 root root    6 May 16  2022 ftp
drwxr-xr-x.  2 root root    6 May 16  2022 games
drwxr-xr-x.  3 root root   18 Dec 18 11:57 kerberos
drwxr-xr-x. 22 root root 4096 Jan 29 11:52 lib
drwxr-xr-x.  2 root root    6 May 16  2022 local
lrwxrwxrwx.  1 root root   11 Dec 18 11:56 lock -> ../run/lock
drwxr-xr-x.  8 root root 4096 Jan 29 11:52 log
lrwxrwxrwx.  1 root root   10 May 16  2022 mail -> spool/mail
drwxr-xr-x.  2 root root    6 May 16  2022 nis
drwxr-xr-x.  2 root root    6 May 16  2022 opt
drwxr-xr-x.  2 root root    6 May 16  2022 preserve
lrwxrwxrwx.  1 root root    6 Dec 18 11:56 run -> ../run
drwxr-xr-x.  6 root root   56 Dec 18 11:57 spool
drwxrwxrwt.  7 root root 4096 Jan 29 18:03 tmp
-rw-r--r--.  1 root root  208 Dec 18 11:57 .updated
drwxr-----.  3 root root   23 Jan 29 18:33 www
drwxr-xr-x.  2 root root    6 May 16  2022 yp

[root@node1 ~]# ls -al /var/www/
total 4
drwxr-----.  3 root root   23 Jan 29 18:33 .
drwxr-xr-x. 20 root root 4096 Jan 29 18:32 ..
drwxr-----.  2 root root   24 Jan 29 18:35 tp3_linux

[root@node1 ~]# ls -al /var/www/tp3_linux/
total 4
drwxr-----. 2 root root 24 Jan 29 18:35 .
drwxr-----. 3 root root 23 Jan 29 18:33 ..
-rwxr-----. 1 root root 15 Jan 29 18:35 index.html
```

```powershell
[root@localhost ~]# echo $RANDOM
14568
[root@node1 ~]# sudo nano /etc/nginx/nginx.conf
[root@node1 ~]# sudo nano /etc/nginx/conf.d/nginxnew.conf
[root@node1 ~]# chmod 744 /etc/nginx/conf.d/nginxnew.conf
[root@node1 ~]# sudo systemctl restart nginx
[root@node1 ~]# sudo firewall-cmd --add-port=14568/tcp --permanent
success
[root@node1 ~]# sudo firewall-cmd --zone=public --permanent --remove-port 80/tcp
success
[root@node1 ~]# sudo firewall-cmd --reload
success

[root@node1 ~]# sudo chown -R nginx /var/www/
[root@node1 ~]# ls -al /var/www/
total 4
drwxr-----.  3 root  root   23 Jan 29 18:33 .
drwxr-xr-x. 20 root  root 4096 Jan 29 18:32 ..
drwxr-----.  2 nginx root   24 Jan 29 18:35 tp3_linux
```

```powershell
[root@node1 ~]# curl http://10.1.1.44:14568
coucou killian
```

# III. Your own services

## 2. Analyse des services existants

```powershell
[root@node1 ~]# systemctl status sshd
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)
     Active: active (running) since Mon 2024-01-29 12:27:02 CET; 7h ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 41754 (sshd)
      Tasks: 1 (limit: 11025)
     Memory: 3.4M
        CPU: 276ms
     CGroup: /system.slice/sshd.service
             └─41754 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Jan 29 17:55:20 node1.tp2.b1 sshd[41925]: Accepted password for root from 10.1.1.1 port 62426 ssh2
Jan 29 17:55:20 node1.tp2.b1 sshd[41925]: pam_unix(sshd:session): session opened for user root(uid=0) by (uid=0)
Jan 29 18:03:03 node1.tp2.b1 sshd[41978]: Accepted password for root from 10.1.1.1 port 62461 ssh2
Jan 29 18:03:03 node1.tp2.b1 sshd[41978]: pam_unix(sshd:session): session opened for user root(uid=0) by (uid=0)
Jan 29 18:55:20 node1.tp2.b1 sshd[42093]: Accepted password for root from 10.1.1.1 port 62702 ssh2
Jan 29 18:55:20 node1.tp2.b1 sshd[42093]: pam_unix(sshd:session): session opened for user root(uid=0) by (uid=0)
Jan 29 19:23:38 node1.tp2.b1 sshd[42226]: Accepted password for root from 10.1.1.1 port 62919 ssh2
Jan 29 19:23:38 node1.tp2.b1 sshd[42226]: pam_unix(sshd:session): session opened for user root(uid=0) by (uid=0)
Jan 29 19:25:17 node1.tp2.b1 sshd[42274]: Accepted password for root from 10.1.1.1 port 62931 ssh2
Jan 29 19:25:17 node1.tp2.b1 sshd[42274]: pam_unix(sshd:session): session opened for user root(uid=0) by (uid=0)
[root@node1 ~]# sudo cat /usr/lib/systemd/system/sshd.service | grep "ExecStart="
ExecStart=/usr/sbin/sshd -D $OPTIONS
[root@node1 ~]# systemctl start sshd
```

```powershell
[root@node1 ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Mon 2024-01-29 19:24:36 CET; 6min ago
    Process: 42269 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 42270 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 42271 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 42272 (nginx)
      Tasks: 2 (limit: 11025)
     Memory: 1.9M
        CPU: 13ms
     CGroup: /system.slice/nginx.service
             ├─42272 "nginx: master process /usr/sbin/nginx"
             └─42273 "nginx: worker process"

Jan 29 19:24:36 node1.tp2.b1 systemd[1]: nginx.service: Deactivated successfully.
Jan 29 19:24:36 node1.tp2.b1 systemd[1]: Stopped The nginx HTTP and reverse proxy server.
Jan 29 19:24:36 node1.tp2.b1 systemd[1]: Starting The nginx HTTP and reverse proxy server...
Jan 29 19:24:36 node1.tp2.b1 nginx[42270]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Jan 29 19:24:36 node1.tp2.b1 nginx[42270]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Jan 29 19:24:36 node1.tp2.b1 systemd[1]: Started The nginx HTTP and reverse proxy server.
[root@node1 ~]# sudo cat /usr/lib/systemd/system/nginx.service | grep "ExecStart="
ExecStart=/usr/sbin/nginx
[root@node1 ~]# systemctl start nginx
```

## 3. Creation de service

```powershell
28945
```