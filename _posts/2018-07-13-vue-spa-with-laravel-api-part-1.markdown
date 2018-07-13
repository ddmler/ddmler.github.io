---
layout: post
title:  "Vue SPA with Laravel API Part 1"
date:   2018-07-13 18:00:00 +0100
categories: laravel vue
description: "Part 1: Setting up a Vue SPA with a Laravel API as the backend using JWT authentication."
---

The complete source code of this series is available [on GitHub][source].

- Part 1: Setting up a Vue SPA with a Laravel API as the backend using JWT authentication. (You are here)
- Part 2: Creating everything database related: models, migrations, seeders and factories.
- Part 3: Setting up API routes and creating API resource controllers.
- Part 4: Testing the API with feature tests and sqlite.
- Part 5: Setting up Continuous Deployment with TravisCI and Heroku.
- Part 6: Customizing the frontend skeleton, design and Dashboard component.
- Part 7: Creating the remaining Vue components: Board, List, Card, Modal.
- Part 8: Setting up Jest and testing all Vue components with it.

(Links will be added when the posts are online)

In this multipart series we will be creating a Vue.js single page application with a Laravel backend API. In this part we will be setting up the skeleton, which means just enough to register and login, so if you only want the skeleton of our SPA you can stop after the first part. After that we will continue by building a small Trello clone as our SPA. What you will need to follow along:

1. NPM/Yarn and Composer installed
1. something to run your project on (for example [Laradock][laradock])

This is built with Laravel 5.6 and Vue 2.5 if you use newer versions there could be some problems along the way.

The first step is to create a new Laravel project if you haven't yet:

`composer create-project --prefer-dist laravel/laravel boards`

If everything works we install some npm dependencies next:

`npm install --save-dev vue-router vue-axios @websanova/vue-auth`

We will need vue-router to create a single page application and vue-axios provides a small wrapper for integrating axios into Vue, which vue-auth needs. Vue-auth is a library that takes care of all our authentication needs for the frontend.

So to use vue-router we need to tell Laravel not to handle web routing. We do this by creating a route that matches all urls that will just return the view of our vue router. So your `routes/web.php` should look like this:

```php
<?php

Route::any('{all}', function () {
    return view('app');
})->where(['all' => '.*']);
```

Next we want to create the Laravel view under `resources/views/app.blade.php`:

{% raw %}
```html
<!doctype html>
<html lang="{{ app()->getLocale() }}">
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <meta name="csrf-token" content="{{ csrf_token() }}">
        <meta name="api-base-url" content="{{ config('app.url') }}">

        <title>{{ config('app.name') }}</title>

        <link href="{{ mix('/css/app.css') }}" rel="stylesheet">
    </head>
    <body>

        <div id="app"></div>

        <script src="{{ mix('/js/app.js') }}"></script>
    </body>
</html>
```
{% endraw %}

This will be our only Laravel view and we will mount Vue to the div with the id app later. We use mix to take care of our css and js assets.

The next step is to bootstrap vue and vue-router. For this your `resources/assets/app.js` file should look like this:

```javascript
import Vue from 'vue';
import VueRouter from 'vue-router';
import axios from 'axios';
import VueAxios from 'vue-axios';
import App from './App.vue';
import Dashboard from './components/Dashboard.vue';
import Home from './components/Home.vue';
import Register from './components/Register.vue';
import Login from './components/Login.vue';


Vue.use(VueRouter);
Vue.use(VueAxios, axios);
axios.defaults.baseURL = document.head.querySelector('meta[name="api-base-url"]').content + '/api';

const router = new VueRouter({
    routes: [{
        path: '/',
        name: 'home',
        component: Home
    },{
        path: '/register',
        name: 'register',
        component: Register,
        meta: {
            auth: false
        }
    },{
        path: '/login',
        name: 'login',
        component: Login,
        meta: {
            auth: false
        }
    },{
        path: '/dashboard',
        name: 'dashboard',
        component: Dashboard,
        meta: {
            auth: true
        }
    }]
});


Vue.router = router
Vue.use(require('@websanova/vue-auth'), {
   auth: require('@websanova/vue-auth/drivers/auth/bearer.js'),
   http: require('@websanova/vue-auth/drivers/http/axios.1.x.js'),
   router: require('@websanova/vue-auth/drivers/router/vue-router.2.x.js'),
});
App.router = Vue.router
new Vue(App).$mount('#app');
```

