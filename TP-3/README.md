# Service SSH

## Analyse dy service

```bash
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

```bash
[root@localhost ~]# ps -ef | grep sshd
root         716       1  0 10:31 ?        00:00:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
root        1279     716  0 10:37 ?        00:00:00 sshd: root [priv]
root        1283    1279  0 10:37 ?        00:00:00 sshd: root@pts/0
root        1331    1284  0 10:46 pts/0    00:00:00 grep --color=auto sshd
```

```bash
[root@localhost ~]# sudo ss -alnpt | grep sshd
LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=716,fd=3))
LISTEN 0      128             [::]:22           [::]:*    users:(("sshd",pid=716,fd=4))
```

```bash
[root@localhost ~]# journalctl | grep sshd
Jan 29 10:31:40 localhost systemd[1]: Created slice Slice /system/sshd-keygen.
Jan 29 10:31:44 localhost systemd[1]: Reached target sshd-keygen.target.
Jan 29 10:31:46 localhost.localdomain sshd[716]: Server listening on 0.0.0.0 port 22.
Jan 29 10:31:46 localhost.localdomain sshd[716]: Server listening on :: port 22.
Jan 29 10:37:23 localhost.localdomain sshd[1279]: Accepted password for root from 10.1.1.1 port 59855 ssh2
Jan 29 10:37:23 localhost.localdomain sshd[1279]: pam_unix(sshd:session): session opened for user root(uid=0) by (uid=0)
```

## 2. Modification du service

```bash
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

```bash
[root@localhost ~]# echo $RANDOM
25346
[root@localhost ~]# sudo nano /etc/ssh/sshd_config
[root@localhost ~]#sudo nano /etc/ssh/sshd_config
[root@localhost ~]# sudo cat /etc/ssh/sshd_config | grep Port
Port 25346
#GatewayPorts no
```

```bash

```