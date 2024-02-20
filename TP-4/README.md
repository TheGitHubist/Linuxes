# Partie 1 : partitionnement du serveur de stockage

```powershell
[root@localhost ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   20G  0 disk
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0   19G  0 part
  ├─rl-root 253:0    0   17G  0 lvm  /
  └─rl-swap 253:1    0    2G  0 lvm  [SWAP]
sdb           8:16   0    2G  0 disk
sdc           8:32   0    2G  0 disk
sr0          11:0    1 1024M  0 rom
```

```powershell
[root@localhost ~]# sudo pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
[root@localhost ~]# sudo pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.
[root@localhost ~]# sudo pvs
  PV         VG Fmt  Attr PSize   PFree
  /dev/sda2  rl lvm2 a--  <19.00g     0
  /dev/sdb      lvm2 ---   <2.01g <2.01g
  /dev/sdc      lvm2 ---    2.00g  2.00g
```

```powershell
[root@localhost ~]# sudo vgcreate storage /dev/sdb
  Volume group "storage" successfully created
[root@localhost ~]# sudo vgextend storage /dev/sdc
  Volume group "storage" successfully extended
[root@localhost ~]# sudo vgs
  VG      #PV #LV #SN Attr   VSize   VFree
  rl        1   2   0 wz--n- <19.00g    0
  storage   2   0   0 wz--n-   4.00g 4.00g
```

```powershell
[root@localhost ~]# sudo lvs
  LV        VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root      rl      -wi-ao---- <17.00g
  swap      rl      -wi-ao----   2.00g
  storageLV storage -wi-a-----   4.00g
```

```powershell
[root@localhost ~]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/storage/storageLV
  LV Name                storageLV
  VG Name                storage
  LV UUID                dIGkMO-0awg-MeWc-cIs3-wNl8-GtiC-27Ztob
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2024-02-19 11:13:12 +0100
  LV Status              available
  # open                 0
  LV Size                4.00 GiB
  Current LE             1025
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2

  --- Logical volume ---
  LV Path                /dev/rl/swap
  LV Name                swap
  VG Name                rl
  LV UUID                wPf9Bw-w4CI-Mlj6-HyN4-IN4d-tNlt-6r68Ye
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2023-12-18 11:56:31 +0100
  LV Status              available
  # open                 2
  LV Size                2.00 GiB
  Current LE             512
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1

  --- Logical volume ---
  LV Path                /dev/rl/root
  LV Name                root
  VG Name                rl
  LV UUID                dMfixz-b4qe-L7vI-oZde-2lKQ-1wiv-6Ets8c
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2023-12-18 11:56:31 +0100
  LV Status              available
  # open                 1
  LV Size                <17.00 GiB
  Current LE             4351
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0

[root@localhost ~]# mkfs -t ext4 /dev/storage/storageLV
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 1049600 4k blocks and 262416 inodes
Filesystem UUID: e3b5b101-0400-4e42-b204-5c31211f07f3
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

```powershell
[root@localhost ~]# mkdir /mnt/storage

[root@localhost ~]# df -h | grep storage
/dev/mapper/storage-storageLV  3.9G   24K  3.7G   1% /mnt/storage
```

```powershell
[root@localhost ~]# nano /etc/fstab
[root@localhost ~]# cat /etc/fstab | grep storage
/dev/storage/storageLV  /mnt/storage            ext4    defaults        0 0

[root@localhost ~]# sudo umount /mnt/storage/
[root@localhost ~]# sudo mount -av
/                        : ignored
/boot                    : already mounted
none                     : ignored
mount: /mnt/storage does not contain SELinux labels.
       You just mounted a file system that supports labels which does not
       contain labels, onto an SELinux box. It is likely that confined
       applications will generate AVC messages and not be allowed access to
       this file system.  For more details see restorecon(8) and mount(8).
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
/mnt/storage             : successfully mounted
```


# Partie 2 : 

## Serveur

```powershell
[root@localhost ~]# sudo dnf install nfs-utils
[root@localhost ~]# sudo mkdir /var/nfs/general -p

