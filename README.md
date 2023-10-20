# wildfly_mode_domain
a configuration of domainwildfly

Comprendre le mode domaine Wildfly
![alt text](images/image1.png)

Pour configurer wildfly en mode standalone, j’utiliserai trois machine virtuelle que j’ai crée en utilisant VirtualBox et Vagrant:
![alt text](images/image2.png)
```
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
```

Quand on crée les trois vm dans le même réseau  local virtuel, nous aurons: 
- Machine 1: 192.168.56.9 -> rnds.cyberlink.co.id
- Machine 2: 192.168.56.10 -> dmz1.cyberlink.co.id
- Machine 3: 192.168.56.11 -> dmz2.cyberlink.co.id

J’ai rajouté les 3 entrées suivantes dans chaque fichier /etc/hosts de chaque machine

![alt text](images/image3.png)

Pour que nslookup peut trouver chaque machine à partir de n’importe quelle autre machine:

![alt text](images/image4.png)

La machine 1 va jouer le rôle de contrôleur de domain (master)<br>
La machine 2 va jouer le rôle du contrôleur de host (slave1)<br>
La machine 3 va jouer le rôle du contrôleur de host (slave2)<br>
**Partie 1 : paramétrage du MASTER**
# Aller sur la machine 1:
```bash
root@wildfly:~$ sudo su
root@rnds:~$ cd /opt
root@rnds:~$ mkdir source
root@rnds:~$ cd source
root@rnds:~$ wget https://download.jboss.org/wildfly/13.0.0.Final/wildfly-13.0.0.Final.tar.gz
root@rnds:~$ cd /opt
root@rnds:~$ tar -zxvf source/wildfly-13.0.0.Final.tar.gz -C /opt
root@rnds:~$ mv wildfly-13.0.0.Final/ wildflycd wildfly/docs/contrib/scripts/systemd
```
![alt text](images/image5.png)

```bash
root@rnds:~$ cat README
```
![alt text](images/image6.png)
```bash
 root@rnds:~$ mkdir /etc/wildfly
 root@rnds:~$ cp wildfly.conf /etc/wildfly/
 root@rnds:~$ cp wildfly.service /etc/systemd/system/
 root@rnds:~$ cp launch.sh /opt/wildfly/bin/
 root@rnds:~$ chmod +x /opt/wildfly/bin/launch.sh
 root@rnds:~$ vi /etc/wildfly/wildfly.conf
```
![alt text](images/image7.png)
```bash
 root@rnds:~$ vi /etc/systemd/system/wildfly.service
```
![alt text](images/image8.png)
```bash
 root@rnds:~$ cd /opt/wildfly/domain/configuration
```
![alt text](images/image9.png)
```bash
 root@rnds:~$ cp host.xml host.xml.ori
 root@rnds:~$ cp domain.xml domain.xml.ori
 root@rnds:~$ vi domain.xml
```
Dans la section <server-groups> on ajoute 
![alt text](images/image10.png)

```bash
 root@rnds:~$ vi host.xml
```
Supprimer l’attribut par défaut name=”Master” dans la balise domain.
Cela permet d’avoir inheritance-server:none, cela permet au serveur de jouer seulement le rôle de contrôleur de domaine et le rôle de load balancer 

Commentez toute la partie <servers>
![alt text](images/image11.png)
```bash
 root@rnds:~$ vi logging.properties
```
![alt text](images/image12.png)
```bash
 root@rnds:~$ mkdir /var/log/wildfly.log
 root@rnds:~$ vi  /opt/wildfly/bin/launch.sh
```
![alt text](images/image13.png)
```bash
 root@rnds:~$ cd /
 root@rnds:~$  systemctl restart wildfly
 root@rnds:~$  systemctl status wildfly
```
![alt text](images/image14.png)
```bash
 root@rnds:~$ netstat -pltn
```
![alt text](images/image15.png)

