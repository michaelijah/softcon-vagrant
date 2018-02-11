# softcon-vagrant
A vagrantfile for libvirt fedora-cloud on fedora systems


You'll need to open the http port on the Host Computer. In fedora, run:  

sudo firewall-cmd --add-service=http --permanent  
sudo firewall-cmd --reload  

You may need to reopen ssh on the Host Computer:  
sudo firewall-cmd --add-service=ssh --permanent 
sudo firewall-cmd --reload  

Finally, you'll need to allow httpd to make connections on the Host Computer:  
sudo setsebool -P httpd_can_network_connect 1  


