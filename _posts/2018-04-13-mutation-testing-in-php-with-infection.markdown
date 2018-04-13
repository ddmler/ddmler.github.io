---
layout: post
title:  "Mutation testing in PHP with Infection"
date:   2018-04-13 18:00:00 +0100
categories: mutation testing php
description: Using the Infection testing framework to generate small mutations of your source code to evaluate your test suite.
---

Mutation testing is used to evaluate the quality of your existing software tests and help you to find ways to improve those tests. For this purpose the mutation framework introduces small changes to your source code and sees how your test suite reacts to that. Examples of small changes could be: Changing a plus to a minus, flipping a conditional or changing the visibility from public to protected. [Infection][infection] is a mutation testing framework for PHP. So let's have a look at it, the easiest way to use it is with a docker image:

`docker pull dockerizedphp/infection`

You can now run it in any project you want with this command in the project root directory:

`docker run --rm -ti -v $PWD:/app dockerizedphp/infection run`

It will ask a series of questions to create a configuration file. For your first run the defaults are probably ok.

```
Generate mutants...

Processing source code files: 1/1
Creating mutated files and processes: 86/86
.: killed, M: escaped, S: uncovered, E: fatal error, T: timed out

M......M..........................MM.....TT....M..   (50 / 86)
......MM..M.....M......M......M.MMM.                 (86 / 86)

86 mutations were generated:
      70 mutants were killed
       0 mutants were not covered by tests
      14 covered mutants were not detected
       0 errors were encountered
       2 time outs were encountered

Metrics:
         Mutation Score Indicator (MSI): 84%
         Mutation Code Coverage: 100%
         Covered Code MSI: 84%

Please note that some mutants will inevitably be harmless (i.e. false positives).
```

It will first run your test suite to make sure you don't have any errors at the moment.
We can see that we provided 1 source code file and Infection generated 86 different mutated versions of that. Your test suite will be run on each mutant and there are 5 possible results:
- The mutant is killed, which means some test failed
- The mutant escaped, which means no test failed
- The mutant is uncovered, which means no test covered this mutant
- A fatal error occured
- Timeout

The results tell us that 14 mutants were not detected so let's have a look:


```
1) /app/src/VerbalExpressions.php:29    [M] PublicVisibility
exec /usr/bin/php7 -c /tmp/infectionGKELeA  /app/vendor/phpunit/phpunit/phpunit --configuration /tmp/infection/phpunitConfiguration.96c437cee18b9bd1c10d68de1314ef52.infection.xml --stop-on-failure
--- Original
+++ New
@@ @@
      */
-    public static function sanitize($value)
+    protected static function sanitize($value)
     {
         return $value ? preg_quote($value, '/') : $value;
     }
     /**
      * Add
      *
PHPUnit 7.0.3 by Sebastian Bergmann and contributors.

..............................................                    46 / 46 (100%)

Time: 30 ms, Memory: 4.00MB

OK (46 tests, 180 assertions)
```

As we can see the mutant has changed the visibility of the sanitize method from public to protected and no test failed. The developer has to decide if this is a false positive or not.

```
11) /app/src/VerbalExpressions.php:543    [M] LessThan
exec /usr/bin/php7 -c /tmp/infectionGKELeA  /app/vendor/phpunit/phpunit/phpunit --configuration /tmp/infection/phpunitConfiguration.a94d68e187104b63e0b0deeef38633b4.infection.xml --stop-on-failure
--- Original
+++ New
@@ @@
             $value = '{' . $min . '}';
-        } elseif ($max < $min) {
+        } elseif ($max <= $min) {
             $value = '{' . $min . ',}';
         } else {
             $value = '{' . $min . ',' . $max . '}';
         }
         // check if the expression has * or + for the last expression
         if (preg_match('/\\*|\\+/', $this->lastAdded)) {
PHPUnit 7.0.3 by Sebastian Bergmann and contributors.

..............................................                    46 / 46 (100%)

Time: 30 ms, Memory: 4.00MB

OK (46 tests, 180 assertions)
```

In this case a `<` was changed to `<=` and no test failed, which means there is no test that checks what happens if min and max are equal here. A quick look at the test suite confirms that each case is tested at least once except the one where min and max are equal.

There is a list of all possible mutators the Infection framework offers [here][mutators].

And if you want to run Infection on the same codebase as I did, you can find it [here][expressions].

[infection]: https://github.com/infection/infection
[mutators]: https://infection.github.io/guide/mutators.html
[expressions]: https://github.com/VerbalExpressions/PHPVerbalExpressions
