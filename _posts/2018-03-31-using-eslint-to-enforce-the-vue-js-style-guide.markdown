---
layout: post
title:  "Using ESLint to enforce the Vue.js Style Guide"
date:   2018-03-31 18:00:00 +0100
categories: linter vue javascript
description: Setting up and using ESLint + Vue.js Plugin to enforce the Vue.js Style Guide.
---

Setting up ESLint with the Vue.js plugin to enforce the official [Vue.js Style Guide][styleguide] just needs 2 simple steps and is done in a minute.

First run `npm install --save-dev eslint eslint-plugin-vue`.

Next create a file named `.eslintrc.js` in your project root directory
```js
module.exports = {
  extends: [
    //'eslint:recommended',
    'plugin:vue/recommended'
  ],
  rules: {
    // override/add rules settings here, such as:
    // 'vue/no-unused-vars': 'error'
  },
  env: {
    amd: true
  }
}
```

The amd setting is only needed if you want to use AMD style modules.

Now just run it like this: `./node_modules/.bin/eslint {dir} --fix --ext .vue` and replace dir with the directory you want to run it on to let the plugin fix all style guide violations it can.

Check [the plugins GitHub page][github] for which rules it can fix automatically.


[styleguide]: https://vuejs.org/v2/style-guide/
[github]: https://github.com/vuejs/eslint-plugin-vue
