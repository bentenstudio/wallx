Parse Server
-------
There are many server providers out there, but DigitalOcean is recommended due to its cheap plans. You can get $10 credit (2-month worth) by registering [here](https://m.do.co/c/b32e35e3d2be)
<br>
## Setting Up VPS
#### Step 1: Create a Droplet

 1. Create a new Droplet [here](https://cloud.digitalocean.com/droplets/new)
 2. Choose Ubuntu 14.04
 3. Choose a size that meets your requirement
 4. Choose preferred datacenter region
 5. Check User Data box and enter following script
 6. Fill in other optional information and click Create
 7. Once done, you will receive server information in your email inbox

*User Data script:*

    #cloud-config
    apt_sources:
      # Enable MongoDB repository
      - source: deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.0 multiverse
        keyid: 7F0CEB10
        filename: mongodb.list
    apt_update: true
    packages:
      - mongodb-org
      - git
      - bc
      - nginx
    runcmd:
      - sudo git clone https://github.com/letsencrypt/letsencrypt /opt/letsencrypt
      - cd ~
      - curl -sL https://deb.nodesource.com/setup_5.x -o nodesource_setup.sh
      - sudo -E bash ./nodesource_setup.sh
      - sudo apt-get install -y nodejs build-essential git
      - sudo npm install -g parse-server pm2

#### Step 2: Setup Domain Name
With this setup, you will be able to use your own domain name to use as Parse Server's API endpoint. For a detailed guide, see [How To Set Up a Host Name with DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-host-name-with-digitalocean) tutorial. If you need help with setting the DNS servernames, take a look at [How To Point to DigitalOcean Nameservers from Common Domain Registrars](https://www.digitalocean.com/community/tutorials/how-to-point-to-digitalocean-nameservers-from-common-domain-registrars)

#### Step 3: Installing SSH Client
You will need to install a SSH client to connect to the VPS. Some good clients to use:
  - [PuTTY](http://www.putty.org/)
  - [WinSCP](https://winscp.net)
  - [SuperPuTTY](https://github.com/jimradford/superputty/releases)
  - [PuTTYtray](https://puttytray.goeswhere.com/)
  - [KiTTY](http://www.9bis.net/kitty/)
Follow [this tutorial](https://www.digitalocean.com/community/tutorials/how-to-use-ssh-keys-with-putty-on-digitalocean-droplets-windows-users) to connect to DigitalOcean VPS using PuTTY.
<br>
## Setting Up SSL Certificate (Optional)
Let's Encrypt is a new Certificate Authority that provides an easy way to obtain free TLS/SSL certificates. Because a certificate is necessary to secure both the migration of data to MongoDB and your Parse Server API endpoint, we'll begin by retrieving one with the `letsencrypt` client.

#### Step 1: Install Let's Encrypt
This step is done for you by a custom script entered during Droplet creation.

#### Step 2: Retrieve Initial Certificate
Change to the `letsencrypt` directory:

    cd /opt/letsencrypt
    
Run `letsencrypt` with the Standalone plugin:
    
    sudo service nginx stop
    ./letsencrypt-auto certonly --standalone

You'll be prompted to answer several questions, including your email address, agreement to a Terms of Service, and the domain name(s) for the certificate. Once finished, you'll receive notes much like the following:

```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at
   /etc/letsencrypt/live/your_domain_name/fullchain.pem. Your cert will expire
   on 2016-05-16. To obtain a new version of the certificate in the
   future, simply run Let's Encrypt again.
 - If you like Let's Encrypt, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```
Note the path and expiration date of your certificate, highlighted in the example output. Your certificate files should now be available in /etc/letsencrypt/**your_domain_name**/

#### Step 3: Set Up Let's Encrypt Auto Renewal
!!! note-danger ""
    **_Warning:_** You can safely complete this guide without worrying about certificate renewal, but you will need to address it for any long-lived production environment.

You may have noticed that your Let's Encrypt certificate is due to expire in 90 days. This is a deliberate feature of the Let's Encrypt approach, intended to minimize the amount of time that a compromised certificate can exist in the wild if something goes wrong.

Let's Encrypt is still in beta. Better auto-renewal features are planned, but in the meanwhile you will either have to repeat the certificate retrieval process by hand, or use a scheduled script to handle it for you. The details of automating this process are covered in [How To Secure Nginx with Let's Encrypt on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-14-04), particularly the section on [setting up auto renewal](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-14-04#step-4-%E2%80%94-set-up-auto-renewal).

<br>
## Configure MongoDB
#### Create an Admin User
We will create a `parse` user to add to server configuration. First, we need to enable authentication in order to create a new user. Run below command to edit `/etc/mongid.conf`

    sudo nano /etc/mongod.conf

Under `# security`, enable client authorization:

```yaml
# security
security:
  authorization: enabled
```
Exit and save the file.

Before restarting the `mongod` service, we need to add a user with the `admin` role. Connect to the running MongoDB instance:

    mongo --port 27017


Create an admin user. Be sure to replace !!text-danger|password!! with  a strong password.
```bash
use admin
db.createUser({
  user: "parse",
  pwd: "password",
  roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
})
exit
```
#### Create a Database
If you have `Let's Encrypt` SSL certificates, run following command. Make sure to replace `myUsername` with yours.
```bash
mongo --port 27017 --ssl --sslAllowInvalidCertificates --authenticationDatabase admin --username myUsername --password
```

Otherwise, run this command:
```bash
mongo --port 27017 admin --username myUsername--password
```

You will be prompt to enter the password you created earlier.

Once logged in, you will need to create a database and a user that is used for migration. Run following command with replacing !!text-danger|**database_name**!! with your own.

	use database_name
	db.createUser({ user: "parse", pwd: "password", roles: [ "readWrite", "dbAdmin" ] })

Restart the mongod service:

    sudo service mongod restart

<br>
## Installing Parse Server
<!--- 
#### Step 1: Install Node.js and Development Tools
Connect to VPS using SSH channel. In PuTTY command window, type (or copy/paste) following commands into the terminal.

```bash
cd ~
curl -sL https://deb.nodesource.com/setup_5.x -o nodesource_setup.sh
sudo -E bash ./nodesource_setup.sh
sudo apt-get install -y nodejs build-essential git
```

#### Step 2: Install Parse Server module
 --->
Use `npm` to install the `parse-server` utility, the `pm2` process manager, and their dependencies, globally:

    sudo npm install -g parse-server pm2

Instead of running `parse-server` as **root** or your `sudo` user, we'll create a system user called **parse**:

```bash
sudo useradd --create-home --system parse
```

Now set a password for **parse**:

	sudo passwd parse

You'll be prompted to enter a password twice.

Now, use the `su` command to become the **parse** user:

	sudo su parse

<br>
## Retrieve Keys
You'll need to retrieve some of the keys for your app. In the Parse dashboard, click on **App Settings** followed by **Security & Keys**:

![](/assets/images/small-007.png)

Since we only have Parse integrated for Android, you only need to collect **Master Key**, **Application ID** and **Client Key** to proceed with further setup. Using the same keys as before will not require you setup the source code again.
<br>
## Configure Parse Server
**PM2** is a feature-rich process manager, popular with Node.js developers. We'll use the `pm2` utility to configure our `parse-server` instance and keep it running over the long term.

#### Step 1: Configure pm2
Change to **parse**'s home directory:

	cd ~

We will edit the file called `/home/parse/ecosystem.json`

    nano ecosystem.json

Paste the following, changing configuration values to reflect your MongoDB connection string, Application ID, and Master Key:

```json
{
  "apps" : [{
    "name"        : "parse-wrapper",
    "script"      : "/usr/bin/parse-server",
    "watch"       : true,
    "merge_logs"  : true,
    "cwd"         : "/home/parse",
    "env": {
      "PARSE_SERVER_CLOUD_CODE_MAIN": "/home/parse/cloud/main.js",
      "PARSE_SERVER_DATABASE_URI": "mongodb://parse:password@your_domain_name:27017/database_name?ssl=true",
      "PARSE_SERVER_APPLICATION_ID": "your_application_id",
      "PARSE_SERVER_MASTER_KEY": "your_master_key",
      "PARSE_SERVER_CLIENT_KEY": "your_client_key",
      "PARSE_SERVER_FACEBOOK_APP_IDS": "your_facebook_app_id"
    }
  }]
}
```

You can also configure other options below:

- `PARSE_SERVER_URL`
- `PARSE_SERVER_JAVASCRIPT_KEY`
- `PARSE_SERVER_REST_API_KEY`
- `PARSE_SERVER_DOT_NET_KEY`
- `PARSE_SERVER_ENABLE_ANON_USERS`

Other options with explanations can be found at [ParsePlatform/parse-server](https://github.com/ParsePlatform/parse-server/blob/master/src/cli/cli-definitions.js)

Exit and save `ecosystem.json`
#### Step 2: Start parse-server with pm2
Now run the script with `pm2`

    pm2 start ecosystem.json
    
*Sample Output*

    ...
    [PM2] Spawning PM2 daemon
    [PM2] PM2 Successfully daemonized
    [PM2] Process launched
    ┌───────────────┬────┬──────┬──────┬────────┬─────────┬────────┬─────────────┬──────────┐
    │ App name      │ id │ mode │ pid  │ status │ restart │ uptime │ memory      │ watching │
    ├───────────────┼────┼──────┼──────┼────────┼─────────┼────────┼─────────────┼──────────┤
    │ parse-wrapper │ 0  │ fork │ 3499 │ online │ 0       │ 0s     │ 13.680 MB   │  enabled │
    └───────────────┴────┴──────┴──────┴────────┴─────────┴────────┴─────────────┴──────────┘
    Use `pm2 show <id|name>` to get more details about an app

Now tell `pm2` to save this process list:

    pm2 save
    
!!! sample-output "Sample Output"
    [PM2] Dumping processes
    
The list of processes `pm2` is running for the parse user should now be stored in `/home/parse/.pm2`.

Now we need to make sure the `parse-wrapper` process we defined earlier in ecosystem.json is restored each time the server is restarted. Fortunately, `pm2` can generate and install a script on its own.

Exit to your regular `sudo` user:

    exit
    
Tell pm2 to install initialization scripts for Ubuntu, to be run as the parse user, using /home/parse as its home directory:

    sudo pm2 startup ubuntu -u parse --hp /home/parse/
    
*Sample Output*

    [PM2] Spawning PM2 daemon
    [PM2] PM2 Successfully daemonized
    [PM2] Generating system init script in /etc/init.d/pm2-init.sh
    [PM2] Making script booting at startup...
    [PM2] -ubuntu- Using the command:
        su -c "chmod +x /etc/init.d/pm2-init.sh && update-rc.d pm2-init.sh defaults"
    System start/stop links for /etc/init.d/pm2-init.sh already exist.
    [PM2] Done.    

<br>

## Configure Nginx
We'll use the Nginx web server to provide a reverse proxy to `parse-server`, so that we can serve the Parse API securely over TLS/SSL. If you did not install `Let's Encrypt`, you can skip this part.

Open `/etc/nginx/sites-enabled/default`

    sudo nano /etc/nginx/sites-enabled/default
    
Replace its contents with the following (make sure to replace **your_domain_name**):

    # HTTP - redirect all requests to HTTPS
    server {
        listen 80;
        listen [::]:80 default_server ipv6only=on;
        return 301 https://$host$request_uri;
    }

    # HTTPS - serve HTML from /usr/share/nginx/html, proxy requests to /parse/
    # through to Parse Server
    server {
            listen 443;
            server_name your_domain_name;

            root /usr/share/nginx/html;
            index index.html index.htm;

            ssl on;
            # Use certificate and key provided by Let's Encrypt:
            ssl_certificate /etc/letsencrypt/live/your_domain_name/fullchain.pem;
            ssl_certificate_key /etc/letsencrypt/live/your_domain_name/privkey.pem;
            ssl_session_timeout 5m;
            ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
            ssl_prefer_server_ciphers on;
            ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';

            # Pass requests for /parse/ to Parse Server instance at localhost:1337
            location /parse/ {
                    proxy_set_header X-Real-IP $remote_addr;
                    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header X-NginX-Proxy true;
                    proxy_pass http://localhost:1337/;
                    proxy_ssl_session_reuse off;
                    proxy_set_header Host $http_host;
                    proxy_redirect off;
            }

            location / {
                    try_files $uri $uri/ =404;
            }
    }

Exit the editor and save the file. Restart Nginx so that changes take effect:

    sudo service nginx restart
    

<br>
##  Storing files with AWS S3 (Optional)
Please note that current files stored on Parse are not migrated yet since Parse team is working on a solution at [ParsePlatform/parse-server/wiki/Files-Migration](https://github.com/ParsePlatform/parse-server/wiki/Files-Migration). Stay tuned until we have the final solution from them. In the meantime, your files are still working and newly uploaded files will be stored on your server. 

To use AWS S3 as file storage rather than current server, check tutorial on Github: [ParsePlatform/parse-server/wiki/Storing-Files-in-AWS-S3](https://github.com/ParsePlatform/parse-server/wiki/Storing-Files-in-AWS-S3)