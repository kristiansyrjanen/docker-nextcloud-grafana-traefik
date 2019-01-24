# docker-nextcloud-grafana-traefik
Cloud tech project, Cloud storage with Nextcloud, Traefik and Statistics with Grafana all in Docker containers.

## Cloud technologies course project

## The assignment is as follows; Create a cloud service as a group of 2-4 members.

### Project members
*Axel Rusanen, Miikka Valtonen, Roope Varttila and Kristian Syrjänen.*

#### Initial plan

We will create a cloud service that contains Nextcloud (for cloudstorage), Grafana (for statistics) and Traefik. These will all be run in Docker containers.

### List of services
1. Dockered **Nextcloud**
2. Dockered **Grafana**
3. Dockered **Traefik**

**1-3**. Run on *Amazon AWS EC2* instances.

*Possible implementations*:

* ~Reverse proxy (NGINX/Apache)~ Replaced with Traefik
* Kubernetes (**High Priority**)

## Start off
Walkthrough of the steps made to complete the project.

### AWS EC2 Instance creation, user creation and connection management.

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

### Point your domain name to the EC2 instance.

Assign an Elastic IP address to your instance from the Network & Security tab, Elastic IPs section. Click *"Allocate new address"* and choose your EC2 instance. This assings an IP-address to your instance. 

Next open up Route 53 from the Services menu, under Networking & Content Delivery tab. We need to create a hosted zone. Once you press "Create hosted zone", you fill the form on the right with your domain name and select *Public Hosted Zone*. It will automatically create two records, NS and SOA record sets. Then we need two more **Type A** records. Press *Create Record Set*, leave the *name* field blank and select *A - IPv4 Address*. Enter the Elastic IP in the *Value* field and press the *Create* button. For the second button repeat the same steps but add **www** in the name field. Now all we need to is add the nameservers from the NS and SOA records to your domain name providers custom DNS settings.

## Security groups

To make the following applications exposed and visible to the internet we need to open up a number of different ports. These are done in the AWS Security Groups which you've already associated with your instance while creating it. At the moment we've only got one port open and that is port 22/tcp which is used for SSH.

## Traefik with Docker-Compose

To run containers with Docker-Compose you need to configure a YAML file called docker-compose.yml, this file contains all needed configurations your containers need. You can see what image Docker uses, which ports are exposed and more. Below is the docker-compose.yml file for running Traefik.

    version: '3'

    services:
      proxy:
        image: traefik:v1.6.6-alpine
        command: --web --docker --logLevel=INFO
        restart: unless-stopped
        networks:
          - web
        ports:
          - "80:80"
          - "443:443"
        labels:
          - "traefik.frontend.rule=Host:traefik.cloudgang.online"
          - "traefik.port=8080"
          - "traefik.frontend.auth.basic=admin:$$apr1$$okLV2vIj$$sQ5fMc.LH88SsD4uYbiMT1"
        volumes:
          - /var/run/docker.sock:/var/run/docker.sock
          - /opt/traefik/traefik.toml:/traefik.toml
          - /opt/traefik/acme.json:/acme.json
    networks:
      web:
        external: true

We also need a configuration file for traefik itself, it's called traefik.toml. Here you configure your ports, domain names, Let's Encrypt details and more. Below you can see the traefik.toml file.

    debug = false

    defaultEntryPoints = ["https","http"]

    [entryPoints]
      [entryPoints.http]
      address = ":80"
        [entryPoints.http.redirect]
        entryPoint = "https"
      [entryPoints.https]
      address = ":443"
      [entryPoints.https.tls]

    [retry]

    [docker]
    endpoint = "unix:///var/run/docker.sock"
    domain = "traefik.cloudgang.online"
    watch = true
    exposedbydefault = false

    [acme]
    email = "your@email"
    storage = "acme.json"
    entryPoint = "https"
    OnHostRule = true
    [acme.httpChallenge]
    entryPoint = "http"

    # Enable web configuration backend
    [web]

    # Traefik proxied GUI
    address = ":8080"

Lastly we need some basic authorization as we expose our traefik GUI to the internet. We add one more line to our docker-compose.yml file which gives us our basic auth. We use htpasswd to do this for our desired username.

    htpasswd -n admin

Type your password, retype it and then you will receive a string which is then used in you docker-compose.yml file. Dollar-signs ($) need to be escaped with another $ in YAML.

    admin:$$apr1$$1vNovEMY$$78uSc43ntXVyZCkRklESS0

Let's add the following string to our docker-compose.yml under the *- labels* section.

    - "traefik.frontend.auth.basic=admin:$$apr1$$1vNovEMY$$78uSc43ntXVyZCkRklESS0"

Now we need to run Docker-Compose and we should have Traefik up and running with basic authorization.

    sudo docker-compose up -d



## References and materials
1. [Key generation with SSH](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)
2. [Amazon Web Services](https://aws.amazon.com/)
3. third
4. fourth
