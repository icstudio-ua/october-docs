# Basic Usage

Connecting to databases and running queries is a simple process, supported by using either raw SQL, the [query builder](./query.md) or [active record models](./model.md). Managing database tables and populating seed data is handled by the [migration and seeder process](./structure.md).

Raw SQL and using the query builder will perform faster and should be used for simple tasks. Active Record is an approach used by the popular framework, Ruby On Rails. It allows an easy interface for performing repetitive tasks like creating, reading, updating and deleting database records. You can learn more about the [active record pattern on Wikipedia](http://en.wikipedia.org/wiki/Active_record_pattern).

## Running Raw SQL Queries

Once you have configured your database connection, you may run queries using the `Db` facade. The `Db` facade provides methods for each type of query: `select`, `update`, `insert`, `delete`, and `statement`.

### Selecting Records

To run a basic query, use the `select` method on the `Db` facade.

```php
$users = Db::select('select * from users where active = ?', [1]);
```

The first argument passed to the `select` method is the raw SQL query, while the second argument is any parameter bindings that need to be bound to the query. Typically, these are the values of the `where` clause constraints. Parameter binding provides protection against SQL injection.

The `select` method will always return an `array` of results. Each result within the array will be a PHP `stdClass` object, allowing you to access the values of the results:

```php
foreach ($users as $user) {
    echo $user->name;
}
```

Instead of using `?` to represent your parameter bindings, you may execute a query using named bindings.

```php
$results = Db::select('select * from users where id = :id', ['id' => 1]);
```

### Modifying Records

To execute an `insert` statement, you may use the `insert` method on the `Db` facade. Like `select`, this method takes the raw SQL query as its first argument and bindings as the second argument.

```php
Db::insert('insert into users (id, name) values (?, ?)', [1, 'Joe']);
```

The `update` method should be used to update existing records in the database. The number of rows affected by the statement will be returned by the method:

```php
$affected = Db::update('update users set votes = 100 where name = ?', ['John']);
```

The `delete` method should be used to delete records from the database. Like `update`, the number of rows deleted will be returned:

```php
$deleted = Db::delete('delete from users');
```

### General Statements

Some database statements should not return any value. For these types of operations, you may use the `statement` method on the `Db` facade.

```php
Db::statement('drop table users');
```

## Multiple Database Connections

When [using multiple connections](../../setup/database-config.md), you may access each connection via the `connection` method on the `Db` facade. The `name` passed to the `connection` method should correspond to one of the connections listed in your `config/database.php` configuration file.

```php
$users = Db::connection('foo')->select(...);
```

You may also access the raw, underlying PDO instance using the `getPdo` method on a connection instance.

```php
$pdo = Db::connection()->getPdo();
```

## Database Transactions

To run a set of operations within a database transaction, you may use the `transaction` method on the `Db` facade. If an exception is thrown within the transaction `Closure`, the transaction will automatically be rolled back. If the `Closure` executes successfully, the transaction will automatically be committed. You don't need to worry about manually rolling back or committing while using the `transaction` method.

```php
Db::transaction(function () {
    Db::table('users')->update(['votes' => 1]);

    Db::table('posts')->delete();
});
```

If you would like to begin a transaction manually and have complete control over rollbacks and commits, you may use the `beginTransaction` method on the `Db` facade:

```php
Db::beginTransaction();
```

You can rollback the transaction via the `rollBack` method:

```php
Db::rollBack();
```

Lastly, you can commit a transaction via the `commit` method:

```php
Db::commit();
```

::: tip
Using the `Db` facade's transaction methods also controls transactions for the [query builder](./query.md) and [model queries](./model.md).
:::

## Database Events

If you would like to receive each SQL query executed by your application, you may use the `listen` method. This method is useful for logging queries or debugging.

```php
Db::listen(function($sql, $bindings, $time) {
    //
});
```

You may register your query listener in the `boot` method of a [plugin registration file](../extending.md). Alternatively, plugins can supply a file named **init.php** in the plugin directory that you can use to place this logic.

### Query Logging

When query logging is enabled, a log is kept in memory of all queries that have been run for the current request. Call the `enableQueryLog` method to enable this feature.

```php
Db::connection()->enableQueryLog();
```

To get an array of the executed queries, you may use the `getQueryLog` method:

```php
$queries = Db::getQueryLog();
```

However, in some cases, such as when inserting a large number of rows, this can cause the application to use excess memory. To disable the log, you may use the `disableQueryLog` method:

```php
Db::connection()->disableQueryLog();
```

::: tip
For quicker debugging it may be more useful to call the `trace_sql` [helper function](../services/log.md) instead.
:::
