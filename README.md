# TP-Prometheus-Grafana-Nagios

Par Clément JELEFF

# Partie 1 : Prometheus & Grafana
### 1) installer Prometheus & Grafana sous docker (faire un net host pour partager le réseau)

#### Prometheus

''' 
   prometheus:
    image: prom/prometheus:latest
    ports:
      - 9090:9090
    volumes:
      - ./prometheus:/etc/prometheus
      - ./prometheus/data:/var/lib/prometheus
    command: --web.enable-lifecycle  --config.file=/etc/prometheus/prometheus.yml
    networks:
      - hosts
'''

#### Grafana

'''
 image: grafana/grafana:latest
    ports:
      - 3000:3000
    volumes:
      - ./grafana:/var/lib/grafana
      - ./grafana/grafana.ini:/etc/grafana/grafana.ini
    environment:
      - GF_PATHS_CONFIG=/etc/grafana/grafana.ini
    depends_on:
      - prometheus
    networks:
      - hosts
'''


### 2) créer un node exporter de votre machine

#### Node Exporter
'''
node-exporter:
    image: prom/node-exporter
    ports:
      - 9100:9100
    networks:
      - hosts
'''
### 3) remonter les métrique du CPU et de l’espace disque utilisé

#### CPU mectric

> 100 - (avg by (instance) (irate(node_cpu_seconds_total[5m])) * 100)

#### Disk space mectric
> 100 - ((node_filesystem_avail_bytes * 100) / node_filesystem_size_bytes)

### 4) Créer des labels pour les métriques en question



### 5) sur grafana, importer un template, mettre un temps de rafraîchissement toutes les 20 secondes et mettre un seuil d’alerte à 75% d’utilisation pour les métriques cités ci dessus.

#### Template import
![Import Dashboard](https://user-images.githubusercontent.com/56600546/161280140-63478103-f846-482a-b525-9e9723a251bd.PNG)

#### Rafraîchissement 20s
![autorefresh](https://user-images.githubusercontent.com/56600546/161280187-d9c378c0-a5cf-45ac-8e30-a53bff08e101.PNG)

#### Alerte 75%

### 6) Sous docker, mettre en place un serveur de stockage (ex: nextcloud)


### 7) faire un node exporter pour votre serveur cloud et mettre la target sur Prometheus
### 8) remonter les métriques permettant de voir la bande passante de ce serveur (si possible essayer de simuler l’envoie de données vers votre serveur cloud)
#### Bonus : mettre en place la découverte automatique de vos conteneurs


# Partie 2 : Nagios
### 1) Installer Nagios Core sur un serveur linux

Nagios 4.4.6 à été installé sur une VM Ubuntu

User: jean
MDP : jean123

### 2) Installer les plugins nécessaires

>sudo apt install -y build-essential apache2 php openssl perl make php-gd libgd-dev libapache2-mod-php libperl-dev libssl-dev daemon wget apache2-utils unzip

### 3) Définir des alias de commandes pour les commandes utilisés dans la suite du TP

> 

### 4) Exécuter le plugin (commande) check_ping sous le compte nagios avec un seuil WARNING à 20 ms et un seuil CRITICAL à 30 % de paquets perdus.
'''
 ./check_ping -H 192.168.1.78 -w 20,20% -c 30,30%
PING OK -  Paquets perdus = 0%, RTA = 0.03 ms|rta=0.033000ms;20.000000;30.000000;0.000000 pl=0%;20;30;0
'''
### 5) faites la meme commande pour sonder https://duckduckgo.com avec 10ms pour le WARNING et 20% pour le CRITICAL

'''
./check_ping -H 40.114.177.156 -w 10,10% -c 20,20%
PING CRITICAL -  Paquets perdus = 100%|rta=20.000000ms;10.000000;20.000000;0.000000 pl=100%;10;20;0
'''


### 6) Installer le plugin check_http et executer le sur votre propre serveur, aidez vous de la commande -help pour les arguments (utilisez les argument -H, -u et -a dans la même commande et observez le résultat)
'''
root@jean-virtual-machine:/usr/local/nagios/libexec# ./check_http -H 192.168.1.78 -u /nagios/ -a nagiosadmin:jean123
HTTP OK: HTTP/1.1 200 OK - 1248 octets en 0,002 secondes de temps de réponse |time=0,002097s;;;0,000000 size=1248B;;;0
'''
### 7) Supervisez son serveur Nagios en créant un fichier “serveur_nagios.cfg” et en trouvant la configuration adéquate ( avec command, host, service… )

> cd /usr/local/nagios/libexec/ 

'''
 define command {

    command_name    check_ping_localhost
    command_line /usr/local/nagios/libexec/check_ping -H 192.168.1.78 -w 20,20% -c 80,80%
}
'''
'''
 # Define a host for the local machine

define host {

    use                     linux-server            ; Name of host template to use
                                                    ; This host definition will inherit all variables that are defined
                                                    ; in (or inherited by) the linux-server host template definition.
    host_name               server_nagios
    alias                   server_nagios
    address                 192.168.1.78
'''

'''
# Define a service to "ping" the local machine

define service {

    use                     local-service           ; Name of service template to use
    host_name               server_nagios
    service_description     PING
    check_command           check_ping_localhost
}
'''


## Déclaration du  répertoire créé dans le fichier nagios.cfg


#### Dans /usr/local/nagios/etc/nagios.cfg :
> cfg_file=/usr/local/nagios/etc/objects/serveur_nagios.cfg


### 8) Vérifier le bon fonctionnement sur l'interface web

### Screenshot Host
![host](https://user-images.githubusercontent.com/56600546/161245355-3444fdb6-546b-44d0-afd8-51ece8ba4f33.PNG)

### Screenshot Host
![services](https://user-images.githubusercontent.com/56600546/161246014-01262deb-eb10-44fb-bc4f-93f2595cad1e.PNG)

 #### Bonus : Créer une VM linux supplémentaire et installer NRPE

Sur votre serveur nagios,configurer NRPE et vérifier son bon fonctionnement sur l’interface web
