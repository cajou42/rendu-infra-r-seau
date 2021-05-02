Projet infra-SI : 
Mise en place d'un site internet
=
_Nicolas de Bengy_
_Leo Sery_
_Matthias Flament_
_Louis Merlaud_
***
### __Prérequis :__
* Une machine avec Centos 7

***

__Installation de nginx :__

```
sudo yum update
sudo yum install epel-release
sudo yum install nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

***

__Config firewall :__

```
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```

***

__Création serveur web en go :__

Installation (from binary) de golang : 
```
wget https://dl.google.com/go/go1.13.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.13.linux-amd64.tar.gz
vi  ~/.bash_profile
export PATH=$PATH:/usr/local/go/bin

sudo mkdir $GOPATH/go-web

sudo cd $GOPATH/go-web

```
Création du Systemd Service:
```
sudo nano /etc/systemd/system/carpentic.service
```
```
[Unit]
Description="Carpentic golang web server"

[Service]
ExectStart=/home/admin/go/carpentic/src/main
WorkingDirectory=/home/admin/go/carpentic/src/
User=admin

[Install]
WantedBy=default.target
```
***

__Installation du monitoring (netdata) :__

```
sudo yum update

sudo yum install epel-release

sudo yum install gcc make git curl zlib-devel git automake libuuid-devel libmnl autoconf pkgconfig findutils

git clone https://github.com/netdata/netdata.git --depth=100

cd netdata/

./packaging/installer/install-required-packages.sh --dont-wait --non-interactive netdata

sudo ./netdata-installer.sh

sudo systemctl start netdata

sudo systemctl enable netdata

sudo systemctl status netdata

sudo netstat -pnltu | grep netdata

sudo firewall-cmd --add-port=19999/tcp --permanent

sudo firewall-cmd --reload

(à entrer dans l'url)
http://centos8-ip:19999/
```
 
***

__Mise en place de Fail2Ban :__

Installer Fail2Ban :
```
* yum install epel-release
* yum install fail2ban fail2ban-systemd
```
Configuration de Fail2Ban :
```
* cp -pf /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
* nano /etc/fail2ban/jail.local
```

Protection du ssh :

* nano /etc/fail2ban/jail.d/sshd.local

```
[sshd]
enabled = true
port = ssh
#action = firewallcmd-ipset
logpath = %(sshd_log)s
maxretry = 5
bantime = 86400
```

Démarage de Fail2Ban :

Redémarage du firewall
```
* systemctl enable firewalld
* systemctl start firewalld
```
Redémarrage du firewall
```
* systemctl enable fail2ban
* systemctl start fail2ban 
```

__Installation de Rkhunter :__

(Rkhunter est actuellement un logiciel obsolète, il faudrait aujourd'hui utiliser Lynis ou Chkrootkit)
```
yum install rkhunter
cp /etc/rkhunter.conf{,.ori}
vi /etc/rkhunter.conf
```
```
COPY_LOG_ON_ERROR=1
ALLOW_SSH_ROOT_USER=no
```
```
rkhunter --update
rkhunter --propupdate
```

***
Pour le backup, les dossiers importants sont le fichiers de conf de nginx, la base de donnée ainsi que les fichiers de frontend et backend.

__Backup avec logrotate :__

Logrotate va, grâce à un script, compresser les fichiers incriminés et va les enregistrer dans le dossier ~/backupfolder chaque semaine. Si le nombre de backup est supérieur à 5, le plus vieux backup sera supprimé.

__Configuration et installation de logrotate :__ 

```
sudo yum update && sudo yum install logrotate
mkdir ~/backupfolder
```
* logrotate.conf :
```
sudo nano /etc/logrotate.conf

# see "man logrotate" for details
# rotate log files weekly
weekly

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
dateext

# uncomment this if you want your log files compressed
#compress

# RPM packages drop log rotation information into this directory
include /etc/logrotate.d

# no packages own wtmp and btmp -- we'll rotate them here
/var/log/wtmp {
    monthly
    create 0664 root utmp
    rotate 5
}

/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 5
}

~/backupfolder {
    daily
    rotate 5
    sharedscripts
    postrotate
    $(scriptbackup.sh)
    endscript
}

# system-specific logs may be also be configured here.
```

* Script :
```
sudo nano ~/scriptbackup.sh

#!/bin/bash


tar -zcvf backup_$(date +'%d-%m-%Y_%H') ~/go/carpentic/src
mv backup_$(date +'%d-%m-%Y_%H') ~/backupfolder
FILENUMBER=$(ls ~/backupfolder | wc -l)
BACKUPLIMIT=5

if (("$FILENUMBER" > "$BACKUPLIMIT")); then
        rm ~/backupfolder/$(ls -t ~/backupfolder/ | tail -1)
fi
```



__Monitoring :__

* Nombre de personnes présentent sur le site
* Vérifier si le site est up ou down
* La santé de la base de donnée
* Un service de ticketing comme OsTicket
