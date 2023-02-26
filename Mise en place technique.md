# **Notre mise en place technique d'un SIEM**


vm-network : 

Debian sans interface |Mise a disposition d'un DNS |d'un DHCP Domaine DNS : haribo.lan 
1. Installer clamAv
2. installer portsentry
3. installer openssh : 
	- root auth désactivé même par clé
	- 2 tentatives de connections échoues en 6h : ban 2 minutes
	- log de chaque connexon échouée ou réussie

vm-gate : 

DNS auto
Debian sans interface ||Fournis l'accès web aux machines vm-servers et vm-users
- mise à disposition d'un proxy http permettant d'accèder à l'intranet et au web
- Toutes les requêtes doivent passer par le proxy, intra ou inter
- Passerelle de toutes les machines
1. Installer clamAV
2. Installer portsentry
3. installer openssh :
	- root désactivé même par clé
	- 2 tentatives échoues en 6h : ban 2 minutes
	- log de chaque connexion échouée ou réussie

vm-servers : 

DNS auto| IP fixe auto |Debian sans interface |
Serveur apache :
	- si 3 tentatives échoues en 6h : ban 24h
- intranet : intra.haribo.lan
- le programme doit être executé sous user : intranet-server
- dossier home de intranet-server : log/ & www/
- authentification basique obligatoire
- version du serveur http ne doit pas apparaître
- SSL/TLS a mettre en place
- certificats signés par CA Haribo
- redirection 80 vers 443
dossiers log/ et www/ ne peuvent être lus écrits ou executes par les autres users de la machine sauf le super user

1. Installer clamAV
2. installer Portsentry
3. installer openssh :
	- root désactivé même par clé
	- 2 tentatives échoues en 6h : ban 2 minutes
	- log de chaque connexion échouée ou réussie


vm-users : 

Debian graphique
IP auto
DNS auto

# **Mise en place technique de chaque config**

## **Config DNS:**
sudo nano /etc/bind/named.conf.options
acl "trusted" {
        10.0.0.10;    # ns1 
        10.0.0.11;    # ns2
        10.0.0.12;  # host1
        10.0.0.13;  # host2
};

options {
        directory "/var/cache/bind";
        
        recursion yes;                 # enables recursive queries
        allow-recursion { trusted; };  # allows recursive queries from "trusted" clients
        listen-on { 10.0.0.11; };   # ns1 private IP address - listen on private network only
        allow-transfer { none; };      # disable zone transfers by default

        forwarders {
                8.8.8.8;
                8.8.4.4;
        };

       sudo nano /etc/bind/named.conf.local
       systmctl restart bind9
       sudo mkdir /etc/bind/zones
       sudo cp /etc/bind/db.local /etc/bind/zones/db.haribo.lan
       
       $TTL    604800
@       IN      SOA     vm-network.haribo.lan root.haribo.lan (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
![2023-02-26_17h27_47](https://user-images.githubusercontent.com/111575087/221423195-cfaad2b7-3468-42c8-a925-ddc9e55f07d6.png)

## **Config du DHCP**
![2023-02-26_17h29_44](https://user-images.githubusercontent.com/111575087/221423383-9eda40fa-7d82-4c5e-a4fb-a3798089cdf9.png)

![2023-02-26_17h31_29](https://user-images.githubusercontent.com/111575087/221423400-e4c3cb88-4a7b-44f1-81dc-24a3cb0d4a18.png)
 # **PortSentry config*
 - sudo apt-get install portsentry
 ## *De base la configuration de portsentry par defaut ne bloque rien*
 Si je desire le personnaliser afin qu'il bloque, il faut: 
 1. Modifier le fichier : **/etc/default/portsentry ou /etc/portsentry/portsentry.ignore.static**
 2. A l'aide de nano nous allons ouvrir et modifier le fichier : **nano /etc/portsentry/portsentry.conf**
 Dans ce fichier, onr echerche la section " TCP_PORTS " et ajoutez les numéros de port qu'on souhaite sureillé. Par exemple, pour surveiller les port 22(ssh) et 80 (http).
 3. cela donne : TCP_PORTS="22,80"
 4. Par sécurité il faut rechercher la ligne #**BLOCK_TCP="1" S'assuré quel est bien commenté
 5. ENsuite ouvrir le fichier : nano /etc/portsentry/portsentry.ignore.static
 6. Pour verifiez que les informations de balayage de port sont enregistrées dans le fichier de journalisation la cmd est : **tail-f /var/log/syslog | grep portsentry**
