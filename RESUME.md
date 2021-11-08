# CREATION DE LA VM

Prerequis :
* Avoir une connexion reseau
* Installation de Vagrant et VirtualBox
* Savoir utiliser Powershell
* Avoir un editeur de texte
* Creation VM-Centos 7.6 from vagrantfile

### Depuis la machine local (Host):
1. Creation d'un dossier qui va contenir mon vagrantfile
```bash
PS H:\PROJETS\repo\formation\DevOps\docker\miniprojet\VM> 
mkdir H:\PROJETS\repo\formation\DevOps\docker\miniprojet\VM\
```
2. Initialisation d'un vagrantfile (option -m = configuration minimum)
```bash
PS H:\PROJETS\repo\formation\DevOps\docker\miniprojet\VM> 
vagrant init -m
```
CAPTURE-02-VM-Initialisation d'un vagrantfile.PNG

3. On recupere la version Centos 7.6 pour son provider VirtualBox
```bash
https://app.vagrantup.com/boxes/search?utf8=%E2%9C%93&sort=downloads&provider=virtualbox&q=centos+7.6
```
```bash
Vagrant.configure("2") do |config|
  config.vm.box = "komlevv/centos-7.6"
  config.vm.box_version = "1.0.0"
end
```
4. Edition du vagrantfile d'apres les prerquis pour la creation de la VM
* CentOS 7.6
* Docker
```bash
$ vi vagrantfile
```
```bash
# -*- mode: ruby -*-
# vi: set ft=ruby :

RAM = 2048
CPU = 2
IP = "10.0.0.200"

# Check Vagrant plugins
# If you want to ensure that vagrant-winnfsd is installed and enabled every time you run your vagrant box, just add the following to your vagrant file, just before the code block
if Vagrant::Util::Platform.windows? then
  unless Vagrant.has_plugin?("vagrant-winnfsd")
    raise  Vagrant::Errors::VagrantError.new, "vagrant-winnfsd plugin is missing. Please install it using 'vagrant plugin install vagrant-winnfsd' and rerun 'vagrant up'"
  end
end
# If you want to ensure that vagrant-vbguest is installed and enabled every time you run your vagrant box, just add the following to your vagrant file, just before the code block
if Vagrant::Util::Platform.windows? then
  unless Vagrant.has_plugin?("vagrant-vbguest")
    raise  Vagrant::Errors::VagrantError.new, "vagrant-vbguest plugin is missing. Please install it using 'vagrant plugin install vagrant-vbguest' and rerun 'vagrant up'"
  end
end
# If you want to ensure that vagrant-share is installed and enabled every time you run your vagrant box, just add the following to your vagrant file, just before the code block
if Vagrant::Util::Platform.windows? then
  unless Vagrant.has_plugin?("vagrant-share")
    raise  Vagrant::Errors::VagrantError.new, "vagrant-share plugin is missing. Please install it using 'vagrant plugin install vagrant-share' and rerun 'vagrant up'"
  end
end

$no_update_system = <<SCRIPT
echo Stop Update System...
sudo systemctl stop packagekit.service && sudo systemctl disable packagekit.service
echo "For this virtual machine, no update will be made the version Centos 7.6 will remain fixed"
SCRIPT
$install_git = <<SCRIPT
echo Install and configure git...
sudo yum install git -y
git config --global user.name "christophe-freijanes"
git config --global user.email "cfreijanes@gmx.fr"
git config --global core.editor vi
sleep 5
SCRIPT
$install_Dev_Tools = <<SCRIPT
echo Installing Development Tools...
sudo yum groups mark install "Development Tools"
sudo yum groups mark convert "Development Tools"
sudo yum groupinstall "Development Tools" -y
sleep 5
SCRIPT
$install_docker_script = <<SCRIPT
echo Installing Docker...
curl -sSL https://get.docker.com/ | sh
usermod -aG docker "$USER"
sudo systemctl start docker
echo "For this Stack, you will use $(ip -f inet addr show eth1 | sed -En -e 's/.*inet ([0-9.]+).*/\1/p') IP Address"
sleep 5
SCRIPT

Vagrant.configure('2') do |config|
  config.vm.define :mpdocker, primary: true  do |mpdocker|
    mpdocker.vm.box = "komlevv/centos-7.6"
    mpdocker.vm.box_version = "1.0.0"
    mpdocker.vbguest.auto_update = false
    mpdocker.vm.network :private_network, ip: IP
    mpdocker.vm.network :forwarded_port, guest: 5000, host: 5000
    mpdocker.vm.hostname = "mpdocker"
    mpdocker.vm.synced_folder ".", "/vagrant"
    mpdocker.vm.provision "shell", inline: $install_git, privileged: true
    mpdocker.vm.provision "shell", inline: $install_Dev_Tools, privileged: true
    mpdocker.vm.provision "shell", inline: $install_docker_script, privileged: true
    mpdocker.vm.provider "virtualbox" do |vb|
      vb.name = "mpdocker"
      vb.memory = RAM
  	  vb.cpus = CPU
    end
  end
end
```
5. Installation des plugins vagrant
```bash
PS H:\PROJETS\repo\formation\DevOps\docker\miniprojet\VM> 
vagrant plugin install vagrant-winnfsd
PS H:\PROJETS\repo\formation\DevOps\docker\miniprojet\VM> 
vagrant plugin install vagrant-vbguest
PS H:\PROJETS\repo\formation\DevOps\docker\miniprojet\VM> 
vagrant plugin install vagrant-share
```	
6. Verification de son vagrantfile
```bash
PS H:\PROJETS\repo\formation\DevOps\docker\miniprojet\VM> 
vagrant validate
Vagrantfile validated successfully.
```
7. Creation de la VM avec docker
```bash
PS H:\PROJETS\repo\formation\DevOps\docker\miniprojet\VM>
vagrant up
```
8. Connexion a la VM en ssh
```bash
PS H:\PROJETS\repo\formation\DevOps\docker\miniprojet\VM> 
vagrant ssh mpdocker
```
9. Verification apres installation
```bash
$ cat /etc/redhat-release 
  CentOS Linux release 7.6.1810 (Core) 
$ docker -v
  Docker version 20.10.10, build b485636
$ ip a
  10.0.0.200
```
# RECUPERATION DU CODE
1. Depuis mpdocker (Host) copier le code de l' API a la racine "/"
```bash
$ cd /
$ git clone https://github.com/diranetafen/student-list.git
```
2. Verification de l'emplacement du code
```bash
$ ls -alh /student-list/
```
# CREATION DU DOCKERFILE
1. Ce deplacer dans le dossier de l'application
```bash
$ cd /student-list/simple_api/
```
2. Edition du Dockerfile
```bash
FROM python:2.7-stretch
LABEL maintainer=Christophe-Freijanes mail=cfreijanes@gmx.fr  
# Dependencies for the system
RUN DEBIAN_FRONTEND=noninteractive apt-get update -y && apt-get install python-dev python3-dev libsasl2-dev python-dev libldap2-dev libssl-dev -y
# Installation Flask
RUN pip install flask==1.1.2 flask_httpauth==4.1.0 flask_simpleldap python-dotenv==0.14.0
# Configure network for API
EXPOSE 5000
# Configuration volume /data
VOLUME [ "/data" ]
# Copy the script student_age.py to /
COPY student_age.py /
# Run the server python and start api
CMD [ "python", "./student_age.py" ]
```
# EDITION INDEX.PHP
1. Edition de la page index.php
```bash
$ vi /student-list/website/index.php
```
2. Modifer la ligne 29 du Code
```bash
$url = 'http://<api_ip_or_name:port>/pozos/api/v1.0/get_student_ages';
par
$url = 'http://$hostname:5000/pozos/api/v1.0/get_student_ages' ;
```
3. Enregistrer les modifications part 1 ;)
# EDITION DU FICHIER DES VARIABLES
1. Creation et edition d'un fichier contenant les variables
```bash
$ vi /student-list/.env_prod
USERNAME=toto
PASSWORD=python
APIHOST=api
```
2. Enregistrer les modifications part 1 ;)
# CREATION VOLUME
1. Creation d'un volume persistant
```bash
$ sudo docker volume create data
```
2. Lister les volumes
```bash
$ sudo docker volume ls
DRIVER    VOLUME NAME
local     data
```  
3. Detail du volume data
```bash
$ sudo docker volume inspect data
[
    {
        "CreatedAt": "2021-11-07T17:46:02Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/data/_data",
        "Name": "data",
        "Options": {},
        "Scope": "local"
    }
]
```
# CREATION NETWORK
1. Creation d'un reseau pour permettre aux conteneurs de communiquer mais aussi d'etre accessible depuis l'exterieur du host
```bash
$ docker network create -d bridge study-net
93acfcd1907eb7cda5c5cb01c73750522abd1505703eba12a07a5069a12f1685
```
2. Liste des network disponible
```bash
$ docker network ls
NETWORK ID     NAME        DRIVER    SCOPE
8740347493ef   bridge      bridge    local
752df8a6eaeb   host        host      local
d4795f64a489   none        null      local
93acfcd1907e   study-net   bridge    local
```
# BUILD AND RUN DOCKER IMAGE  =================================================================
1. Creation de l'image pour notre conteneur api
```bash
$ docker build -t student-list_api:v1.0 .
```
2. Verification de l'image
```bash
$ docker images
REPOSITORY         TAG           IMAGE ID       CREATED         SIZE
student-list_api   v1.0          4c056fe48362   6 seconds ago   1.13GB
python             2.7-stretch   e71fc5c0fcb1   18 months ago   928MB
```
3. Tag de l'image de l'apllication
```bash
$ docker tag bfbaca16bb5f cfreijanes/pozosapi:v1.0
```  
4. Creation d'un repositorie depuis son docker-hub
```bash
https://hub.docker.com/repositories
```
CAPTURE
5. Push de l'image vers le repositorie de son docker-hub
```bash
$ docker push cfreijanes/pozosapi:v1.0
```
6. Verification de nos microservices actifs
```bash
$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
7. Creation du conteneur
* Demarrer l'image docker
* Creation er execution du conteneur student-list_web puis execution du conteneur
```bash
$ docker run -it --name student-list_web --network study-net -d -p 8080:80 -v /website:/var/www/html/ php:apache
```
```bash
$ docker run -it --name student-list_api --network study-net -d -p 5000:5000 -v /student-list/simple_api/student_age.json:/data/student_age.json student-list_api:v1
```
8. Verification de nos microservices actifs
```bash
$ docker ps -a
```
# NETTOYAGE DE NOTRE API
1. Stopper les conteneurs a nettoyer
```bash
$ docker ps
$ docker stop student-list_web
$ docker stop student-list_api
```
2. Lister les images
```bash
$ docker images -a
```
3. Suppression des images
```bash
$ docker rmi Image Image
```
# INSTALLER DOCKER-COMPOSE
Derniere release : https://docs.docker.com/compose/install/

1. Installation de docker-compose
```bash
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```
2. Verification de la version
```bash
$ docker-compose -v
docker-compose version 1.29.2, build 5becea4c
```
* Si ne fonctionne pas erreur "command not found"
```bash
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
# BUILD AND RUN DOCKER-COMPOSE
1. Edition du docker-compose
```bash
$ vi /student_list/docker-compose.yml
version: '3.1'
services:
  api:
    build: /student-list/simple_api/
    volumes:
      - ./simple_api/student_age.json:/data/student_age.json
    ports:
      - "5000:5000"
    env_file:
      - .env_prod
    environment:
      - $USERNAME
      - $PASSWORD
    networks:
      - study-net
    restart: always
  web:
    image: php:apache
    volumes:
      - ./website:/var/www/html/
    restart: always
    env_file:
      - .env_prod
    environment:
      - $USERNAME
      - $PASSWORD
      - $APIHOST
    ports:
      - "8080:80"
    networks:
      - study-net
    depends_on:
      - api
networks:
  study-net:
```
2. Run le docker-compose
```bash
$ docker-compose up -d
```


