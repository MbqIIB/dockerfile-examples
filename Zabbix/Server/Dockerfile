########## Dockerfile for Zabbix-server version 3.4.6 #########
#
# This Dockerfile builds a basic installation of Zabbix-server.
#
# Zabbix server is the central process of Zabbix software
#
# To build this image, from the directory containing this Dockerfile
# (assuming that the file is named Dockerfile):
# docker build -t <image_name> .
#
# To Start Zabbix server run the below command:
# docker run --name <container_name> -p <portnumber>:80 -d <image>  
# 
# Test in the browser by using the following url:
# http://<hostname>:<port_number>/zabbix
#
##################################################################################

# Base Image
FROM s390x/ubuntu:16.04

# The author
MAINTAINER  LoZ Open Source Ecosystem (https://www.ibm.com/developerworks/community/groups/community/lozopensource)

ENV DEBIAN_FRONTEND=noninteractive
ENV PATH=$PATH:/usr/local/sbin/
ENV SOURCE_DIR=/
WORKDIR $SOURCE_DIR

# Install dependencies
RUN	apt-get update && apt-get install -y mysql-server \
	&& apt-get -y install \
		apache2 \
		ceph \
		curl \
		gcc \
		git \
		libapache2-mod-php \
		libcurl3 \
		libcurl4-openssl-dev \
		libmysqlclient-dev \
		libmysqld-dev \
		libsnmp-dev \
		libxml2-dev \
		make \
		php \
		php7.0-gd \
		php7.0-ldap \
		php7.0-xml \
		php-bcmath \
		php-mbstring \
		php-mysql \
		snmp \
		snmptrapd \
		vim \
		wget \
		libevent-dev \
		libpcre3-dev \
	
# Enable PHP support by modifying Apache configuration file
	&&	cd /etc/apache2 \
	&&	chmod 766 apache2.conf \
	&&	echo "ServerName localhost" >> apache2.conf \
	&&	echo "AddType application/x-httpd-php .php" >> apache2.conf \
	&&	echo "<Directory />" >> apache2.conf \
	&&	echo " DirectoryIndex index.php " >> apache2.conf \
	&&	echo "</Directory>" >> apache2.conf \
	
# Download and install Zabbix server
	&&	cd $SOURCE_DIR \
	&&	wget http://sourceforge.net/projects/zabbix/files/ZABBIX%20Latest%20Stable/3.4.6/zabbix-3.4.6.tar.gz/download -O zabbix-3.4.6.tar.gz \
	&&	tar -xvf zabbix-3.4.6.tar.gz \
	&&	cd zabbix-3.4.6/ \
	&&	./configure --enable-server --with-mysql --enable-ipv6 --with-net-snmp --with-libcurl --with-libxml2 \
	&&	make \
	&&	make install \

# Create a 'zabbix' user required to start Zabbix server daemon	
	&&	/usr/sbin/groupadd zabbix \
	&&	/usr/sbin/useradd -g zabbix zabbix \

# Installing Zabbix web interface	
	&&	cd /zabbix-3.4.6/frontends/php/ \
	&&	mkdir -p /var/www/html/zabbix \
	&&	cp -rf * /var/www/html/zabbix/ \
	&&	cd /var/www/html/zabbix \
	&&	chown -R zabbix:zabbix conf \
	
# Prerequisites to start Zabbix server
	&&	service apache2 start \
	&&	service mysql start \

# Create database and grant privileges to zabbix user	
	&&	/usr/bin/mysql -e "create database zabbix" \
	&& 	/usr/bin/mysql -e "create user 'zabbix'@'localhost'" \
	&&	/usr/bin/mysql -e "grant all privileges on zabbix.* to 'zabbix'@'localhost'" \
	
	&&	cd /zabbix-3.4.6/database/mysql \
	&&	mysql -uzabbix zabbix < schema.sql \
	&&	mysql -uzabbix zabbix < images.sql \
	&&	mysql -uzabbix zabbix < data.sql \
	
# Change php.ini file 
	&&	sed -i 's/max_execution_time = 30/max_execution_time = 300/g' /etc/php/7.0/apache2/php.ini \
	&&	sed -i 's/max_input_time = 60/max_input_time = 300/g' /etc/php/7.0/apache2/php.ini \
	&&	sed -i 's/post_max_size = 8M/post_max_size = 16M/g' /etc/php/7.0/apache2/php.ini \
	&&	sed -i "s/;date.timezone =/date.timezone = 'Asia\/Kolkata'/g" /etc/php/7.0/apache2/php.ini \
	
# Start Zabbix server
	&&	zabbix_server \
	
# Clean up cache data and remove dependencies which are not required
	&&	apt-get -y remove \
			curl \
			gcc \
			git \
			make \
			vim \
			wget \
	&& 	apt autoremove -y && apt-get clean \
	&&	rm -rf /zabbix-3.4.6.tar.gz && rm -rf /var/lib/apt/lists/* 
	
EXPOSE 80

CMD ["apachectl","-D","FOREGROUND"]

# End of Dockerfile