[root@localhost ~]# sudo chown nobody /var/nfs/general
[root@localhost ~]# ls -dl /var/nfs/general
drwxr-xr-x. 2 nobody root 6 Feb 19 11:50 /var/nfs/general

[root@localhost ~]# sudo nano /etc/exports
[root@localhost ~]# cat /etc/exports
/var/nfs/general    10.1.1.42(rw,sync,no_subtree_check)


[root@localhost ~]# sudo systemctl status nfs-server
○ nfs-server.service - NFS server and services
     Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; disabled; preset: disabled)
    Drop-In: /run/systemd/generator/nfs-server.service.d
             └─order-with-mounts.conf
     Active: inactive (dead)
[root@localhost ~]# sudo systemctl enable nfs-server
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
[root@localhost ~]# sudo systemctl start nfs-server
[root@localhost ~]# sudo systemctl status nfs-server
● nfs-server.service - NFS server and services
     Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; preset: disabled)
    Drop-In: /run/systemd/generator/nfs-server.service.d
             └─order-with-mounts.conf
     Active: active (exited) since Mon 2024-02-19 11:57:24 CET; 2s ago
    Process: 11711 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
    Process: 11712 ExecStart=/usr/sbin/rpc.nfsd (code=exited, status=0/SUCCESS)
    Process: 11729 ExecStart=/bin/sh -c if systemctl -q is-active gssproxy; then systemctl reload gssproxy ; fi (code=exited, status=0/SUCCESS)
   Main PID: 11729 (code=exited, status=0/SUCCESS)
        CPU: 13ms

Feb 19 11:57:24 localhost.localdomain systemd[1]: Starting NFS server and services...
Feb 19 11:57:24 localhost.localdomain systemd[1]: Finished NFS server and services.


[root@localhost ~]# firewall-cmd --permanent --list-all | grep services
  services: cockpit dhcpv6-client ssh

[root@localhost ~]#firewall-cmd --permanent --add-service=nfs
  success
[root@localhost ~]#firewall-cmd --permanent --add-service=mountd
  success
[root@localhost ~]#firewall-cmd --permanent --add-service=rpc-bind
  success
[root@localhost ~]#firewall-cmd --reload
  success


[root@localhost ~]# firewall-cmd --permanent --list-all | grep services
  services: cockpit dhcpv6-client mountd nfs rpc-bind ssh
```
---
## Client

```powershell
[root@localhost ~]# sudo dnf install nfs-utils

[root@localhost ~]# sudo mkdir -p /nfs/general
[root@localhost ~]# sudo mount 10.1.1.44:/var/nfs/general /nfs/general
[root@localhost ~]# df -h
Filesystem                  Size  Used Avail Use% Mounted on
devtmpfs                    4.0M     0  4.0M   0% /dev
tmpfs                       882M     0  882M   0% /dev/shm
tmpfs                       353M  5.0M  348M   2% /run
/dev/mapper/rl-root          17G  1.2G   16G   8% /
/dev/sda1                   960M  223M  738M  24% /boot
tmpfs                       177M     0  177M   0% /run/user/0
10.1.1.44:/var/nfs/general   17G  1.2G   16G   8% /nfs/general

[root@localhost ~]# sudo nano /etc/fstab
[root@localhost ~]# cat /etc/fstab | grep general
10.1.1.44:/var/nfs/general    /nfs/general   nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0

