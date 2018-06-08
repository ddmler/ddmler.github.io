---
layout: post
title:  "PHP Security Advisories"
date:   2018-06-08 18:00:00 +0100
categories: php security
description: The PHP Security Advisories Database project and multiple ways to consume it to check your project for known security vulnerabilities.
---

The PHP Security Advisories Database project tries to provide easy access to known security vulnerabilities for PHP projects and libraries. It does not try to include every vulnerability there is so you still need to check for vulnerabilities with other methods. However due to the database being really easy to use it is a good idea to include it in your development process somehow, like for example in a CI process. The database itself can be found [here][database]. If you want to add a security vulnerability you know of create a new pull request there. There are multiple possible ways for automated checking of your project.

1. [SensioLabs web checker][web]: Just upload your composer.lock file.

2. You can also use the [SensioLabs Security Checker CLI tool][cli] or an API:

```sh
php security-checker security:check composer.lock
```

3. As a [Composer requirement][composer] creating requirement conflicts for vulnerable packages:

```sh
composer require --dev roave/security-advisories:dev-master
```

Whenever you try to require or update a new package that has a known security vulnerability in the database, this package will force a conflict in composer so you can't install it. Deployment will not be secured because a simple composer install with a valid composer.lock file will not generate any conflicts.

[database]: https://github.com/FriendsOfPHP/security-advisories
[web]: https://security.sensiolabs.org/check
[cli]: https://github.com/sensiolabs/security-checker
[composer]: https://github.com/Roave/SecurityAdvisories
