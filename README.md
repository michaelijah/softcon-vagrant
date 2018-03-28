# softcon-vagrant
A vagrantfile for libvirt fedora-cloud on fedora systems


You'll need to do a couple of things to prep your system for use.

THIS VIRTUALIZATION USES NFS. NFS IS ONLY COMPATIBLE WITH A WINDOWS HOST BY USING A 3RD PARTY PLUGIN

1. Install Vagrant `sudo dnf install vagrant`
2. If you plan on unleashing your creation onto the open internet then you'll need to install nginx. Otherwise, you can skip step 2.
   * sudo dnf install nginx
   * You'll need to setup your nginx as a reverse proxy. Change EXAMPLE and com to match the domain and tld (top-level domain) of your website. Save the text below in a file to /etc/nginx/conf.d/reverse_proxy.conf (or into your sites-enable/sites-available) hierarchy.
   * The below code example will proxy requests for jenkins.example.com to localhost:8080, nexus.example.com to localhost:8081, etc. 
```
map $subdomain $subdomain_port {
               default   8081;
               www       8081;
               jenkins   8080;
               nexus     8081;
               gitbucket 8082;
}

server {
        listen 80;
        listen [::]:80;

        server_name ~^(?P<subdomain>.+)\.EXAMPLE\.com$;

        location / {
                proxy_pass http://127.0.0.1:$subdomain_port;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
        }

}
```
   3. You'll need to open the http port on the Host Computer. In fedora, run:  
   * `sudo firewall-cmd --add-service=http --permanent`
   * `sudo firewall-cmd --reload`
   4. You need to install and setup NFS (networked file system). The vagrant community offers a few plugins but they are outdated, unsupported, and abandoned. The best thing to do is to install and setup NFS onto the host operating system.
   * The following instructions were shamelessly pilfered from https://www.itzgeek.com/how-tos/linux/centos-how-tos/how-to-setup-nfs-server-on-centos-7-rhel-7-fedora-22.html. Plese visit them to show your support!
```
sudo dnf install nfs-utils libnfsidmap
sudo systemctl enable rpcbind
sudo systemctl start rpcbind
sudo systemctl start nfs-server
sudo systemctl start rpc-statd
sudo systemctl start nfs-idmapd
sudo systemctl enable nfs-server
#Configure the firewall to allow NFS connectivity
firewall-cmd --permanent --add-service mountd
firewall-cmd --permanent --add-service rpc-bind
firewall-cmd --permanent --add-service nfs
firewall-cmd --reload
``` 
5. Finally, you'll need to allow httpd to make connections on the Host Computer:  
   * `sudo setsebool -P httpd_can_network_connect 1`
   ***
   **You can stop here if your vm won't be on the open internet**
   ***
6. You should consider securing your website if it is going to be served on the open web. Let's secure our HTTPS with Let's Encrypt
   * Uncomment "proxy_set_header X-Forwarded-Proto https;" in the file you created in step 2 (/etc/nginx/conf.d/reverse_proxy.conf)
   * `sudo dnf install certbot-nginx`
   * `sudo firewall-cmd --permanent --add-service=https`
   * Change example.com in the following step to match your domain and top-level domain
   * `sudo certbot --nginx -d example.com -d www.example.com -d jenkins.example.com -d gitbucket.example.com -d nexus.example.com`
   * Enter your email address
   * Allow the Let's Encrypt to modify your conf files to redirect all traffic to https.
   * `sudo nginx -s reload`
7. If you have set up an ssh server then you should open a port for the service
   * You may need to reopen ssh on the Host Computer:  
   * sudo firewall-cmd --add-service=ssh --permanent 
   * sudo firewall-cmd --reload 

