---
layout: post
title:  "Vue SPA with Laravel API Part 2"
date:   2018-07-17 18:00:00 +0100
categories: laravel vue
description: "Part 2: Creating everything database related: models, migrations, seeders and factories."
---

The complete source code of this series is available [on GitHub][source].

- Part 1: [Setting up a Vue SPA with a Laravel API as the backend using JWT authentication.][part1]
- Part 2: Creating everything database related: models, migrations, seeders and factories. (You are here)
- Part 3: [Setting up API routes and creating API resource controllers.][part3]
- Part 4: [Testing the API with feature tests and sqlite.][part4]
- Part 5: [Setting up Continuous Deployment with TravisCI and Heroku.][part5]
- Part 6: [Customizing the frontend skeleton, design and Dashboard component.][part6]
- Part 7: [Creating the remaining Vue components: Board, List, Card, Modal.][part7]
- Part 8: [Setting up Jest and testing all Vue components with it.][part8]

In this part we will first create all models and their migrations needed for our small Trello clone. After that we will create seeders and factories to easily populate our database with test data.

We will create 3 models: Board, BoardList and Card. A User has many Boards, a Board contains many Lists and each List contains many Cards. So our Eloquent relationships are simple hasMany relationships.

We let artisan create the three models and a migration for each with these commands:

```sh
php artisan make:model Board --migration
php artisan make:model BoardList --migration
php artisan make:model Card --migration
```

Next let's set up the Eloquent relationships. Add this to `App/User.php`:

```php
public function boards()
{
    return $this->hasMany("App\Board");
}
```

The `App/Board.php` model will look like this:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Board extends Model
{
    protected $fillable = [
        'name'
    ];

    public function user()
    {
        return $this->belongsTo("App\User");
    }

    public function boardLists()
    {
        return $this->hasMany("App\BoardList");
    }
}
```

The `App/BoardList.php` model will look like this:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class BoardList extends Model
{
    protected $table = "board_lists";

    public function board()
    {
        return $this->belongsTo("App\Board");
    }

    public function cards()
    {
        return $this->hasMany("App\Card");
    }
}
```

And finally, the `App/Card.php` model will look like this:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Card extends Model
{
    public function boardList()
    {
        return $this->belongsTo("App\BoardList");
    }
}
```

And with that our models are already done. Now open up the created migrations in the `database/migrations` folder and lets start with the Boards table:

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateBoardsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('boards', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->integer('user_id')->unsigned();
            $table->timestamps();

            $table->foreign('user_id')->references('id')->on('users')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('boards');
    }
}
```

We need an id, a name of the board, a user_id field containing our User relation and the created_at and updated_at timestamps. We also define a foreign key which defines that if a user gets deleted, his boards will be deleted too. Next the same for the BoardLists table:

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateBoardListsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('board_lists', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->integer('board_id')->unsigned();
            $table->integer('order')->unsigned()->default(0);
            $table->timestamps();

            $table->foreign('board_id')->references('id')->on('boards')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('board_lists');
    }
}
```

This time with a board_id instead of a user_id since we only have a relationship with the board model. The order field will indicate how the user sorted his Lists on the Board (they will be sortable via drag&drop). We also define a foreign key which cascades the deletes from boards to all lists of a board again. Finally we create a Cards table migration (The User migration already comes out of the box in Laravel):

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateCardsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('cards', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->integer('board_list_id')->unsigned();
            $table->integer('order')->unsigned()->default(0);
            $table->text('description');
            $table->timestamps();

            $table->foreign('board_list_id')->references('id')->on('board_lists')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('cards');
    }
}
```

A lot is pretty much the same: id, name, board_list_id and order. We also have another foreign key, which means if a user deletes his account a cascade of deletes will be triggered that delete all his boards, his lists and his cards as well. Notice that the board_list_id has to be written in snake case for Eloquent to find it since the class name is in camel case and Eloquent automatically converts this. Finally we add a simple text field called description.

And with that we can run: `php artisan migrate` to create all tables in the database. If you got an error use `php artisan migrate:fresh` since up to the error everything that worked doesn't get rolled back and the fresh command deletes everything first.

So now we have models and database tables but they're still empty. It would be nice if we had data in them for testing purposes. We achieve this by creating Seeder and Factory classes next.

