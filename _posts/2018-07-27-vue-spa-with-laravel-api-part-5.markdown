---
layout: post
title:  "Vue SPA with Laravel API Part 5"
date:   2018-07-27 18:00:00 +0100
categories: laravel vue
description: "Part 5: Setting up Continuous Deployment with TravisCI and Heroku."
---

The complete source code of this series is available [on GitHub][source].

- Part 1: [Setting up a Vue SPA with a Laravel API as the backend using JWT authentication.][part1]
- Part 2: [Creating everything database related: models, migrations, seeders and factories.][part2]
- Part 3: [Setting up API routes and creating API resource controllers.][part3]
- Part 4: [Testing the API with feature tests and sqlite.][part4]
- Part 5: Setting up Continuous Deployment with TravisCI and Heroku. (You are here)
- Part 6: [Customizing the frontend skeleton, design and Dashboard component.][part6]
- Part 7: [Creating the remaining Vue components: Board, List, Card, Modal.][part7]
- Part 8: [Setting up Jest and testing all Vue components with it.][part8]

In this part we want to setup TravisCI so that it runs all our feature tests on different php versions and if everything works it should automatically deploy everything to heroku.

First we need to install the cli tools of TravisCI and heroku:

```sh
gem install travis -v 1.8.8 --no-rdoc --no-ri
sudo snap install heroku --classic
```

Go to [Travis website][travis] and sign in with your GitHub account. Next click on the switch of your repository to activate travis.

Next we have to create the heroku app. So go to [herokus website][heroku] and sign up/login. Choose create new app, put in a name and a region and create it. Next open the app and configure Add-ons. Here search for JawsDB MySQL and choose Kitefin Shared (Free) and add it to your app.

Next go to the settings of your app and add a buildpack: heroku/php (the officially supported php buildpack). Next reveal the config vars and first copy the JAWSDB_URL to DATABASE_URL (so that their contents are the same). Next create APP_KEY and paste in your Laravel APP_KEY. Also create an APP_URL and paste in the url of your heroku app. You can find this url below in the settings. Now we still need: DB_CONNECTION = heroku, SESSION_DRIVER = cookie and JWT_SECRET set to the secret key we created in the beginning of this tutorial with: `php artisan jwt:secret`. Now we only have to create and change some files in our project:

First create a file called `Profile` for heroku:

```
web: vendor/bin/heroku-php-nginx -C nginx.conf public/
```

and another file called `nginx.conf` that we used in the Procfile:

```
location / {
    # try to serve file directly, fallback to rewrite
    try_files $uri @rewriteapp;
}

location @rewriteapp {
    # rewrite all to app.php
    rewrite ^(.*)$ /index.php$1 last;
}
```

Next create the Travis config file called `.travis.yml`:

```yaml
language: php

php:
- 7.1
- 7.2

before_script:
- phpenv config-rm xdebug.ini
- cp .env.example .env
- rm phpunit.xml
- cp phpunit.xml.travis phpunit.xml
- mysql -e 'create database boards_test;' -u root
- composer install --no-interaction
- php artisan key:generate
- npm install

script:
- vendor/bin/phpunit
- npm run production

services:
- mysql

deploy:
  provider: heroku
  api_key:
    secure: LF7+Viu6rjrt84eXNPJOqEwJW3i8omDRFZoAUwrxUDBC+7G3+AFXV+Rn+BXm+XnniWdpy9N0k0LNyuDaSajrBNtETYcgnadzR/u29g1+iZJuQyOScmtQ44OeEPBxXP8ZbOhAWFDGI0yM8cWOGayAfcxvtu9YJFremZ0K6ZXCQC1l5FBHShvG7w5m2G+WXkFBSk2VNaJndnfIH8XrgRBJTgTXFHZT64wXh7UEevoZWW/Y+rwmCDhOEzbrBxVXlvyZoGmhKMSyWOPe2oJ0dtRuH440ZJsVES01YBqaFNJpnPKX14JkSxz1oHBi7KnRzdnFO2GUIxaPG3ZCPlsZ8xIYG7mwuU5pzo2KwsB37k9L76Tw4ZIQbQOQnJ5/M3H7jsdFS0/2rMaGKw8ONm7ckRRsMSRGWcABc/r30ImjQAS+s9Y9IVej6TettOUNFGZ3USR26AIhQMgyTkn1t/mhCsnXroWfccLzlVnn1SYjjmWrEIzNHm0aNsSnzOP+e2JloratrZ6km/fSTJYIDooLBKLl/7VnsOP7Fjk7FmU/pxd0PsMb39M1304FyfTvCEJ//J1RYHOLnR827HKaIXfKWfHZVWL9Ukf1+n4SBIaQ6OMXUC2U1ZftzcMY4wXIZTfNzUhnsKtWSXIUoEgyxP6uD4/HgupbemO2yQi9gEfnuuN+9G8=
  app: ddmler-boards
  skip_cleanup: true
  on:
    repo: ddmler/boards
    php: 7.1
```