First we tell vue to use vue-router and the vue-axios wrapper and set the default baseurl of our api. This way we save a lot of typing later on. Next we create the VueRouter object. The attributes of each route should be self-explanatory. The auth attribute in meta defines for which users the route is accessible. `auth: true` means only loggedin users can access this route, `auth: false` means only not loggedin users can access this route and nothing means this route is accessible to anyone. Lastly we setup the vue-auth library with the correct drivers we want to use and mount our Vue object.

Now lets create the SPA navigation, for this create `resources/assets/App.vue`:

{% raw %}
```html
<template>
    <div>
        <div v-if="$auth.ready()">
            <div>
                <nav>
                    <ul>
                        <li>
                            <router-link :to="{ name: 'home' }">Home</router-link>
                        </li>
                        <li v-if="!$auth.check()">
                            <router-link :to="{ name: 'login' }">Login</router-link>
                        </li>
                        <li v-if="!$auth.check()">
                            <router-link :to="{ name: 'register' }">Register</router-link>
                        </li>
                        <li v-if="$auth.check()">
                            <router-link :to="{ name: 'dashboard' }">Dashboard</router-link>
                        </li>
                        <li v-if="$auth.check()">
                            <a href="#" @click.prevent="$auth.logout()">Logout ({{ $auth.user().name }})</a>
                        </li>
                    </ul>
                </nav>
            </div>
            <div>
                <router-view></router-view>
            </div>
        </div>
        <div v-if="!$auth.ready()">
            Loading...
        </div>
    </div>
</template>
```
{% endraw %}

We will show a loading message while our auth object is not yet ready. If it is ready we render a navigation and the router view. Notice that the auth object provides a lot of useful functions like check, logout and the username. With these we can create a navigation that changes depending on your authentication state and the logout functionality is done already.


We still need some components so our router-view can render something. `resources/assets/components/Home.vue` is our homepage, is accessible to anyone and looks like this:

```html
<template>
    <h1>Laravel Boards</h1>
</template>
```

`resources/assets/components/Dashboard.vue` is accessible only to loggedin users and looks like this:

```html
<template>
    <h1>Laravel Dashboard</h1>
</template>
```

The last two steps for the frontend are a register and login component.

`resources/assets/components/Register.vue`:

{% raw %}
```html
<template>
    <div>
        <div v-if="error && !success">
            <p>There was an error.</p>
        </div>
        <div v-if="success">
            <p>Registration successful. You can now <router-link :to="{name:'login'}">login.</router-link></p>
        </div>
        <form autocomplete="off" @submit.prevent="register" v-if="!success" method="post">
            <div>
                <label for="name">Username</label>
                <input type="text" id="name" v-model="name" required>
                <span v-if="error && errors.name">{{ errors.name }}</span>
            </div>
            <div>
                <label for="email">E-mail</label>
                <input type="email" id="email" placeholder="user@example.com" v-model="email" required>
                <span v-if="error && errors.email">{{ errors.email }}</span>
            </div>
            <div>
                <label for="password">Password</label>
                <input type="password" id="password" v-model="password" required>
                <span v-if="error && errors.password">{{ errors.password }}</span>
            </div>
            <button type="submit">Register</button>
        </form>
    </div>
</template>
<script>
    export default {
        data(){
            return {
                name: '',
                email: '',
                password: '',
                error: false,
                errors: {},
                success: false
            };
        },
        methods: {
            register(){
                var app = this
                this.$auth.register({
                    params: {
                        name: app.name,
                        email: app.email,
                        password: app.password
                    },
                    success: function () {
                        app.success = true
                    },
                    error: function (resp) {
                        app.error = true;
                        app.errors = resp.response.data.errors;
                    },
                    redirect: null
                });
            }
        }
    }
</script>
```
{% endraw %}

