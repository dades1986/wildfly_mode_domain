# wildfly_mode_domain
a configuration of domainwildfly

Comprendre le mode domaine Wildfly
![alt text](http://url/to/img.png)

Pour configurer wildfly en mode standalone, j’utiliserai trois machine virtuelle que j’ai crée en utilisant VirtualBox et Vagrant:

Vagrant.configure("2") do |config|
  # VM1 Configuration
  config.vm.define "machine1" do |machine1|
    machine1.vm.box = "ubuntu/bionic62"
    machine1.vm.network "private_network", type: "dhcp"
    machine1.vm.network "public_network", type: "dhcp"
	config.vm.boot_timeout = 600
    config.ssh.insert_key = true
  end

  # VM2 Configuration
  config.vm.define "machine2" do |machine2|
    machine2.vm.box = "ubuntu/bionic62"
    machine2.vm.network "private_network", type: "dhcp"
    machine2.vm.network "public_network", type: "dhcp"
	config.vm.boot_timeout = 600
    config.ssh.insert_key = true
  end

  # VM3 Configuration (machine3)
  config.vm.define "machine3" do |machine3|
    machine3.vm.box = "ubuntu/bionic62"
    machine3.vm.network "private_network", type: "dhcp"
	machine3.vm.network "public_network", type: "dhcp"
	config.vm.boot_timeout = 600
    config.ssh.insert_key = true
  end
end


Quand on crée les trois vm dans le même réseau  local virtuel, nous aurons: 
 Machine 1:  192.168.56.9 -> rnds.cyberlink.co.id
Machine 2 : 192.168.56.10 ->dmz1.cyberlink.co.id
Machine3:192.168.56.11 ->dmz2.cyberlink.co.id
J’ai rajouté les 3 entrées suivantes dans chaque fichier /etc/hosts de chaque machine



Comme nslookup peut trouver chaque machine à partir de n’importe quelle autre machine:

La machine 1 va jouer le rôle de contrôleur de domain (master)
La machine 2 va jouer le rôle du contrôleur de host (slave1)
La machine 3 va jouer le rôle du contrôleur de host (slave2)
Partie 1 : paramétrage du MASTER
Aller sur la machine 1:
-sudo su
-cd /opt
-mkdir source
-wget https://download.jboss.org/wildfly/13.0.0.Final/wildfly-13.0.0.Final.tar.gz
-cd /opt
-tar -zxvf source/wildfly-13.0.0.Final.tar.gz -C /opt
-mv wildfly-13.0.0.Final/  wildfly
-cd wildfly/docs/contrib/scripts/systemd

-cat README


 -mkdir /etc/wildfly
 -cp wildfly.conf /etc/wildfly/
 -cp wildfly.service /etc/systemd/system/
 -cp launch.sh /opt/wildfly/bin/
 -chmod +x /opt/wildfly/bin/launch.sh
-vi /etc/wildfly/wildfly.conf

-vi /etc/systemd/system/wildfly.service

-cd /opt/wildfly/domain/configuration

-cp host.xml host.xml.ori
-cp domain.xml domain.xml.ori
-vi domain.xml

Dans la section <server-groups> on ajoute 

-vi host.xml

Supprimer l’attribut par défaut name=”Master” dans la balise domain.
Cela permet d’avoir inheritance-server:none, cela permet au serveur de jouer seulement le rôle de contrôleur de domaine et le rôle de load balancer 

Commentez toute la partie <servers>

-vi logging.properties

-mkdir /var/log/wildfly.log
-vi  /opt/wildfly/bin/launch.sh

-cd /
- systemctl restart wildfly
- systemctl status wildfly

-netstat -pltn


