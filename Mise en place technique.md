# **Notre mise en place technique d'un SIEM**


- vm-network : 

Debian sans interface |Mise a disposition d'un DNS |d'un DHCP Domaine DNS : haribo.lan

1. Installer clamAv  | clamaV daemon | search antimalware en real-time | OK
2. installer portsentry  | OK
3. installer openssh : 
	- root auth désactivé même par clé | OK
	- log de chaque connexon échouée ou réussie ( log type : **VERBOSE** )| OK ![image](https://user-images.githubusercontent.com/111575087/221578677-3949cd7a-ad83-4000-82ad-7969eda09282.png)
![image](https://user-images.githubusercontent.com/111575087/221578707-3f3b1ba7-d1bd-4258-a270-d9942b21eecc.png)

4. Installer fail2ban
	- 2 tentatives de connections échoues en 6h : ban 2 minutes | OK
	
5. Installer Filebeat
	- 
6. Installer Metricbeat
	 
vm-gate : 

DNS auto
Debian sans interface ||Fournis l'accès web aux machines vm-servers et vm-users
- mise à disposition d'un proxy http permettant d'accèder à l'intranet et au web pour VM-server et VM-user
- Toutes les requêtes externe doivent passer par le proxy, intra ou inter
-  *recherche d'amélioration du filtrage proxy*
- Passerelle de toutes les machines
1. Installer clamAV
2. Installer portsentry
3. installer openssh :
	- root désactivé même par clé
	- 2 tentatives échoues en 6h : ban 2 minutes
	- log de chaque connexion échouée ou réussie ( log type : **VERBOSE** )

vm-servers : 

DNS auto| IP fixe auto |Debian sans interface |
Serveur apache :
	- si 3 tentatives échoues en 6h : ban 24h
- intranet : intra.haribo.lan
- le programme doit être executé sous user : intranet-server
- dossier home de intranet-server : log/ & www/
- **log:** log d'erreur et de connection 
- **www:** doit contenir les fichiers mis à disposition
- authentification basique obligatoire
- version du serveur http ne doit pas apparaître
- SSL/TLS a mettre en place | certificats signés par CA Haribo
- redirection 80 vers 443
dossiers log/ et www/ ne peuvent être lus écrits ou executes par les autres users de la machine sauf le super user ( root )
**Trouver une amélioration de la sécurité de l'internet**



1. Installer clamAV
2. installer Portsentry
3. installer openssh :
	- root désactivé même par clé
	- 2 tentatives échoues en 6h : ban 2 minutes
	- log de chaque connexion échouée ou réussie
### **Proposition d'amélioration de la sécurité SSH**
Première amélioration: modification du port SSH ( 22 => 2220)

vm-users : 

Debian graphique
IP auto
DNS auto
1. Installer clamAV
2. installer Portsentry
3. installer openssh :
	- root désactivé même par clé
	- 2 tentatives échoues en 6h : ban 2 minutes
	- log de chaque connexion échouée ou réussie

Installation de Metric & filebeat



SIEM Composition
##########################################################################################"
Hebergé sur AZURE


- Outils de collecte elasticsearch
- dispositif d'alerte : Filebeat & Metricbeat
- Envoi de log vers https://elasticsearch 
- les certificat doivent provenir d'haribo CA (Configurer filebeat pour envoyer logs en SSL)
- chaque service qui envoi des logs disposera de son propres indexe: 
	- fail2ban : 
			detection brute force sur SSH
			
	- Security ( clamAv ): 
			detection malware
			
	- HTTP ( Apache2 ): 
			detection de brute force
			
	- portscan ( Portsentry):
			detection de scan
			
	- proxy ( Squid ):
			detection si tentative de site interdit
			
	- health ( Metricbeat Module system) :
			Consommation RAM | CPU | Disk 
			states check filebeat / metricbeat
						

- Kibana
	- 3 utilisateurs : analyste : Ne peut creer d'indexe | peut creer des dashboards
	 		   client : Ne peut creer d'indexe/données | ni de dashboards | ne peut que visualisé les dashboards des analysts et les données des indexes
			   admin : Accès admin sur toute les fonctionnalités
			   
	- Possibilité d'effectuer une recherche de log grâce au paramètre 'timestamp' en fonction de leur date de création
	

Create a file named authors.hrb
	- 1st line : Noms en majuscule spéarés par des virgules et dans l'ordre croissant des membres du group suivi d'un retour à la ligne
	- 2e ligne : Prénoms en minuscule, séparés par des birgules et dans l'ordre croissant des membres du groupe suivi d'un retout à la ligne
	 

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
 Si je desire le personnaliser afin qu'il bloque, il faut: Recherchez la ligne **BLOCK_UDP="1"** et changez-la en **BLOCK_UDP="0"** pour désactiver le blocage des paquets UDP. 
 
 1. Modifier le fichier : **/etc/default/portsentry ou /etc/portsentry/portsentry.ignore.static**
 2. A l'aide de nano nous allons ouvrir et modifier le fichier : **nano /etc/portsentry/portsentry.conf**
 Dans ce fichier, onr echerche la section " TCP_PORTS " et ajoutez les numéros de port qu'on souhaite sureillé. Par exemple, pour surveiller les port 22(ssh) et 80 (http).
 3. cela donne : **TCP_PORTS="22,80"**
 4. Par sécurité il faut rechercher la ligne #**BLOCK_TCP="1" S'assuré quel est bien commenté
 5. ENsuite ouvrir le fichier : **nano /etc/portsentry/portsentry.ignore.static**
 6. Pour verifiez que les informations de balayage de port sont enregistrées dans le fichier de journalisation la cmd est : **tail-f /var/log/syslog | grep portsentry**
 7. Pour annulé le blocage d'ip : Recherchez la ligne **KILL_ROUTE="/sbin/route add -host $TARGET$ reject"**
### **Proposition d'amélioration**
Actuellement nous ne blocons pas les IPs qui scans mais les log.
- nous pourrions implémenté un blocages ip+log 

## **Les CA d'autorité Elasticsearch** 

Pour utiliser des certificats dans Elasticsearch, vous devez créer une autorité de certification qui émettra des certificats pour chaque composant du système qui doit communiquer de manière sécurisée. Voici les étapes générales pour configurer une autorité de certification pour Elasticsearch :

1. Générer une **clé privée pour l'autorité de certification (CA).**

2. Utiliser la clé privée pour **générer un certificat de l'autorité de certification.**

3. **Distribuer le certificat de l'autorité de certification** à tous les composants Elasticsearch qui doivent communiquer de manière sécurisée.

4. Générer une clé privée **pour chaque composant Elasticsearch** qui doit communiquer de manière sécurisée.

5. Utiliser chaque clé privée pour générer un certificat pour chaque composant Elasticsearch.

6. **Signer chaque certificat avec le certificat de l'autorité de certification.**

