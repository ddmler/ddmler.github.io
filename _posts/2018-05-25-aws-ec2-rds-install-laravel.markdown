---
layout: post
title:  "AWS EC2 & RDS install Laravel"
date:   2018-05-25 18:00:00 +0100
categories: aws
description: Launching a webserver with AWS EC2 and a database with AWS RDS and installing a Laravel project on them.
---

Part 1: [AWS VPC creating a public and private subnet][part1]

Part 2: AWS EC2 & RDS install Laravel (you are here)

Now we want to launch some stuff in our VPC and try to get a Laravel application running. The first step is to create a database in the private subnet and a webserver in the public subnet.

We will start with the database, for that go to the AWS RDS page and create a new instance. Step 1 and step 2 can be filled in however you like. In step 3 choose our created VPC and for VPC security groups choose existing VPC security groups and select our DBServer security group. Launch the instance and your database is all set.

The next step is to launch a Webserver. For this we will use EC2. Launch a new instance with an AMI that you like, I chose Ubuntu server. In step 3 choose our VPC for network and for subnet select the public subnet. You can enable auto-assign public IP or associate an elastic ip after launching. In step 6 select the WebServer security group and launch the instance. Download the ssh key and use the chmod command on it:

`chmod 400 <path and keyname>.pem`

The ssh command to connect will look like this:
`ssh -i <path and keyname>.pem ubuntu@<instance IP or endpoint>`

Once connected we can start by installing a webserver and php:

```sh
sudo apt-get install apache2 zip
sudo apt-get install php libapache2-mod-php php-mbstring php-xml php-mysql
sudo chmod 777 /var/www/html/
cd /var/www/html/
rm index.html
touch index.php
nano index.php
```

Set the content of the new file to something like this:

```php
<?php

echo "Hello World!";
```
Now access the page and check that it works. The next step is to get composer:

```sh
curl -O https://getcomposer.org/composer.phar
mv composer.phar composer
chmod +x composer
sudo mv composer /usr/local/bin
```

Again check that it works with: `composer`

Now lets remove the file again and clone a git repository to `/var/www/html`:

```sh
rm index.php && cd ..

git clone <your laravel project> html
```

Since laravels entrypoint is the public folder we need to change our DocumentRoot to that:

```sh
sudo nano /etc/apache2/sites-available/000-default.conf
```

Change: `DocumentRoot /var/www/html/` to `DocumentRoot /var/www/html/public`

To enable mod_rewrite we have to change another setting of apache:

```sh
sudo nano /etc/apache2/apache2.conf
```

Search for: `<Directory /var/www/>`

in that change: `AllowOverride None` to `AllowOverride All`

And with this the webserver setup is done. The next step is to restart the webserver for the changes to take effect, install composer dependencies, set the Laravel storage permissions and edit the `.env` file:

```sh
sudo service apache2 restart

cd html
composer install --no-dev

sudo chmod -R 777 storage
mv .env.example .env
nano .env
```

Set DB_HOST to the endpoint of your RDS instance, the other settings were set when creating the RDS instance. You can find all of that information by going to the RDS instances page and clicking on the name of your instance. Now scroll down to Details and look for endpoint, db name and username. Now we just need to generate an application key with artisan and if there are some run migrations and database seeders.

```sh
php artisan key:generate
php artisan migrate
php artisan db:seed
```

The website should now be fully functioning.


[part1]: https://ddmler.github.io/aws/2018/05/18/aws-vpc-creating-a-public-and-private-subnet.html
