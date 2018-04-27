---
layout: post
title:  "TravisCI quick setup and some advanced features"
date:   2018-04-27 18:00:00 +0100
categories: ci
description: How to quickly get TravisCI to run your continuous integration process for GitHub projects and a short overview of it's features.
---

TravisCI for GitHub projects makes CI so easy you won't want to change to any other service after using it once. The first step is to go to the [Travis website][travis] and login using your GitHub account. Here you will see a list of all your repositories with a switch in front of each. For the repository that you want to use Travis for flip the switch to on.

The next step is to create a `travis.yml` file in your project root directory. To start keep it simple, an example for PHP could be:

```yaml
language: php

php:
- 7.0
- 7.1
- 7.2

before_script:
- composer install --no-interaction

script:
- vendor/bin/phpunit
```

With your next git push the build is automatically triggered and will run your script (phpunit in this case) for all php versions you chose.

And that's already it. Travis will now build for each push or pull request and signal the result in GitHub or in more detail on travis.

Now lets have a look at some more features Travis offers.

```yaml
deploy:
  provider: heroku
  api_key:
    secure: l92erYktjkDv+Rc8N79dvq8vT+g9GEI/wTJGOp7RhM9s1LhV4hk8eBJQ1W+/YMT3a7pbpgGIubms70gQ+3iYEivhrY4Wpe8Ty8Vzq7+7aUehtOJ2Kbi74q2OCXXDIbNcjLzoH99z6wbOGMzHTXt0aLlGawyE3sBfexmVpqXOZwH/0EqCR92lx3d2xCPVHEnYgDUcyOX+gVO79CPWd/T/E9ECRVjM0XCJkIQjWNzoi/SpS+g1mH1d3W4VG1/A+h0VnHP7nJ341nsLdnt/HRjLze5J20bJKqS3Z3PmclikMwtm6MNYhKcwsmp4vbeNTKXci+lHSTsQaZ56yUNlTXwVkvCQ15JI0s1mvgZFjJDBZL5QqX9g1UnhT5BB118lHj978OodnVql0cqQIxZAGB37oVZnakvbgETJbIIIIe/718NhfJlewPBpf6DhflsNg1wcn6BTAp4QT+jIRSI1DgvXErP7t2YYjVKTxAugMtjle8vP3frEVPeE+IC16oVBsCE/0YuJIDVmV3c6dF1AMvCR/mJkJ6BcqZgIxH3cXhrwhlSohEULehSmAnGQEotci85yPmbMjGoVZaiLP49g8CRWttAepGbD7mE8RNtRFE8NAsAlgrmYCLiuj6/VZAG3l1dpZK/qK4lmfgIlKhZeQCLcLlX8Q8CIjbtSYf4gPuuiw80=
  app: tutoringup
```

Travis also offers the ability to automatically deploy to a given provider if your build is successful. It supports more than 30 different providers to deploy to, in this example heroku. The api_key token is encrypted with the help of travis cli tools.


```yaml
notifications:
  email:
    recipients:
      - you@example.com
    on_success: never
    on_failure: always
```

Another option is to configure notifications. This example sends an email whenever the build was not successful. There are also a lot of other options supported like Slack instead of email.

```yaml
services:
- postgresql
```

Travis also provides services like for example databases. This will provide the psql command line tool in travis various scripts sections and you can easily test your code with a real database. How to do that exactly will be the content of a new blog post soon.

```yaml
matrix:
    include:
        - php: 7.1
          env: dbdep=mysql
        - php: 7.1
          env: dbdep=postgres

    fast_finish: true
```

The matrix lets you define more granularly what version to test and lets you add environment variables. These environment variables can then be used to for example install specific dependencies. The example would test the same PHP version twice with different databases, in case you need to test both for your project. The fast finish option lets the build fail as soon as one version fails.

There are a lot more features Travis offers but I think those are the most useful ones for now.

[travis]: https://travis-ci.org
