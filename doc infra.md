Projet infra-SI : 
Mise en place d'un site internet
=
Nicolas De Bengy
Leo Sery
Matthias Flament
Louis Merlaud
***

__Installation de nginx sur CENTOS 7 :__

à faire avec la commande sudo : 
```
yum update
yum install epel-release
yum install nginx
systemctl start nginx
systemctl enable nginx
```

***

__Config firewall :__

```
firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --permanent --zone=public --add-service=https
firewall-cmd --reload
```

***

__Création serveur web en go :__

installation (from binary) de golang : 
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

Information complémentaire concernant le monitoring :

Voir les stats du sites : RAM, utilisation du CPU, débit entrant/sortant, etc...

Ces informations nous permetterons d'optimiser le site au fil du temps. 

***

__Mise en place de Fail2Ban :__

InstallerFail2Ban :
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

remplir le fichier avec ce qui suit : 
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

redémarage du firewall
```
* systemctl enable firewalld
* systemctl start firewalld
```
redémarrage du firewall
```
* systemctl enable fail2ban
* systemctl start fail2ban 
```

__Installation de Rkhunter :__

(Rkhunter est actuellement un logiciel obsolète)
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

__backup indispensable :__

projet web : 
* backend (golang dans notre cas)
* frontend (html et css)
* base de donnée

__monitoring :__

* nombre de personne présente sur le site
* vérifier si le site et up ou down
* le nombre d'incident présent
