# VM---webserver-documentation
Brief documentation with the commands to enter in the command line to initialize a web server.

(Use 'sudo' if necessary before the command)

- apt update & sudo apt upgrade //update packages

- apt install openssh-server //install the SSH server

- nano /etc/netplan/00-installer-config.yaml //modify the configuration file


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

- netplan try or sudo netplan apply //apply the changes on the previous file

- apt install apache2 //install the Apache2 server

- nano /etc/hosts & nano /etc/hostname //modify the name of the server and the hostname
