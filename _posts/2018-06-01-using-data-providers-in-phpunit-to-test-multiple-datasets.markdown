---
layout: post
title:  "Using data providers in PHPUnit to test multiple datasets"
date:   2018-06-01 18:00:00 +0100
categories: php testing
description: How to use a data provider in PHPUnit to run the same test with multiple input datasets.
---

Sometimes you have to test your code the exact same way with different input values. Maybe you need to check edge cases, negative numbers as input or completely different data types as input. PHPUnit allows you to define data providers to make this simple. Instead of having to copy and paste the test code which is always the same you can just tell PHPUnit to run the same test with different values. For this to work we need to create a public method that returns an array or an iterable object with the test data. In this example we want to pass in 3 values per test: 2 inputs and the expected output. In the following example the testAdd test will be run 4 times each time with a different input from the additionProvider. PHPUnit will provide each element as an argument so the order in our data provider is: a, b, expected.

```php
class AddTest extends TestCase
{
    /**
     * @dataProvider additionProvider
     */
    public function testAdd($a, $b, $expected)
    {
        $this->assertSame($expected, $a + $b);
    }

    public function additionProvider()
    {
        return [
            [0, 0, 0],
            [0, 1, 1],
            [0, -1, -1],
            [true, false, 1]
        ];
    }
}
```

If however any of the test cases fail PHPUnit will just output the array element which failed and to make it easier to understand what broke you can name each dataset and PHPUnit will output that name with the error message:

```php
class AddTest extends TestCase
{
    /**
     * @dataProvider additionProvider
     */
    public function testAdd($a, $b, $expected)
    {
        $this->assertSame($expected, $a + $b);
    }

    public function additionProvider()
    {
        return [
            "0 + 0 = 0" => [0, 0, 0],
            "0 + 1 = 1" => [0, 1, 1],
            "0 + -1 = -1" => [0, -1, -1],
            "true + false = 1" => [true, false, 1]
        ];
    }
}
```

