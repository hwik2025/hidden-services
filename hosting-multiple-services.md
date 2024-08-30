## Prerequisites

1.  **A VPS with root access**
2.  **Nginx installed**
3.  **Tor installed**

## Step 1: Install Nginx and Tor

First, let’s get Nginx and Tor up and running. Fire up your terminal and execute these commands:
_Update your package list and install Nginx and Tor:_
```
sudo apt update
sudo apt install nginx tor -y
```
## Step 3: Configure Tor for Multiple Hidden Services

Time to set up Tor to understand your master plan. Open Tor’s configuration file:
```
sudo nano /etc/tor/torrc
```
Add configurations for each hidden service:
```
HiddenServiceDir /var/lib/tor/mysite1/
HiddenServicePort 80 127.0.0.1:8080
HiddenServiceDir /var/lib/tor/mysite2/
HiddenServicePort 80 127.0.0.1:8080
```
**Note:** We’re reusing port 8080 internally for both services, but this is fine because they will be distinguished by their .onion addresses as _server_name_ in the config.

## Step 3: Restart Tor

Do this to generate your .onion addresses.
```
sudo systemctl restart tor
```
## Step 4: Get Your .onion Addresses

After restarting Tor, it will generate the .onion addresses for your hidden services. To find out what they are, check the respective hostname files:
```
sudo cat /var/lib/tor/mysite1/hostname
sudo cat /var/lib/tor/mysite2/hostname
```
## Step 5: Configure Nginx for Multiple Sites
We will be using long server_name directives, so we need to change the limit in /etc/nginx/nginx.conf.
```
sudo nano /etc/nginx/nginx.conf
```
Add (or uncomment) this line, change the value from 64 to 128:
```
server_names_hash_bucket_size 128;
```
It’s better to create configuration files for each site you want to host. Let’s set up two sites: `mysite1.onion` and `mysite2.onion`.
Navigate to Nginx’s configuration directory:
```
cd /etc/nginx/sites-available/
```
Create configuration files for each site:
```
sudo nano mysite1_onion
```
Add the following content for `mysite1` (adjust paths and settings as needed):
```
server {
    listen 127.0.0.1:8080;
    server_name [your first onion address];   # This is very important!
    root /var/www/mysite1;   # Adjust this to your site's root directory
    location / {
        try_files $uri $uri/ =404;
    }
}
```
Do the same for `mysite2`:
```
sudo nano mysite2_onion
```
Add the Nginx configuration:
```
server {
    listen 127.0.0.1:8080;
    server_name [your second onion address];  # This is very important!
    root /var/www/mysite2;   # Adjust this to your site's root directory

    location / {
        try_files $uri $uri/ =404;
    }
}
```
Enable these sites by creating symlinks in the `sites-enabled` directory:
```
sudo ln -s /etc/nginx/sites-available/mysite1_onion /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/mysite2_onion /etc/nginx/sites-enabled/
```
## Step 6: Create Web Directories

Ensure you have the directories for your sites:
```
sudo mkdir -p /var/www/mysite1
sudo mkdir -p /var/www/mysite2
```
And put some content in there to test:
```
echo "<html><body><h1>Welcome to mysite1.onion</h1></body></html>" | sudo tee /var/www/mysite1/index.html
echo "<html><body><h1>Welcome to mysite2.onion</h1></body></html>" | sudo tee /var/www/mysite2/index.html
```
## Step 7: Restart Nginx

Make sure your changes take effect by restarting Tor and Nginx:
```
sudo systemctl reload nginx
```
## Tying It All Together

So what’s happening under the hood? Here’s the cheat sheet:

-   **Nginx** is handling virtual hosts for your sites.
-   **Tor** is mapping those sites to unique .onion addresses.
-   **VPS** knows how to route and serve content request coming through Tor for each address.

Why Nginx?Nginx is known for its low resource usage and high performance, hosting multiple sites without breaking a sweat. Compared to other servers, its configuration for this setup is pretty straightforward.

## Troubleshooting Common Issues

-   **Site not loading**: Double-check those config files and make sure Nginx and Tor services are running without errors.
-   **404 Errors**: Ensure the root directories have the correct content and permissions.
-   **Overlap Issues**: If you’re seeing one site for another’s URL, ensure each site has a separate `HiddenServicePort` if you're using different ports.

And there you have it. With this setup, your single VPS is a multi-talented Tor host, running multiple hidden services like a pro. Happy hosting — cryptic style! If you hit any roadblocks, dive into the logs and configs. They’re your roadmap to the dark web!

h2 { margin-top: 0.14in; margin-bottom: 0.08in; background: transparent; page-break-after: avoid }h2.western { font-family: "Liberation Serif", serif; font-size: 18pt; font-weight: bold }h2.cjk { font-family: "Noto Serif CJK SC"; font-size: 18pt; font-weight: bold }h2.ctl { font-family: "Noto Sans Devanagari"; font-size: 18pt; font-weight: bold }pre { background: transparent }pre.western { font-family: "Liberation Mono", monospace; font-size: 10pt }pre.cjk { font-family: "Noto Sans Mono CJK SC", monospace; font-size: 10pt }pre.ctl { font-family: "Liberation Mono", monospace; font-size: 10pt }p { line-height: 115%; margin-bottom: 0.1in; background: transparent }em { font-style: italic }code.western { font-family: "Liberation Mono", monospace }code.cjk { font-family: "Noto Sans Mono CJK SC", monospace }code.ctl { font-family: "Liberation Mono", monospace }strong { font-weight: bold }
