# webserver
Breve documentazione per inizializzare un server web Ubuntu (versione in questione: 20.04.1 LTS)

:heavy_exclamation_mark: Usare **sudo** prima del comando se non si hanno i diritti necessari.

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

---

### :ghost: INSTALLAZIONE SSH & APACHE2

- _apt install openssh-server_

- _apt install apache2_

:pushpin:`Checkpoint: verificare l'installazione del server Apache aprendo il browser e mettendo nella barra degli indirizzi l'IP del server web, se viene visualizzata la pagina di default di Apache l'installazione è andata a buon fine.`

      172.16.29.105/index.html

---

### :ghost: CREAZIONE DI UN UTENTE PER UN SITO SPECIFICO

- _useradd -s /bin/bash -d /var/www/'nome_cartella_sito' -m 'nome_user'_

- _passwd 'password'_

:pushpin:`Checkpoint: fare il login da remoto per verificare la corretta aggiunta dell'utente.`

---

### :ghost: AGGIUNTA DEL SITO WEB

- _cd /var/www/'nome_cartella_sito'_

- _mkdir log_

- _cd /etc/apache2/sites-available_

- _cp 000-default.conf 'nome_cartella_sito'.conf_

- _nano 'nome_cartella_sito'.conf_

`Togliere il commento nella riga ServerName e inserire il dominio del sito; In DocumentRoot inserire il percorso della cartella del sito; In ErrorLog e CustomLog togliere il $, le parentesi graffe e inserire il percorso della cartella del sito /log.`

---

### :ghost: ABILITAZIONE DEL SITO

- _a2ensite 'file_configurazione_sito'.conf_

- _systemctl restart apache2.service_

:pushpin:`Checkpoint: immettere nella barra degli indirizzi del browser 'nome_cartella_sito.dominio', se vengono visualizzati i file inseriti dall'utente registrato il sito funziona correttamente. Di seguito alcuni esempi:`

      sitoa-105.virtual.marconi
      sitob-105.virtual.marconi
      ubuntu-srv105.virtual.marconi

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

**3. Installazione Tomcat**

- _wget http://www-eu.apache.org/dist/tomcat/tomcat-9/v9.0.27/bin/apache-tomcat-9.0.27.tar.gz -P /tmp_

- _tar xf /tmp/apache-tomcat-9*.tar.gz -C /opt/tomcat_, estraiamo l'archivio scaricato dentro la cartella /opt/tomcat.

- _ln -s /opt/tomcat/apache-tomcat-9.0.27 /opt/tomcat/latest_

- _chown -RH tomcat: /opt/tomcat/latest_

- _sh -c 'chmod +x /opt/tomcat/latest/bin/*.sh'_

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
