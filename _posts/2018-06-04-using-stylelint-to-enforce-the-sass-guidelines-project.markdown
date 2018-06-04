---
layout: post
title:  "Using stylelint to enforce the Sass Guidelines project"
date:   2018-06-04 18:00:00 +0100
categories: linter css
description: How to set up stylelint to automatically fix issues with your Sass coding style.
---

[Stylelint][stylelint] is a linter for CSS (and CSS-like syntaxes like Sass). It is designed similar to ESLint, so if you have worked with that stylelint will be very easy to pick up for you. For this little guide we want to use a plugin to enforce the [Sass Guidelines defined here][styleguide].

The first step is to install it using npm:

`npm install --save-dev stylelint stylelint-config-sass-guidelines`

Next create a .stylelintrc file with this content:
```json
{
  "extends": "stylelint-config-sass-guidelines"
}
```

You can also add a rules object to override specific rules. Now with running the following command:

`./node_modules/.bin/stylelint "resources/assets/sass/*.scss" --fix`

Stylelint will automatically enumerate all scss files in this directory and fix all your code to adhere to the Sass guidelines. You will have to use quotes so that your shell won't enumerate the files in that directory but stylelint will.

Check the [GitHub Repo][github] for the plugin to see what rules it configures.

[styleguide]: https://sass-guidelin.es/
[github]: https://github.com/bjankord/stylelint-config-sass-guidelines
[stylelint]: https://stylelint.io/
