---
layout: post
title:  "Getting started with Laradock"
date:   2018-03-08 16:30:20 +0100
categories: laravel docker
description: How to use Laradock to get a complete development environment running in just 5 steps.
---

Setting up a complete development environment for Laravel using Laradock just takes a few minutes. You will need: git, docker, docker-compose and a laravel project (can be a brand new one).

First clone the Laradock project anywhere you like.

`git clone https://github.com/Laradock/laradock.git`

Copy the env example file to .env

`cp env-example .env`

Change the Application setting to your project root directory.

`Application=~/project`

Now you can just choose the services you want to use and start them with docker-compose. Laradock offers a ton of different services already configured for you to use. [Check the laradock site here to get more info.][laradock]

`docker-compose up -d mysql nginx`

This may take a while since it has to download all the docker images you chose. To use the mysql docker service in your Laravel project you only have to change the `DB_HOST` config in your `.env` file like this:

`DB_HOST=mysql`

And that's it. You should now be able to see your laravel project in your browser on http://localhost/

If you try to find out which containers are currently running with

`docker-compose ps`

You should see the ones you have chosen before and a php-fpm as well as a workspace container. You can access the workspace container to use things like composer or php artisan commands by starting a shell on it

`docker-compose exec -u laradock workspace bash`

This workspace should provide you with all the command line tools you need.




Links to find out more:

[Laradock website][laradock]

[Laravel website][laravel]


[laradock]: http://laradock.io/
[laravel]: https://laravel.com/