The api_key was added automatically with this command:

```sh
heroku login
travis encrypt $(heroku auth:token) --add deploy.api_key
```

Which will encrypt your heroku auth:token so that only Travis can decrypt and use it. You should never publish your auth token from heroku, only the encrypted version here.

We let Travis test PHP 7.1 and 7.2. First we remove XDebug, then we copy the files into the correct place so that Travis has the correct environment variables. Next we use MySQL to create our database and let composer and npm install our dependencies. We also set the Laravel APP_KEY. For our script we run our PHPUnit tests with MySQL and if that is a success we create our npm production assets. At the last step we deploy to heroku with our api_key. We only deploy on one php version (or else we would have 2 deploys for each commit).

Since we use MySQL we have to change the defaultStringLength because we use utf8mb4 as a character encoding which uses 4 bytes and we would use too much space for our unique email column in the users table. (Indexes can only be about 750 bytes, in our encoding it would be above 1000 with standard values)

We do that by opening `app/Providers/AppServiceProvider.php` and changing it to:

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Schema;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Schema::defaultStringLength(191); // Hack for older MySQL version in TravisCI
    }

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

Next let's create the database connection we told heroku to use (we named it heroku in it's environment vars). Add this to the connections array in `config/database.php`:

```php
'heroku' => [
            'driver'   => 'mysql',
            'host'     => @parse_url(getenv("DATABASE_URL"))["host"],
            'database' => @substr(parse_url(getenv("DATABASE_URL"))["path"], 1),
            'username' => @parse_url(getenv("DATABASE_URL"))["user"],
            'password' => @parse_url(getenv("DATABASE_URL"))["pass"],
            'charset' => 'utf8mb4',
            'collation' => 'utf8mb4_unicode_ci',
            'prefix' => '',
            'strict' => true,
            'engine' => null,
        ],
```

Now copy `phpunit.xml` to `phpunit.xml.travis` (so that we have both) and remove these 2 lines from `phpunit.xml.travis` (we added those in a post before):

```xml
<env name="DB_CONNECTION" value="sqlite"/>
<env name="DB_DATABASE" value=":memory:"/>
```

With this our local tests with phpunit will run with sqlite (set in phpunit.xml) and Travis will run them with MySQL. With this setup we have very fast local tests and Travis tests that everything works in MySQL which we use in production in heroku.

Lastly we need to change our `.env.example` file and set these DB variables:

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=boards_test
DB_USERNAME=root
DB_PASSWORD=
```

The reason for that is that Travis will copy the example file and use it as it's environment file. So these are the credentials for Travis.

Now commit everything and if your phpunit tests pass, Travis will run your npm production script und deploy everything to heroku.

Since we have migrations and seeders we want to run we have to use herokus run command (or use the web console in your app dashboard):

`heroku run "php artisan migrate && php artisan db:seed"`

And that's it you should now be able to visit your heroku app and login. In [part 6][part6] we will work a bit on the frontend markup and design.

[travis]: https://travis-ci.org/
[heroku]: https://heroku.com/
[source]: https://github.com/ddmler/boards
[part1]: https://ddmler.github.io/laravel/vue/2018/07/13/vue-spa-with-laravel-api-part-1.html
[part2]: https://ddmler.github.io/laravel/vue/2018/07/17/vue-spa-with-laravel-api-part-2.html
[part3]: https://ddmler.github.io/laravel/vue/2018/07/20/vue-spa-with-laravel-api-part-3.html
[part4]: https://ddmler.github.io/laravel/vue/2018/07/24/vue-spa-with-laravel-api-part-4.html
[part6]: https://ddmler.github.io/laravel/vue/2018/07/31/vue-spa-with-laravel-api-part-6.html
[part7]: https://ddmler.github.io/laravel/vue/2018/08/07/vue-spa-with-laravel-api-part-7.html
[part8]: https://ddmler.github.io/laravel/vue/2018/08/10/vue-spa-with-laravel-api-part-8.html
