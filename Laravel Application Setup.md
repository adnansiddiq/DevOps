
# Laravel App Setup on Amazon AMI 2 with Letsencrypt Certificate

## Prepare Server

### Install Apache and Php 7.4
sudo yum update -y
sudo amazon-linux-extras install -y php7.4
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
sudo systemctl is-enabled httpd
php -v

### Set permissions of web root directory
sudo usermod -a -G apache ec2-user
exit
ssh ec2-user@ip
groups
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
find /var/www -type f -exec sudo chmod 0664 {} \;

### Install git and composer
sudo yum install git
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '55ce33d7678c5a611085589f1f3ddf8b3c52d662cd01d4ba75c0ee0459970c2200a51f492d557530c71c15d8dba01eae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer

### clone the application
cd /var/www/html/
git clone {repo_url}
cd {app}

### Install Dependency
sudo yum install openssl php-common php-mbstring php-json php-curl php-xml php-zip php-gd php-sodium
composer install

### copy env
cp .env.example .env
php artisan key:generate
nano .env
php artisan migrate:status

### Virtual Host Setup
cd /etc/httpd/conf.d/
sudo nano vhost.conf

```
<VirtualHost *:80>
    DocumentRoot "/var/www/html/{app}/public"
    ServerName app.dev
    <Directory "/var/www/html/{app}/public">
            AllowOverride All
            Options +FollowSymLhtmlinks +Indexes
            Order allow,deny
            Allow from all
    </Directory>
</VirtualHost>
```
sudo service httpd restart

### Insatll SSL using Letsencrypt
sudo amazon-linux-extras enable php7.4
sudo yum clean metadata
sudo yum install httpd mod_ssl -y
sudo yum install php php-cli php-mysqlnd php-pdo php-common -y
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo yum-config-manager --enable epel
sudo yum install certbot python2-certbot-apache -y
sudo certbot --apache

### links
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-lamp-amazon-linux-2.html
https://awswithatiq.com/letsencrypt-with-amazon-linux-2-centos-7/
