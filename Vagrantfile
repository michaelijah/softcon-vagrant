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
      config.hostmanager.aliases = %w(nexus.localdomain gitbucket.localdomain jenkins.localdomain toplevel.localdomain)
      config.hostmanager.ignore_private_ip = false
  end
  
  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.network :private_network, ip:"192.168.122.100" 
  
  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"
  config.vm.network :public_network, :dev => "virbr0", :mode => "bridge", :type => "bridge" 

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  #For nexus
  config.vm.network "forwarded_port", guest: 8081, host: 8081, protocol: "tcp", auto_correct: true, host_ip: "0.0.0.0"
  #For gitbucket
  config.vm.network "forwarded_port", guest: 8082, host: 8082, protocol: "tcp", auto_correct: true, host_ip: "0.0.0.0" 
  #For jenkins
  config.vm.network "forwarded_port", guest: 8080, host: 8080, protocol: "tcp", auto_correct: true, host_ip: "0.0.0.0" 
  
  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"
  # To cache update packages (which is helpful if frequently doing `vagrant destroy && vagrant up`)
  # you can create a local directory and share it to the guest's DNF cache. Uncomment the lines below
  # to create and use a dnf cache directory
  Dir.mkdir('.dnf-cache') unless File.exists?('.dnf-cache')
  config.vm.synced_folder ".dnf-cache", "/var/cache/dnf", type: "sshfs", sshfs_opts_append: "-o nonempty"
  
  NEXUS_UID = 10011
  NEXUS_GID = 10011
  Dir.mkdir('nexus-install') unless File.exists?('nexus-install')
  config.vm.synced_folder "nexus-install", "/var/lib/nexus-install", type: "sshfs", sshfs_opts_append: "-o uid=#{NEXUS_UID},gid=#{NEXUS_GID},nonempty"
  Dir.mkdir('nexus') unless File.exists?('nexus')
  config.vm.synced_folder "nexus", "/opt/nexus", type: "sshfs", sshfs_opts_append: "-o uid=#{NEXUS_UID},gid=#{NEXUS_GID},nonempty"
  Dir.mkdir('sonatype-work') unless File.exists?('sonatype-work')
  config.vm.synced_folder "sonatype-work", "/opt/sonatype-work", type: "sshfs", sshfs_opts_append: "-o uid=#{NEXUS_UID},gid=#{NEXUS_GID},nonempty"
  
  GITBUCKET_UID = 10001
  GITBUCKET_GID = 10001
  Dir.mkdir('gitbucket-home') unless File.exists?('gitbucket-home')
  config.vm.synced_folder "gitbucket-home", "/var/lib/gitbucket", type: "sshfs", sshfs_opts_append: "-o uid=#{GITBUCKET_UID},gid=#{GITBUCKET_GID},nonempty"
  Dir.mkdir('gitbucket-install') unless File.exists?('gitbucket-install')
  config.vm.synced_folder "gitbucket-install", "/var/lib/gitbucket-install", type: "sshfs", sshfs_opts_append: "-o uid=#{GITBUCKET_UID},gid=#{GITBUCKET_GID},nonempty"

  JENKINS_UID = 10002 
  JENKINS_GID = 10002
  Dir.mkdir('jenkins') unless File.exists?('jenkins')
  config.vm.synced_folder "jenkins", "/var/lib/jenkins", type: "sshfs", sshfs_opts_append: "-o uid=#{JENKINS_UID},gid=#{JENKINS_GID},nonempty"

  MYSQL_UID = 27 
  MYSQL_GID = 27 
  MYSQL_PASSWORD = 'G!tbuck3t' #This needs to have 1 Capital, 1 lower, 1 number and 1special character...! doesn't seem to work well
  Dir.mkdir('mysql') unless File.exists?('mysql')
  #The additional selinux context is crucial for mysql to run in Fedora.
  config.vm.synced_folder "mysql", "/var/lib/mysql", type: "sshfs", sshfs_opts_append: "-o uid=#{MYSQL_UID},gid=#{MYSQL_GID},nonempty,context=system_u:object_r:mysql_db_t:s0" 

  Dir.mkdir('eatmydata-install') unless File.exists?('eatmydata-install')
  config.vm.synced_folder "eatmydata-install", "/var/lib/eatmydata-install", type: "sshfs", sshfs_opts_append: "-o nonempty"

  Dir.mkdir('etc') unless File.exists?('etc')
  config.vm.synced_folder ".", "/vagrant", type: "sshfs", sshfs_opts_append: "-o nonempty"


  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  #
  config.vm.provider "libvirt" do |lv|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
    lv.memory = "2048"
    lv.cpu_mode = "host-passthrough"
    
  end
  
  #If the RAM available for VM (lv.memory) is less than 8GB then you should consider
  #Setting the variable below to 1 to force provisioning script to setup and use swap.
  #If it is more than 8GB you may be able to set to 0.
  SETUP_SWAP = 1
  
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  #
  
  # This shell updates the VM, grabs and installs nexus sonatype, jenkins, gitbucket, creates a 4GB swap (for my slow server)
  config.vm.provision "shell", inline: <<-SHELL
    #If there are any environment variables you want to
    #Add then place them here
    cp /vagrant/etc/environment /etc/environment
    #The host-side dnf.conf allows caching to be used
    #which speeds up subsequent spool-ups
    cp -f /vagrant/etc/dnf/dnf.conf /etc/dnf/dnf.conf
    source /etc/environment

    
    # Start by creating new user and group.
    groupadd -g #{NEXUS_GID} nexus
    useradd -c 'Nexus Repo User' -d /opt/nexus -g nexus -u #{NEXUS_UID} -s /bin/bash nexus
    passwd -d nexus
    # Creating new user and group.
    groupadd -g #{GITBUCKET_GID} gitbucket
    useradd -c 'Gitbucket UseR' -d /var/lib/gitbucket -g gitbucket -u #{GITBUCKET_UID} -s /sbin/nologin gitbucket
    #Add a jenkins group and user
    groupadd -g #{JENKINS_GID} jenkins
    useradd -c 'Jenkins Continuous Build Server' -d /var/lib/jenkins -g jenkins -u #{JENKINS_UID} -s /sbin/nologin jenkins
    #Add a mysql group and user
    groupadd -g #{MYSQL_GID} mysql
    useradd -c 'MySQL Server' -d /var/lib/mysql -g mysql -u #{MYSQL_UID} -s /sbin/nologin mysql


    #Create swap space if it doesn't exist
    if [ ! -f  /swapfile ] && [ #{SETUP_SWAP} -eq 1 ]; then
      touch /swapfile;
      #6GB of swapspace 6 * 2^20 = 4194304
      dd if=/dev/zero of=/swapfile bs=1024 count=6291456;
      chmod 0600 /swapfile;
      #make the swapspace file system
      mkswap /swapfile;
      #add the swapspace to /etc/fstab so it's always available on boot.
      sudo sed -i '$a /swapfile swap swap defaults 0 0' /etc/fstab;
      #Turn swap on so that it's immediately available
      swapon /swapfile;
    fi

    dnf install -y wget
    dnf install -y gcc automake autoconf libtool 
    #Install eatmydata to speed up provisioning process
    if [ "$(ls -A /var/lib/eatmydata-install 2> /dev/null)" == "" ]; then
       cd /var/lib/eatmydata-install
       wget https://launchpad.net/libeatmydata/trunk/release-105/+download/libeatmydata-105.tar.gz
       sudo tar -xvf /var/lib/eatmydata-install/libeatmydata-105.tar.gz --transform='s,/*libeatmydata-[^/]*/,libeatmydata/,'
    fi
    cd /var/lib/eatmydata-install/libeatmydata
    ./configure
    make
    make check
    sudo make install
    cd ~
    sudo echo "alias dnf='eatmydata dnf'" > /root/.bash_aliases

    dnf install -y jenkins
    dnf install -y vim
    dnf install -y links
    #To allow changes to security policy for nexus
    dnf install -y policycoreutils-devel

    sudo semanage permissive -a usr_t
    sudo semanage permissive -a var_t
    sudo semanage permissive -a init_t
    sudo semanage permissive -a fusefs_t
    sudo semanage permissive -a mysqld_db_t
    sudo semanage permissive -a mysqld_t

    #####################
    #Setup for Gitbucket
    #####################
    #
    source /etc/os-release
    if [ "$(ls -A /var/lib/gitbucket-install/noarch 2> /dev/null)" == "" ]; then
      sudo dnf install -y fedora-packager fedora-review
      sudo usermod -aG mock vagrant
      newgrp mock
    
      #grab the fedora core id number from the operating system
      #We are going to use that number to create/tag our rpm so
      #that we can move between versions of virtualization without ill effect
      echo "Using Fedora Core "$VERSION_ID
      cd /var/lib/gitbucket-install
      fedpkg --release f${VERSION_ID} local
    fi
    
    if [ "$(ls -A /var/lib/gitbucket 2> /dev/null)" == "" ]; then
      sudo rpm -i /var/lib/gitbucket-install/noarch/gitbucket*fc$VERSION_ID.noarch.rpm 
    else
      sudo rpm -i --excludepath /var/lib/gitbucket /var/lib/gitbucket-install/noarch/gitbucket*fc$VERSION_ID.noarch.rpm 
    fi

    ###
    #I don't really like the default h2 database
    #Let's configure an external database instead
    ###
    source /etc/os-release
    sudo dnf install -y http://dev.mysql.com/get/mysql57-community-release-fc$VERSION_ID-10.noarch.rpm
    sudo dnf install -y mysql-community-server

    sudo systemctl enable mysqld.service
    sudo systemctl start mysqld.service
    TEMP_MYSQL_PASSWORD=$(grep -oP '(?<=A temporary password is generated for root@localhost: )(.*)' /var/log/mysqld.log)
    echo "SET PASSWORD = PASSWORD('#{MYSQL_PASSWORD}');" | mysql -h 127.0.0.1 -u root -p'$TEMP_MYSQL_PASSWORD'
    echo "create database gitbucket;" | mysql -h 127.0.0.1 -uroot -p'#{MYSQL_PASSWORD}'
    #The next line means that the root and gitbucket user passwords are the same...you can change this or not.
    echo "grant all privileges on `gitbucket`.* to gitbucket@localhost identified by '#{MYSQL_PASSWORD}';" | mysql -h 127.0.0.1 -uroot -p'#{MYSQL_PASSWORD}'
    echo "flush privileges;" | mysql -h 127.0.0.1 -uroot -p'#{MYSQL_PASSWORD}'
   

    #####################
    #Setup Nexus
    #####################
    #
    if [ ! -f /var/lib/nexus-install/latest-unix.tar.gz ]; then
       #change to work dir
       cd /var/lib/nexus-install/
       #Then download fresh version of nexus. In my case v3
       wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
    fi

    if [ "$(ls -A /opt/nexus 2> /dev/null)" == "" ]; then
       cd $(dirname $NEXUS_HOME)
       su - nexus -c "tar -xvf /var/lib/nexus-install/latest-unix.tar.gz --transform='s,/*nexus-[^/]*/,nexus/,' nexus-*"
    fi

    if [ "$(ls -A /opt/sonatype-work 2> /dev/null)" == "" ]; then
       cd $(dirname $NEXUS_HOME)
       su - nexus -c "tar -xvf /var/lib/nexus-install/latest-unix.tar.gz --transform='s,/*nexus-[^/]*/,nexus/,'"
    fi

    sudo sh -c 'echo "export NEXUS_HOME=/opt/nexus" >> /etc/profile.d/nexus.sh'
    source /etc/profile.d/nexus.sh
    #
    #change to local
    #cd $(dirname $NEXUS_HOME)
    #
    #Extract the files out of tar but change nexus-3.x.x to simply nexus in the folder name
    #sudo tar -xvf /tmp/latest-unix.tar.gz --transform='s,/*nexus-[^/]*/,nexus/,'
    #Nexus needs at least 65536 available file descriptors let's change that here.
    sudo echo '* - nofile 65536' >> /etc/security/limits.conf
    sudo ln -s $NEXUS_HOME/bin/nexus /etc/init.d/nexus
    #
    # Creating new symlink to avoid version in path.
    # sudo echo 'run_as_user="nexus"' > $NEXUS_HOME/bin/nexus.rc
    su - nexus -c 'echo "run_as_user=nexus" > /opt/nexus/bin/nexus.rc' 

    sudo cp /vagrant/etc/systemd/system/nexus.service /etc/systemd/system/nexus.service
    cd /etc/init.d
    sudo chkconfig --add nexus
    sudo chkconfig --levels 345 nexus on
    sudo systemctl enable nexus.service
    cd ~
    #This will likely fail...pipe the failures into MyNexus security module and then activate
    sudo grep nexus /var/log/audit/audit.log | sudo audit2allow -a -M MyNexus
    sudo semodule -i MyNexus.pp
    sudo systemctl daemon-reload
    sudo systemctl start nexus.service
    sudo systemctl enable nexus.service

    #Install and setup reverse proxy with nginx
    #Artifactory's X-Artifactory-Override-Base-Url only works in 
    #nginx v1.3.9 or higher. The normal dnf install method won't work
    #First Let's grab dependencies for nginx
    #rpm -Uvh http://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-stable.noarch.rpm 
    #rpm -Uvh http://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-stable.noarch.rpm
    #This dependency uses the version_id 
    #rpm -Uvh http://rpms.famillecollet.com/fedora/remi-release-$VERSION_ID.rpm


    #install nginx with php
    dnf -y install nginx
    #Setting this SE Linux policy allows http connect inside the host
    sudo setsebool -P httpd_can_network_connect true
    cp /vagrant/etc/nginx/conf.d/* /etc/nginx/conf.d/
    sudo systemctl start nginx
    sudo systemctl enable nginx

    sudo systemctl start jenkins
    sudo systemctl start gitbucket
    sudo systemctl enable jenkins
    sudo systemctl enable gitbucket
  SHELL
end
