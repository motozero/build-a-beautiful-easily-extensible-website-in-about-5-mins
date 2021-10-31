# Ubuntu 20.04 to Awesome Website in ~5 mins 

## WHAT

How to spin up a basic yet beautiful and extensible website on any IaaS (like AWS, Azure, GCP) with a custom hostname, HTTPS, and SFTP, all nearly free and all within about 5 minutes!  

And although the resulting website can definitely stand on it's own, it's really intended to act as an origin for an Akamai CDN config which we'll be walking through setting up in a future post.

## WHY

Akamai is incredible at deliverying and securing the world's most trafficked and targeted web applications but it's not always so great at making it as frictionless as possible for people to learn about, demo and experiment with Akamai solutions.  Although this has defintely been improving, I still saw a need for some Hello World style jumping off points to facilitate the adoption of Akamai solutions.

This is why I started HelloWorld.ByAkamai.com, to try to make it as easy as I can for anyone (like customers, fellow SEs, curious web devs, interviewees, etc.) to try out Akamai solutions.

HelloWorld.ByAkamai.com will feature several How-To style trainings, all aimming to be ~5 minutes in length, and all building on eachother.  The intent here is to support those who are looking to dive deeper into solutioning with Akamai vs. those that may only want a quick, clear understanding of a specific solution without getting having to wade through information they don't want.

The purpose for this specific post is to walkthrough going from a linux install on an IaaS provider to hcing a fully functional, beautiful custom website ready to act as an origin for an Akamai config where various Akamai solutions can bolted on as desired.

## PREREQUISITES

1. An IaaS account (like AWS, Azure, GCP, Digital Ocean, Linode, etc.)  
  
    - You'll most likely have to put down a credit card if your company can't get you an account to use.
    
    - Most have a free tier for prototyping which will work fine for most cases where there's minmal traffic.
    
    - I plan on posting a video walkthrough of opening an account, spinning up Ubuntu 20.04 and SSHing in for each of the IaaS providers mentioned above.

    - I plan on reviewing the 5 IaaS providers above to compare their prototyping pricing tiers to see which is the cheapest.
 
2. A custom hostname purchsased through a Registrar (like Google Domains, Go Daddy, NameCheap, etc.) or you could ask someone like me if I can set up a custom subdomain on a domain I own.

3. A DNS resolver.  I beleive most Domain Registrars can act as a DNS resolver on your behalf.  I use domains.google.com as my registrar and they provide this service.  Akamai has a solution called Edge DNS that can do this job just as easily and potentially faster.

4. A Command line tool (like Terminal if you're on a Mac like me or Putty if your on Windows), personally I prefer iTerm 2 which is free)

5. (Optional) A Code Editor, I use the free version of Visual Studio Code.

6. (Optional) A SFTP GUI, I use the free version of Filezilla.

## INSTRUCTIONS

### 1. Setup Ubuntu 20.04

A. SSH in to Ubuntu 20.04 and run...
```
sudo su
apt update
```

### 2. Setup Nginx

A. Run...
```
apt install nginx
```
... then run...
```
ufw allow 80
ufw allow 443
ufw allow 22
curl -4 icanhazip.com
```
... then copy that IP and drop it into a browser.  You should see the Nginx Welcome page via HTTP (not HTTPS)

### 3. Setup your Custom Domain

A. Update the DNS of the custom hostname to include an A record pointing at the server's IP address.

B. Once this propagates, the same Nginx Welcome Page should be visible via the custom hostname via HTTP.

### 4. Setup a Cert for HTTPS delivery sing Let's Encrypt and Certbot

A. Run...
```
apt install letsencrypt
```
... then run...
```
systemctl status certbot.timer
```
... then run...
```
apt install python3-certbot-nginx
```
... then run...
```
certbot --nginx --agree-tos --preferred-challenges http -d YOUR-DOMAIN
```
... replacing YOUR-DOMAIN with your custom hostnamme.

B. From a browser navigate to the custom hostname and you should see the Nginx Welcome page via HTTPS instead of HTTP.

### 5. Setup SFTP 

A. Run...
```
apt-get install ssh-server -y
vi /etc/ssh/sshd_config
```

B. Comment out this...
```
Subsystem       sftp    /usr/lib/openssh/sftp-server
```
... like this...
```
#Subsystem       sftp    /usr/lib/openssh/sftp-server
```
... and change this...
```
PasswordAuthentication no
```
... to this...
```
PasswordAuthentication yes
```
... then add this at the end of the file...

```
Subsystem sftp internal-sftp
Match group ftpaccess
ChrootDirectory %h
X11Forwarding no
AllowTcpForwarding no
ForceCommand internal-sftp
```
... then save.

C. Run...
```
addgroup sftp
useradd -m sftpUser -g sftp
passwd sftpUser
```
... then enter a new password and then run...
```
chmod 700 /home/sftpUser # /home/sftpUser
systemctl restart ssh
```
... then run...
```
sftp sftpUser@YOUR-IP
```
... replacing YOUR-IP with the server's IP address.  Next you should be prompted for the password you created and once you enter it, the prompt should change to sftp>.  From here you can just type `exit` to get out of sftp.

D. Run...
```
chmod 777 -R /var/www/html  
```
  
### 6. Access the server via SFTP using Filezilla
  
A. Download a beautiful static site template for free from https://html5up.net

B. SFTP in via Filezilla and upload the template
  
### 7. Navigate to your hostname via HTTPS and see the completed site ready to use as an origin for an Akamai CDN config.

### 8. Set Up Security Response Headers

A. Run...

vi /etc/nginx/sites-available/default

B. In each server block, so within server { add...

    # Security Headers
        add_header X-Frame-Options DENY;
        add_header X-XSS-Protection "1";
        add_header Content-Security-Policy "default-src 'self'; font-src *;img-src * data:; script-src *; style-src *";

C. Run...

systemctl restart nginx
