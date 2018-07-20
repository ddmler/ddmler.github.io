---
layout: post
title:  "Vue SPA with Laravel API Part 3"
date:   2018-07-20 18:00:00 +0100
categories: laravel vue
description: "Part 3: Setting up API routes and creating API resource controllers."
---

The complete source code of this series is available [on GitHub][source].

- Part 1: [Setting up a Vue SPA with a Laravel API as the backend using JWT authentication.][part1]
- Part 2: [Creating everything database related: models, migrations, seeders and factories.][part2]
- Part 3: Setting up API routes and creating API resource controllers. (You are here)
- Part 4: Testing the API with feature tests and sqlite.
- Part 5: Setting up Continuous Deployment with TravisCI and Heroku.
- Part 6: Customizing the frontend skeleton, design and Dashboard component.
- Part 7: Creating the remaining Vue components: Board, List, Card, Modal.
- Part 8: Setting up Jest and testing all Vue components with it.

(Links will be added when the posts are online)

In this part we will create all necessary API routes and our controllers so we can consume our data via the API.

First lets add these routes to `routes/api.php`:

```php
Route::group([

    'middleware' => ['api', 'auth:api']

], function($router) {
    Route::apiResources([
      'boards' => 'API\BoardController',
      'boardLists' => 'API\BoardListController',
      'cards' => 'API\CardController'
    ]);

    Route::patch('board/updateOrder', 'API\BoardController@updateOrder');
    Route::patch('board/updateListOrder', 'API\BoardController@updateListOrder');
});
```

And let artisan create the controllers with an `--api` flag which will automatically create a lot of methods we may want:

```sh
php artisan make:controller API/BoardController --api --model=Board
php artisan make:controller API/BoardListController --api --model=BoardList
php artisan make:controller API/CardController --api --model=Card
```

Let's start with the `app/Http/Controllers/API/BoardController.php`. Don't worry we'll go through each method after the code.

```php
<?php

namespace App\Http\Controllers\API;

use App\Board;
use App\Card;
use App\BoardList;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Auth;

class BoardController extends Controller
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        $boards = Board::with('boardLists.cards')->where('user_id', Auth::id())->get();
        return response()->json($boards);
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $request->validate([
            'name' => 'required',
        ]);

        $board = new Board;
        $board->name = $request->name;
        $board->user()->associate(Auth::id());
        $board->save();

        return response()->json($board, 201);
    }

    /**
     * Display the specified resource.
     *
     * @param  \App\Board  $board
     * @return \Illuminate\Http\Response
     */
    public function show(Board $board)
    {
        if (Auth::id() !== $board->user->id) {
            abort(403, 'Unauthorized for this action.');
        }

        $board->load(['boardLists' => function ($query) {
            $query->with(['cards' => function ($query) {
                $query->orderBy('order');
            }])->orderBy('order');
        }]);

        return response()->json($board);
    }

    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \App\Board  $board
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, Board $board)
    {
        if (Auth::id() !== $board->user->id) {
            abort(403, 'Unauthorized for this action.');
        }

        $request->validate([
            'name' => 'required',
        ]);

        $board->name = $request->name;
        $board->save();

        return response()->json($board);
    }

    public function updateOrder(Request $request)
    {
        $board = Board::findOrFail($request->board["id"]);

        if (Auth::id() !== $board->user->id) {
            abort(403, 'Unauthorized for this action.');
        }

        foreach ($request->board["board_lists"] as $boardList) {
            foreach ($boardList["cards"] as $card) {
                $cardModel = Card::find($card["id"]);
                $cardModel->board_list_id = $boardList["id"];
                $cardModel->order = $card["order"];
                $cardModel->save();
            }
        }

        return response()->json("OK");
    }

    public function updateListOrder(Request $request)
    {
        $board = Board::findOrFail($request->board["id"]);

        if (Auth::id() !== $board->user->id) {
            abort(403, 'Unauthorized for this action.');
        }

        foreach ($request->board["board_lists"] as $boardList) {
            $listModel = BoardList::find($boardList["id"]);
            $listModel->order = $boardList["order"];
            $listModel->save();
        }

        return response()->json("OK");
    }

    /**
     * Remove the specified resource from storage.
     *
     * @param  \App\Board  $board
     * @return \Illuminate\Http\Response
     */
    public function destroy(Board $board)
    {
        if (Auth::id() !== $board->user->id) {
            abort(403, 'Unauthorized for this action.');
        }

        $board->delete();
        return response()->json(null, 204);
    }
}
```

The index method is very simple: We get all boards of the user and return it as json. The store method is also very simple in that we validate that a name was given, create the board, associate it to the user, save it and return it as json with a http status code. The update method is basically the same, we just check that the board belongs to the user who is currently loggedin. In the show method we have to change our query a bit because we want our results to contain an ordered list of BoardLists and for each of those an ordered list of Cards. The destroy method is also pretty straightforward: Check that the board belongs to the user and if so delete it.

