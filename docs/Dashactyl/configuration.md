---
sidebar_position: 3
---

# Configuration
This page goes over the `settings.yml` configuration and setting up the Nginx webserver for Dashactyl.

## Configuring your Settings
Because the `settings.yml` file is so large, this page will break down and explain each individual section.

```yaml
website:
  port: 8000
  secret: 'Enter your session password here.'
  secure: false

discord:
  id: "Enter your Discord OAuth2 ID here."
  secret: "Enter your Discord OAuth2 secret here."
  callbackpath: "http://localhost:8000/accounts/callback"
  prompt: false 

  token: "Insert your bot token here."
  guild: "000000000000000000"
```

The start of the settings file; The `port` is where Dashactyl will be running. Your Discord credentials can be found [at the developer portal](https://discord.com/developers/applications) with the application you are using for Dashactyl. When setting up the `callbackpath`, make sure it is also whitelisted in the portal: __Your Application > Oauth2 > Redirects__. The `token` is the token of your bot application for Dashactyl. This must be kept secret __at all times__ as it can be easily abused. The `guild` is your server ID if applicable. This is optional and is used to add clients to your server when logging into the dashboard.

```yaml
pterodactyl:
  domain: "Enter your Pterodactyl Panel domain here."
  key: "Enter your Pterodactyl Panel application API key here."
  generate_password_on_sign_up: true
```

This section focuses on the panel side of configuration. `domain` is your Pterodactyl domain. This must be prefixed with `https://` to work. If you are hosting locally then `localhost:<PORT>` can be used (with additional paths if applicable). `key` is the Pterodactyl Application API key for Dashactyl. This can be found or created by going to `your.pterodactyl.domain/admin/api`. This must be kept secret __at all times__ as it can be used to expose confidential information and destroy your panel. `generate_password_on_sign_up` is whether to create a new password for new accounts or use a default password.

```yaml
database:
  host: "Enter your database IP here."
  port: "Enter your database port here."
  user: "Enter your database user here."
  password: "Enter your database password here."
  database: "dashactyl"
```

(WIP) This section is for setting up the MySQL/MariaDB database for Dashactyl.

```yaml
api:
  apicodepassword:
    user info: true
    blacklist user: true
    unblacklist user: true
    set coins: true
    set package: true
    set resources: true
    create coupon: true
    revoke coupon: true
```

This section is for managing the Dashactyl API endpoints. Each option toggles whether the endpoint can be used publicly.

```yaml
locations:
  "1":
    name: "Location Name"
    enabled: true
    package: null

    # package:
    # - default
    # - another_package_name

    renewal: true
```

(WIP) This section is for the packages associated with user accounts.

```yaml
eggs:
  paper:
    display: "Paper"
    minimum:
      memory: 100
      disk: 100
      cpu: 10
    maximum:
      memory: null
      disk: null
      cpu: null
    info:
      egg: 3
      docker_image: quay.io/pterodactyl/core:java
      startup: java -Xms128M -Xmx{{SERVER_MEMORY}}M -Dterminal.jline=false -Dterminal.ansi=true -jar {{SERVER_JARFILE}}
      environment:
        SERVER_JARFILE: 'server.jar'
        BUILD_NUMBER: 'latest'
      feature_limits:
        databases: 1
        backups: 1
```

(WIP) This section is for the server configuration eggs in Pterodactyl. When creating a server through Dashactyl, the package associated with this egg will be used to create it.

```yaml
packages:
  default: "default"
  list:
    default:
      display: "The package name."
      memory: 1024
      disk: 1024
      cpu: 100
      servers: 1
    pro:
      display: "Pro Package"
      memory: 2048
      disk: 2048
      cpu: 200
      servers: 2
```

The packages displayed when a user is creating a server through Dashactyl. These packages can be customised and used with Dashactyl's currency system. The options shown are the specifications for the server(s) that will be given to the user in that package.

```yaml
store:
  memory:
    enabled: true
    cost: 10
    per: 10

  disk:
    enabled: true
    cost: 10
    per: 10

  cpu:
    enabled: true
    cost: 10
    per: 10

  servers:
    enabled: true
    cost: 10
    per: 10
```

This section is for the Dashactyl store configuration. `enabled` is whether the item should be purchasable in the shop. `cost` is how much the item costs and `per` is the amount of that item to give per purchase.

```yaml
afk:
  domain_lock: 
    - localhost:8000
  redirect_on_attempt_to_steal_code: https://www.youtube.com/watch?v=dQw4w9WgXcQ

  everywhat: 60
  gaincoins: 1

renewal:
  renewal_time: 6.048e+8
  deletion_time: 8.64e+7

  renew_fee: 10
```

(WIP)

## Setting Up Nginx
The Nginx webserver will allow us to use a custom domain name and apply SSL to it.

First, make sure you have Nginx and certbot installed:
```bash
sudo apt install nginx
sudo apt install certbot
sudo apt install -y python3-certbot-nginx
```

Now you can install your SSL certificate:
```bash
systemctl start nginx
certbot certonly --nginx -d <DASHACTYL_DOMAIN>
```

Make sure to replace `<DASHACTYL_DOMAIN>` with your domain name. If you have done this correctly you should see something similar to the following:
```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/your.dashactyl.domain/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/your.dashactyl.domain/privkey.pem
   Your cert will expire on date. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

Next you need to go to the nginx sites directory and create a configuration file:
```bash
cd /etc/nginx/sites-enabled
nano dashactyl.conf
```

Now paste the following into the file. Make sure to replace `<DOMAIN>` and `<PORT>` with your Dashactyl domain and the port Dashactyl is running on.
```conf
server {
  listen 80;
  server_name <DOMAIN>;
  return 301 https://$server_name$request_uri;
}
server {
  listen 443 ssl http2;

  server_name <DOMAIN>;
  ssl_certificate /etc/letsencrypt/live/<DOMAIN>/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/<DOMAIN>/privkey.pem;
  ssl_session_cache shared:SSL:10m;
  ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers  HIGH:!aNULL:!MD5;
  ssl_prefer_server_ciphers on;

  location / {
    proxy_pass http://localhost:<PORT>/;
    proxy_buffering off;
    proxy_set_header X-Real-IP $remote_addr;
  }
}
```

Once you have edited and saved your configuration file, restart Nginx with `systemctl restart nginx` and restart Dashactyl. You should see it running on that domain with SSL!