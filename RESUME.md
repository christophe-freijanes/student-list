##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/dockerhub/flask.jpg)
##
## 1. CREATION DE LA VM
##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/VBoxVagrantCentOS.jpg)
Notre machine virtuelle se nommera "mpdocker" elle sera le host qui permettra de faire fonctionner nos conteneurs.
### Prerequis:
* Une machine local sous Linux, Windows, MacOS
* Avoir une connexion reseau
* Installation de Vagrant et VirtualBox
* Savoir utiliser Powershell
* Avoir un editeur de texte (Notepad++, VSCode)
* Creation d'une virtual machine avec un OS:Centos 7.6 depuis un vagrantfile
* Le provisionement du syteme doit avoir au minimun ses paquets d'installer: *git*, *docker*, *docker-compose*
### Depuis votre machine local (Host):
1. Creation d'un dossier qui va contenir mon vagrantfile
```bash
PS H:\PROJETS\repo\student-list>
```
```bash
mkdir H:\PROJETS\repo\student-list\mpdocker\
```
2. Initialisation d'un vagrantfile (option -m = configuration minimum)
```bash
PS H:\PROJETS\repo\student-list\mpdocker>
```
```bash 
vagrant init -m
```
3. Copier/Coller ce vagrantfile
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
# SCRIPT for provisioning the magic system :)
$no_update_system = <<SCRIPT
echo Stop Update System...
sudo systemctl stop packagekit.service && sudo systemctl disable packagekit.service
echo "For this virtual machine, no update will be made the version Centos 7.6 will remain fixed"
SCRIPT

$install_git = <<SCRIPT
sudo yum install -y yum-utils
echo Install and configure git...
sudo yum install git -y
git config --global user.name "christophe-freijanes"
git config --global user.email "cfreijanes@gmx.fr"
git config --global core.editor vi
sleep 5
SCRIPT

$install_docker_script = <<SCRIPT
echo Installing Docker...
curl -sSL https://get.docker.com/ | sh
sudo usermod -aG docker $USER
sudo systemctl enable docker
sudo systemctl start docker
echo Docker has been installed...
echo Installing Docker-compose...
sudo mkdir -p /opt/bin/
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
echo Docker-Compose has been installed...
echo "For this Stack, you will use $(ip -f inet addr show eth1 | sed -En -e 's/.*inet ([0-9.]+).*/\1/p') IP Address"
sleep 5
echo "The Virtual machine Centos 7.6 has been installed >>> Last reboot !!!"
sudo reboot
SCRIPT

# The vagrant config for virtual machine named mpdocker
Vagrant.configure('2') do |config|
  config.vm.define :mpdocker, primary: true  do |mpdocker|
    mpdocker.vm.box = "komlevv/centos-7.6"
    mpdocker.vm.box_version = "1.0.0"
    mpdocker.vbguest.auto_update = false
    mpdocker.vm.network :private_network, ip: IP
#    mpdocker.vm.network :forwarded_port, guest: 5000, host: 5000
#    mpdocker.vm.network :forwarded_port, guest: 80, host: 8080
    mpdocker.vm.hostname = "mpdocker"
    mpdocker.vm.synced_folder ".", "/vagrant"
    mpdocker.vm.provision "shell", inline: $install_git, privileged: true
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
PS H:\PROJETS\repo\student-list\mpdocker>
```
```bash
vagrant plugin install vagrant-winnfsd
```
```bash
PS H:\PROJETS\repo\student-list\mpdocker>
```
```bash
vagrant plugin install vagrant-vbguest
```
```bash
PS H:\PROJETS\repo\student-list\mpdocker>
```
```bash
vagrant plugin install vagrant-share
```	
6. Verification de son vagrantfile
```bash
PS H:\PROJETS\repo\student-list\mpdocker>
```
Output:
```bash
vagrant validate
Vagrantfile validated successfully.
```
7. Creation de la machine virtuelle "mpdocker"
```bash
PS H:\PROJETS\repo\student-list\mpdocker>
vagrant up
```
8. Connexion en ssh depuis votre prompt
```bash
PS H:\PROJETS\repo\student-list\mpdocker>
```
```bash
vagrant ssh mpdocker
```
9. Verification apres installation
```bash
cat /etc/redhat-release
```
```bash
CentOS Linux release 7.6.1810 (Core)
```
```bash
docker -v
```
```bash
Docker version 20.10.10, build b485636
```
```bash
docker-compose -v
```
```bash
docker-compose version 1.29.2, build 5becea4c
```
```bash
ip a
```
```bash
...
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:24:c7:d8 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.200/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe24:c7d8/64 scope link
       valid_lft forever preferred_lft forever
