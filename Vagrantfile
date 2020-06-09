# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "debian/contrib-buster64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.network "private_network", ip: "192.168.50.11"
  config.vm.hostname = "ancgis.dev.net"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Getting BrowserSync working with Vagrant
  #config.vm.network :forwarded_port, guest: 80, host: 8080, auto_correct: true
  #config.vm.network :forwarded_port, guest: 3001, host: 3001, auto_correct: true

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Disable the default root
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.synced_folder "./ancgis", "/var/www/ancgis/sources", create: true, owner: "vagrant", group: "vagrant"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.
  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 2
    v.name = "ancgis-server"
  end

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
  config.vm.provision "shell", privileged: true, inline: <<-SHELL

    ########################
    # Packages and Sources #
    ########################
    # apt update and upgrade
    apt-get update
    DEBIAN_FRONTEND=noninteractive apt-get -y -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold" upgrade

    # Node and npm (npm is distributed with Node.js)
    apt-get install -y curl
    curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
    apt-get install -y nodejs
    # To compile and install native addons from npm you may also need to install build tools
    apt-get install -y build-essential

    # Port 80 requires elevated privileges
    # https://docs.requarks.io/wiki/troubleshooting#error-listening-on-port-xx-requires-elevated-privileges
    apt-get install -y libcap2-bin
    setcap 'cap_net_bind_service=+ep' `which node`

    # MongoDB
    wget -qO - https://www.mongodb.org/static/pgp/server-4.2.asc | sudo apt-key add -
    echo "deb http://repo.mongodb.org/apt/debian buster/mongodb-org/4.2 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.2.list
    apt-get update
    apt-get install -y mongodb-org
    sed -i 's/  bindIp: 127.0.0.1/#  bindIp: 127.0.0.1/g' /etc/mongod.conf
    service mongod restart
    # Enables MongoDB service on system start
    systemctl enable mongod.service

    # Sources
    apt-get install -y git
    # AncGIS
    rm -rdf /var/www/ancgis/sources/* # Required in case of previous VM build
    git clone -b develop https://github.com/sgalopin/ancgis.git /home/vagrant/ancgis # Sources can not be cloned directly into a synchronized folder
    mkdir -p /var/www/ancgis/sources
    cp -fa /home/vagrant/ancgis/. /var/www/ancgis/sources # -f option required in case of previous VM build
    rm -rdf /home/vagrant/ancgis
    echo 'PATH="$PATH:/var/www/ancgis/node_modules/.bin"' >> /home/vagrant/.profile # Required by the run scripts of package.json
    # Solves permission issue
    chgrp -R vagrant /var/www/ancgis
    chmod -R g+w /var/www/ancgis
    # AncDB
    rm -rdf /var/www/ancdb/sources/* # Required in case of previous VM build
    git clone -b develop https://github.com/sgalopin/ancdb.git /home/vagrant/ancdb # Sources can not be cloned directly into a synchronized folder
    mkdir -p /var/www/ancdb/sources
    cp -fa /home/vagrant/ancdb/. /var/www/ancdb/sources # -f option required in case of previous VM build
    rm -rdf /home/vagrant/ancdb
    echo 'PATH="$PATH:/var/www/ancdb/node_modules/.bin"' >> /home/vagrant/.profile # Required by the run scripts of package.json
    # Solves permission issue
    chgrp -R vagrant /var/www/ancdb
    chmod -R g+w /var/www/ancdb

    # AdminMongo
    git clone https://github.com/mrvautin/adminMongo.git /var/www/adminMongo
    cd /var/www/adminMongo && npm install electron-prebuilt --unsafe-perm=true # Bugfix for the electron-prebuilt post-installation (required by AdminMongo)
    cd /var/www/adminMongo && npm install
    cp /var/www/ancdb/sources/admin/config.json /var/www/adminMongo/config/config.json
    cp /var/www/ancdb/sources/admin/app.json /var/www/adminMongo/config/app.json
    # Solves permission issue
    chgrp -R vagrant /var/www/adminMongo
    chmod -R g+w /var/www/adminMongo

    # SendGrid
    echo "export SENDGRID_API_KEY='YOUR_API_KEY'" > sendgrid.env
    echo "sendgrid.env" >> .gitignore
    source ./sendgrid.env

  SHELL

  # Provision "npm-install"
  config.vm.provision "npm-install", type: "shell", privileged: false,  inline: <<-SHELL
    # Make the .env file
    cp /var/www/ancgis/sources/.env.dist /var/www/ancgis/sources/.env
    # Install the node_modules outside of the synced folder
    cp /var/www/ancgis/sources/package.json /var/www/ancgis
    cp /var/www/ancgis/sources/package-lock.json /var/www/ancgis
    cd /var/www/ancgis && npm install
    rm /var/www/ancgis/package.json
    rm /var/www/ancgis/package-lock.json
    # Build the package (node_modules/.bin must be into the PATH)
    cd /var/www/ancgis/sources && npm run build
  SHELL

  # Provision "populate-db"
  config.vm.provision "populate-db", type: "shell", privileged: false, inline: <<-SHELL
    # Make the .env file
    cp /var/www/ancdb/sources/shell/.env.dist /var/www/ancdb/sources/shell/.env
    # Populate the database
    cd /var/www/ancdb/sources/data && /bin/bash /var/www/ancdb/sources/shell/populate-db.sh
  SHELL

  # The following provisions are only run when called explicitly
  if ARGV.include? '--provision-with'

    # Provision "launch-app"
    config.vm.provision "launch-app", type: "shell", privileged: false, inline: <<-SHELL
      # cd /var/www/ancgis/ && sudo DEBUG=app:* npm start
      cd /var/www/ancgis/sources && npm run start:dev
    SHELL

    # Provision "launch-dba"
    config.vm.provision "launch-dba", type: "shell", privileged: false, inline: <<-SHELL
      cd /var/www/adminMongo && npm start
    SHELL

    # Provision "install-devtools"
    config.vm.provision "install-devtools", type: "shell", privileged: true, inline: <<-SHELL
      # Atom
      wget -qO - https://packagecloud.io/AtomEditor/atom/gpgkey | sudo apt-key add -
      sudo sh -c 'echo "deb [arch=amd64] https://packagecloud.io/AtomEditor/atom/any/ any main" > /etc/apt/sources.list.d/atom.list'
      # Chrome (for UI testing)
      wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
      echo "deb http://dl.google.com/linux/chrome/deb/ stable main" | sudo tee /etc/apt/sources.list.d/google-chrome.list

      sudo apt-get update
      sudo apt-get install -y atom google-chrome-stable meld
      # TODO: Add a file manager
      # https://wiki.debian.org/FileManager
      # https://www.tecmint.com/top-best-lightweight-linux-file-managers/
    SHELL

  end

end
