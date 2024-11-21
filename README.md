INSTALL OPENVAS COMMUNITY EDITION USING DOCKER CONTAINER WITH HTTPS ENABLED (Ubuntu 24.04)

For this, I've used the official guide https://greenbone.github.io/docs/latest/22.4/container/index.html and made a few changes to the docker-compose.yml file in order to enable HTTPS in the OpenVAS container. 
The steps of how to enable HTTPS are also here https://greenbone.github.io/docs/latest/22.4/container/workflows.html#setting-up-ssl-tls-for-gsa , however, they are out-of-date and won't work, so I had to take some parts of that guide and troubleshoot to make it work.

1. If you haven't install docker yet, make sure to install it. You can follow the steps below to install it:

https://greenbone.github.io/docs/latest/22.4/container/index.html#installing-docker

2. After you have docker up and running, you have to create the SSL certificate that will be used by the OpenVAS web interface.

sudo openssl req -x509 -newkey rsa:4096 -keyout serverkey.pem -out servercert.pem -nodes -days 397

3. Make sure the SSL key and certificate belong to the user 1001 and group 1001 byt doing:

sudo chown 1001:1001 *.pem

4. After creating the SSL certificate, you have to create a folder where you will store it, this command creates a hidden folder and move the SSL keys and cert to that folder.

sudo mkdir $HOME/.ssl && sudo mv serverkey.pem servercert.pem $HOME/.ssl  

You can check the contents of the folder by doing cat ls -l /.ssl

5. As a next step you should create a folder where you will store the docker compose file that will have all the instructions to pull and activate the OpenVAS containers.

sudo export DOWNLOAD_DIR=$HOME/greenbone-community-container && mkdir -p $DOWNLOAD_DIR

6. Go to the newly created folder:

cd /greenbone-community-container

7. The OpenVAS container uses a stack of containers (many containers) to work properly and syncronize feeds, but the one that handles the web interface is the GSA container and that's the one we are going to focus on.

While you're inside the greenbone-community-container folder, you can download the docker-compose.yml file that I've modified to make this work using the following command: wget https://raw.githubusercontent.com/jhurta05/openvas-docker-https/refs/heads/main/docker-compose.yml?token=GHSAT0AAAAAAC2YMK6X2DMTKKNIEXVTMWB6ZZ7QK7Q

8. After you have download the docker-compose.yml file, you have to modify the following lines:

sudo nano docker-compose.yml 

Modify lines: 
- Lines 241 to 245, make sure that the path is pointing to the right place where the server cert and key are stored.

9. Now we should be ready to pull and run the containers, however, we are not quite reeady yet.

Within the OpenVAS container Stack there's one container with the name gvm, this container is configure to use the user group 1001 by default, so is important that you have the user and groups 1001 configured under your OS, and that the user has sudo privileges. (This doesn't have to be an actual user that will access your console, but it can be something symbolic just to make this work), you can remove SSH access to that user from the ssh_config file.
Is important that this user has sudo privileges because only sudo users can bind ports below 1024, and port 443 is below 1024, also, you might get permission error when the deployment tries to move the certificate files from the OS to the docker container. 

You can check user and groups by using:

sudo cat /etc/passwd

If you don't have a user with ID and group 1001, you have to create one. The result should look like this:

ubuntu:x:1001:1001:Ubuntu:/home/ubuntu:/bin/bash

10. To pull the docker images just do:

sudo docker compose pull

11. After you have pulled the containers, you can proceed to run it:

sudo docker compose up

12. You now should be able to access the web interface of the OpenVAS container (the docker compose file is already configured to allow remote access to the web interface).

https://IP_ADDRESS/

13. Credentials:

Username: admin
Password: admin


I hope you enjoy. Please let me know if you have any questions.


