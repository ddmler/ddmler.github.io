---
layout: post
title:  "Hot reloading and similar concepts in Laravel Mix"
date:   2018-06-15 18:00:00 +0100
categories: build
description: A short explanation of hot reloading and webpacks watch mode and how to use them in Laravel Mix.
---

When developing you often find yourself in a situation where you need to compile some things like Sass to CSS or Vue files to javascript and so on. To not have to use the command line every time after making a small change, we got `webpack --watch`, which will automatically recompile your changed files on each save. Now you just have to reload the page and do all the things you had done before to get to the same state of your page to see the changes. The logical next step was live reloading, in which a watch command is still running and recompiling everything, but on top of that it reloads the page automatically. However you still had to take all the actions again to get to the same state of the page. To solve this hot reloading or hot module replacement was created. Hot reloading will watch for changes and instead of reloading the page completely just swap out old components with the new components while conserving the state. An example to illustrate: If you had a counter and just made a change to the style of it, with hot reloading it would update the style and the counter would still be ticking at the current number, whereas with live reloading the counter would start again fresh.

One example where hot reloading is available is Laravel with Laravel Mix:

```json
"scripts": {
        "watch": "cross-env NODE_ENV=development node_modules/webpack/bin/webpack.js --watch --progress --hide-modules --config=node_modules/laravel-mix/setup/webpack.config.js",
        "watch-poll": "npm run watch -- --watch-poll",
        "hot": "cross-env NODE_ENV=development node_modules/webpack-dev-server/bin/webpack-dev-server.js --inline --hot --config=node_modules/laravel-mix/setup/webpack.config.js",
    }
```

As you can see Laravel Mix provides a watch command which uses `webpack --watch` internally and a hot command which provides the hot reloading.

You can run them with npm like this:

`npm run hot`

If you develop your app on HTTPS you need to add the `--https` flag to the hot command.

Also make sure to use the Laravel mix helper to get the correct url in the src attribute of your assets when including them in your page. This will make everything work automatically in development and production.

{% raw %}
```html
<script src="{{ mix('js/app.js') }}"></script>
```
{% endraw %}
