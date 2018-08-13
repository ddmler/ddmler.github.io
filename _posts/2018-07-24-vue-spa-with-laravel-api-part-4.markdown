---
layout: post
title:  "Vue SPA with Laravel API Part 4"
date:   2018-07-24 18:00:00 +0100
categories: laravel vue
description: "Part 4: Testing the API with feature tests and sqlite."
---

The complete source code of this series is available [on GitHub][source].

- Part 1: [Setting up a Vue SPA with a Laravel API as the backend using JWT authentication.][part1]
- Part 2: [Creating everything database related: models, migrations, seeders and factories.][part2]
- Part 3: [Setting up API routes and creating API resource controllers.][part3]
- Part 4: Testing the API with feature tests and sqlite. (You are here)
- Part 5: [Setting up Continuous Deployment with TravisCI and Heroku.][part5]
- Part 6: [Customizing the frontend skeleton, design and Dashboard component.][part6]
- Part 7: [Creating the remaining Vue components: Board, List, Card, Modal.][part7]
- Part 8: [Setting up Jest and testing all Vue components with it.][part8]

In this part we want to setup a sqlite database for our testsuite and create feature tests for our API. The feature tests here will have a simple design: make a json call to each API endpoint, assert the correct HTTP response code and assert that the intended functionality is working correctly if the provided data is correct.

We will also add some tests that test validation errors and authorization.

To set this up we first have to add some more environment variables to `phpunit.xml` below the other env entries:

```xml
<env name="DB_CONNECTION" value="sqlite"/>
<env name="DB_DATABASE" value=":memory:"/>
```

Next lets create a new `tests/DatabaseTestCase.php` file that will look like this:

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Support\Facades\Artisan;

abstract class DatabaseTestCase extends TestCase
{
    use RefreshDatabase;

    public function setUp()
    {
        parent::setUp();
        // Deactivated - right now we create all objects ourselves
        // but you could use this to generate dummy data
        // Artisan::call('db:seed');
    }
}
```

The reason for that is that we add the RefreshDatabase which will use `php artisan migrate:fresh` before each test and maybe want to let artisan call our seeders. Since we may don't want every TestCase to do that we create an extra class here which we will use if we need that functionality. Next we let artisan create our feature test classes:

```sh
php artisan make:test BoardTest
php artisan make:test BoardListTest
php artisan make:test CardTest
```

Let's, as always, begin with `tests/Feature/BoardTest.php`:

```php
<?php

namespace Tests\Feature;

use Tests\DatabaseTestCase;
use Illuminate\Foundation\Testing\WithFaker;
use App\User;
use App\Board;
use App\BoardList;

class BoardTest extends DatabaseTestCase
{
    protected $user;
    protected $board;

    public function setUp()
    {
        parent::setUp();
        $this->user = factory(User::class)->create();
        $this->board = factory(Board::class)->create(['user_id' => $this->user->id]);
    }

    public function testLoginRequired()
    {
        $this->json('GET', 'api/boards')
            ->assertStatus(401);
    }

    public function testUserCanSeeBoards()
    {
        $this->actingAs($this->user)
            ->json('GET', 'api/boards')
            ->assertStatus(200)
            ->assertJson([
                ['name' => $this->board->name]
            ])
            ->assertJsonCount(1);
    }

    public function testUserCanViewHisBoard()
    {
        $boardLists = factory(BoardList::class, 2)->create(['board_id' => $this->board->id]);

        $this->actingAs($this->user)
            ->json('GET', 'api/boards/' . $this->board->id)
            ->assertStatus(200)
            ->assertJsonCount(2, 'board_lists');
    }

    public function testUserCanCreateBoards()
    {
        $payload = ['name' => 'My Board'];

        $this->actingAs($this->user)
            ->json('POST', 'api/boards', $payload)
            ->assertStatus(201)
            ->assertJson(['name' => $payload['name']]);
    }

    public function testCreateBoardValidation()
    {
        $this->actingAs($this->user)
            ->json('POST', 'api/boards')
            ->assertStatus(422)
            ->assertJson([
                "message" => "The given data was invalid.",
                "errors" => [
                    "name" => ["The name field is required."]
                ]
            ]);
    }

    public function testUserCanUpdateBoard()
    {
        $payload = ['name' => 'My Board'];

        $this->actingAs($this->user)
            ->json('PATCH', 'api/boards/' . $this->board->id, $payload)
            ->assertStatus(200)
            ->assertJson(['name' => $payload['name']])
            ->assertJsonMissing(['name' => $this->board->name]);
    }

    public function testUpdateValidation()
    {
        $this->actingAs($this->user)
            ->json('PATCH', 'api/boards/' . $this->board->id)
            ->assertStatus(422)
            ->assertJson([
                "message" => "The given data was invalid.",
                "errors" => [
                    "name" => ["The name field is required."]
                ]
            ]);
    }

    public function testUserCanDeleteBoards()
    {
        $this->actingAs($this->user)
            ->json('DELETE', 'api/boards/' . $this->board->id)
            ->assertStatus(204);

        // Check that it is really gone with a GET request of all boards of the user
        $this->actingAs($this->user)
            ->json('GET', 'api/boards')
            ->assertJsonCount(0);
    }

