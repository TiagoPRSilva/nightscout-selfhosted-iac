# Self Hosted Nightscout IaC Configuration

The purpose of this repository is the define an easy to set up self hosted nightscout instance on docker. This configuration makes use of the fine tailored [linuxserver.io](https://docs.linuxserver.io/) docker images. When finished following this set up you will have a Nightscout instance:
- Accessible anywhere;
- Protected by tls with a lets encrypt self signed certificate;
- Certificate is automatically regenerated via certbot;
- Mechanism to automatically point your duckdns subdomain to your public ip, if your ISP changes you public ip, the changes are applied automatically on your duckdns.

**Table of Contents**

- [Requirements](#reference-documentation)
- [Install](#install)
- [Reference Documentation](#reference-documentation)

# Requirements

In order to self host these services you require:

- Duckdns account;
- Access to your home router to do some port forwarding;
- Any kind of home server (NAS, raspberry or regular pc) with docker installed;

# Install

## Duckdns

Create a subdomain on your duckdns account, put in your public ip (check your public ip https://www.whatismyip.com/).

## Port Forward

On your isp router define a static ip for your server on your LAN and forward port 443 to internal 1443 and 80 to internal 180 pointing to your servers local ip. (For port forwarding i had to contact my ISP, because that configuration was unfortunately controlled by them). i wont show specific here because this varies between IPS and routers. Just search for your router model on 'how to port forward'.

## Docker Environment variables configuration

After router configurations are done, clone this repository to a folder of your choice, and edit the .env file to your needs. 
For password variables, it is a good practise to use a password manager to generate strong passwords. Personally i use keepass and save the keepass file on dropbox, then i save and generate all of my passwords from there. Since its on dropbox you can access them easly on your smartphone, simply by installing a KPass on your android and point it to your dropbox/onedrive synced file.

- RESTART_MODE - the way containers handle restart (defaults to 'unless-stopped');
- TZ - the timezone to set in all the containers, find your appropriate value in this link https://en.wikipedia.org/wiki/List_of_tz_database_time_zones (use the value from column 'TZ database name');
- BASE_CONFIG_PATH - the base path for the containers bind mounts related to configurations. I suggest you use the cloned repository folder as the base path;
- BASE_DATA_PATH - the base path for the containers bind mounts related to data. This setting will be used as the base path of bind mounts related to data generated by the containers (for instance your mongo nightscout database). I diferentiated this env variable from the above, because data is more important than configurations, it has more volume and it should be placed somewhere with a backup mechanism implemented (like a dropbox folder, or a place in your filesystem that has a raid0);
- PUID - A user id that exists in your linux host, that you want to be the owner of the container files (run a 'cat /etc/passwd' to find your system users);
- PGID - Same as user id but for group (run a 'cat /etc/group' to find your system groups);
- MONGODB_PORT - The port to use in your mongo db instance;
- MONGODB_USER - The database user to define in your mongo db on container creation;
- MONGODB_PASSWORD - The database password to define in your mongo db on container creation;
- MONGODB_ROOT_USER - The root database user to define in your mongo db on container creation;
- MONGODB_ROOT_PASSWORD - The root database password to define in your mongo db on container creation;
- EMAIL - The email that will receive the certificate expiring notifications from swag service;
- DUCKDNS_FQDN - After creating a sub domain on your duckdns define this as pattern 'your-sub-domain.duckdns.org';
- DUCKDNS_SUBDOMAINS - Your duckdns subdomain 'your-sub-domain';
- DUCKDNS_TOKEN - Access your duckdns account, on the upper part of the main page there is a 'token' property, put its value here;
- NS_API_SECRET - Define the api-secret of your nightscout;
- NS_DB_NAME - The name of your nightscout database;
- NS_PORT - The port to use on your nightscout service (defaults to 1337);
- NS_TITLE - The title that appears on your nightscout instance main page;
- For the variables taht start with NS_, see [nightscout documentation](https://github.com/nightscout/cgm-remote-monitor#environment)

## Run the services

After all the configuration:

```bash
# change to the directory where the docker-compose.yml is.
cd to/cloned/repository/folder

# start the services. Let it finish the creation
sudo docker-compose up -d
```

After this you can see that everything is up and running by accessing portainer (portainer an optinal application that allows you to see your running containers easly and see their logs and a lot more). Portainer should be accesible by your lan server ip on port 9000.


Now in order to make your nightscout instance available via https a last step is required. Stop your services by executing the commands bellow

```bash
# change to the directory where the docker-compose.yml is.
cd to/cloned/repository/folder

# start the services. Let it finish the creation
sudo docker-compose down
```

## Nginx configuration

The final step is to configure your reverse proxy (nginx inside swag container) to route external requests to your nightscout instance, exposing it to the internet via https.

Navigate to this folder in your system 'BASE_CONFIG_PATH/swag/config/nginx/proxy-confs/', then create a file with the following content and add it to that folder.

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name nightscout.PUT-YOUR-DUCKDNS-SUBDOMAIN-HERE.duckdns.org;
    include /config/nginx/ssl.conf;
    client_max_body_size 0;

    location / {
        include /config/nginx/proxy.conf;
        include /config/nginx/resolver.conf;
        set $upstream_app nightscout;
        set $upstream_port 1337;
        set $upstream_proto http;
        proxy_pass $upstream_proto://$upstream_app:$upstream_port;
    }
}
```

Then navigate to this folder in your system 'BASE_CONFIG_PATH/swag/config/nginx/site-confs/', and open the file.

On this file ensure the follow content is not commented:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name _;
    return 301 https://$host$request_uri;
}
```

After this start the services again:

```bash
# change to the directory where the docker-compose.yml is.
cd to/cloned/repository/folder

# start the services. Let it finish the creation
sudo docker-compose up -d
```

Now if you access nightscout.you-duck-dns-subdomain.duckdns.org your nightscout instance should be available and ready for use.

# Questions/Issues

If you have any questions or issues submit it [here].

[here]: https://github.com/TiagoPRSilva/nightscout-selfhosted-iac/issues

# Reference Documentation

This is a list of important related documentation the services used in this configuration:

- linuxserverio swag image https://docs.linuxserver.io/images/docker-swag
- linuxserverio duckdns image https://docs.linuxserver.io/images/docker-duckdns
- bitnami mongodb image https://hub.docker.com/r/bitnami/mongodb
- nightscout documentation https://github.com/nightscout/cgm-remote-monitor