...
```
##
## 2. RECUPERATION DU CODE
##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/logoGithub.PNG)
##
1. Depuis mpdocker (Host) copier le code de l' API a la racine "/"
```bash
cd /
```
```bash
sudo git clone https://github.com/diranetafen/student-list.git
```
2. Verification de l'emplacement du code
```bash
ls -alh /student-list/
```
Output:
```bash
total 48K
drwxr-xr-x.  8 root root  193 Nov 11 13:52 .
dr-xr-xr-x. 19 root root  259 Nov 11 13:52 ..
-rw-r--r--.  1 root root  800 Nov 11 13:52 docker-compose.yml
drwxr-xr-x.  8 root root  163 Nov 11 13:52 .git
-rw-r--r--.  1 root root 7.1K Nov 11 13:52 README.md
drwxr-xr-x.  2 root root   70 Nov 11 13:52 simple_api
drwxr-xr-x.  2 root root   23 Nov 11 13:52 website
```
##
## 3. CREATION DU DOCKERFILE
##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/Docker.jpg)
##
1. Edition du Dockerfile
```bash
sudo vi /student-list/simple_api/Dockerfile
```
NB: La touche I c'est la vie ! :)
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
##
## 4. EDITION INDEX.PHP
1. Edition de la page index.php
```bash
sudo vi /student-list/website/index.php
```
2. Modifer la ligne 21 et 20 ainsi que la ligne 29
```bash
              if ( empty($username) ) $username = 'toto';
              if ( empty($password) ) $password = 'cHI0aG9u';
