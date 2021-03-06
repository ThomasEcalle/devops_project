* Pré-requis 

Génération d'une clef privée ssh
Installation de Virtualbox v 5.1.x
Installation de Vagrant v 1.9.x
Installation d'Ansible v 2.1.1.0

* Création des VMs avec Vagrant (fichier `Vagrantfile` fourni)

** ubuntu/trusty64 ayant les caractéristiques suivantes 
hostname master
vb name master-esgi
adresse privée 192.168.56.210
cpus 2
mémoire 4g
Provisioning (de type shell)
ajout de la clef privée dans les clefs authorisées par l'utilisateur vagrant

** centos/7 ayant les caractéristiques suivantes
hostname slave
vb name slave-esgi
adresse privée 192.168.56.211
cpus 2
mémoire 4g
Provisioning (de type shell)
ajout de la clef privée dans les clefs authorisées par l'utilisateur vagrant

`vagrant up` puis `vagrant halt`

Démarrage des VMs depuis VirtualBox

* Création de l'inventaire de machines ansible : `platforms/vagrant/inventory.ini`

L'inventaire devra définir les groupes de machines suivants :
master contenant master-esgi 
slave contenant slave-esgi 
swarm contenant les sous-groupes de machines master et slave
jenkins contenant master-esgi
registry contenant master-esgi
zookeeper contenant le groupe de machines master
middleware contenant le groupe de machines swarm

* Ajout des variables `ansible_user` et `ansible_ssh_host` pour les 2 machines : `platforms/vagrant/host_vars/master-esgi.yml` et `platforms/vagrant/host_vars/slave-esgi.yml`

* Ajout des variables `vault_docker_registry_username` (identifiant pour le registry docker) et `vault_docker_registry_password` (mot de passe pour le registry docker) au sein du fichier vault `platforms/vagrant/group_vars/all/vault.yml` 

Le mot de passe utilisé pour encrypter le fichier devra être rajouté au sein du fichier `platforms/vagrant/vault_password.txt`

* Développement d'un playbook docker.yml et du rôle associé permettant d'installer docker-ce sur tous les noeuds

`ansible-playbook docker.yml -i platforms/vagrant/inventory.ini --vault-password-file platforms/vagrant/vault_password.txt -vv`

* Installation de docker-registry (playbook et rôle fournis) sur le master

`ansible-playbook registry.yml -i platforms/vagrant/inventory.ini --vault-password-file platforms/vagrant/vault_password.txt -vv`

* Développement d'un playbook swarm.yml et du rôle associé permettant d'installer un cluster swarm sur tous les noeuds

`ansible-playbook swarm.yml -i platforms/vagrant/inventory.ini --vault-password-file platforms/vagrant/vault_password.txt -vv`

* Installation de zookeeper sur le master (playbook et rôle associé fourni)

`ansible-playbook zookeeper.yml -i platforms/vagrant/inventory.ini --vault-password-file platforms/vagrant/vault_password.txt -vv`

* Installation d'un brocker kafka sur tous les noeuds et du topic kafka `esgi` (plyabooks fournis)

`ansible-playbook kafka.yml -i platforms/vagrant/inventory.ini --vault-password-file platforms/vagrant/vault_password.txt -vv`

* Installation de jenkins sur le master (playbook et rôle associé fournis)

`ansible-playbook jenkins.yml -i platforms/vagrant/inventory.ini --vault-password-file platforms/vagrant/vault_password.txt -vv`

* Développement du micro-service réactif

** application spring-boot (deux routes `/` et `/health`) qui, lorsqu'elle sera sollicitée sur la route principale, devra poster le message `Hello world` dans le topic kafka `esgi`

* Pipeline Jenkins

packaging docker de l'application via gradle, maven ou sbt
publication de l'image dans le private registry docker `master:443/app:latest` via ansible
déploiement de l'application (3 instances) sur le cluster swarm via ansible
