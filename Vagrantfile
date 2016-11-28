# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  config.vm.box = "debian/jessie64"

  config.vm.define "server" do |server|
   server.vm.hostname = "server.drupal.local"
   server.vm.network "forwarded_port", guest: 80, host: 8080
   server.vm.synced_folder "./", "/vagrant"
   server.vm.synced_folder "./portal", "/var/www/portal"
   server.vm.synced_folder "./files/apache", "/etc/apache2/sites-available/"
   server.vm.synced_folder "./files/mysql", "/etc/mysql/"

   server.vm.provider :virtualbox do |v|
    v.name = "server.drupal.local"
    v.customize [
     "modifyvm", :id,
     "--name", "server.drupal.local",
     "--memory", 512,
     "--cpus", 1
    ]
   end

   server.vm.provision "shell", inline: <<-SHELL
    # APT UPDATE
	apt-get update --fix-missing -y
	apt-get autoclean

    # APACHE 2
	apt-get install -y apache2 apache2-utils apache2-mpm-prefork curl build-essential vim git subversion
	a2enmod rewrite headers
	sed -i "s/LockFile ${APACHE_LOCK_DIR}/accept.lock/Mutex file:${APACHE_LOCK_DIR} default/g" /etc/apache2/apache2.conf
	echo "ServerName localhost" >> /etc/apache2/httpd.conf
	/etc/init.d/apache2 restart

	# PHP
	apt-get install -y php5 php5-cli php5-curl php5-gd php5-mysql php5-recode php5-gmp php5-xmlrpc php5-xsl php5-intl php5-mcrypt php5-imagick php5-json libapache2-mod-php5 php5-xdebug
	sed -i "s/short_open_tag = On/short_open_tag = Off/g" /etc/php5/apache2/php.ini
	sed -i "s/;date.timezone =/date.timezone = America\/Sao_Paulo/g" /etc/php5/apache2/php.ini
	sed -i "s/memory_limit = 128M/memory_limit = 256M/g" /etc/php5/apache2/php.ini
	sed -i "s/_errors = Off/_errors = On/g" /etc/php5/apache2/php.ini
	sed -i "s/error_reporting = -1/error_reporting = 30711/g" /etc/php5/apache2/php.ini #E_ALL & ~E_NOTICE & ~E_STRICT

	/etc/init.d/apache2 restart

	# COMPOSER
	curl -sS https://getcomposer.org/installer | php
	mv composer.phar /usr/local/bin/composer

	echo "
		NameVirtualHost *:80
		Listen 80
	" >> /etc/apache2/ports.conf

	a2ensite portal.drupal.conf

	echo "127.0.1.1		portal.drupal.local" >> /etc/hosts
	/etc/init.d/apache2 restart


	MYSQL_USER_ADMIN='vagrant'
    MYSQL_PASSWORD='123456789'
    MYSQL_PORT='3306'
    MYSQL_USER='mysql'
    MYSQL_KEY_BUFFER='128M'
    MYSQL_MAX_CONNECTIONS='1000'

    echo "mysql-server-5.5 mysql-server/root_password password $MYSQL_PASSWORD" | debconf-set-selections
    echo "mysql-server-5.5 mysql-server/root_password_again password $MYSQL_PASSWORD" | debconf-set-selections

    apt-get install -y mysql-server mysql-client

    QUERY="CREATE USER '$MYSQL_USER_ADMIN'@'%' IDENTIFIED BY '$MYSQL_PASSWORD'; \
    GRANT ALL PRIVILEGES ON *.* TO '$MYSQL_USER_ADMIN'@'%' WITH GRANT OPTION;   \
    FLUSH PRIVILEGES;"

    mysql -uroot -p$MYSQL_PASSWORD -e "$QUERY"

    # touch /etc/my.cnf
    # touch /etc/mysql/my.cnf
    # echo $MYSQL_CNF >> /etc/my.cnf
    # echo $MYSQL_CNF >> /etc/mysql/my.cnf

    service mysql restart
    mysqladmin -uroot -p$MYSQL_PASSWORD status

	# CUSTOMIZE VIM
	# wget https://gist.githubusercontent.com/avandrevitor/86319d80065f35616ff914580ea59327/raw/53220a893521dd8b46955270f00adbf5d81b2bb5/.vimrc && cp ~/.vimrc /home/vagrant/


	SHELL
 end
end
