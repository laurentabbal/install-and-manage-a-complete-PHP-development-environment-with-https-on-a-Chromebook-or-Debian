# Install and manage a complete PHP development environment (with https) on a Chromebook (or Debian) with just a few copy-paste

This guide explains how to install and manage a complete PHP development environment (PHP, Apache, MySQL, PhpMyAdmin, Chromium, Codium...) with HTTPS suppport for Virtual Hosts on a Chromebook (or Debian/Ubuntu) with just a few copy-paste.

## A. APPLICATIONS
### 1. INITIALISATION and TOOLS
⏱️ Copy the content of the cell below, paste it into the terminal (shift+ctrl+v), press the “enter” key and wait.
```shell
DEVSERVER=~/DEVSERVER &&
LOCALHOST=~/DEVSERVER/localhost &&
VIRTUALHOSTS=~/DEVSERVER/virtualhosts &&
mkdir $DEVSERVER $LOCALHOST $VIRTUALHOSTS &&
touch $LOCALHOST/phpinfo.php &&
sudo apt update &&
sudo apt upgrade -y &&
sudo apt install curl -y &&
sudo apt install wget -y &&
sudo apt install nano -y &&
sudo apt install ufw -y &&
sudo apt install wget libnss3-tools -y &&
curl -JLO "https://dl.filippo.io/mkcert/latest?for=linux/amd64" &&
chmod +x mkcert-v*-linux-amd64 &&
sudo mv mkcert-v*-linux-amd64 /usr/local/bin/mkcert &&
sudo bash -c "cat <<EOT >> $LOCALHOST/phpinfo.php
<?php phpinfo(); ?>
EOT"
```

### 2. APACHE
⏱️ Copy the content of the cell below, paste it into the terminal (shift+ctrl+v), press the “enter” key and wait.
```shell
LOCALHOST=~/DEVSERVER/localhost &&
sudo apt install apache2 -y &&
sudo ufw allow OpenSSH &&
sudo ufw allow 'WWW Full' &&
sudo apt update &&
sudo apt upgrade -y &&
sudo a2dissite 000-default &&
sudo systemctl reload apache2 &&
sudo touch /etc/apache2/sites-available/localhost.conf &&
sudo chmod 755 /etc/apache2/sites-available/localhost.conf &&
sudo bash -c "cat <<EOT >> /etc/apache2/sites-available/localhost.conf
<VirtualHost *:80>
    ServerName localhost
    DocumentRoot $LOCALHOST
    <Directory $LOCALHOST>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog localhost_error.log 
    CustomLog localhost_access.log combined 
</VirtualHost>
EOT" &&
sudo a2ensite localhost.conf &&
sudo systemctl reload apache2
```

### 3. PHP
⏱️ Copy the content of the cell below, paste it into the terminal (shift+ctrl+v), press the “enter” key and wait.
```shell
PHP_VERSION=8.1
sudo apt -y install lsb-release apt-transport-https ca-certificates && 
sudo wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg &&
sudo echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list &&
sudo apt update &&
sudo apt install php$PHP_VERSION -y &&
sudo apt install php$PHP_VERSION-{bcmath,bz2,cgi,curl,fpm,xml,mysql,zip,imap,intl,imagick,ldap,gd,mbstring,mysql,pgsql,soap,xmlrpc} -y &&
sudo a2enmod proxy_fcgi setenvif &&
sudo a2enconf php$PHP_VERSION-fpm &&
sudo systemctl reload apache2
```

### 4. MYSQL
⏱️ Copy the content of the cell below, paste it into the terminal (shift+ctrl+v), press the “enter” key and wait.
```shell
sudo wget https://dev.mysql.com/get/mysql-apt-config_0.8.22-1_all.deb &&
sudo apt install ./mysql-apt-config_0.8.22-1_all.deb -y &&
sudo apt update &&
sudo apt upgrade -y &&
sudo apt install mysql-server -y &&
sudo rm mysql-apt-config_*.deb
```

* 🚨 First window : select Ok (not <Ok>)
* 🚨 Second window : select Ok (not <Ok>)
* 🚨 Third window : no password, select <Ok> directly
* 🚨 Fourth windows : select "Use legacy Authentication Method"