    public function testAuthorization()
    {
        $user2 = factory(User::class)->create();

        $this->actingAs($user2)
            ->json('GET', 'api/boards/' . $this->board->id)
            ->assertStatus(403);

        $this->actingAs($user2)
            ->json('DELETE', 'api/boards/' . $this->board->id)
            ->assertStatus(403);

        $this->actingAs($user2)
            ->json('PATCH', 'api/boards/' . $this->board->id)
            ->assertStatus(403);
        
        $this->actingAs($user2)
            ->json('PATCH', 'api/board/updateOrder', ['board' => $this->board])
            ->assertStatus(403);

        $this->actingAs($user2)
            ->json('PATCH', 'api/board/updateListOrder', ['board' => $this->board])
            ->assertStatus(403);
    }
}
```

Let's break it down a bit: We use a setup method that will be called before each test in this file. In it we call the parent setup (of our DatabaseTestCase, which means it will drop all tables and run the migrations) and then create a user and a board belonging to the user. This way we don't have to copy the setup code into every test we need a user with a board (which is pretty much every test here). 

The first test loginRequired checks that we get the Unauthenticated status code when we're not acting as the created user. In the second test we act as the user and assert that we see the board in the json response of our board overview (index method in the controller). The next test checks that we can see 2 BoardLists in the view of a specific board.

The next two tests check the store method of the controller. We first check that with a correct payload provided the board is created correctly. The other test checks that without a payload we get the validation error that the name is required. The same is done for the update method with the next two tests. The delete test just deletes the board and checks that we get 0 boards when we call the index method after that.

The last test checks for authorization at all controller methods that need to be protected from access of another user. For this we create a second user and try to access all API endpoints that should be protected of the first users board. We assert that all of them fail with an Unauthorized message.

Let's continue with `tests/Feature/BoardListTest.php`:

```php
<?php

namespace Tests\Feature;

use Tests\DatabaseTestCase;
use Illuminate\Foundation\Testing\WithFaker;
use App\User;
use App\Board;
use App\BoardList;

class BoardListTest extends DatabaseTestCase
{
    protected $user;
    protected $board;
    protected $boardList;

    public function setUp()
    {
        parent::setUp();
        $this->user = factory(User::class)->create();
        $this->board = factory(Board::class)->create(['user_id' => $this->user->id]);
        $this->boardList = factory(BoardList::class)->create(['board_id' => $this->board->id]);
    }

    public function testLoginRequired()
    {
        $payload = ['name' => 'My List', 'order' => 1, 'board_id' => $this->board->id];

        $this->json('POST', 'api/boardLists', $payload)
            ->assertStatus(401);
    }

    public function testUserCanCreateLists()
    {
        $payload = ['name' => 'My List', 'order' => 1, 'board_id' => $this->board->id];

        $this->actingAs($this->user)
            ->json('POST', 'api/boardLists', $payload)
            ->assertStatus(201)
            ->assertJson(['name' => $payload['name']]);
    }

    public function testCreateValidation()
    {
        $this->actingAs($this->user)
            ->json('POST', 'api/boardLists')
            ->assertStatus(422)
            ->assertJson([
                "message" => "The given data was invalid.",
                "errors" => [
                    "name" => ["The name field is required."],
                    "order" => ["The order field is required."],
                    "board_id" => ["The board id field is required."],
                ]
            ]);
    }

    public function testUserCanUpdateLists()
    {
        $payload = ['name' => 'My List'];

        $this->actingAs($this->user)
            ->json('PATCH', 'api/boardLists/' . $this->boardList->id, $payload)
            ->assertStatus(200)
            ->assertJson(['name' => $payload['name']])
            ->assertJsonMissing(['name' => $this->boardList->name]);
    }

    public function testUpdateValidation()
    {
        $this->actingAs($this->user)
            ->json('PATCH', 'api/boardLists/' . $this->boardList->id)
            ->assertStatus(422)
            ->assertJson([
                "message" => "The given data was invalid.",
                "errors" => [
                    "name" => ["The name field is required."]
                ]
            ]);
    }

    public function testUserCanDeleteLists()
    {
        $this->actingAs($this->user)
            ->json('DELETE', 'api/boardLists/' . $this->boardList->id)
            ->assertStatus(204);

        // Check that it is really gone with a GET request of the board of the user
        $this->actingAs($this->user)
            ->json('GET', 'api/boards/' . $this->board->id)
            ->assertJsonCount(0, 'board_lists');
    }

    public function testAuthorization()
    {
        $user2 = factory(User::class)->create();
        $payload = ['name' => 'My List', 'order' => 1, 'board_id' => $this->board->id];

        $this->actingAs($user2)
            ->json('POST', 'api/boardLists', $payload)
            ->assertStatus(403);

        $this->actingAs($user2)
            ->json('PATCH', 'api/boardLists/' . $this->boardList->id, $payload)
            ->assertStatus(403);

        $this->actingAs($user2)
            ->json('DELETE', 'api/boardLists/' . $this->boardList->id)
            ->assertStatus(403);
    }
}
```

It's pretty much the same as before. We test store, update, delete, loginRequired and authorization needed. We added a created boardList into our setup since we now need a boardList too. If you understood the Board tests you should also quickly understand these here.

And finally `tests/Feature/CardTest.php`:

```php
<?php

