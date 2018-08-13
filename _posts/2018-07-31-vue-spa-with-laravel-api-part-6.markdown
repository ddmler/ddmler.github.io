---
layout: post
title:  "Vue SPA with Laravel API Part 6"
date:   2018-07-31 18:00:00 +0100
categories: laravel vue
description: "Part 6: Customizing the frontend skeleton, design and Dashboard component."
---

The complete source code of this series is available [on GitHub][source].

- Part 1: [Setting up a Vue SPA with a Laravel API as the backend using JWT authentication.][part1]
- Part 2: [Creating everything database related: models, migrations, seeders and factories.][part2]
- Part 3: [Setting up API routes and creating API resource controllers.][part3]
- Part 4: [Testing the API with feature tests and sqlite.][part4]
- Part 5: [Setting up Continuous Deployment with TravisCI and Heroku.][part5]
- Part 6: Customizing the frontend skeleton, design and Dashboard component. (You are here)
- Part 7: [Creating the remaining Vue components: Board, List, Card, Modal.][part7]
- Part 8: [Setting up Jest and testing all Vue components with it.][part8]

(Links will be added when the posts are online)

Lets work a bit on our frontend after all that backend stuff. For this we need to install some additional dependencies:

`npm install --save-dev @fortawesome/fontawesome-free bulma vuedraggable`

We use fontawesome for icons, bulma for our general design and vuedraggable to let our lists and cards be reordered via drag&drop however the user wants.

We also have [ESLint][eslint], [stylelint][stylelint] and [php-cs-fixer][cs-fixer] installed and made an npm script similar to [this][npm-script]. This allows us to run all three linters with the fix option at the same time which will fix all our php, vue (only template and js) and scss files.

We want to override some bulma variables to change our design so add these to the `resources/assets/sass/_variables.scss` file:

```sass
$text: hsl(0, 0%, 94%);
$text-light: hsl(0, 0%, 94%);
$text-strong: hsl(0, 0%, 94%);
$link: hsl(348, 100%, 61%);
$link-hover: hsl(348, 100%, 41%);
$primary: #242424;
$input-background-color: #363636;
$input-color: #fff;
$input-border-color: #242424;
$input-hover-border-color: #242424;
$button-color: #fff;
$button-background-color: #242424;
$button-hover-color: #fff;
$button-border-width: 0;
$button-focus-color: #fff;
$button-active-color: #fff;
$message-body-padding: 0.75rem;
```

Also lets import bulma and fontawesome and set some general styles in our `resources/assets/sass/app.scss` file:

```sass
// Fonts
@import url('https://fonts.googleapis.com/css?family=Raleway:300,400,600');

// Variables
@import 'variables';

// Bulma
// change variables before import
@import './node_modules/bulma/bulma.sass';

// FontAwesome
@import './node_modules/@fortawesome/fontawesome-free/scss/fontawesome.scss';
@import './node_modules/@fortawesome/fontawesome-free/scss/fa-regular.scss';
@import './node_modules/@fortawesome/fontawesome-free/scss/fa-brands.scss';
@import './node_modules/@fortawesome/fontawesome-free/scss/fa-solid.scss';

html {
  background-color: #121212;
}

.main-wrapper {
  padding: 10px;
}

.label {
  color: #fff;
}

.button:hover {
  background-color: #363636;
}

.message.is-danger {
  background-color: #242424;
}

.card {
  background-color: #232323;
  border-radius: 5px;
}
```

In our `resources/assets/js/bootstrap.js` we remove jquery and bootstrap since we're not using these two.

Lets get to the Javascript part. We will need an EventBus later. Luckily Vue is an implementation of an EventBus so we just create this file `resources/assets/js/EventBus.js`:

```js
import Vue from 'vue';

export const EventBus = new Vue();
```

And our EventBus is done. Next we update our `resources/assets/js/app.js` file:

```js
import Vue from 'vue';
import VueRouter from 'vue-router';
import axios from 'axios';
import VueAxios from 'vue-axios';
import App from './App.vue';
import Dashboard from './components/Dashboard.vue';
import Board from './components/Board.vue';
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
    },{
        path: '/board/:id',
        name: 'board_view',
        component: Board,
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
App.data = {
    loading: false,
    error: false,
    errors: {},
}
App.methods = {
    closeErrors() {
        this.error = false;
        this.errors = {};
    }
}
const vueApp = new Vue(App).$mount('#app');

axios.interceptors.request.use(config => {
    vueApp.loading = true;
    return config;
});

axios.interceptors.response.use(config => {
    vueApp.loading = false;
    vueApp.error = false;
    return config;
}, error => {
    vueApp.loading = false;
    vueApp.errors = error.response.data.errors || { "none": [error.response.data.message] };
    vueApp.error = true;
    return Promise.reject(error);
});
```

