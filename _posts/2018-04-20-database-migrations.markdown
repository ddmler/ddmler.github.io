---
layout: post
title:  "Database migrations"
date:   2018-04-20 18:00:00 +0100
categories: database
description: What are database migrations and first steps in using them.
---

Database migrations are incremental, reversible changes to schemas of relational databases. This means a database migration has 2 functions it needs to provide: one to make a change to the schema and one to undo that change. With database migrations the database schema doesn't need to be designed up front but instead can evolve almost as easily as the source code. This works especially well in agile environments. That means database migrations are to the database schema what git is to the source code: a solution to the versioning problem. Whenever something needs to be changed in the database just create a new migration and you will have a complete history of the evolution of your database. Laravel includes migrations out of the box but there are other packages that provide this kind of functionality for php out there.

To get started:

`php artisan make:migration create_board_lists_table`

This command creates a file with boilerplate where we can define how the migration should look like. In this case we want to create a new table:

```php
class CreateBoardListsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('board_lists', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->integer('board_id')->unsigned();
            $table->timestamps();
            $table->foreign('board_id')->references('id')->on('boards')->onDelete('cascade');
        });
    }
    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('board_lists');
    }
}
```

When the migration is run, it will create a new table called board_lists and the columns of it are defined in the function. There is a method for probably everything you could express with SQL, a list can be found in the [Laravel documentation][docs]. The first three should be pretty self-explanatory. The timestamps method will create two columns called created_at and updated_at. The last line defines a foreign key that should reference the id column in the boards table.

To run all migrations that have not yet been run:
`php artisan migrate`

When developing I often find it useful to use the following command to drop all tables and run all migrations after that:
`php artisan migrate:fresh`

There are also commands to rollback all or some of your migrations if you don't want to drop all the tables.

In production you should be careful with the down method of the migration. If it is accidentally used all your data could be wiped. So you could make it impossible to run the down methods when the environment variable is set to production or some other way to disable them.

If you use migrations in combination with database seeders you can easily create a test database with test data. How to do that exactly will be the content of a different post soon.

[docs]: https://laravel.com/docs/5.6/migrations