[root@localhost ~]# sudo umount /nfs/general
[root@localhost ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                882M     0  882M   0% /dev/shm
tmpfs                353M  5.0M  348M   2% /run
/dev/mapper/rl-root   17G  1.2G   16G   8% /
/dev/sda1            960M  223M  738M  24% /boot
tmpfs                177M     0  177M   0% /run/user/0
```


# Partie 3 : Serveur web

## 2. Install

```powershell
[root@localhost ~]# sudo dnf install nginx
```

## 3. Analyse

```powershell
[root@localhost ~]# sudo systemctl enable nginx
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /usr/lib/systemd/system/nginx.service.
[root@localhost ~]# sudo systemctl start nginx
[root@localhost ~]# sudo systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: disabled)
     Active: active (running) since Mon 2024-02-19 12:25:48 CET; 5s ago
    Process: 2794 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 2795 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 2796 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 2797 (nginx)
      Tasks: 2 (limit: 11025)
     Memory: 1.9M
        CPU: 15ms
     CGroup: /system.slice/nginx.service
             ├─2797 "nginx: master process /usr/sbin/nginx"
             └─2798 "nginx: worker process"

Feb 19 12:25:48 localhost.localdomain systemd[1]: Starting The nginx HTTP and reverse proxy server...
Feb 19 12:25:48 localhost.localdomain nginx[2795]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Feb 19 12:25:48 localhost.localdomain nginx[2795]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Feb 19 12:25:48 localhost.localdomain systemd[1]: Started The nginx HTTP and reverse proxy server.
```

```powershell
[root@localhost ~]# ps -ef | grep nginx
root        2797       1  0 12:25 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx       2798    2797  0 12:25 ?        00:00:00 nginx: worker process
root        2867    2526  0 19:24 pts/0    00:00:00 grep --color=auto nginx

[root@localhost ~]# sudo ss -alnpt | grep nginx
LISTEN 0      511          0.0.0.0:80        0.0.0.0:*    users:(("nginx",pid=2798,fd=6),("nginx",pid=2797,fd=6))
LISTEN 0      511             [::]:80           [::]:*    users:(("nginx",pid=2798,fd=7),("nginx",pid=2797,fd=7))
```

```powershell
[root@localhost nginx]# cat /etc/nginx/nginx.conf | grep server -A 20
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
```

```powershell
[root@localhost nginx]# ls -al /usr/share/nginx/html
total 12
drwxr-xr-x. 3 root root  143 Feb 19 12:22 .
drwxr-xr-x. 4 root root   33 Feb 19 12:22 ..
-rw-r--r--. 1 root root 3332 Oct 16 19:58 404.html
-rw-r--r--. 1 root root 3404 Oct 16 19:58 50x.html
drwxr-xr-x. 2 root root   27 Feb 19 12:22 icons
lrwxrwxrwx. 1 root root   25 Oct 16 20:00 index.html -> ../../testpage/index.html
-rw-r--r--. 1 root root  368 Oct 16 19:58 nginx-logo.png
lrwxrwxrwx. 1 root root   14 Oct 16 20:00 poweredby.png -> nginx-logo.png
lrwxrwxrwx. 1 root root   37 Oct 16 20:00 system_noindex_logo.png -> ../../pixmaps/system-noindex-logo.png
```

## 4 : Visite du site web

```powershell
[root@localhost nginx]# sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[root@localhost nginx]# sudo firewall-cmd --add-port=80/tcp --permanent
success
[root@localhost nginx]# sudo firewall-cmd --reload
success
[root@localhost nginx]# sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: cockpit dhcpv6-client ssh
  ports: 80/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

```powershell
[root@localhost nginx]# curl http://10.1.1.42
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/

      html {
        height: 100%;
        width: 100%;
      }
        body {
  background: rgb(20,72,50);
  background: -moz-linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%)  ;
  background: -webkit-linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%) ;
  background: linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%);
  background-repeat: no-repeat;
  background-attachment: fixed;
  filter: progid:DXImageTransform.Microsoft.gradient(startColorstr="#3c6eb4",endColorstr="#3c95b4",GradientType=1);
        color: white;
        font-size: 0.9em;
        font-weight: 400;
        font-family: 'Montserrat', sans-serif;
        margin: 0;
        padding: 10em 6em 10em 6em;
        box-sizing: border-box;

      }


  h1 {
    text-align: center;
    margin: 0;
    padding: 0.6em 2em 0.4em;
    color: #fff;
    font-weight: bold;
    font-family: 'Montserrat', sans-serif;
    font-size: 2em;
  }
  h1 strong {
    font-weight: bolder;
    font-family: 'Montserrat', sans-serif;
  }
  h2 {
    font-size: 1.5em;
    font-weight:bold;
  }

  .title {
    border: 1px solid black;
    font-weight: bold;
    position: relative;
    float: right;
    width: 150px;
    text-align: center;
    padding: 10px 0 10px 0;
    margin-top: 0;
  }

  .description {
    padding: 45px 10px 5px 10px;
    clear: right;
    padding: 15px;
  }

  .section {
    padding-left: 3%;
   margin-bottom: 10px;
  }

  img {

    padding: 2px;
    margin: 2px;
  }
  a:hover img {
    padding: 2px;
    margin: 2px;
  }

  :link {
    color: rgb(199, 252, 77);
    text-shadow:
  }
  :visited {
    color: rgb(122, 206, 255);
  }
  a:hover {
    color: rgb(16, 44, 122);
  }
  .row {
    width: 100%;
    padding: 0 10px 0 10px;
  }

  footer {
    padding-top: 6em;
    margin-bottom: 6em;
    text-align: center;
    font-size: xx-small;
    overflow:hidden;
    clear: both;
  }

  .summary {
    font-size: 140%;
    text-align: center;
  }

  #rocky-poweredby img {
    margin-left: -10px;
  }

  #logos img {
    vertical-align: top;
  }

  /* Desktop  View Options */

  @media (min-width: 768px)  {

    body {
      padding: 10em 20% !important;
    }

    .col-md-1, .col-md-2, .col-md-3, .col-md-4, .col-md-5, .col-md-6,
    .col-md-7, .col-md-8, .col-md-9, .col-md-10, .col-md-11, .col-md-12 {
      float: left;
    }

    .col-md-1 {
      width: 8.33%;
    }
    .col-md-2 {
      width: 16.66%;
    }
    .col-md-3 {
      width: 25%;
    }
    .col-md-4 {
      width: 33%;
    }
    .col-md-5 {
      width: 41.66%;
    }
    .col-md-6 {
      border-left:3px ;
      width: 50%;


    }
    .col-md-7 {
      width: 58.33%;
    }
    .col-md-8 {
      width: 66.66%;
    }
    .col-md-9 {
      width: 74.99%;
    }
    .col-md-10 {
      width: 83.33%;
    }
    .col-md-11 {
      width: 91.66%;
    }
    .col-md-12 {
      width: 100%;
    }
  }

  /* Mobile View Options */
  @media (max-width: 767px) {
    .col-sm-1, .col-sm-2, .col-sm-3, .col-sm-4, .col-sm-5, .col-sm-6,
    .col-sm-7, .col-sm-8, .col-sm-9, .col-sm-10, .col-sm-11, .col-sm-12 {
      float: left;
    }

    .col-sm-1 {
      width: 8.33%;
    }
    .col-sm-2 {
      width: 16.66%;
    }
    .col-sm-3 {
      width: 25%;
    }
    .col-sm-4 {
      width: 33%;
    }
    .col-sm-5 {
      width: 41.66%;
    }
    .col-sm-6 {
      width: 50%;
    }
    .col-sm-7 {
      width: 58.33%;
    }
    .col-sm-8 {
      width: 66.66%;
    }
    .col-sm-9 {
      width: 74.99%;
    }
    .col-sm-10 {
      width: 83.33%;
    }
    .col-sm-11 {
      width: 91.66%;
    }
    .col-sm-12 {
      width: 100%;
    }
    h1 {
      padding: 0 !important;
    }
  }


  </style>
  </head>
  <body>
    <h1>HTTP Server <strong>Test Page</strong></h1>

    <div class='row'>

      <div class='col-sm-12 col-md-6 col-md-6 '></div>
          <p class="summary">This page is used to test the proper operation of
            an HTTP server after it has been installed on a Rocky Linux system.
            If you can read this page, it means that the software is working
            correctly.</p>
      </div>

      <div class='col-sm-12 col-md-6 col-md-6 col-md-offset-12'>


        <div class='section'>
          <h2>Just visiting?</h2>

          <p>This website you are visiting is either experiencing problems or
          could be going through maintenance.</p>

          <p>If you would like the let the administrators of this website know
          that you've seen this page instead of the page you've expected, you
          should send them an email. In general, mail sent to the name
          "webmaster" and directed to the website's domain should reach the
          appropriate person.</p>

          <p>The most common email address to send to is:
          <strong>"webmaster@example.com"</strong></p>

          <h2>Note:</h2>
          <p>The Rocky Linux distribution is a stable and reproduceable platform
          based on the sources of Red Hat Enterprise Linux (RHEL). With this in
          mind, please understand that:

        <ul>
          <li>Neither the <strong>Rocky Linux Project</strong> nor the
          <strong>Rocky Enterprise Software Foundation</strong> have anything to
          do with this website or its content.</li>
          <li>The Rocky Linux Project nor the <strong>RESF</strong> have
          "hacked" this webserver: This test page is included with the
          distribution.</li>
        </ul>
        <p>For more information about Rocky Linux, please visit the
          <a href="https://rockylinux.org/"><strong>Rocky Linux
          website</strong></a>.
        </p>
        </div>
      </div>
      <div class='col-sm-12 col-md-6 col-md-6 col-md-offset-12'>
        <div class='section'>

          <h2>I am the admin, what do I do?</h2>

        <p>You may now add content to the webroot directory for your
        software.</p>

        <p><strong>For systems using the
        <a href="https://httpd.apache.org/">Apache Webserver</strong></a>:
        You can add content to the directory <code>/var/www/html/</code>.
        Until you do so, people visiting your website will see this page. If
        you would like this page to not be shown, follow the instructions in:
        <code>/etc/httpd/conf.d/welcome.conf</code>.</p>

        <p><strong>For systems using
        <a href="https://nginx.org">Nginx</strong></a>:
        You can add your content in a location of your
        choice and edit the <code>root</code> configuration directive
        in <code>/etc/nginx/nginx.conf</code>.</p>

        <div id="logos">
          <a href="https://rockylinux.org/" id="rocky-poweredby"><img src="icons/poweredby.png" alt="[ Powered by Rocky Linux ]" /></a> <!-- Rocky -->
          <img src="poweredby.png" /> <!-- webserver -->
        </div>
      </div>
      </div>

      <footer class="col-sm-12">
      <a href="https://apache.org">Apache&trade;</a> is a registered trademark of <a href="https://apache.org">the Apache Software Foundation</a> in the United States and/or other countries.<br />
      <a href="https://nginx.org">NGINX&trade;</a> is a registered trademark of <a href="https://">F5 Networks, Inc.</a>.
      </footer>

  </body>
</html>
```

```powershell
[root@localhost ~]# cat /var/log/nginx/access.log | tail -3
10.1.1.1 - - [19/Feb/2024:19:43:06 +0100] "GET /poweredby.png HTTP/1.1" 200 368 "http://10.1.1.42/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36 OPR/106.0.0.0" "-"
10.1.1.1 - - [19/Feb/2024:19:43:06 +0100] "GET /favicon.ico HTTP/1.1" 404 3332 "http://10.1.1.42/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36 OPR/106.0.0.0" "-"
10.1.1.42 - - [19/Feb/2024:19:43:23 +0100] "GET / HTTP/1.1" 200 7620 "-" "curl/7.76.1" "-"
```

```powershell
[root@localhost ~]# sudo nano /etc/nginx/nginx.conf
[root@localhost ~]# cat /etc/nginx/nginx.conf | grep listen
        listen       8080;
        listen       [::]:8080;
#        listen       443 ssl http2;
#        listen       [::]:443 ssl http2;
```

```powershell
[root@localhost ~]# sudo systemwtl restart nginx
[root@localhost ~]# sudo systemwtl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: disabled)
     Active: active (running) since Mon 2024-02-19 20:16:09 CET; 6s ago
    Process: 2949 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 2950 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 2951 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 2952 (nginx)
      Tasks: 2 (limit: 11025)
     Memory: 1.9M
        CPU: 18ms
     CGroup: /system.slice/nginx.service
             ├─2952 "nginx: master process /usr/sbin/nginx"
             └─2954 "nginx: worker process"

Feb 19 20:16:09 localhost.localdomain systemd[1]: nginx.service: Deactivated successfully.
Feb 19 20:16:09 localhost.localdomain systemd[1]: Stopped The nginx HTTP and reverse proxy server.
Feb 19 20:16:09 localhost.localdomain systemd[1]: Starting The nginx HTTP and reverse proxy server...
Feb 19 20:16:09 localhost.localdomain nginx[2950]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Feb 19 20:16:09 localhost.localdomain nginx[2950]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Feb 19 20:16:09 localhost.localdomain systemd[1]: Started The nginx HTTP and reverse proxy server.
```

## 5. Modif de la conf du serveur web

```powershell
[root@localhost ~]# sudo ss -alnpt | grep nginx
LISTEN 0      511          0.0.0.0:8080      0.0.0.0:*    users:(("nginx",pid=2954,fd=6),("nginx",pid=2952,fd=6))
LISTEN 0      511             [::]:8080         [::]:*    users:(("nginx",pid=2954,fd=7),("nginx",pid=2952,fd=7))
```

```powershell
[root@localhost ~]# sudo firewall-cmd --remove-port=80/tcp --permanent
success
[root@localhost ~]# sudo firewall-cmd --add-port=8080/tcp --permanent
success
[root@localhost ~]# sudo firewall-cmd --reload
success
[root@localhost ~]# sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: cockpit dhcpv6-client ssh
  ports: 8080/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

```powershell
[root@localhost ~]# curl http://10.1.1.42:8080
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/

      html {
        height: 100%;
        width: 100%;
      }
        body {
  background: rgb(20,72,50);
  background: -moz-linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%)  ;
  background: -webkit-linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%) ;
  background: linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%);
  background-repeat: no-repeat;
  background-attachment: fixed;
  filter: progid:DXImageTransform.Microsoft.gradient(startColorstr="#3c6eb4",endColorstr="#3c95b4",GradientType=1);
        color: white;
        font-size: 0.9em;
        font-weight: 400;
        font-family: 'Montserrat', sans-serif;
        margin: 0;
        padding: 10em 6em 10em 6em;
        box-sizing: border-box;

      }


  h1 {
    text-align: center;
    margin: 0;
    padding: 0.6em 2em 0.4em;
    color: #fff;
    font-weight: bold;
    font-family: 'Montserrat', sans-serif;
    font-size: 2em;
  }
  h1 strong {
    font-weight: bolder;
    font-family: 'Montserrat', sans-serif;
  }
  h2 {
    font-size: 1.5em;
    font-weight:bold;
  }

  .title {
    border: 1px solid black;
    font-weight: bold;
    position: relative;
    float: right;
    width: 150px;
    text-align: center;
    padding: 10px 0 10px 0;
    margin-top: 0;
  }

  .description {
    padding: 45px 10px 5px 10px;
    clear: right;
    padding: 15px;
  }

  .section {
    padding-left: 3%;
   margin-bottom: 10px;
  }

  img {

    padding: 2px;
    margin: 2px;
  }
  a:hover img {
    padding: 2px;
    margin: 2px;
  }

  :link {
    color: rgb(199, 252, 77);
    text-shadow:
  }
  :visited {
    color: rgb(122, 206, 255);
  }
  a:hover {
    color: rgb(16, 44, 122);
  }
  .row {
    width: 100%;
    padding: 0 10px 0 10px;
  }

  footer {
    padding-top: 6em;
    margin-bottom: 6em;
    text-align: center;
    font-size: xx-small;
    overflow:hidden;
    clear: both;
  }

  .summary {
    font-size: 140%;
    text-align: center;
  }

  #rocky-poweredby img {
    margin-left: -10px;
  }

  #logos img {
    vertical-align: top;
  }

  /* Desktop  View Options */

  @media (min-width: 768px)  {

    body {
      padding: 10em 20% !important;
    }

    .col-md-1, .col-md-2, .col-md-3, .col-md-4, .col-md-5, .col-md-6,
    .col-md-7, .col-md-8, .col-md-9, .col-md-10, .col-md-11, .col-md-12 {
      float: left;
    }

    .col-md-1 {
      width: 8.33%;
    }
    .col-md-2 {
      width: 16.66%;
    }
    .col-md-3 {
      width: 25%;
    }
    .col-md-4 {
      width: 33%;
    }
    .col-md-5 {
      width: 41.66%;
    }
    .col-md-6 {
      border-left:3px ;
      width: 50%;


    }
    .col-md-7 {
      width: 58.33%;
    }
    .col-md-8 {
      width: 66.66%;
    }
    .col-md-9 {
      width: 74.99%;
    }
    .col-md-10 {
      width: 83.33%;
    }
    .col-md-11 {
      width: 91.66%;
    }
    .col-md-12 {
      width: 100%;
    }
  }

  /* Mobile View Options */
  @media (max-width: 767px) {
    .col-sm-1, .col-sm-2, .col-sm-3, .col-sm-4, .col-sm-5, .col-sm-6,
    .col-sm-7, .col-sm-8, .col-sm-9, .col-sm-10, .col-sm-11, .col-sm-12 {
      float: left;
    }

    .col-sm-1 {
      width: 8.33%;
    }
    .col-sm-2 {
      width: 16.66%;
    }
    .col-sm-3 {
      width: 25%;
    }
    .col-sm-4 {
      width: 33%;
    }
    .col-sm-5 {
      width: 41.66%;
    }
    .col-sm-6 {
      width: 50%;
    }
    .col-sm-7 {
      width: 58.33%;
    }
    .col-sm-8 {
      width: 66.66%;
    }
    .col-sm-9 {
      width: 74.99%;
    }
    .col-sm-10 {
      width: 83.33%;
    }
    .col-sm-11 {
      width: 91.66%;
    }
    .col-sm-12 {
      width: 100%;
    }
    h1 {
      padding: 0 !important;
    }
  }


  </style>
  </head>
  <body>
    <h1>HTTP Server <strong>Test Page</strong></h1>

    <div class='row'>

      <div class='col-sm-12 col-md-6 col-md-6 '></div>
          <p class="summary">This page is used to test the proper operation of
            an HTTP server after it has been installed on a Rocky Linux system.
            If you can read this page, it means that the software is working
            correctly.</p>
      </div>

      <div class='col-sm-12 col-md-6 col-md-6 col-md-offset-12'>


        <div class='section'>
          <h2>Just visiting?</h2>

          <p>This website you are visiting is either experiencing problems or
          could be going through maintenance.</p>

          <p>If you would like the let the administrators of this website know
          that you've seen this page instead of the page you've expected, you
          should send them an email. In general, mail sent to the name
          "webmaster" and directed to the website's domain should reach the
          appropriate person.</p>

          <p>The most common email address to send to is:
          <strong>"webmaster@example.com"</strong></p>

          <h2>Note:</h2>
          <p>The Rocky Linux distribution is a stable and reproduceable platform
          based on the sources of Red Hat Enterprise Linux (RHEL). With this in
          mind, please understand that:

        <ul>
          <li>Neither the <strong>Rocky Linux Project</strong> nor the
          <strong>Rocky Enterprise Software Foundation</strong> have anything to
          do with this website or its content.</li>
          <li>The Rocky Linux Project nor the <strong>RESF</strong> have
          "hacked" this webserver: This test page is included with the
          distribution.</li>
        </ul>
        <p>For more information about Rocky Linux, please visit the
          <a href="https://rockylinux.org/"><strong>Rocky Linux
          website</strong></a>.
        </p>
        </div>
      </div>
      <div class='col-sm-12 col-md-6 col-md-6 col-md-offset-12'>
        <div class='section'>

          <h2>I am the admin, what do I do?</h2>

        <p>You may now add content to the webroot directory for your
        software.</p>

        <p><strong>For systems using the
        <a href="https://httpd.apache.org/">Apache Webserver</strong></a>:
        You can add content to the directory <code>/var/www/html/</code>.
        Until you do so, people visiting your website will see this page. If
        you would like this page to not be shown, follow the instructions in:
        <code>/etc/httpd/conf.d/welcome.conf</code>.</p>

        <p><strong>For systems using
        <a href="https://nginx.org">Nginx</strong></a>:
        You can add your content in a location of your
        choice and edit the <code>root</code> configuration directive
        in <code>/etc/nginx/nginx.conf</code>.</p>

        <div id="logos">
          <a href="https://rockylinux.org/" id="rocky-poweredby"><img src="icons/poweredby.png" alt="[ Powered by Rocky Linux ]" /></a> <!-- Rocky -->
          <img src="poweredby.png" /> <!-- webserver -->
        </div>
      </div>
      </div>

      <footer class="col-sm-12">
      <a href="https://apache.org">Apache&trade;</a> is a registered trademark of <a href="https://apache.org">the Apache Software Foundation</a> in the United States and/or other countries.<br />
      <a href="https://nginx.org">NGINX&trade;</a> is a registered trademark of <a href="https://">F5 Networks, Inc.</a>.
      </footer>

  </body>
