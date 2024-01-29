# Partie 1

## I. Find me.

### 1. Chemin vers le répertoire personnel de l'utilisateur.

```bash
┌──(kali㉿kali)-[~]
└─$ pwd               
/home/kali
```

### 2. Chemin vers les logs SSH

```bash
┌──(kali㉿kali)-[~]
└─$ cd /var/log 
                                                                             
┌──(kali㉿kali)-[/var/log]
└─$ la   
alternatives.log  gvm             notus-scanner  speech-dispatcher
apache2           inetsim         openvpn        stunnel4
apt               journal         postgresql     sysstat
boot.log          lastlog         private        wtmp
btmp              lightdm         README         Xorg.0.log
dpkg.log          macchanger.log  redis          Xorg.0.log.old
faillog           mosquitto       runit          Xorg.1.log
fontconfig.log    nginx           samba          Xorg.1.log.old
                                                                             
┌──(kali㉿kali)-[/var/log]
└─$ pwd
/var/log
```

### 3. Chemin vers les configs SSH

```bash
┌──(kali㉿kali)-[/]
└─$ cd /etc/ssh
                                                                             
┌──(kali㉿kali)-[/etc/ssh]
└─$ la
moduli        sshd_config.d           ssh_host_ed25519_key.pub
ssh_config    ssh_host_ecdsa_key      ssh_host_rsa_key
ssh_config.d  ssh_host_ecdsa_key.pub  ssh_host_rsa_key.pub
sshd_config   ssh_host_ed25519_key
                                                                             
┌──(kali㉿kali)-[/etc/ssh]
└─$ pwd
/etc/ssh
```

## II. Users

### 1. Nouveau User

```bash
┌──(kali㉿kali)-[/etc/ssh]
└─$ sudo useradd -m -d /home/papier_alu marmotte
     
┌──(kali㉿kali)-[/etc/ssh]
└─$ sudo passwd marmotte
New password: 
Retype new password: 
passwd: password updated successfully
                                                                             
┌──(kali㉿kali)-[/etc/ssh]
└─$ cat /etc/passwd
root:x:0:0:root:/root:/usr/bin/zsh
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nolog
...
kali:x:1000:1000:,,,:/home/kali:/usr/bin/zsh
marmotte:x:1001:1001::/home/marmotte:/bin/sh
```

### 2. Infos enregistrées par le système

```bash
┌──(kali㉿kali)-[/]
└─$ cat etc/passwd | grep marmotte
marmotte:x:1001:1001::/home/papier_alu:/bin/sh
```

```bash
┌──(kali㉿kali)-[/]
└─$ sudo cat /etc/shadow | grep marmotte
marmotte:$y$j9T$fg6Em5RPUPXJlg3aaQ/MU.$eT9ZZAZ9v1nULzOsp8xuA1ZHw3RMA9EuNG8qXdV55d/:19744:0:99999:7:::
```

### 3. Connexion avec le nouvel utilisateur

```bash
┌──(kali㉿kali)-[/]
└─$ su - marmotte
Password: 
$ ls -a
.   .bash_logout  .bashrc.original  .face       .java   .profile
..  .bashrc       .config           .face.icon  .local  .zshrc
$ pwd
/home/papier_alu
$ ls /home/kali
ls: cannot open directory '/home/kali': Permission denied
$ exit
                                                                             
┌──(kali㉿kali)-[/]
└─$ 
```

# Partie 2

## I. Programmes et processus

### 1. Run then kill

```
Terminal 1 :
```
```bash                                                
┌──(kali㉿kali)-[/]
└─$ sleep 1000   
```
---
```
Terminal 2 :
```
```bash
┌──(kali㉿kali)-[/]
└─$ ps -ef | grep sleep
kali       19676    1920  0 04:56 pts/0    00:00:00 sleep 1000
kali       20607   19723  0 04:58 pts/1    00:00:00 grep --color=auto sleep
                                                                             
┌──(kali㉿kali)-[/]
└─$ kill 1920 
```

### 2. Taches de fond

```bash
┌──(kali㉿kali)-[/]
└─$ sleep 1000 &
[1] 22648
                                                                             
┌──(kali㉿kali)-[/]
└─$ jobs 
[1]  + running    sleep 1000
                                                                             
┌──(kali㉿kali)-[/]
└─$ kill %1
                                                                             
[1]  + terminated  sleep 1000
```