Vue-auth provides a register method which we use and provide the data from the form above. We then also provide a success and an error function and don't redirect on success. If there is an error we will highlight the field and provide an error message (from Laravel).


`resources/assets/components/Login.vue`:

```html
<template>
    <div>
        <div v-if="error">
            <p>It looks like those credentials are not working.</p>
        </div>
        <form autocomplete="off" @submit.prevent="login" method="post">
            <div>
                <label for="email">E-mail</label>
                <input type="email" id="email" placeholder="user@example.com" v-model="email" required>
            </div>
            <div>
                <label for="password">Password</label>
                <input type="password" id="password" v-model="password" required>
            </div>
            <button type="submit">Sign in</button>
        </form>
    </div>
</template>
<script>
  export default {
    data(){
      return {
        email: null,
        password: null,
        error: false
      }
    },
    methods: {
      login(){
        var app = this
        this.$auth.login({
            params: {
              email: app.email,
              password: app.password
            },
            success: function () {},
            error: function () { this.error = true; },
            rememberMe: true,
            redirect: '/dashboard',
            fetchUser: true,
        });
      },
    }
  }
</script>
```

The login component is very similar to the register one. We use the login method provided by vue-auth and add a rememberMe option, a redirect after a successful login to the dashboard component and a fetchUser option. The fetchUser option set to true will automatically fetch the users data which we used in our navigation earlier. And we're done with the frontend skeleton.

Now lets begin with our API. Our first step here will be to install a package that provides JWT support for Laravels auth system: [jwt-auth][jwt].

To install simply run `composer require tymon/jwt-auth`.

Now open `config/app.php` and add this to the `providers` array: `Tymon\JWTAuth\Providers\LaravelServiceProvider::class,`

Next run `php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"`

And `php artisan jwt:secret`

There is a new config file: `config/jwt.php` where you can configure this package.

Next open `config/auth.php` and set the default guard to api like this:

```php
'defaults' => [
    'guard' => 'api',
    'passwords' => 'users',
],
```

Below that change the api guard driver to jwt like this:

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```

Our User model needs to implement the `JWTSubject` interface and we need to implement two methods: `getJWTIdentifier()` and `getJWTCustomClaims`:

```php
<?php

namespace App;

use Tymon\JWTAuth\Contracts\JWTSubject;
use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements JWTSubject
{
    use Notifiable;
    
    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name', 'email', 'password',
    ];
    
    /**
     * The attributes that should be hidden for arrays.
     *
     * @var array
     */
    protected $hidden = [
        'password', 'remember_token',
    ];
    
    /**
    * Get the identifier that will be stored in the subject claim of the JWT.
    *
    * @return mixed
    */
    public function getJWTIdentifier()
    {
        return $this->getKey();
    }
    
    /**
     * Return a key value array, containing any custom claims to be added to the JWT.
     *
     * @return array
     */
    public function getJWTCustomClaims()
    {
        return [];
    }
}
```

Finally add these routes to your `routes/api.php` file:

```php
Route::group([
    'middleware' => 'api',
    'prefix' => 'auth'
], function ($router) {
    Route::post('register', 'AuthController@register');
    Route::post('login', 'AuthController@login');
    Route::post('logout', 'AuthController@logout');
    Route::post('refresh', 'AuthController@refresh');
    Route::post('me', 'AuthController@me');
    Route::get('user', 'AuthController@user');
});
```

You should now be able to register, login and logout. You can stop the tutorial here if you just wanted to set up a starter boilerplate Vue SPA with a Laravel API as a backend. For everyone else we now begin developing our Trello clone by creating all the database stuff in Part 2.

[laradock]: https://ddmler.github.io/laravel/docker/2018/03/08/getting-started-with-laradock.html
[jwt]: https://github.com/tymondesigns/jwt-auth
[source]: https://github.com/ddmler/boards