### 5. PHPMYADMIN
⏱️ Copy the content of the cell below, paste it into the terminal (shift+ctrl+v), press the “enter” key and wait.
```shell
PMA_VERSION=5.2.0
LOCALHOST=~/DEVSERVER/localhost &&
sudo wget https://files.phpmyadmin.net/phpMyAdmin/$PMA_VERSION/phpMyAdmin-$PMA_VERSION-all-languages.tar.gz &&
sudo tar xvf phpMyAdmin-$PMA_VERSION-all-languages.tar.gz &&
sudo mv phpMyAdmin-$PMA_VERSION-all-languages/ $LOCALHOST/phpmyadmin &&
sudo rm phpMyAdmin-*-languages.tar.gz
sudo mkdir $LOCALHOST/phpmyadmin/tmp &&
sudo touch $LOCALHOST/phpmyadmin/config.inc.php &&
CONFIG_CONTENT='<?php
\$i = 0;
\$i++;
\$cfg["Servers"][\$i]["auth_type"] = "config";
\$cfg["Servers"][\$i]["user"] = "pma";
\$cfg["Servers"][\$i]["password"] = "pmapwd";
\$cfg["Servers"][\$i]["host"] = "127.0.0.1";
\$cfg["Servers"][\$i]["hide_db"] = "information_schema|performance_schema|mysql|phpmyadmin|sys";
\$cfg["Servers"][\$i]["compress"] = false;
\$cfg["Servers"][\$i]["pmadb"] = "phpmyadmin";
\$cfg["Servers"][\$i]["bookmarktable"] = "pma__bookmark";
\$cfg["Servers"][\$i]["relation"] = "pma__relation";
\$cfg["Servers"][\$i]["table_info"] = "pma__table_info";
\$cfg["Servers"][\$i]["table_coords"] = "pma__table_coords";
\$cfg["Servers"][\$i]["pdf_pages"] = "pma__pdf_pages";
\$cfg["Servers"][\$i]["column_info"] = "pma__column_info";
\$cfg["Servers"][\$i]["history"] = "pma__history";
\$cfg["Servers"][\$i]["table_uiprefs"] = "pma__table_uiprefs";
\$cfg["Servers"][\$i]["tracking"] = "pma__tracking";
\$cfg["Servers"][\$i]["userconfig"] = "pma__userconfig";
\$cfg["Servers"][\$i]["recent"] = "pma__recent";
\$cfg["Servers"][\$i]["favorite"] = "pma__favorite";
\$cfg["Servers"][\$i]["users"] = "pma__users";
\$cfg["Servers"][\$i]["usergroups"] = "pma__usergroups";
\$cfg["Servers"][\$i]["navigationhiding"] = "pma__navigationhiding";
\$cfg["Servers"][\$i]["savedsearches"] = "pma__savedsearches";
\$cfg["Servers"][\$i]["central_columns"] = "pma__central_columns";
\$cfg["Servers"][\$i]["designer_settings"] = "pma__designer_settings";
\$cfg["Servers"][\$i]["export_templates"] = "pma__export_templates";'
sudo bash -c "cat <<EOT >> $LOCALHOST/phpmyadmin/config.inc.php
$CONFIG_CONTENT
EOT" &&
sudo mysql < $LOCALHOST/phpmyadmin/sql/create_tables.sql &&
sudo mysql -u root -Bse "CREATE USER 'pma'@'localhost' IDENTIFIED BY 'pmapwd'; GRANT ALL PRIVILEGES ON *.* TO 'pma'@'localhost'; FLUSH PRIVILEGES;" &&
sudo service mysql restart
```

### 6. CHROMIUM
⏱️ Copy the content of the cell below, paste it into the terminal (shift+ctrl+v), press the “enter” key and wait.
```shell
sudo apt update &&
sudo apt upgrade -y &&
sudo apt install chromium -y &&
chromium
```
🚨 Chromium must be started once before adding virtual hosts.

### 7. CODIUM
⏱️ Copy the content of the cell below, paste it into the terminal (shift+ctrl+v), press the “enter” key and wait
```shell
wget -qO - https://gitlab.com/paulcarroty/vscodium-deb-rpm-repo/raw/master/pub.gpg | gpg --dearmor | sudo dd of=/usr/share/keyrings/vscodium-archive-keyring.gpg &&
echo 'deb [ signed-by=/usr/share/keyrings/vscodium-archive-keyring.gpg ] https://download.vscodium.com/debs vscodium main' | sudo tee /etc/apt/sources.list.d/vscodium.list &&
sudo apt update &&
sudo apt install codium -y
```

## B. VIRTUAL HOSTS

### 1. ADD A VIRTUAL HOST
Copy the content of the cell below, paste it into the terminal (shift+ctrl+v), change the name of the virtual host  and press the “enter” key.
```shell
VH_NAME="name_of_the_virtual_host_you_want_to_create"
```
Copy the content of the cell below, paste it into the terminal (shift+ctrl+v), press the “enter” key.
```shell
DEVSERVER=~/DEVSERVER &&
VIRTUALHOSTS=~/DEVSERVER/virtualhosts &&
mkdir $VIRTUALHOSTS/$VH_NAME &&
sudo touch /etc/apache2/sites-available/$VH_NAME.conf &&
sudo chmod 755 /etc/apache2/sites-available/$VH_NAME.conf &&
sudo bash -c "cat <<EOT >> /etc/apache2/sites-available/$VH_NAME.conf
<VirtualHost *:443>
    ServerName $VH_NAME
    DocumentRoot $VIRTUALHOSTS/$VH_NAME
    SSLEngine on
    SSLCertificateKeyFile /etc/ssl/private/$VH_NAME.key
    SSLCertificateFile /etc/ssl/certs/$VH_NAME.crt
    <Directory $VIRTUALHOSTS/$VH_NAME>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog ${VH_NAME}_error.log 
    CustomLog ${VH_NAME}_access.log combined  
</VirtualHost>
EOT" &&
sudo a2ensite $VH_NAME.conf &&
sudo bash -c "cat <<EOT >> /etc/hosts
127.0.0.1 $VH_NAME
EOT" &&
sudo a2enmod ssl &&
mkcert -install &&
mkcert -key-file $VH_NAME.key -cert-file $VH_NAME.crt $VH_NAME &&
sudo mv $VH_NAME.key /etc/ssl/private/$VH_NAME.key &&
sudo mv $VH_NAME.crt /etc/ssl/certs/$VH_NAME.crt &&
sudo systemctl reload apache2
```

### 2. DELETE A VIRTUAL HOST
Copy the content of the cell below, paste it into the terminal (shift+ctrl+v), change the name of the virtual host  and press the “enter” key.
```shell
VH_NAME="name_of_the_virtual_host_you_want_to_delete"
```
Copy the content of the cell below, paste it into the terminal (shift+ctrl+v), press the “enter” key.
```shell
sudo a2dissite $VH_NAME &&
sudo rm /etc/apache2/sites-available/$VH_NAME.conf
sudo sed -i "/$VH_NAME/d" /etc/hosts &&
sudo systemctl reload apache2
```

## C. USEFUL COMMANDS
```shell
codium ~/DEVSERVER
```
```shell
codium ~/DEVSERVER/localhost
```
```shell
codium ~/DEVSERVER/virtualhosts
```