### 3. Find paths

```bash
┌──(kali㉿kali)-[/]
└─$ which sleep
/usr/bin/sleep

┌──(kali㉿kali)-[/]
└─$ ls -al /usr/bin | grep sleep
-rwxr-xr-x  1 root root         43888 Sep 20  2022 sleep
```

```bash
┌──(kali㉿kali)-[/]
└─$ sudo find / -name "*.bashrc"
/usr/share/kali-defaults/etc/skel/.bashrc
/usr/share/doc/adduser/examples/adduser.local.conf.examples/skel/dot.bashrc
/usr/share/doc/adduser/examples/adduser.local.conf.examples/bash.bashrc
/usr/share/base-files/dot.bashrc
find: ‘/run/user/1000/gvfs’: Permission denied
/etc/skel/.bashrc
/etc/bash.bashrc
/home/papier_alu/.bashrc
/home/marmotte/.bashrc
/home/kali/.bashrc
/root/.bashrc
```

### 4. la variable PATH

```bash
┌──(kali㉿kali)-[/]
└─$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:             /usr/bin:            /sbin:/bin:/usr/local/games:/usr/games
                                                                             
┌──(kali㉿kali)-[/]
└─$ which sleep
/usr/bin/sleep
                                                                             
┌──(kali㉿kali)-[/]
└─$ which ssh  
/usr/bin/ssh
                                                                             
┌──(kali㉿kali)-[/]
└─$ which ping
/usr/bin/ping
```

## II.Paquets

```bash
┌──(kali㉿kali)-[/]
└─$ firefox

┌──(kali㉿kali)-[/]
└─$ which firefox
/usr/bin/firefox

┌──(kali㉿kali)-[/]
└─$ sudo apt install nginx      
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
nginx is already the newest version (1.24.0-2).
nginx set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 951 not upgraded.

┌──(kali㉿kali)-[/]
└─$ which nginx  
/usr/sbin/nginx

┌──(kali㉿kali)-[/]
└─$ ls -a /var/log/nginx       
.  ..  access.log  error.log

┌──(kali㉿kali)-[/]
└─$ nginx -t     
2024/01/22 05:34:23 [warn] 38885#38885: the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /etc/nginx/nginx.conf:1
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
2024/01/22 05:34:23 [emerg] 38885#38885: open() "/run/nginx.pid" failed (13: Permission denied)
nginx: configuration file /etc/nginx/nginx.conf test failed
                                                                             
┌──(kali㉿kali)-[/]
└─$ ls -a /etc/nginx    
.               koi-utf            nginx.conf       snippets
..              koi-win            proxy_params     uwsgi_params
conf.d          mime.types         scgi_params      win-utf
fastcgi.conf    modules-available  sites-available
fastcgi_params  modules-enabled    sites-enabled

┌──(kali㉿kali)-[/]
└─$ grep -v '#' /etc/apt/sources.list | sort -u

deb http://http.kali.org/kali kali-rolling main contrib non-free non-free-firmware
```

# Partie 3

```
Téléchargement du fichier :
```
```bash                                                               
┌──(kali㉿kali)-[~]
└─$ wget https://gitlab.com/it4lik/b1-linux-2023/-/raw/master/tp/2/meow?ref_type=heads&inline=false

```
---
```
Extraction du fichier :
```

