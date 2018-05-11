---
layout: post
title:  "Database seeders"
date:   2018-05-11 18:00:00 +0100
categories: database
description: Using database seeders with factories to generate test data in Laravel.
---

Database seeding is the automated creation of (test) data for a database. This step usually follows the creation of the database schema for example with something like [database migrations][migrations]. In Laravel you can easily create your first seeder class with: `php artisan make:seeder UsersTableSeeder`. The generated file can then be modified like follows to create an admin account.

```php
<?php
use Illuminate\Database\Seeder;
use App\User;

class UsersTableSeeder extends Seeder
{
    public function run()
    {
        User::create([
          'name' => 'admin',
          'email' => 'admin@example.com',
          'password' => bcrypt('admin')
        ]);
    }
}
```

Now we just need to make sure that the main DatabaseSeeder class in the same folder as our seeder will run our UsersTableSeeder. You have to add all your seeders here otherwise they won't be executed.

```php
<?php
use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run()
    {
        $this->call(UsersTableSeeder::class);
        $this->call(CardsTableSeeder::class);
        // all other Seeder classes
    }
}
```

Now just run: `php artisan db:seed` to run all your seeder classes or use `php artisan db:seed --class=UsersTableSeeder` to run a specific seeder individually and your admin account will be created.

In case you need to create a few more entries in your database you can use factories. These are also very useful for testing since you can easily generate a given number of objects, in this case 32. The create method will save the objects to the database. If you don't want to save them, for example in your test cases, use `make()` instead.

```php
<?php
use Illuminate\Database\Seeder;

class CardsTableSeeder extends Seeder
{
    public function run()
    {
        factory(App\Card::class, 32)->create();
    }
}
```

To use the factory we need to create it first. For this we use the define method with the first argument set to the model which we want to generate and the second argument as a closure that returns an array with the values we want to use for each created object. In this example we use Faker to generate a random name, but you could also use static values if you don't need randomness. We also set a random id from another model for our 1:n model relationship. For this the order of the seeding that we set in DatabaseSeeder is important.

```php
<?php
use Faker\Generator as Faker;

$factory->define(App\Card::class, function (Faker $faker) {
    return [
        'name' => $faker->name,
        'board_list_id' => App\BoardList::all()->random()->id,
    ];
});
```

[migrations]: https://ddmler.github.io/database/2018/04/20/database-migrations.html
