# docker-nextcloud-grafana-plex
Cloud tech project, Cloud storage with Nextcloud, Statistics with Grafana and Plex for streaming, all in Docker containers.

## Cloud technologies course project

## The assignment is as follows; Create a cloud service as a group of 2-4 members.

### Project members
*Axel Rusanen, Miikka Valtonen, Roope Varttila and Kristian Syrjänen.*

#### Initial plan

We will create a cloud service that contains Nextcloud (for cloudstorage), Grafana (for statistics) and Plex (for streaming). These will all be run in Docker containers.

### List of services
1. Dockered **Nextcloud**
2. Dockered **Grafana**
3. Dockered **Plex**

**1-3**. Run on *Amazon AWS EC2* instances.

*Possible implementations*:

* Reverse proxy (NGINX/Apache)
* Kubernetes (**High Priority**)

## Start off
Walkthrough of the steps made to complete the project.

### AWS EC2 Instance creation

Launching EC2 Instances with Amazon Web Services. Headed to AWS console and launched an **Ubuntu 16.04 LTS EC2 instance** (t2.small) with 20GB of Standard SSD storage. Connected to the VPS using the generated **private key** and default user, which in this case is *ubuntu*.

    ~$ ssh -i cloud_key.txt ubuntu@IP-ADDRESS

Creating users for group members.

    ~$ sudo adduser kristian
    (Provided information for user creation)
    ~$ sudo adduser kristian sudo
    ~$ sudo adduser kristian adm
    ~$ sudo adduser kristian admin

Completed same steps for all group members.

Next up is creation of public and private keys that are required to connect to the server.
Each step must be done with each user.

Switching user from *ubuntu* to *kristian*.

     ~$ sudo su kristian
The public keys are stored in the users home directory under .ssh/authorized_keys so we need to create those and give the right permissions for them.

     ~$ cd
     ~$ mkdir .ssh
     ~$ touch .ssh/authorized_keys
     ~$ cd
     ~$ chmod 700 .ssh
     ~$ chmod 600 .ssh/authorized_keys
After we've created the necessary directory and file we need to generate the actual public and private keys.

     ~$ ssh-keygen -t rsa -b 4096 -C "Kristians Key"
     Name it and save it.

Copy the public key you just created and add it to your **authorized_keys** file.

Save the private key on your desktop/laptop which you are working from and use it to connect to the instance.

     ~$ ssh -i $LOCATION\my-private-key.txt kristian@IP-ADDRESS

## References and materials
1. [Key generation with SSH](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)
2. second
3. third
4. fourth
