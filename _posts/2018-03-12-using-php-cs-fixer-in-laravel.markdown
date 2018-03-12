---
layout: post
title:  "Using PHP-CS-Fixer in Laravel"
date:   2018-03-12 12:00:00 +0100
categories: laravel linter
description: How to set up and use PHP-CS-Fixer in a project as shown with Laravel for example.
---

With multiple people working on a project there will almost certainly come a time where even though you all agreed on a coding style convention the violations against this convention pile up (at least if you don't have your CI set up to look for those things, which you probably should then). Maybe it's just a quick bug fix or someone new to the codebase or you are stuck with a legacy application that doesn't even have a coding style convention.

Luckily PHP-CS-Fixer can help you with that. With a very simple setup it will automatically fix your code to adhere to standards like PSR2 or something you define on your own.
To get started first install it with composer. You can either install it globally or per project in the dev requirements:

`composer require --dev friendsofphp/php-cs-fixer`

Now you will need to create a config file named `.php_cs.dist`. The following example for the config file will configure PHP-CS-Fixer to apply the PSR2 standard to a Laravel project. If you don't use a laravel project just change the exclude directories.

```php
<?php
$finder = PhpCsFixer\Finder::create()
    ->exclude('bootstrap/')
    ->exclude('public/')
    ->exclude('resources/')
    ->exclude('storage/')
    ->in(__DIR__)
;
return PhpCsFixer\Config::create()
    ->setRules([
        '@PSR2' => true,
        'array_syntax' => ['syntax' => 'short'],
    ])
    ->setFinder($finder)
;
```

And that's it! With these two simple steps you can fix your entire codebase at once with the command:

`./vendor/bin/php-cs-fixer fix`