All we did is add a new route to view a board, add a data and a methods object to our root vue object and create some axios interceptors. The axios interceptors will set the loading state of our application on our root object before each request it makes and when it gets a response it will automatically check for Laravel errors (we handle validation and other errors the same here and display all, you could also just display all validation errors) and set the loading state to false again. We use these root element data objects in our `resources/assets/js/App.vue` file but we could use them anywhere:

{% raw %}
```html
<template>
  <div>
    <div v-if="$auth.ready()">
      <nav class="navbar is-primary">
        <div class="navbar-brand">
          <router-link 
            v-if="!$auth.check()" 
            :to="{ name: 'home' }" 
            class="navbar-item">Boards</router-link>
          <router-link 
            v-if="$auth.check()" 
            :to="{ name: 'dashboard' }" 
            class="navbar-item">Boards</router-link>
        </div>
        <div class="navbar-menu">
          <div class="navbar-start"/>
          <div class="navbar-end">
            <div 
              v-if="$root.loading" 
              class="navbar-item">
              <i 
                class="fas fa-spinner fa-spin fa-lg" 
                title="Loading/Saving"/>
            </div>
            <div 
              v-if="!$root.loading" 
              class="navbar-item">
              <i 
                class="fas fa-cloud fa-lg" 
                title="Everything is saved"/>
            </div>
            <router-link 
              v-if="!$auth.check()" 
              :to="{ name: 'login' }" 
              class="navbar-item">Login</router-link>
            <router-link 
              v-if="!$auth.check()" 
              :to="{ name: 'register' }" 
              class="navbar-item">Register</router-link>
            <div 
              v-if="$auth.check()" 
              class="navbar-item"><i class="fas fa-user"/> {{ $auth.user().name }}</div>
            <a 
              v-if="$auth.check()" 
              href="#" 
              class="navbar-item" 
              @click.prevent="$auth.logout()">Logout</a>
          </div>
        </div>
      </nav>
      <div class="main-wrapper">
        <div 
          v-if="$root.error" 
          class="message is-danger">
          <div class="message-body">
            <span class="error-close">
              <a @click.prevent="closeErrors"><i class="fas fa-times"/></a>
            </span>
            Oops we have some errors: <ul>
              <li 
                v-for="(e,index) in $root.errors" 
                :key="index">
                <span 
                  v-for="message in e" 
                  :key="message">{{ message }}</span>
              </li>
            </ul>
          </div>
        </div>
        <router-view/>
      </div>
    </div>
    <div v-if="!$auth.ready()">
      <section class="hero is-fullheight">
        <div class="hero-head"/>
        <div class="hero-body">
          <div class="container has-text-centered">
            <i class="fas fa-spinner fa-spin fa-3x"/>
          </div>
        </div>
        <div class="hero-foot"/>
      </section>
    </div>
  </div>
</template>
<style>
.navbar.is-primary .navbar-end > a.navbar-item:hover,
.navbar.is-primary .navbar-brand > a.navbar-item:hover {
  background-color: #363636;
}

.navbar-laravel {
  background-color: #fff;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.04);
}

.fa-user {
  margin-right: 5px;
}

.error-close {
  position: absolute;
  right: 25px;
}

.error-close i {
  font-size: 1.75rem;
}
</style>
```
{% endraw %}

We changed quite a few things here. First we added two icons in our menu which will change depending on the `$root.loading` state in our root vue object. If loading is set to true, we will display a spinning icon which is signaling the user that we are doing some kind of data fetching or saving in the background. If loading is set to false, we display a cloud icon which is signaling that everything is synced with the backend. These two icons are from fontawesome. We then display a user icon and the user name and next to it the logout button. We also put a div in front of our router view that is only showing when our axios interceptor got an Laravel error. It will then display all errors, so if you have multiple validation errors on your form it will list all of them. It also has a red cross icon that can close the message, else it will disappear after the next axios request. Lastly we changed our loading message for the entire page (after pressing F5 or when loading for the first time). Instead of the simple loading text we now use a bulma fullheight hero. This allows us to horizontally and vertically center the same fontawesome spinning icon we used before but this time a bit larger. This will indicate to the user that the page is trying to figure out if the user is authenticated or not.