We want to add data to each model we have. So we will need 4 seeder classes in total, which we let artisan create for us:

```sh
php artisan make:seeder UsersTableSeeder
php artisan make:seeder BoardsTableSeeder
php artisan make:seeder BoardListsTableSeeder
php artisan make:seeder CardsTableSeeder
```

When we use the seed command of artisan, it will call the main DatabaseSeeder class located at `database/seeds/DatabaseSeeder.php`, so we will have to call all other seeders from it like this:

```php
<?php

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $this->call(UsersTableSeeder::class);
        $this->call(BoardsTableSeeder::class);
        $this->call(BoardListsTableSeeder::class);
        $this->call(CardsTableSeeder::class);
    }
}
```

Next lets create the UserTableSeeder like this so we get one account with which we can login:

```php
<?php

use Illuminate\Database\Seeder;
use App\User;

class UsersTableSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
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

For the other 3 seeders we will be using the factories we will create after this:

```php
<?php

use Illuminate\Database\Seeder;

class BoardsTableSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        factory(App\Board::class, 2)->create();
    }
}
```

This factory call creates (and saves them in the database) 2 Board objects. If we used make instead of create the factory would not save them to the database. Since we didn't supply any argument to the create method, it will use the default values but we could also supply specific values by using an array as it's argument.

```php
<?php

use Illuminate\Database\Seeder;

class BoardListsTableSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        factory(App\BoardList::class, 8)->create();
    }
}
```

It's the same for BoardList with 8 created instances.

```php
<?php

use Illuminate\Database\Seeder;

class CardsTableSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        factory(App\Card::class, 32)->create();
    }
}
```

And also the same for Card with 32 created instances. These numbers are just randomly chosen by me so we have something more than 1 of each.

Next let's create the factories. The UserFactory is already provided by Laravel so we only have to create the three others with the help of artisan:

```sh
php artisan make:factory BoardFactory --model=Board
php artisan make:factory BoardListFactory --model=BoardList
php artisan make:factory CardFactory --model=Card
```

In the factory we just return an array with the values of each field we want to fill. These values are the defaults if we don't supply any values ourselves when using the factory.

```php
<?php

use Faker\Generator as Faker;

$factory->define(App\Board::class, function (Faker $faker) {
    return [
        'name' => $faker->name,
        'user_id' => App\User::all()->random()->id,
    ];
});
```

For the Board factory we just provide a random name from Faker and a random user_id (which will always be our only created user in this case).

```php
<?php

use Faker\Generator as Faker;

$factory->define(App\BoardList::class, function (Faker $faker) {
    return [
        'name' => $faker->name,
        'board_id' => App\Board::all()->random()->id,
        'order' => random_int(0, 10000),
    ];
});
```

The BoardList factory gets a random name and a random Board it belongs to, as well as a random number for its position.

```php
<?php

use Faker\Generator as Faker;

$factory->define(App\Card::class, function (Faker $faker) {
    return [
        'name' => $faker->name,
        'board_list_id' => App\BoardList::all()->random()->id,
        'order' => random_int(0, 10000),
        'description' => $faker->paragraph,
    ];
});
```

And the same for the card factory with the addition of a description text.

And with that we can run `php artisan db:seed` to seed our database with some testing data and use the factories in our tests for easy creation of objects.

In our next step we want to create the API routes and controllers so that we can consume the data via an API. For that see [Part 3][part3].

[source]: https://github.com/ddmler/boards
[part1]: https://ddmler.github.io/laravel/vue/2018/07/13/vue-spa-with-laravel-api-part-1.html
[part3]: https://ddmler.github.io/laravel/vue/2018/07/20/vue-spa-with-laravel-api-part-3.html
[part4]: https://ddmler.github.io/laravel/vue/2018/07/24/vue-spa-with-laravel-api-part-4.html
[part5]: https://ddmler.github.io/laravel/vue/2018/07/27/vue-spa-with-laravel-api-part-5.html
[part6]: https://ddmler.github.io/laravel/vue/2018/07/31/vue-spa-with-laravel-api-part-6.html
[part7]: https://ddmler.github.io/laravel/vue/2018/08/07/vue-spa-with-laravel-api-part-7.html
[part8]: https://ddmler.github.io/laravel/vue/2018/08/10/vue-spa-with-laravel-api-part-8.html