Now we have two methods left: updateListOrder and updateCardOrder. Both are pretty similar in that we get as input a json Board object (the same as we would output in our show method). The only difference here will be that javascript has changed the order values of the lists/cards and moved them around inside the array. So we just have to loop through all lists/cards and update their order value in the database to match the frontend. For cards we also change the BoardList association, because you are able to drag a card from one list to another as well as changing the simple order inside one list.

And that's it for our BoardController. The rest will be very simple, trust me. Let's create the `app/Http/Controllers/API/BoardListController.php` next:


```php
<?php

namespace App\Http\Controllers\API;

use App\BoardList;
use App\Board;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Auth;

class BoardListController extends Controller
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        //
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $request->validate([
            'name' => 'required',
            'order' => 'required|integer',
            'board_id' => 'required|integer',
        ]);

        $board = Board::findOrFail($request->board_id);

        if (Auth::id() !== $board->user->id) {
            abort(403, 'Unauthorized for this action.');
        }

        $boardList = new BoardList;
        $boardList->name = $request->name;
        $boardList->order = $request->order;
        $boardList->board()->associate($request->board_id);
        $boardList->save();

        return response()->json($boardList->load('cards'), 201);
    }

    /**
     * Display the specified resource.
     *
     * @param  \App\BoardList  $boardList
     * @return \Illuminate\Http\Response
     */
    public function show(BoardList $boardList)
    {
        //
    }

    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \App\BoardList  $boardList
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, BoardList $boardList)
    {
        if (Auth::id() !== $boardList->board->user->id) {
            abort(403, 'Unauthorized for this action.');
        }

        $request->validate([
            'name' => 'required',
        ]);

        $boardList->name = $request->name;
        $boardList->save();

        return response()->json($boardList);
    }

    /**
     * Remove the specified resource from storage.
     *
     * @param  \App\BoardList  $boardList
     * @return \Illuminate\Http\Response
     */
    public function destroy(BoardList $boardList)
    {
        if (Auth::id() !== $boardList->board->user->id) {
            abort(403, 'Unauthorized for this action.');
        }

        $boardList->delete();
        return response()->json(null, 204);
    }
}
```

We only use the store, update and destroy method because a list will only be requested together with the board. Those three methods are basically the same as in board, just that we added the auth check in our store method.

And finally the `app/Http/Controllers/API/CardController.php`:

```php
<?php

namespace App\Http\Controllers\API;

use App\Card;
use App\BoardList;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Auth;

class CardController extends Controller
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        //
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $request->validate([
            'name' => 'required',
            'order' => 'required|integer',
            'list_id' => 'required|integer',
        ]);

        $boardList = BoardList::findOrFail($request->list_id);

        if (Auth::id() !== $boardList->board->user->id) {
            abort(403, 'Unauthorized for this action.');
        }

        $card = new Card;
        $card->name = $request->name;
        $card->order = $request->order;
        $card->description = "";
        $card->boardList()->associate($request->list_id);
        $card->save();

        return response()->json($card, 201);
    }

    /**
     * Display the specified resource.
     *
     * @param  \App\Card  $card
     * @return \Illuminate\Http\Response
     */
    public function show(Card $card)
    {
        return response()->json($card);
    }

    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \App\Card  $card
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, Card $card)
    {
        if (Auth::id() !== $card->boardList->board->user->id) {
            abort(403, 'Unauthorized for this action.');
        }

        $request->validate([
            'name' => 'required_without:description',
            'description' => 'required_without:name',
        ]);

        $card->name = $request->name ?: $card->name;
        $card->description = $request->description ?: $card->description;
        $card->save();

        return response()->json($card);
    }

    /**
     * Remove the specified resource from storage.
     *
     * @param  \App\Card  $card
     * @return \Illuminate\Http\Response
     */
    public function destroy(Card $card)
    {
        if (Auth::id() !== $card->boardList->board->user->id) {
            abort(403, 'Unauthorized for this action.');
        }

        $card->delete();
        return response()->json(null, 204);
    }
}
```

Again almost the same but here we use update in two different ways: once to update the name and once to update the description (these will be two different forms in the frontend as we will see later)

And now we have a fully functioning API for authentication, Boards, BoardLists and Cards. But we still lack some tests for our API so we will create some feature tests next. You can find these in part 4.

[source]: https://github.com/ddmler/boards
[part1]: https://ddmler.github.io/laravel/vue/2018/07/13/vue-spa-with-laravel-api-part-1.html
[part2]: https://ddmler.github.io/laravel/vue/2018/07/17/vue-spa-with-laravel-api-part-2.html