And with this we have our application skeleton ready. All that's left is to create all necessary vue components for our dashboard and the board view.
We will need: The already existing Dashboard.vue which will display a list of all of the users boards, Board.vue which will be the main board view and it contains List.vue which is a single list on the board which contains Card.vue which is a single card in a given list and Modal.vue which will overlay a modal above the board view to show more details of a card. In our project the only detail shown here is it's description, but we could extend it with more features like tags, comments etc.

Lets start with the simplest of those, the Dashboard located in `resources/assets/js/components/Dashboard.vue`:

{% raw %}
```html
<template>
  <div>
    <h1>Your Boards</h1>
    <div class="flex-wrapper">
      <div 
        v-for="board in boards"
        :key="board.id" 
        class="flex-item board">

        <div class="card">
          <div class="card-content">
            <router-link :to="{ name: 'board_view', params: { id : board.id }}">{{ board.name }}</router-link>
            <span class="board-navs">
              <a @click.prevent="deleteThis(board)"><i class="fas fa-trash"/></a>
            </span>
          </div>
        </div>
      </div>

      <div class="flex-item">
        <form @submit.prevent>
          <input 
            v-model="name" 
            type="text" 
            class="input" 
            placeholder="New Board name"
            required>
          <button 
            class="button" 
            @click="createNew">Create</button>
        </form>
      </div>
    </div>
  </div>
</template>
<style scoped>
h1 {
    font-size: 1.75rem;
    margin-left: 10px;
}

.flex-wrapper {
    display: flex;
    flex-wrap: wrap;
}

.flex-item {
    flex: 0 0 auto;
    width: 270px;
    margin: 5px;
}

.board-navs {
  display: none;
  position: absolute;
  right: 12px;
  top: 25px;
}

.board:hover .board-navs {
  display: block;
}
</style>
<script>
import axios from 'axios';

export default {
    data() {
        return {
            boards: null,
            name: "",
        };
    },
    created() {
        this.fetchData();
    },
    methods: {
    fetchData() {
        axios
            .get('/boards')
            .then(response => {
                this.boards = response.data;
            }).catch(() => {});
    },
    createNew() {
        axios
            .post('/boards', { name: this.name })
            .then(response => {
                this.boards.push(response.data);
            }).catch(() => {});
    },
    deleteThis: function(board) {
        axios
            .delete('/boards/' + board.id)
            .then(() => {
                this.boards.splice(this.boards.indexOf(board), 1);
            }).catch(() => {});
    }
}
}
</script>
```
{% endraw %}

When the component is created, we fetch the data, which is the index method of our BoardController. We list each board in a flexbox as a bulma card. Each card contains a router link to the board view and on hover shows a trash icon. When clicking the trash icon vue will execute the deleteThis method which will simply use the delete API endpoint. The last flexbox item is a form to create a new board. The component is very straightforward since we just have 3 crud methods implemented here with a bit of design.

The other 4 components are a bit more complex since they will include drag&drop and more crud functionality. We will look at them in the next part.

[eslint]: https://ddmler.github.io/linter/vue/javascript/2018/03/31/using-eslint-to-enforce-the-vue-js-style-guide.html
[stylelint]: https://ddmler.github.io/linter/css/2018/06/04/using-stylelint-to-enforce-the-sass-guidelines-project.html
[cs-fixer]: https://ddmler.github.io/laravel/linter/2018/03/12/using-php-cs-fixer-in-laravel.html
[npm-script]: https://ddmler.github.io/cli/npm/2018/03/14/quick-tip-simplify-command-line-scripts-with-npm-scripts.html
[source]: https://github.com/ddmler/boards
[part1]: https://ddmler.github.io/laravel/vue/2018/07/13/vue-spa-with-laravel-api-part-1.html
[part2]: https://ddmler.github.io/laravel/vue/2018/07/17/vue-spa-with-laravel-api-part-2.html
[part3]: https://ddmler.github.io/laravel/vue/2018/07/20/vue-spa-with-laravel-api-part-3.html
[part4]: https://ddmler.github.io/laravel/vue/2018/07/24/vue-spa-with-laravel-api-part-4.html
[part5]: https://ddmler.github.io/laravel/vue/2018/07/27/vue-spa-with-laravel-api-part-5.html
[part7]: https://ddmler.github.io/laravel/vue/2018/08/07/vue-spa-with-laravel-api-part-7.html
[part8]: https://ddmler.github.io/laravel/vue/2018/08/10/vue-spa-with-laravel-api-part-8.html
