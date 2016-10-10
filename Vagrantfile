# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

settings = YAML.load_file 'config.yml'

Vagrant.configure("2") do |config|

    config.vm.box = "scotch/box"
    config.vm.network "private_network", ip: settings['site']['ip']
    config.vm.hostname = settings['site']['sitename']
    config.vm.synced_folder ".", "/var/www/public", :mount_options => ["dmode=777", "fmode=666"]
    
    # Optional NFS. Make sure to remove other synced_folder line too
    #config.vm.synced_folder ".", "/var/www/public", :nfs => { :mount_options => ["dmode=777","fmode=666"] }
	
    config.vm.provider "virtualbox" do |vb|
         # Display the VirtualBox GUI when booting the machine
         # vb.gui = true

         # Customize the amount of memory on the VM:
         vb.memory = "1024"
    end

    config.vm.provision "shell", :args => [settings['site']['sitename']], inline: <<-SHELL

	sitename=$1
	scotchbox="/etc/apache2/sites-enabled/scotchbox.local.conf"
	if [ -f "$scotchbox" ]
	then
		echo "Disabling scotchbox.local.conf. Will probably tell you to restart Apache..."
		sudo a2dissite scotchbox.local.conf
		echo "So let's restart apache..."
		sudo service apache2 restart
	else
		echo "scotchbox.local.conf not found. No cleanup needed."
	fi

	phpmyadmin="/etc/phpmyadmin"
	if [ -d "$phpmyadmin" ]
	then
		echo "PHPMyAdmin already installed."
	else
		echo "Installing PHPMyAdmin..."
		PASSWORD='password'
		mysql -uroot -proot -e "SET PASSWORD FOR 'root'@'localhost' = PASSWORD('$PASSWORD'); FLUSH PRIVILEGES;"
		sudo debconf-set-selections <<< "phpmyadmin phpmyadmin/dbconfig-install boolean true"
		sudo debconf-set-selections <<< "phpmyadmin phpmyadmin/app-password-confirm password $PASSWORD"
		sudo debconf-set-selections <<< "phpmyadmin phpmyadmin/mysql/admin-pass password $PASSWORD"
		sudo debconf-set-selections <<< "phpmyadmin phpmyadmin/mysql/app-pass password $PASSWORD"
		sudo debconf-set-selections <<< "phpmyadmin phpmyadmin/reconfigure-webserver multiselect apache2"
		sudo apt-get -y install phpmyadmin
	fi
    SHELL
end