```
Lien : [Encode-Decode Base64 ](https://base64decode.org)
```bash
$url = 'http://<api_ip_or_name:port>/pozos/api/v1.0/get_student_ages';
par
$url = 'http://10.0.0.200:5000/pozos/api/v1.0/get_student_ages';
```
3. Enregistrer les modifications part 1 ;)
##
## 5. EDITION DU FICHIER DES VARIABLES
1. Creation et edition d'un fichier contenant les variables
```bash
sudo vi /student-list/.env_prod
```
```bash
SOME_USERNAME=toto
SOME_PWD_VAR=python
```
2. Enregistrer les modifications ;)
##
## 6. CREATION VOLUME
1. Creation d'un volume persistant
```bash
sudo docker volume create data
```
2. Lister les volumes
```bash
sudo docker volume ls
```
```bash
DRIVER    VOLUME NAME
local     data
```  
3. Detail du volume data
```bash
sudo docker volume inspect data
```
Output :
```bash
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
##
## 7. CREATION NETWORK
1. Creation d'un reseau pour permettre aux conteneurs de communiquer entre eux
```bash
sudo docker network create study-net
```
Exemple output :
```bash
93acfcd1907eb7cda5c5cb01c73750522abd1505703eba12a07a5069a12f1685
```
2. Liste des network disponible
```bash
sudo docker network ls
```
```bash
NETWORK ID     NAME        DRIVER    SCOPE
8740347493ef   bridge      bridge    local
752df8a6eaeb   host        host      local
d4795f64a489   none        null      local
93acfcd1907e   study-net   bridge    local
```
##
## 8. BUILD AND RUN DOCKER IMAGE
##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/dockerhub/dockerrun.jpeg)
##
1. Creation de l'image pour notre conteneur api
```bash
cd /student-list/simple_api/
```
```bash
sudo docker build -t student-list_api:v1.0 .
```
2. Verification de l'image
```bash
sudo docker images
```
```bash
REPOSITORY         TAG           IMAGE ID       CREATED         SIZE
student-list_api   v1.0          4c056fe48362   6 seconds ago   1.13GB
python             2.7-stretch   e71fc5c0fcb1   18 months ago   928MB
```
3. Tag de l'image de l'apllication
```bash
sudo docker tag <IMAGE ID> cfreijanes/student-list_api:v1.0
```  
4. Creation d'un repositorie depuis son docker-hub
##
Lien : [Docker-Hub repositories ](https://hub.docker.com/repositories)
##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/dockerhub/0-1.png)
##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/dockerhub/0-2.png)
##
5. Connexion au Docker-hub
```bash
sudo docker login -u <USERNAME> -p <PASSWORD>
```
6. Push de l'image vers le repositorie de son docker-hub
```bash
sudo docker push cfreijanes/student-list_api:v1.0
```
7. Verification de nos microservices actifs ou non
```bash
sudo docker ps -a
```
```bash
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
8. Creation du deuxieme conteneur student-list_web
* Demarrer l'image docker
* Creation et execution du conteneur student-list_web puis execution du conteneur contenant notre API
```bash
sudo docker run -it --name student-list_web --network study-net -d -p 8080:80 -v /website:/var/www/html/ php:apache
```
```bash
sudo docker run -it --name student-list_api --network study-net -d -p 5000:5000 -v /student-list/simple_api/student_age.json:/data/student_age.json student-list_api:v1.0
```
9. Verification de nos microservices actifs
```bash
sudo docker ps -a
```
```bash
$ sudo docker ps -a
CONTAINER ID   IMAGE                   COMMAND                  CREATED          STATUS          PORTS                                       NAMES
f86db3416969   student-list_api:v1.0   "python ./student_ag…"   9 seconds ago    Up 8 seconds    0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   student-list_api
807d1dcdbd88   php:apache              "docker-php-entrypoi…"   35 seconds ago   Up 34 seconds   0.0.0.0:8080->80/tcp, :::8080->80/tcp       student-list_web
```
10. Test de l'API
```bash
curl -u toto:python -X GET http://$HOSTNAME:5000/pozos/api/v1.0/get_student_ages
```
```bash
{
  "student_ages": {
    "alice": "12",
    "bob": "13"
  }
}
```
##
## 9. AUTOMATISATION AVEC GITHUB
##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/dockergithub.png)
##
Prerequis :
- Avoir un compte Docker-hub permettant le build (payant).
- Avoir creer une image depuis votre host.
- Avoir pusher cette image sur votre repository Docker-hub.
- Avoir un repository sur Github dans lequel sera construite votre Dockerfile.
Dans cet exemple j'ai creer un repository docker-pozos sur mon Github.
### Depuis son PC local vers Github
1. Cloner le repository que vous venez de creer depuis votre Github
```bash
git clone git@github.com:christophe-freijanes/docker-pozos.git
```
2. Depuis le prompt on ce rends dans notre repertoire
```bash
 cd .\docker-pozos\
```
3. On va rendre dynamique le build de notre Dockerfile, biensur pour que cela fonctionne vous devez avoir le Dockerfile, student_age.py et student_age.json entre votre machine local et votre VM. Utiliser un editeur de text pour copier coller votre code et l'enregistrer.
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
4. Commande git a executer pour pusher le repo local
```bash
git init
```
```bash
git add Dockerfile README.md student_age.py student_age.json
```
```bash
git commit -am "Creation repo docker-pozos"
```
```bash
git remote add origin https://github.com/christophe-freijanes/docker-pozos.git
```
```bash
git push -u origin master
```
### Depuis Docker-Hub vers Github
1. Choisir le repository ou ce situe votre image
##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/dockerhub/01.PNG)
2. Parametrage a faire depuis votre compte
##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/dockerhub/02.png)
##
3. Connexion a faire entre votre Github et Docker-Hub
##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/dockerhub/03.png)
##
4. Aller dans votre repository contenant votre image
##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/dockerhub/04.png)
##
5. Parametrage de votre Builds
##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/dockerhub/05.png)
##
6. Choisir le repository de Github que vous souhaitez synchroniser
##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/dockerhub/06.png)
##
7. Il ne reste plus cas faire ???
##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/dockerhub/07.png)
##
NB: Attention ne fermer pas la page cela peut-etre long, tout depend de la taille de l'image of course ;)
##
8. Si le build ne se fait pas verifier depuis les settings que le lien vers votre Docker-hub et votre Github est implementer
##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/dockerhub/08.png)
##
## 10. NETTOYAGE
##
1. Lister les conteneurs
```bash
sudo docker ps -a
```
```bash
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS                                       NAMES
4d3626d0af8c   student-list_api:v1.0   "python ./student_ag…"   3 minutes ago   Up 3 minutes   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   student-list_api
a01dcea3b59d   php:apache            "docker-php-entrypoi…"   8 minutes ago   Up 8 minutes   0.0.0.0:8080->80/tcp, :::8080->80/tcp       student-list_web
```
2. Stopper les conteneurs a nettoyer
```bash
sudo docker ps
```
```bash
sudo docker stop student-list_web
```
```bash
sudo docker stop student-list_api
```
3. Suppression des conteneurs et des images
- Supprimer les deux conteneurs
```bash
docker rm <CONTAINER ID>
```
```bash
docker images
```
- Supprimer les 4 images de haut en bas c'est mieux :)
```bash
docker rmi <IMAGE ID>
```
4. Verification que les conteneurs ne sont plus present
```bash
sudo docker ps -a
```
```bash
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
5. Verification que les images ne sont plus presentes
```bash
sudo docker images
```
```bash
REPOSITORY         TAG           IMAGE ID       CREATED         SIZE
```
##
## 11. BUILD AND RUN DOCKER-COMPOSE
##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/dockerhub/docker-compose.png)
##
1. Edition du docker-compose
```bash
cd ..
```
```bash
sudo vi docker-compose.yml
```
```bash
version: '3.3'
services:
    api:
        build: '/student-list/simple_api/'
        container_name: student-list_api
        network_mode: study-net
        ports:
            - '5000:5000'
        env_file:
            - .env_prod
        volumes:
            - '/student-list/simple_api/student_age.json:/data/student_age.json'
            - '/student-list/simple_api/student_age.py:/data/student_age.py'
    web:
        container_name: student-list_web
        network_mode: study-net
        ports:
            - '8080:80'
        env_file:
            - .env_prod
        volumes:
            - '/website:/var/www/html/'
        image: 'php:apache'
        depends_on:
            - 'api'
networks:
    study-net:
```
2. Installation de docker-compose pour Centos 7.6
```bash
sudo curl -L https://github.com/docker/compose/releases/download/1.21.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
```
```bash
sudo chmod +x /usr/local/bin/docker-compose
```
```bash
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
```bash
docker-compose --version
```
Output:
```bash
docker-compose version 1.21.0, build 5920eb0
```
3. Run le docker-compose
```bash
sudo docker-compose up -d
```
```bash
Creating student-list_api_1 ... done
Creating student-list_web_1 ... done
```
4. Verification de nos microservices
```bash
docker ps -a
```
```bash
CONTAINER ID   IMAGE              COMMAND                  CREATED         STATUS         PORTS                                       NAMES
78378f485a3e   php:apache         "docker-php-entrypoi…"   2 minutes ago   Up 2 minutes   0.0.0.0:8080->80/tcp, :::8080->80/tcp       student-list_web_1
e247a778b5c3   student-list_api   "python ./student_ag…"   2 minutes ago   Up 2 minutes   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   student-list_api_1
```
##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/dockerhub/result.png)
##
Congratulation !
##
## 12. PRIVATE DOCKER-REGISTRY
##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/privateRegistry.jpg)
##
1. Deployer d'une autre machine virtuelle  dans cet exemple on la nommera "regdocker"
```bash
# -*- mode: ruby -*-
# vi: set ft=ruby :

RAM = 4096
CPU = 4
IP = "10.0.0.201"

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

Vagrant.configure("2") do |config|
  config.vm.define "regdocker" do |regdocker|
    regdocker.vm.box = "ubuntu/xenial64"
    regdocker.vm.network "private_network", ip: IP
	  regdocker.vm.hostname = "regdocker"
    regdocker.vm.provider "virtualbox" do |v|
  	  v.name = "regdocker"
  	  v.memory = RAM
  	  v.cpus = CPU
  	end
  	regdocker.vm.provision :shell do |shell|
  	  shell.path = "install_docker.sh"
    end 
  end
end
```
* A mettre dans le dossier de votre machine le script bash install_docker.sh a coter de votre vagrantfile ;)
```bash
#!/bin/bash
sudo yum -y update

# install docker
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
sudo usermod -aG docker vagrant
sudo systemctl enable docker
sudo systemctl start docker
echo "For this Stack, you will use $(ip -f inet addr show enp0s8 | sed -En -e 's/.*inet ([0-9.]+).*/\1/p') IP Address"
```
2. Deploiement de regdocker
```bash
vagrant up
```
3. Ce connecter en ssh
```bash
ssh vagrant@<IP_REGISTRY>
```
4. Creation d'un reseau "reg-study-net"
```bash
docker network create reg-study-net
```
5. Creation du conteneur BACKEND
```bash
docker run -d -p 5000:5000 -e REGISTRY_STORAGE_DELETE_ENABLED=true --net reg-study-net --name registry-study-net registry:2
```
6. Creation du conteneur FRONTEND 
```bash
docker run -d -p 8090:80 --net reg-study-net -e REGISTRY_URL=http://registry-study-net:5000 -e DELETE_IMAGES=true -e REGISTRY_TITLE=reg-study-net --name frontend-study-net joxit/docker-registry-ui:1.5-static
```
7. Verification de l' activitee de nos conteneurs
```bash
docker ps
```
```bash
CONTAINER ID   IMAGE                                 COMMAND                  CREATED          STATUS          PORTS                                       NAMES
4104823b1746   joxit/docker-registry-ui:1.5-static   "/docker-entrypoint.…"   9 seconds ago    Up 8 seconds    0.0.0.0:8090->80/tcp, :::8090->80/tcp       frontend-study-net
47d021115527   registry:2                            "/entrypoint.sh /etc…"   27 seconds ago   Up 26 seconds   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   registry-study-net
```
8. Connexion a la WEBGUI de notre registry
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/dockerhub/private-registry.png)
##
Lien : [Private Registry](http://10.0.0.201:8090/)
##
## 13. PULL IMAGE DOCKER HUB TO PUSH PRIVATE REGISTRY
1. Pull d'une image depuis le Docker-hub
```bash
docker pull nginx:latest
```
2. Preparation pour tag de notre image 
```bash
docker images
```
```bash
REPOSITORY                 TAG          IMAGE ID       CREATED        SIZE
nginx                      latest       04661cdce581   18 hours ago   141MB
registry                   2            b2cb11db9d3d   2 months ago   26.2MB
joxit/docker-registry-ui   1.5-static   74416e0cd8ba   8 months ago   24.2MB
```
3. Tag de l'image "nginx" dans cette exemple
```bash
docker tag <IMAGE ID> localhost:5000/nginx:private-registry
```
4. Push image en local depuis notre host
```bash
docker push localhost:5000/nginx:private-registry
```
```bash
docker images
```
```bash
REPOSITORY                 TAG                IMAGE ID       CREATED        SIZE
nginx                      latest             04661cdce581   18 hours ago   141MB
localhost:5000/nginx       private-registry   04661cdce581   18 hours ago   141MB
registry                   2                  b2cb11db9d3d   2 months ago   26.2MB
joxit/docker-registry-ui   1.5-static         74416e0cd8ba   8 months ago   24.2MB
```
##
* Details de notre image depuis notre WebGUI:
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/dockerhub/tag.png)
##
NB: On remarque que l'on a la possibiliter de supprimer notre image.
##
## 14. PUSH D'UNE IMAGE DEPUIS UNE AUTRE MACHINE (DISTANTE) VERS NOTRE PRIVATE REGISTRY
1. Pull d'une immage par exemple stocker dans notre Docker hub, on va la telecharger vers mpdocker puis l' envoyer vers regdocker (Private Registry)
```bash
sudo docker login
```
```bash
sudo docker pull cfreijanes/student-list_api:latest
```
2. Depuis notre machine mpdocker on va lister nos images pour voir si on la dans notre referentiel sur notre machine mpdocker of Course !
```bash
sudo docker images
```
```bash
REPOSITORY                    TAG              IMAGE ID       CREATED        SIZE  
nginx                         latest           04661cdce581   20 hours ago   141MB 
10.0.0.201:5000/nginx         private-remote   04661cdce581   20 hours ago   141MB 
cfreijanes/student-list_api   latest           e85d03d15757   24 hours ago   1.13GB
```
3. On va Tagger l'image de notre API avec la ref <IMAGE ID> 
```bash
sudo docker tag <IMAGE ID> <IP_REGISTRY>:5000/student-list_api:centos-remote
```
4. Dans le cadre de ce mini-projet je vais proposer une solution de contournement pour permettre a notre machine mpdocker de pouvoir pusher notre image par http. :)
```bash
sudo vi /usr/lib/systemd/system/docker.service
```
* Remplacer <IP_REGISTRY> par l'IP de votre private registry 
```bash
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service containerd.service
Wants=network-online.target
Requires=docker.socket containerd.service
[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd --insecure-registry <IP_REGISTRY>:5000
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always
# Note that StartLimit* options were moved from "Service" to "Unit" in systemd 229.
# Both the old, and new location are accepted by systemd 229 and up, so using the old location
# to make them work for either version of systemd.
StartLimitBurst=3
# Note that StartLimitInterval was renamed to StartLimitIntervalSec in systemd 230.
# Both the old, and new name are accepted by systemd 230 and up, so using the old name to make
# this option work for either version of systemd.
StartLimitInterval=60s
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Comment TasksMax if your systemd version does not support it.
# Only systemd 226 and above support this option.
TasksMax=infinity
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
OOMScoreAdjust=-500
[Install]
WantedBy=multi-user.target
```
* Une fois cela fait on va redemarrer le daemon et le service docker
```bash
sudo systemctl daemon-reload
```
```bash
sudo systemctl restart docker
```
```bash
sudo systemctl status docker
```
5. Push de l'image de notre API vers notre private registry (regdoker)
```bash
sudo docker push <IP_REGISTRY>:5000/student-list_api:centos-remote
```
6. On peut aussi supprimer nos images
##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/dockerhub/delete.png)
##
## 15. CHECK DEPUIS NOTRE WEGUI PRIVATE REGISTRY
Lien : [Private Registry](http://10.0.0.201:8090/)
##
![alt text](https://github.com/christophe-freijanes/student-list/blob/master/images/dockerhub/api.png)
##