namespace Tests\Feature;

use Tests\DatabaseTestCase;
use Illuminate\Foundation\Testing\WithFaker;
use App\User;
use App\Board;
use App\BoardList;
use App\Card;

class CardTest extends DatabaseTestCase
{
    protected $user;
    protected $board;
    protected $boardList;
    protected $card;

    public function setUp()
    {
        parent::setUp();
        $this->user = factory(User::class)->create();
        $this->board = factory(Board::class)->create(['user_id' => $this->user->id]);
        $this->boardList = factory(BoardList::class)->create(['board_id' => $this->board->id]);
        $this->card = factory(Card::class)->create(['board_list_id' => $this->boardList->id]);
    }

    public function testLoginRequired()
    {
        $payload = ['name' => 'My Card', 'order' => 1, 'list_id' => $this->boardList->id];

        $this->json('POST', 'api/cards', $payload)
            ->assertStatus(401);
    }

    public function testUserCanCreateCards()
    {
        $payload = ['name' => 'My Card', 'order' => 1, 'list_id' => $this->boardList->id];

        $this->actingAs($this->user)
            ->json('POST', 'api/cards', $payload)
            ->assertStatus(201)
            ->assertJson(['name' => $payload['name']]);
    }

    public function testCreateValidation()
    {
        $this->actingAs($this->user)
            ->json('POST', 'api/cards')
            ->assertStatus(422)
            ->assertJson([
                "message" => "The given data was invalid.",
                "errors" => [
                    "name" => ["The name field is required."],
                    "order" => ["The order field is required."],
                    "list_id" => ["The list id field is required."],
                ]
            ]);
    }

    public function testUserCanUpdateCards()
    {
        $payload = ['name' => 'My Card'];

        $this->actingAs($this->user)
            ->json('PATCH', 'api/cards/' . $this->card->id, $payload)
            ->assertStatus(200)
            ->assertJson(['name' => $payload['name']])
            ->assertJsonMissing(['name' => $this->card->name]);
    }

    public function testUpdateValidation()
    {
        $this->actingAs($this->user)
            ->json('PATCH', 'api/cards/' . $this->card->id)
            ->assertStatus(422)
            ->assertJson([
                "message" => "The given data was invalid.",
                "errors" => [
                    "name" => ["The name field is required when description is not present."]
                ]
            ]);
    }

    public function testUserCanDeleteCards()
    {
        $card2 = factory(Card::class)->create(['board_list_id' => $this->boardList->id]);

        $this->actingAs($this->user)
            ->json('DELETE', 'api/cards/' . $this->card->id)
            ->assertStatus(204);

        // Check that it is really gone with a GET request of the board of the user
        $this->actingAs($this->user)
            ->json('GET', 'api/boards/' . $this->board->id)
            ->assertDontSee($this->card->name)
            ->assertSee($card2->name);
    }

    public function testAuthorization()
    {
        $user2 = factory(User::class)->create();
        $payload = ['name' => 'My Card', 'order' => 1, 'list_id' => $this->boardList->id];

        $this->actingAs($user2)
            ->json('POST', 'api/cards', $payload)
            ->assertStatus(403);

        $this->actingAs($user2)
            ->json('PATCH', 'api/cards/' . $this->card->id, $payload)
            ->assertStatus(403);        

        $this->actingAs($user2)
            ->json('DELETE', 'api/cards/' . $this->card->id)
            ->assertStatus(403);
    }
}
```

Again we added a card to our setup but the tests are pretty much the same.

We make heavy use of the factories and often provide an array to the create method which overwrites the default values which we defined in the factories. All test code is very easy to understand and read which makes developing these tests really enjoyable thanks to Laravels helper methods. And since we're using Laravels auth system with our JWT authentication we can simply use actingAs instead of having to get and send a bearer token and all that stuff. So this was a lot of fun, I really like feature testing Laravel applications.

[In the next post][part5] we want to setup TravisCI to run these tests automatically on different PHP versions, and if everything works deploy the code to heroku.

[source]: https://github.com/ddmler/boards
[part1]: https://ddmler.github.io/laravel/vue/2018/07/13/vue-spa-with-laravel-api-part-1.html
[part2]: https://ddmler.github.io/laravel/vue/2018/07/17/vue-spa-with-laravel-api-part-2.html
[part3]: https://ddmler.github.io/laravel/vue/2018/07/20/vue-spa-with-laravel-api-part-3.html
[part5]: https://ddmler.github.io/laravel/vue/2018/07/27/vue-spa-with-laravel-api-part-5.html
[part6]: https://ddmler.github.io/laravel/vue/2018/07/31/vue-spa-with-laravel-api-part-6.html
[part7]: https://ddmler.github.io/laravel/vue/2018/08/07/vue-spa-with-laravel-api-part-7.html
[part8]: https://ddmler.github.io/laravel/vue/2018/08/10/vue-spa-with-laravel-api-part-8.html