```bash
┌──(kali㉿kali)-[~]
└─$ file meow
meow: Zip archive data, at least v2.0 to extract, compression method=deflate
                                                                                                                        
┌──(kali㉿kali)-[~]
└─$ mkdir meow.d    
                                                                                                                        
┌──(kali㉿kali)-[~]
└─$ unzip meow -d meow.d
Archive:  meow
  inflating: meow.d/meow             
                                                                                                                        
┌──(kali㉿kali)-[~]
└─$ la meow.d  
meow
                                                                                                                        
┌──(kali㉿kali)-[~]
└─$ file meow.d/meow
meow.d/meow: XZ compressed data, checksum CRC64
                                                                                                                        
┌──(kali㉿kali)-[~]
└─$ xz --decompress meow           
xz: meow: File format not recognized
                                                                                                                        
┌──(kali㉿kali)-[~]
└─$ xz --decompress meow.d/meow
xz: meow.d/meow: Filename has an unknown suffix, skipping
                                                                                                                        
┌──(kali㉿kali)-[~]
└─$ cd meow.d
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ xz --decompress meow       
xz: meow: Filename has an unknown suffix, skipping
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ mv meow meow.xz 
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ xz --decompress meow.xz    
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ la       
meow
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ file meow       
meow: bzip2 compressed data, block size = 900k
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ mv meow meow.bz2
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ bzip2 -d meow.bz2    
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ la
meow
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ file meow
meow: RAR archive data, v5
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ mv meow meow.rar    
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ 7zr x meow.rar

7-Zip (a) [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,2 CPUs 11th Gen Intel(R) Core(TM) i5-11400H @ 2.70GHz (806D1),ASM,AES-NI)

Scanning the drive for archives:
1 file, 17934823 bytes (18 MiB)

Extracting archive: meow.rar
ERROR: meow.rar
Can not open the file as archive

    
Can't open as archive: 1
Files: 0
Size:       0
Compressed: 0
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ la
meow.rar
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ unrar meow.rar      

UNRAR 7.00 beta 3 freeware      Copyright (c) 1993-2023 Alexander Roshal

Usage:     unrar <command> -<switch 1> -<switch N> <archive> <files...>
               <@listfiles...> <path_to_extract/>

<Commands>
  e             Extract files without archived paths
  l[t[a],b]     List archive contents [technical[all], bare]
  p             Print file to stdout
  t             Test archive files
  v[t[a],b]     Verbosely list archive contents [technical[all],bare]
  x             Extract files with full path

<Switches>
  -             Stop switches scanning
  @[+]          Disable [enable] file lists
  ad[1,2]       Alternate destination path
  ag[format]    Generate archive name using the current date
  ai            Ignore file attributes
  ap<path>      Set path inside archive
  c-            Disable comments show
  cfg-          Disable read configuration
  cl            Convert names to lower case
  cu            Convert names to upper case
  dh            Open shared files
  ep            Exclude paths from names
  ep3           Expand paths to full including the drive letter
  ep4<path>     Exclude the path prefix from names
  f             Freshen files
  id[c,d,n,p,q] Display or disable messages
  ierr          Send all messages to stderr
  inul          Disable all messages
  kb            Keep broken extracted files
  me[par]       Set encryption parameters
  n<file>       Additionally filter included files
  n@            Read additional filter masks from stdin
  n@<list>      Read additional filter masks from list file
  o[+|-]        Set the overwrite mode
  ol[a,-]       Process symbolic links as the link [absolute paths, skip]
  op<path>      Set the output path for extracted files
  or            Rename files automatically
  ow            Save or restore file owner and group
  p[password]   Set password
  r             Recurse subdirectories
  sc<chr>[obj]  Specify the character set
  si[name]      Read data from standard input (stdin)
  sl<size>[u]   Process files with size less than specified
  sm<size>[u]   Process files with size more than specified
  ta[mcao]<d>   Process files modified after <d> YYYYMMDDHHMMSS date
  tb[mcao]<d>   Process files modified before <d> YYYYMMDDHHMMSS date
  tn[mcao]<t>   Process files newer than <t> time
  to[mcao]<t>   Process files older than <t> time
  ts[m,c,a,p]   Save or restore time (modification, creation, access, preserve)
  u             Update files
  v             List all volumes
  ver[n]        File version control
  vp            Pause before each volume
  x<file>       Exclude specified file
  x@            Read file names to exclude from stdin
  x@<list>      Exclude files listed in specified list file
  y             Assume Yes on all queries

                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ unrar x meow.rar

UNRAR 7.00 beta 3 freeware      Copyright (c) 1993-2023 Alexander Roshal


Extracting from meow.rar

Extracting  meow                                                      OK 
All OK
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ la
meow  meow.rar
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ file meow
meow: gzip compressed data, from Unix, original size modulo 2^32 145049600 gzip compressed data, reserved method, has CRC, extra field, has comment, from FAT filesystem (MS-DOS, OS/2, NT), original size modulo 2^32 145049600
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ rm meow.rar
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ la
meow
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ mv meow meow.gz 
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ gzip -d meow.gz                                                                                                   
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ la
meow
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ file meow
meow: POSIX tar archive (GNU)
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ mv meow meow.tar
                                                                                                                        
┌──(kali㉿kali)-[~/meow.d]
└─$ tar xvf meow.tar          
```
---
```
Chercher les fichiers :
```
```bash

```