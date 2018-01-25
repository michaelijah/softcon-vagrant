# -*- mode: ruby -*-
# vi: set ft=ruby :

#The format of the box, box_url, use of hostmanager plugin, sshfs, dnf caching, and more
#all come from https://fedoraproject.org/wiki/Vagrant

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  #config.vm.box = "fedora/27-cloud-base"
  #config.vm.box_url = "https://download.fedoraproject.org/pub/fedora/linux/releases"\
  #                    "/27/CloudImages/x86_64/images/Fedora-Cloud-Base-Vagrant"\
  #                    "-27-1.6.x86_64.vagrant-libvirt.box"
  config.vm.box = "fedora/26-cloud-base"
  config.vm.box_url = "http://mirror.us.leaseweb.net/fedora/linux/releases"\
                      "/26/CloudImages/x86_64/images/"\
                      "Fedora-Cloud-Base-Vagrant-26-1.5.x86_64.vagrant-libvirt.box"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  config.vm.box_check_update = false

  # This is an optional plugin that, if installed, updates the host's /etc/hosts
  # file with the hostname of the guest VM. In Fedora it is packaged as
  # ``vagrant-hostmanager``
  # This removes the need to care about or remember the vm's ip
  if Vagrant.has_plugin?("vagrant-hostmanager")
      config.hostmanager.enabled = true
      config.hostmanager.manage_host = true
      config.hostmanager.aliases = %w(artifactory.local.com gitbucket.local.com jenkins.local.com toplevel.local.com)
  end


  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # To cache update packages (which is helpful if frequently doing `vagrant destroy && vagrant up`)
  # you can create a local directory and share it to the guest's DNF cache. Uncomment the lines below
  # to create and use a dnf cache directory
  #
  Dir.mkdir('.dnf-cache') unless File.exists?('.dnf-cache')
  config.vm.synced_folder ".dnf-cache", "/var/cache/dnf", type: "sshfs", sshfs_opts_append: "-o nonempty"
  Dir.mkdir('gitbucket') unless File.exists?('gitbucket')
  config.vm.synced_folder "gitbucket", "/var/opt/gitbucket", type: "sshfs", sshfs_opts_append: "-o nonempty"
  Dir.mkdir('etc') unless File.exists?('etc')
  config.vm.synced_folder ".", "/vagrant", type: "sshfs", sshfs_opts_append: "-o nonempty"


  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "libvirt" do |lv|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
    lv.memory = "2048"
    lv.cpu_mode = "host-passthrough"

  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  #
  #This increases the startup timeout to 120 for artifactory's tomcat install
  config.vm.provision "shell", inline: "export START_TMO=120",run: "always"
  # This shell updates the VM, grabs and install artifactory, jenkins, gitbucket, creates a 4GB swap (for my slow server)
  config.vm.provision "shell", inline: <<-SHELL
    #Artifactory on my slowpoke server needs more than 1 minute to start
    #START_TMO is adjusted in this environment file.
    cp /vagrant/etc/environment /etc/environment
    source /etc/environment
    
    #Create swap space if it doesn't exist
    if [ ! -f  /swapfile ]; then
      touch /swapfile;
      #4GB of swapspace 4 * 2^20 = 4194304
      dd if=/dev/zero of=/swapfile bs=1024 count=4194304;
      chmod 0600 /swapfile;
      #make the swapspace file system
      mkswap /swapfile;
      #add the swapspace to /etc/fstab so it's always available on boot.
      sudo sed -i '$a /swapfile swap swap defaults 0 0' /etc/fstab;
      #Turn swap on so that it's immediately available
      swapon /swapfile;
    fi


#    dnf -y update
    dnf install -y wget
#    dnf install -y mysql
    wget -nc https://bintray.com/jfrog/artifactory-rpms/rpm -O bintray-jfrog-artifactory-rpms.repo
    sudo mv bintray-jfrog-artifactory-rpms.repo /etc/yum.repos.d/
    sudo dnf -y install jfrog-artifactory-oss
    chown -R artifactory:artifactory /var/opt/jfrog/run/
    rm /var/opt/jfrog/run/artifactory.pid
    #/opt/jfrog/artifactory/bin/configure.mysql.sh    
    dnf install -y jenkins
    dnf install -y vim
    dnf install -y links

    #setup for gitbucket
    sudo dnf install -y fedora-packager fedora-review
    sudo usermod -aG mock vagrant
    newgrp mock
    
    #grab the fedora core id number from the operating system
    #We are going to use that number to create/tag our rpm so
    #that we can move between versions of virtualization without ill effect
    source /etc/os-release
    echo "Using Fedora Core "$VERSION_ID
    cd /var/opt/gitbucket
    fedpkg --release f${VERSION_ID} local
    sudo rpm -i /var/opt/gitbucket/noarch/gitbucket*fc$VERSION_ID.noarch.rpm 
#    dnf remove -y fedora-packager fedora-review
    sudo systemctl enable jenkins
    sudo systemctl enable artifactory
    sudo systemctl enable gitbucket

    #Install and setup reverse proxy with nginx
    #Artifactory's X-Artifactory-Override-Base-Url only works in 
    #nginx v1.3.9 or higher. The normal dnf install method won't work
    #First Let's grab dependencies for nginx
    #rpm -Uvh http://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-stable.noarch.rpm 
    #rpm -Uvh http://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-stable.noarch.rpm
    #This dependency uses the version_id 
    #rpm -Uvh http://rpms.famillecollet.com/fedora/remi-release-$VERSION_ID.rpm

    #copy the repo file into the operating system so that we can pull the right version of software
    #cp /vagrant/etc/yum.repos.d/* /etc/yum.repos.d/

    #install nginx with php
    #dnf --enablerepo=remi --enablerepo=remi-php72 install nginx php-fpm php-common
    dnf -y install nginx
    #Setting this SE Linux policy allows http connect inside the host
    sudo setsebool -P httpd_can_network_connect true
    cp /vagrant/etc/nginx/conf.d/* /etc/nginx/conf.d/
    sudo systemctl enable nginx

  SHELL
end
