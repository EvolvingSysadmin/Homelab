# RancherOS

# RancherOS Docker Config

## TOC
* [RancherOS Installation](#rancheros-installation)
* [RancherOS Initial Config](#rancheros-initial-config)
* [RancherOS Share Config](#rancheros-share-config)
* [RancherOS NFS Volume for Containers through Portainer](#rancheros-nfs-volume-for-containers-through-portainer)
* [Rancher Server Docker Config](#rancher-server-docker-config)
* [Resources](#resources)
* [TODO](#todo)

### Variables
* RancherOS IP Address
* Password for RancherOS rancher user
* RancherOS SSH keys
* Disks for target install
* freeNAS IP/volumes

## RancherOS Installation 

Resource <sup>[1](#Resources)</sup>

### Installation and SSH Access
* Boot ISO in ProxMox
* Get ip address on eth0: ```ip addr```
* Change password for rancher user: ```sudo passwd rancher```
* Edit cloud-config.yml
  * ```wget https://raw.githubusercontent.com/EvolvingSysadmin/Homelab/main/RancherOS/cloud-config.yml```
  * Add RacherOS IP address
  * Create SSH keygen pair:
    * run puttygen save public and private
    * add public ssh key as single line to cloud-config.yml
* Putty into RancherOS with rancher / password set
* Create cloud-config.yml file: 
  * ```sudo vi cloud-config.yml```
  * Copy contents from local cloud-configy.yml that has public ssh key as a single string
* List disks: ```sudo fdisk -l```
* run rancher installer on desired disk: ```sudo ros install -c cloud-config.yml -d /dev/sda```
* Choose N for reboot
* Powerdown VM: ```sudo poweroff```
* Remove iso from hardware of VM in ProxMox
* Boot VM in ProxMox
* Add private key in putty: ssh --> auth --> browse
* Login as rancher

## RancherOS Initial Config 
Resource <sup>[2](#Resources)</sup>

* Switch to root: ```sudo su -```
* Switch to Ubuntu console: ```sudo ros console enable ubuntu```
  * To switch back: ```sudo ros console switch default```
* Enable Kernal extras: ```sudo ros service enable kernel-extras```
* Start Kernel extras: ```sudo ros service up kernel-extras```
* Reboot system to apply the changes: ```Sudo reboot /f```
* Update the system: ```sudo apt-get update```
* Create share user with same ID as in freeNAS: ```adduser -u FREENAS_USER_ID share_user```
* Add share_user to sudoers: <sup>[3](#Resources)</sup>
  * ```sudo su -```
  * ```visudo```
  * Add to bottom of file: ```share_user  ALL=(ALL) NOPASSWD:ALL```

## RancherOS Share Config
Resource <sup>[4](#Resources)</sup>
* SSH@rancher@RancherOS
* ```sudo su -```
* cd ```/var/lib/rancher/conf/cloud-config.d/```
* ```wget https://raw.githubusercontent.com/EvolvingSysadmin/Homelab/main/RancherOS/rancheros-cloud-config.yml``` [4](#Resources)</sup>
  * edit SERVER and SHARE (i for insert, esc for exit out of insert, :wq for quit with save)
*  Apply changes: ```sudo reboot /f```
* check that nfs share is now avilable, ssh in again and:
  *  ```cd /mnt/nfs-1/```
  * ```su share_user```
  * ```touch test.txt```

## RancherOS NFS Volume for Containers through Portainer 
Resources <sup>[5](#Resources)</sup>
* Run portainer docker on port 9000: ```docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce```
* Add portainer volume: portainer --> volume --> add:
  * Enable NFS Volume
  * Address: ```insert actual freeNAS IP Address```
  * Mount Point: ```/mnt/tank/lab/rancher```
  * Attach volume to containers: portainer --> containers --> Add Container:
    *  Enable random host network port
    * Volumes -> map additional volumes: 
      * Container: /mnt (bind)
      * Host: /mnt/nfs-1/

      https://www.portainer.io/products/community-edition
      https://hub.docker.com/r/portainer/portainer-ce 

## Rancher Server Docker Config
* Add rancher container in docker: sudo docker run -d --restart=unless-stopped -p 8080:8080 rancher/server
* Or add rancher container through portainer https://hub.docker.com/r/rancher/server

## Resources
1. RancherOS Installation Youtube Tutorial: https://www.youtube.com/watch?v=_Od8Z3iv54M
2. RancherOS Installation Ride Along: https://gist.github.com/s1lvester/fc5bcd4d3eb8e0c93b11854a01a0a034
3. Add user to sudoers: https://linuxize.com/post/how-to-add-user-to-sudoers-in-ubuntu/
4. wget https://raw.githubusercontent.com/redshift-s/rancheros-docker-media/master/rancheros-cloud-config.yml
5. RancherOS Share Config: https://github.com/redshift-s/rancheros-docker-media
6. Portainer Docker Documentation: https://documentation.portainer.io/v2.0/deploy/ceinstalldocker/

TODO
* [X] Add links to bottom and number for better flow
* [X] Add config files to github (enable git in commands)
* [X] Preserve helpful docs
* [X] add backups to walter documentation
* [x] Check command syntax in new VM & run files
* [X] Rebuild VM with backups at every point
  * [X] Test restoring from backups
* [x] Make TOC w/ Links and Links from root
* [] Add variables in docs
* [] Make public/private version of docs for sharing based off variables
* [] Script where possible

Make freenas docs //configure calibre testing


NFS storage youtube vid: https://www.youtube.com/watch?v=RNhqvx8y_8A 
https://github.com/rancher/os/blob/master/README.md