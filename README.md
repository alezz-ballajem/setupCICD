# Sonod docker image for running Odoo CI workloads

This Odoo image has the following characteristics:

- Based on Ubuntu 22.04.
- Minimal dependencies to run Odoo CI jobs

  - git (with user.name=GitLab and user.email=gitlab@opsivist.io)
  - git-autoshare
  - mercurial
  - openssh-client
  - rsync
  - make
  - python3.8/3.9/3.10/3.11
  - virtualenv
  - postgresql client
  - lessc
  - wkhtmltopdf
  - unbuffer
  - gettext: useful to manipulate .po files
  - graphviz, poppler-utils, antiword: used by some Odoo versions
  - libreoffice-writer, libreoffice-calc: for py3o

- [pipx](https://pypi.org/project:pipx>)
- [manifestoo](https://pypi.org/project/manifestoo>)
- ssh client configuration

  - disable strict host key checking

- Odoo 16 is supported.
- Odoo is not preinstalled.
- Runs as non-privileged user named _gitlab-runner_

## Recommended mounts

It is recommended to mount the following volumes, for better build performance:

- ```/home/gitlab-runner/.cache/pip```
- ```/home/gitlab-runner/.cache/git-autoshare```
- ```/home/gitlab-runner/.cache/acsoo-wheel```, if you use the deprecated ```acsoo wheel``` command
--------------------------------------------------------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------------------------------------------------------

#Setup CI/CD FOR PROJECTS:

steps for create CI/CD for odoo projects:

use spacefic image "alezz77122/odoo-ci:22.4"

1- install gitlab-runner:
run this command to get gitlab-repository : 

	curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
run this command to install gitlab-runner latest version : 

	sudo apt-get install gitlab-runner

register in this gitlab-runner by run this command: 


       gitlab-runner register  --url https://gitlab.com  --token token_value
       
       choose name for this runner 
       
       we choose docker excuter in this runner 
       
       we choose this image "alezz77122/odoo-ci:22.4"


run this command to start gitlab-runner : 

	sudo service gitlab-runner start
------------------------------------------------------------------------------------------------------------------------------------------------------

create volumes for this gitlab runner to caches like git-autoshare and pip/cache

2- create two folders git-autoshare  and pip-cache
 and we give it privilage like the owner give it 1000:1000 and the 1000 is a user id for gitlab-runner in our container
 
 give this foldes full permisions  777
 and we add this folders in gitlab-runner config "/etc/runner-name/config.toml" file like volumes 
 volumes = ["/cache","/etc/pip-cache:/home/gitlab-runner/.cache/pip:rw","/etc/git-autoshare-cache:/home/gitlab-runner/.cache/git-autoshare:rw"]
 
 ------------------------------------------------------------------------------------------------------------------------------------------------------
 
 3- install postgresql in docker container or install it in all system 
    if you install it in docker container you have must add the ip address for this container in gitlab-runner config file 
    like this extra_hosts = ["pgdb:ip address for container"]
    extra_hosts = ["pgdb:172.17.0.5"]
-------------------------------------------------------------------------------------------------------------------------------------------------------

4- after all these steps we restart the gitlab-runner by run this command : 

   	sudo service gitlab-runner start
--------------------------------------------------------------------------------------------------------------------------------------------------------

5- when we create new project we rename it and copy all files and update almost files 
   remove the requirment.txt from the project and we can create spetial requirment.txt for this project by run this command :
   
   		pip df sync
  
---------------------------------------------------------------------------------------------------------------------------------------------------------

6- after that we run some commands inside the container:

	to entroll inside container run this command : docker exec -it pgdb /bin/bash
	
	we change diroctory by run this command : cd /var/lib/postgresql/data
	
	we run this command : "ls"  to list all files in this diroctory  
	
	we choose pg_hba.conf 	
	
	we change the configration by run this command : nano pg_hba.conf
	
	add this under IPv4 local connections:
	 # IPv4 local connections:
         host     all      gitlab-runner      172.17.0.0/16      trust
	
	 
-----------------------------------------------------------------------------------------------------------------------------------------------------------------                 
7- create ssh key in server and we add ssh private key as variable on gitlab repository it name is "ssh_deploy_key"

   we add the public key in this taps in gitlab project: Repository -> deploy keys  for push job 
   
    
