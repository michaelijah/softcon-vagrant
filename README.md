# softcon-vagrant
A vagrantfile for libvirt fedora-cloud on fedora systems


You'll need to do a couple of things to prep your system for use.

1. Install Vagrant
-sudo dnf install vagrant

1a. Install Vagrant plugins needed by this Vagrantfile
-vagrant plugin install vagrant-sshfs
-vagrant plugin install vagrant-libvirt
-vagrant plugin install vagrant-hostmanager

2. If you plan on unleashing your creation onto the internet then you'll need to install nginx. Otherwise, you can skip step 2.
-sudo dnf install nginx

2a. You'll need to setup your nginx as a reverse proxy. Change EXAMPLE and com to match the domain and tld of your website. Save the file to /etc/nginx/conf.d/reverse_proxy.conf (or into your sites-enable/sites-available) hierarchy.
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
                resolver 127.0.0.1 ipv6=off;
        }

}

2b. The above example will proxy requests for jenkins.example.com to localhost:8080, nexus.example.com to localhost:8081, etc. Setting up SSL would be similar, I'll leave that as an exercise to the user.



3. You'll need to open the http port on the Host Computer. In fedora, run:  

sudo firewall-cmd --add-service=http --permanent  
sudo firewall-cmd --reload  

#You may need to reopen ssh on the Host Computer:  
#sudo firewall-cmd --add-service=ssh --permanent 
#sudo firewall-cmd --reload  

#Finally, you'll need to allow httpd to make connections on the Host Computer:  
#sudo setsebool -P httpd_can_network_connect 1  


