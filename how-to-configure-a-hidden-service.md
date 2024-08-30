Prerequisites Checklist
-----------------------

**1. A VPS or dedicated server**

Just make sure you have root access.

**2. Linux OS**

**3. Nginx installed**

**4. Tor installed**

--- -

Step 1: Install Nginx
=====================

Update and Install
------------------

First things first. Update your system packages.
```
sudo apt update
sudo apt upgrade -y
```
Then, install Nginx, enable it's service and start it.
```
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```
Congrats, you've got Nginx!

Step 2: Install Tor
===================

Tor is the secret sauce here. Let's install it.
```
sudo apt install tor -y
sudo systemctl enable tor
sudo systemctl start tor
```
Step 3: Configure Tor for Your .onion Site
==========================================

Open Tor's configuration file:
```
sudo nano /etc/tor/torrc
```
Add these lines to the beginning of the file:
```
HiddenServiceDir /var/lib/tor/hidden_service/
HiddenServicePort 80 127.0.0.1:8080
```
Save and exit (`Ctrl+X`, then `Y`, and `Enter`).
Restart Tor:
```
sudo systemctl restart tor
```
Step 4: Get Your .onion Address
===============================

After restarting, Tor will generate your .onion address. Check the file:
```
sudo cat /var/lib/tor/hidden_service/hostname
```
You should see something like: `abcd1234xyz.onion`

Step 5: Configure Nginx for the .onion Site
===========================================

Open your Nginx config file or create a new one:
```
sudo nano /etc/nginx/sites-available/default
```
Make it look something like this:
```
server {
    listen 127.0.0.1:8080;
    server_name abcd1234xyz.onion;
    root /var/www/html;  # Adjust as needed
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
Change nginx config to enable the long server_name.
```
sudo nano /etc/nginx/nginx.conf
```
Add (or uncomment) this line, change the value from 64 to 128:
```
server_names_hash_bucket_size 128;
```
Test your Nginx config for errors:
```
sudo nginx -t
```
If all looks good, reload Nginx:
```
sudo systemctl reload nginx
```
Step 6: Success!
================

Open the Tor Browser on your local machine and navigate to your .onion address. You should see your site!