</html>
```

```powershell
[root@localhost nginx]# sudo nano /var/www/site_web_1/index.html
[root@localhost nginx]# cat /etc/nginx/nginx.conf
    <!DOCTYPE HTML>
    <html>
        <head>
        </head>
        <body>
            <h1> Sup bro </h1>
        </body>
    </html>
[root@localhost nginx]# sudo nano /etc/nginx/nginx.conf
[root@localhost nginx]# cat /etc/nginx/nginx.conf | grep server -A 20
    server {
        listen       8080;
        listen       [::]:8080;
        server_name  _;
        root         /var/www/site_web_1;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
```

```powershell
[root@localhost nginx]# sudo systemctl restart nginx
[root@localhost nginx]# curl http://10.1.1.42:8080
    <!DOCTYPE HTML>
    <html>
        <head>
        </head>
        <body>
            <h1> Sup bro </h1>
        </body>
    </html>
```

## 6. Deux sites web sur un seul serveur

```powershell
[root@localhost ~]# cat /etc/nginx/nginx.conf | grep include
include /usr/share/nginx/modules/*.conf;
    include             /etc/nginx/mime.types;
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/default.d/*.conf;
#        include /etc/nginx/default.d/*.conf;
```

```powershell
[root@localhost nginx]# sudo nano /etc/nginx/nginx.conf
[root@localhost nginx]# sudo nano /etc/nginx/conf.d/site_web_1.conf
[root@localhost nginx]# cat /etc/nginx/conf.d/site_web_1.conf | grep server -A 20
    server {
        listen       8080;
        listen       [::]:8080;
        server_name  _;
        root         /var/www/site_web_1;
        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
        error_page 404 /404.html;
        location = /404.html {
        }
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
```

```powershell
[root@localhost nginx]# nano /var/www/site_web_2/index.html
[root@localhost nginx]# cat /var/www/site_web_2/index.html
    <!doctype html>
    <html>
        <head>
        </head>
        <body>
            <h1> Sup sista </h1>
        </body>
    </html>
[root@localhost nginx]# sudo nano /etc/nginx/nginx.conf
[root@localhost nginx]# sudo nano /etc/nginx/conf.d/site_web_1.conf
[root@localhost nginx]# cat /etc/nginx/conf.d/site_web_1.conf | grep server -A 20
    server {
        listen       8888;
        listen       [::]:8888;
        server_name  _;
        root         /var/www/site_web_2;
        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
        error_page 404 /404.html;
        location = /404.html {
        }
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }

[root@localhost ~]# sudo firewall-cmd --add-port=8888/tcp --permanent
success
[root@localhost ~]# sudo firewall-cmd --reload
success
[root@localhost ~]# sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: cockpit dhcpv6-client ssh
  ports: 8080/tcp 8888/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

```powershell
[root@localhost nginx]# curl http://10.1.1.42:8080
    <!doctype html>
    <html>
        <head>
        </head>
        <body>
            <h1> Sup bro </h1>
        </body>
    </html>
[root@localhost nginx]# curl http://10.1.1.42:8888
    <!doctype html>
    <html>
        <head>
        </head>
        <body>
            <h1> Sup sista </h1>
        </body>
    </html>
```