# Vagrant-IAAC-Multiple-VMs-Deployment
Infrastructure As A Code Deployment of multiple VMs using Vagrant. The provision script below deploys both a WordPress website & static HTML website onto 2 seperate Ubuntu VMs programmatically.

## Deployment Code
```
Vagrant.configure("2") do |config|
  config.vm.provision "shell", inline: "echo Hello"

  config.vm.define "static" do |static|
    static.vm.box = "ubuntu/focal64"
    static.vm.network "private_network", ip: "192.168.33.14"
    static.vm.network "public_network"
    static.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = "1"
    end

    static.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y apache2
      sudo apt-get install wget unzip -y
      cd /tmp/
      wget https://www.tooplate.com/zip-templates/2119_gymso_fitness.zip
      unzip -o 2119_gymso_fitness.zip
      cp -r 2119_gymso_fitness/* /var/www/html/
    SHELL
  end

  config.vm.define "wp" do |wp|
    wp.vm.box = "ubuntu/focal64"
    wp.vm.network "private_network", ip: "192.168.33.15"
    wp.vm.network "public_network"
    wp.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = "1"
    end

    wp.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install apache2 \
        ghostscript \
        libapache2-mod-php \
        mysql-server \
        php \
        php-bcmath \
        php-curl \
        php-imagick \
        php-intl \
        php-json \
        php-mbstring \
        php-mysql \
        php-xml \
        php-zip -y
      
      sudo mkdir -p /srv/www
      sudo chown www-data: /srv/www
      curl https://wordpress.org/latest.tar.gz | sudo -u www-data tar zx -C /srv/www
      cp /vagrant/wordpress.conf /etc/apache2/sites-available/wordpress.conf

      sudo a2ensite wordpress
      sudo a2enmod rewrite
      sudo a2dissite 000-default
      sudo service apache2 reload

      mysql -u root -e 'CREATE DATABASE wordpress;'
      mysql -u root -e 'CREATE USER wordpress@localhost IDENTIFIED BY "12345";'
      mysql -u root -e 'GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.* TO wordpress@localhost;'
      mysql -u root -e 'FLUSH PRIVILEGES;'

      sudo -u www-data cp /srv/www/wordpress/wp-config-sample.php /srv/www/wordpress/wp-config.php
      sudo -u www-data sed -i 's/database_name_here/wordpress/' /srv/www/wordpress/wp-config.php
      sudo -u www-data sed -i 's/username_here/wordpress/' /srv/www/wordpress/wp-config.php
      sudo -u www-data sed -i 's/password_here/12345/' /srv/www/wordpress/wp-config.php
    SHELL
  end
end
```
