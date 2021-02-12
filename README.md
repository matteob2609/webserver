# webserver
Breve documentazione per inizializzare un server web Ubuntu (versione in questione: 20.04.1 LTS)

:heavy_exclamation_mark: Usare **sudo** prima del comando se non si hanno i diritti necessari.

**Link per visualizzare direttamente la parte interessata:**

[Configurazione IP & installazione pacchetti necessari](https://github.com/matteob2609/webserver#ghost-configurazione-ip--installazione-pacchetti-necessari)

[Installazione SSH & Apache2](https://github.com/matteob2609/webserver#ghost-installazione-ssh--apache2)

[Attivazione HTTPS su Apache2](https://github.com/matteob2609/webserver#ghost-attivazione-https-su-apache2)

[Attivazione del servizio FTP](https://github.com/matteob2609/webserver#ghost-attivazione-del-servizio-ftp)

[Installazione e configurazione del servizio Samba (SMB)](https://github.com/matteob2609/webserver#ghost-installazione-e-configurazione-del-servizio-samba-smb)

[Installazione Tomcat 9](https://github.com/matteob2609/webserver#ghost-installazione-tomcat-9)

---

### :ghost: CONFIGURAZIONE IP & INSTALLAZIONE PACCHETTI NECESSARI

- _nano /etc/hosts_ 

- _nano /etc/hostname_

:pushpin:`Checkpoint: immettere il comando 'reboot' per riavviare il server web e visualizzare l'hostname aggiornato. `

      reboot

- _nano /etc/netplan/00-installer-config.yaml_

`Versione senza DHCP, indirizzi statici`

      network:
        renderer: networkd
        ethernets:
          enp0s3:
            addresses: [172.16.29.105/16]
            gateway4: 172.16.1.7
            nameservers:
                addresses: [172.16.1.10, 1.1.1.1]
         version: 2

`Versione con DHCP, indirizzi dinamici`

        network:
        version: 2
        renderer: networkd
        ethernets:
          enp0s3:
            dhcp4: true
            dhcp6: true

`Questa configurazione IP è stata fatta all'interno della scuola e riguarda il mio caso in particolare. Se il server è virtualizzato occorre impostare la scheda di rete in modalità Bridge.`

- _netplan try_

:pushpin:`Checkpoint: verificare se l'indirizzo è stato modificato correttamente tramite il comando 'ip addr' e verificare la connessione con il comando 'ping'.`

      ip addr
      ping www.google.com

- _apt update_

- _apt upgrade_

[Torna su](https://github.com/matteob2609/webserver#webserver)

---

### :ghost: INSTALLAZIONE SSH & APACHE2

**Introduzione SSH:** protocollo che stabilisce una connessione remota cifrata tramite un'interfaccia a riga di comando con un altro host della rete. Ha sostituito il protocollo insicuro Telnet.

**Introduzione Apache2:** è il server web libero piu' diffuso al mondo, in grado di operare su una grande vastità di S.O. Ha il vantaggio di offrire controlli per la sicurezza come quelle effettuate da un proxy.

**1. Installazione**

- _apt install openssh-server_

- _apt install apache2_

:pushpin:`Checkpoint: verificare l'installazione del server Apache aprendo il browser e mettendo nella barra degli indirizzi l'IP del server web, se viene visualizzata la pagina di default di Apache l'installazione è andata a buon fine.`

      172.16.29.105/index.html
      
**2. Creazione di un utente per un sito specifico**

- _useradd -s /bin/bash -d /var/www/'nome_cartella_sito' -m 'nome_user'_

- _passwd 'password'_

:pushpin:`Checkpoint: fare il login da remoto per verificare la corretta aggiunta dell'utente.`

**3. Aggiunta del sito web**

- _cd /var/www/'nome_cartella_sito'_

- _mkdir log_

- _cd /etc/apache2/sites-available_

- _cp 000-default.conf 'nome_cartella_sito'.conf_

- _nano 'nome_cartella_sito'.conf_

`Togliere il commento nella riga ServerName e inserire il dominio del sito; In DocumentRoot inserire il percorso della cartella del sito; In ErrorLog e CustomLog togliere il $, le parentesi graffe e inserire il percorso della cartella del sito /log.`

**4. Abilitazione del sito**

- _a2ensite 'file_configurazione_sito'.conf_

- _systemctl restart apache2.service_

:pushpin:`Checkpoint: immettere nella barra degli indirizzi del browser 'nome_cartella_sito.dominio', se vengono visualizzati i file inseriti dall'utente registrato il sito funziona correttamente. Di seguito alcuni esempi:`

      sitoa-105.virtual.marconi
      sitob-105.virtual.marconi
      ubuntu-srv105.virtual.marconi

[Torna su](https://github.com/matteob2609/webserver#webserver)

---

### :ghost: ATTIVAZIONE HTTPS SU APACHE2

**Azione preliminare. Controllo della versione OpenSSL**

- _openssl version -a_

:pushpin:`Checkpoint: Controllare l'output, dovrà essere simile al seguente:`

      OpenSSL 1.1.1f  31 Mar 2020
      built on: Mon Apr 20 11:53:50 2020 UTC
      platform: debian-amd64
      options:  bn(64,64) rc4(16x,int) des(int) blowfish(ptr)
      compiler: gcc -fPIC -pthread -m64 -Wa,--noexecstack ...
      OPENSSLDIR: "/usr/lib/ssl"
      ENGINESDIR: "/usr/lib/x86_64-linux-gnu/engines-1.1"
      Seeding source: os-specific
      
- _a2enmod ssl_, in caso SSL non fosse installato.
      
**1. Creazione del certificato SSL**

- _openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/'nome_chiave'.key -out /etc/ssl/certs/'nome_certificato'.crt_

      Country Name (2 letter code) [AU]:
      State or Province Name (full name) [Some-State]:
      Locality Name (eg, city) []:
      Organization Name (eg, company) [Internet Widgits Pty Ltd]:
      Organizational Unit Name (eg, section) []:
      Common Name (e.g. server FQDN or YOUR name) []:sitoa-105.virtual.marconi
      Email Address []:
      
- _openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048_

**2. Configurare Apache per utilizzare SSL**

- _nano /etc/apache2/conf-available/ssl-params.conf_

      # from https://cipherli.st/
      # and https://raymii.org/s/tutorials/Strong_SSL_Security_On_Apache2.html

      SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH
      SSLProtocol All -SSLv2 -SSLv3
      SSLHonorCipherOrder On
      # Disable preloading HSTS for now.  You can use the commented out header line that includes
      # the "preload" directive if you understand the implications.
      #Header always set Strict-Transport-Security "max-age=63072000; includeSubdomains; preload"
      Header always set Strict-Transport-Security "max-age=63072000; includeSubdomains"
      Header always set X-Frame-Options DENY
      Header always set X-Content-Type-Options nosniff
      # Requires Apache >= 2.4
      SSLCompression off
      SSLSessionTickets Off
      SSLUseStapling on
      SSLStaplingCache "shmcb:logs/stapling-cache(150000)"

      SSLOpenSSLConfCmd DHParameters "/etc/ssl/certs/dhparam.pem"
      
- _cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/default-ssl.conf.bak_, prima di modificare il file di configurazione di SSL creiamo un backup del file stesso.

- _nano /etc/apache2/sites-available/default-ssl.conf_

      <IfModule mod_ssl.c>
              <VirtualHost _default_:443>
                      ServerAdmin webmaster@localhost
                      ServerName sitoa-105.virtual.marconi

                      DocumentRoot /var/www/html

                      ErrorLog ${APACHE_LOG_DIR}/error.log
                      CustomLog ${APACHE_LOG_DIR}/access.log combined

                      SSLEngine on

                      SSLCertificateFile      /etc/ssl/certs/'nome_certificato'.crt
                      SSLCertificateKeyFile /etc/ssl/private/'nome_chiave'.key

                      <FilesMatch "\.(cgi|shtml|phtml|php)$">
                                      SSLOptions +StdEnvVars
                      </FilesMatch>
                      <Directory /usr/lib/cgi-bin>
                                      SSLOptions +StdEnvVars
                      </Directory>

                      BrowserMatch "MSIE [2-6]" \
                                     nokeepalive ssl-unclean-shutdown \
                                     downgrade-1.0 force-response-1.0

              </VirtualHost>
      </IfModule>
      
**3. Modificare le impostazioni del firewall per permette il traffico in entrata e uscita**

- _ufw allow 'Apache Full'_

- _ufw delete allow 'Apache'_

- _ufw status_, visualizzare lo stato del firewall.

:pushpin:`Checkpoint: lo status del firewall dovrà essere simile al seguente:`

      Status: active

      To                         Action      From
      --                         ------      ----
      OpenSSH                    ALLOW       Anywhere
      Apache Full                ALLOW       Anywhere
      OpenSSH (v6)               ALLOW       Anywhere (v6)
      Apache Full (v6)           ALLOW       Anywhere (v6)
      
**4. Controllare che le modifiche siano avvenute con successo**

:pushpin:`Checkpoint: immettere il comando 'apache2ctl configtest' per verificare che non ci siano errori di sintassi. Output:`

      AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message
      Syntax OK
      
- _systemctl restart apache2_, solo se il checkpoint non ha dato errori.

**5. Test**

Digitare sul browser _https://sitoa-105.virtual.marconi_ (questo è il mio caso in particolare, il dominio o l'IP può essere differente in base alla configurazione effettuata).
Visto che il certificato non è verificato e confermato da nessuna autorità, riceveremo questo avvertimento:

![your-connection-is-not-private](https://user-images.githubusercontent.com/61114792/107025705-11765100-67aa-11eb-8a1b-a7b0a90ca0a2.png)

Per procedere fare click su 'Advanced' e fare un click sul link per continuare comunque la connessione al sito.

:heavy_exclamation_mark: **N.B. il lucchetto sarà segnato da una X, questo vuol dire che il certificato non è confermato, ma la connessione è comunque criptata.** :heavy_exclamation_mark:
      
[Torna su](https://github.com/matteob2609/webserver#webserver)

---

### :ghost: ATTIVAZIONE DEL SERVIZIO FTP

**Introduzione:** FTP (File Transfer Protocol) è un protocollo utilizzato per il trasferimento di dati basato su un sistema client-server. Consente di caricare, scaricare e spostare file all'interno di un sistema di directory.

- _apt install vsftpd_

- _nano /etc/vsftpd.conf_

      listen=yes

      listen_ipv6=NO

      anonymous_enable=NO

      local_enable=YES

      write_enable=YES

      local_umask=022

      dirmessage_enable=YES

      use_localtime=YES

      xferlog_enable=YES

      connect_from_port_20=YES

      xferlog_file=/var/log/vsftpd.log

      xferlog_std_format=YES

      ftpd_banner=Welcome to our  FTP service.

      chroot_local_user=YES

      local_root=/var/www/$USER

      user_sub_token=$USER

      allow_writeable_chroot=YES

      secure_chroot_dir=/var/run/vsftpd/empty

      pam_service_name=vsftpd

      rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem

      rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key

      ssl_enable=NO

      session_support=YES

      log_ftp_protocol=YES

- _systemctl start vsftpd_

- _systemctl enable vsftpd_

[Torna su](https://github.com/matteob2609/webserver#webserver)

---

### :ghost: INSTALLAZIONE E CONFIGURAZIONE DEL SERVIZIO SAMBA (SMB)

**Introduzione:** SMB (Server Message Block) è un protocollo utilizzato soprattutto dai Microsoft Windows, principalmente per condividere file, stampanti, porte seriali e comunicazioni di varia natura tra diversi nodi di una rete.
Include anche un processo di comunicazione tra processi autenticata.

- _apt update_

- _apt install samba_

:pushpin:`Checkpoint: immettere il comando 'whereis samba' e verificare l'output, che deve essere il seguente:`
      
      samba: /usr/sbin/samba /usr/lib/samba /etc/samba /usr/share/samba /usr/share/man/man7/samba.7.gz /usr/share/man/man8/samba.8.gz
      
- _mkdir /home/'nome_user'/sambashare/_

- _nano /etc/samba/smb.conf_ e inserire in fondo al file queste righe di testo:

      [sambashare]
         comment=Samba on Ubuntu
         path=/home/'nome_user'/sambashare
         read only=no
         browsable=yes
      
      #Condivisione di una cartella
      [shared]
      comment = Cartella condivisa        # Commento sulla condivisione
      path = /cartella/da/condividere     # Percorso della condivisione
      browseable = yes                    # Rende visibile la condivisione
      public = yes                        # Rende la cartella accessibile
      create mask = 0755                  # Permessi dei file
      
      #Condivisione della home
      [homes]
      comment = Home condivisa            # Commento sulla condivisione
      browseable = yes                    # Rende visibile la condivisione
      valid users = %S                    # Utente che vi può accedere
      create mask = 0700                  # Permessi dei file
      directory mask = 0700               # Permessi della home

- _service smbd restart_

- _ufw allow samba_, permette al firewall di lasciar passare il traffico in entrata e in uscita che riguarda il servizio SMB.

- _smbpasswd -a 'nome_user'_

:pushpin:`Checkpoint per Ubuntu: aprire il default file manager, fare click su Connect to Server e inserire 'smb://ip-address/sambashare'.`

:pushpin:`Checkpoint per macOS: nel menu Finder, fare click su Go > Connect to Server e inserire 'smb://ip-address/sambashare'.`

:pushpin:`Checkpoint per Windows: aprire il default file manager e modificare l'indirizzo con '\\ip-address\sambashare'.`

:heavy_exclamation_mark: **N.B. per 'ip-address' si intende l'IP del webserver; 'sambashare' è il nome della cartella per la condivisione.** :heavy_exclamation_mark:

[Torna su](https://github.com/matteob2609/webserver#webserver)

---

### :ghost: INSTALLAZIONE TOMCAT 9

**Introduzione:** Tomcat è un server web che fornisce una piattaforma software per l'esecuzione di applicazioni web sviluppate in linguaggio Java.

**1. Installazione Java**

- _apt update_

- _apt install default-jdk_

:pushpin:`Checkpoint: Verificare la versione Java installata inserendo il comando java -version.
Un esempio di output è questo:`

      openjdk version "11.0.9.1" 2020-11-04
      OpenJDK Runtime Environment (build 11.0.9.1+1-Ubuntu-0ubuntu1.20.04)
      OpenJDK 64-Bit Server VM (build 11.0.9.1+1-Ubuntu-0ubuntu1.20.04, mixed mode, sharing)


**2. Creazione di un nuovo utente che eseguirà il servizio Tomcat**

- _useradd -r -m -U -d /opt/tomcat -s /bin/false tomcat_

**3. Installazione Tomcat e permessi**

- _wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.41/bin/apache-tomcat-9.0.41.tar.gz -P /tmp_

- _tar xf /tmp/apache-tomcat-9*.tar.gz -C /opt/tomcat_, estraiamo l'archivio scaricato dentro la cartella /opt/tomcat.

- _cd /opt/tomcat_

- _sudo chgrp -R tomcat /opt/tomcat_

- _sudo chmod -R g+r conf_

- _sudo chmod g+x conf_

- _sudo chown -R tomcat webapps/ work/ temp/ logs/_

**4. Creazione di un systemd Unit file**

- _nano /etc/systemd/system/tomcat.service_, inserendo le righe di testo di sotto permetterà di eseguirlo come un servizio.

      [Unit]
      Description=Tomcat 9 servlet container
      After=network.target

      [Service]
      Type=forking

      User=tomcat
      Group=tomcat

      Environment="JAVA_HOME=/usr/lib/jvm/default-java"
      Environment="JAVA_OPTS=-Djava.security.egd=file:///dev/urandom -Djava.awt.headless=true"

      Environment="CATALINA_BASE=/opt/tomcat/latest"
      Environment="CATALINA_HOME=/opt/tomcat/latest"
      Environment="CATALINA_PID=/opt/tomcat/latest/temp/tomcat.pid"
      Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"

      ExecStart=/opt/tomcat/latest/bin/startup.sh
      ExecStop=/opt/tomcat/latest/bin/shutdown.sh

      [Install]
      WantedBy=multi-user.target
      
:heavy_exclamation_mark: **N.B. Modificare il percorso JAVA_HOME se la cartella d'installazione di Java è differente** :heavy_exclamation_mark:

- _systemctl daemon-reload_

- _systemctl start tomcat_, esecuzione del servizio Tomcat.

:pushpin:`Checkpoint: verificare lo stato del servizio inserendo il comando 'systemctl status tomcat'. L'output dovrà essere il seguente:`

      * tomcat.service - Tomcat 9 servlet container
         Loaded: loaded (/etc/systemd/system/tomcat.service; disabled; vendor preset: enabled)
         Active: active (running) since Wed 2018-09-05 15:45:28 PDT; 20s ago
        Process: 1582 ExecStart=/opt/tomcat/latest/bin/startup.sh (code=exited, status=0/SUCCESS)
       Main PID: 1604 (java)
          Tasks: 47 (limit: 2319)
         CGroup: /system.slice/tomcat.service
         
- _systemctl enable tomcat_, se non ci sono errori è possibile eseguire il servizio Tomcat all'esecuzione del server utilizzando questo comando.

**5. Modificare il firewall lasciando passare il traffico Tomcat sulla porta 8080**

- _ufw allow 8080/tcp_

**6. Accesso a Tomcat**

Digitare all'interno della barra di ricerca del browser _http://server_domain_or_IP:8080_

![tomcat_page](https://user-images.githubusercontent.com/61114792/106262435-4cfda200-6223-11eb-96a1-b3625513a39b.png)

[Torna su](https://github.com/matteob2609/webserver#webserver)
