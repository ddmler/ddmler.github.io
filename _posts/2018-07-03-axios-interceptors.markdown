---
layout: post
title:  "Axios interceptors"
date:   2018-07-03 18:00:00 +0100
categories: js
description: Using axios interceptors to set reactive vue flags and automatically handle all Laravel errors.
---

In this post we want to use axios interceptors to set some reactive data in our Vue root object. We will set a loading flag which indicated if any axios request is currently executed, an error flag which indicates if we got some error as a response from laravel and an error object which contains the error messages. We then want to display 2 different icons: a spinning loading icon when loading is true and a cloud icon when loading is false (meaning everything is synchronized with the backend). For the error object we want to automatically display all errors when we any.

First we setup our Vue root element with the needed data object and a methods object which contains a method that will clear all errors and with that hiding the displayed errors again. We then mount Vue and define 2 axios interceptors the first will be called each time we start a request and the second will be called each time we get a response back. The first interceptor just sets loading to true so we see in our icons later that a request is taking place right now. The second interceptor takes 2 arguments: a callback for when the response is a success and a callback for when the response is an error (detected with the HTTP statuscode). If it's a success we want to set loading to false and don't display any errors. If it wasn't successful we again set loading to false and our error flag to true. The errors object will then get filled with one of two possibilities: either we have a validation error, or a standard error. Since both have different json we bring the standard errors into the same format as the validation errors are. We then reject the promise so that it fails.

```js
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
    vueApp.errors = {};
    vueApp.error = false;
    return config;
}, error => {
    vueApp.loading = false;
    vueApp.errors = error.response.data.errors || { "none": [error.response.data.message] };
    vueApp.error = true;
    return Promise.reject(error);
});
```

You could use axios now like this:

```js
axios
    .post('/cards', payload)
    .then(response => {
        // success
    }).catch(() => {});
```

As the catch callback we provide an empty function since all of the error handling is already done in the interceptor. You could however still do different error handling if you need it. Next let's use fontawesome icons to display a spinning loading icon while loading and a cloud icon while not loading:

```html
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
```

Pretty easy. You can use this snippet in every vue component since we access the root element to get the loading state. Next up is a display of all the error messages. We use bulma for styling but you don't have to. It will be a bulma danger message that lists each message in our errors array. So if you have multiple validation errors all will be listed here. If you have another Laravel error like model not found this will also be listed. We also add a cross icon to at the top right which uses the method (use $root in front if you use it in any other component than the root one):

```html
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
```

Lastly some css to position our close button and make it a bit bigger.

```css
.error-close {
  position: absolute;
  right: 25px;
}

.error-close i {
  font-size: 1.75rem;
}
```

Just handling all errors like this may not be a good idea, however you can also drop the other errors and only display validation errors. For this purpose this snippet is very powerful. The other errors can then be handled normally in the now empty catch callback.
