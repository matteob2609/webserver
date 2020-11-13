# VM---webserver-documentation
Breve documentazione per inizializzare un server web Ubuntu.

(Usare 'sudo' prima del comando se non si hanno i diritti necessari)

### CONFIGURAZIONE DEL SERVER WEB

- apt update E DOPO apt upgrade //Aggiorna i pacchetti

- apt install openssh-server //Installa il server SSH

- nano /etc/netplan/00-installer-config.yaml //Modifica il file di configurazione. Da modificare in base alla propria rete


      network:
    
        renderer: networkd
      
        ethernets:
      
          enp0s3:
        
            addresses: [x.x.x.x/x]
          
            gateway4: x.x.x.x
          
            nameservers:
          
                search: [mydomain, other domain]
              
                addresses: [x.x.x.x, x.x.x.x]
              
         version: 2

- netplan try OPPURE netplan apply //Applica i cambiamenti effettuati nel file di configurazione

- apt install apache2 //Installa il server Apache2

- nano /etc/hosts E DOPO nano /etc/hostname //File da modificare se si vuole cambiare il nome del server e dell'hostname

### AGGIUNGERE UN SITO SPECIFICO AL SERVER WEB

- cd /var/www //Mi sposto all'interno della cartella www

- mkdir 'nome_cartella' //Sarà la cartella che contiene i file del sito

- cd 'nome_cartella_appena_creata' //Sempre da www mi sposto all'interno della cartella del sito appena creata per fare una cartella di log

- mkdir log //Creo la cartella di log, i file di log vengono creati in automatico

- cd /etc/apache2/sites-available //Entro all'interno della cartella dei siti disponibili di Apache2

- cp 000-default.conf 'nome_cartella_precedente'.conf //Crea un nuovo file facendolo uguale al contenuto che c'è in 000-default.conf

- nano 'nome_cartella_precedente'.conf //Modifico il file di configurazione del sito

- Togliere il commento nella riga ServerName e inserire il dominio del sito; In DocumentRoot inserire il percorso della cartella del sito; In ErrorLog e CustomLog, all'interno delle parentesi graffe inserire il percorso della cartella del sito/log

- a2ensite 'file_configurazione_sito'.conf E DOPO systemctl restart apache2.service //Abilito il sito e riavvio il server Apache
