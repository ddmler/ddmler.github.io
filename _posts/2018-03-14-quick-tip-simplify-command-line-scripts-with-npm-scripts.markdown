---
layout: post
title:  "Quick Tip: Simplify command line scripts with npm scripts"
date:   2018-03-14 12:00:00 +0100
categories: cli npm
description: NPM scripts offers an easy way to run complex scripts with a simple command.
---

If you find yourself typing a specific command a lot you may want to simplify it by using a feature of npm (or another package manager like composer) that allows you to define scripts to run, effectively giving them a simple alias that is easier to remember. To define a script just add a `scripts` object to your package.json file and define some scripts you want to be able to run. An example could look like this:

```json
{
  "scripts": {
    "eslint": "./node_modules/.bin/eslint resources/assets/js/ --fix",
    "stylelint": "./node_modules/.bin/stylelint \"resources/assets/sass/*.scss\" --fix",
    "ci": "npm run eslint && npm run stylelint"
  }
}
```

This will define 3 scripts: `eslint`, `stylelint` and `ci`. You can run a script by just using: `npm run ci` to run the ci command. Note that the ci command itself uses the two other npm scripts. What I like to do is add a `ci` command to all my projects that I can run before committing code that will run all the tests, linters and whatever else is useful. This means I always just have to run one simple command to get everything ready to commit. You can then let your CI tool run the npm scripts, which means you can always execute the current CI process easily on your machine.

Some script names have a special meaning and get executed automatically when a specific action occurs, for example a script with the name `install` will run automatically after the package is installed. You can check the [NPM documentation][npm-docs] to find out which names trigger automatic executions under which circumstances.


[npm-docs]: https://docs.npmjs.com/misc/scripts
