# webserver
Breve documentazione per inizializzare un server web Ubuntu.

(Usare 'sudo' prima del comando se non si hanno i diritti necessari)

### 1. CONFIGURAZIONE DEL SERVER WEB

- apt update 

- apt upgrade

- nano /etc/hosts 

- nano /etc/hostname

Checkpoint --> immettere il comando reboot per riavviare il server web e visualizzare l'hostname aggiornato.

      reboot

- apt install openssh-server

- nano /etc/netplan/00-installer-config.yaml

      network:
        renderer: networkd
        ethernets:
          enp0s3:
            addresses: [172.16.29.100+x/16]
            gateway4: 172.16.1.7
            nameservers:
                addresses: [172.16.1.10, x.x.x.x]
         version: 2

- netplan try

Checkpoint --> verificare se l'indirizzo è stato modificato correttamente tramite il comando ip addr e verificare la connessione con il comando ping.

      ip addr
      ping

- apt install apache2

Checkpoint --> verificare l'installazione del server Apache aprendo il browser e mettendo nella barra degli indirizzi l'IP del server web, se viene visualizzata la pagina di default di Apache l'installazione è andata a buon fine.

### 2. CREARE UN NUOVO UTENTE PER IL SITO

- useradd -s /bin/bash -d /var/www/'nome_cartella_sito' -m 'nome_user'

- passwd 'password'

### 3. AGGIUNGERE UN SITO WEB

- cd /var/www/'nome_cartella_sito'

- mkdir log

- cd /etc/apache2/sites-available

- cp 000-default.conf 'nome_cartella_sito'.conf

- nano 'nome_cartella_sito'.conf

- Togliere il commento nella riga ServerName e inserire il dominio del sito; In DocumentRoot inserire il percorso della cartella del sito; In ErrorLog e CustomLog togliere il $, le parentesi graffe e inserire il percorso della cartella del sito /log.

### 4. ABILITARE IL SITO

- a2ensite 'file_configurazione_sito'.conf

- systemctl restart apache2.service

Checkpoint --> immettere nella barra degli indirizzi del browser IP/nome_cartella_sito, se vengono visualizzati i file inseriti dall'utente registrato il sito funziona correttamente.

### 5. ATTIVARE IL SERVIZIO FTP

- apt install vsftpd

- nano /etc/vsftpd.conf

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

