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