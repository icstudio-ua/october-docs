# Query Builder

::: aside
The query builder uses PDO parameter binding to protect your application against SQL injection attacks. There is no need to clean strings being passed as bindings.
:::

The database query builder provides a convenient, fluent interface to creating and running database queries. It can be used to perform most database operations in your application, and works on all supported database systems.

### Raw SQL Example

Sometimes it makes more sense to perform a query using plain SQL, you can do this with the `Db::select` method.

```php
Db::select('select * from sometable where name = :name', ['name' => 'Charles']);
```

See the article on [running raw SQL queries](./basics.md) for more information.

## Retrieving Results

To begin a fluent query, use the `table` method on the `Db` facade. The `table` method returns a fluent query builder instance for the given table, allowing you to chain more constraints onto the query and then finally get the results. In this example, `get` will return all records from a table.

```php
$users = Db::table('users')->get();
```

Akin to [raw queries](./basics.md), the `get` method returns an `array` of results where each result is an instance of the PHP `stdClass` object. You may access each column's value by accessing the column as a property of the object.

```php
foreach ($users as $user) {
    echo $user->name;
}
```

If you just need to retrieve a single row from the database table, you may use the `first` method. This method will return a single `stdClass` object.

```php
$user = Db::table('users')->where('name', 'John')->first();

echo $user->name;
```

### Plucking Values

If you don't even need an entire row, you may extract a single value from a record using the `value` method. This method will return the value of the column directly:

```php
$email = Db::table('users')->where('name', 'John')->value('email');
```

If you would like to retrieve an array containing the values of a single column, you may use the `pluck` method. In this example, we'll retrieve an array of role titles.

```php
$titles = Db::table('roles')->pluck('title');

foreach ($titles as $title) {
    echo $title;
}
```

You may also specify a custom key column for the returned array.

```php
$roles = Db::table('roles')->pluck('title', 'name');

foreach ($roles as $name => $title) {
    echo $title;
}
```

::: warning
The result from `pluck` will return `Collection` object to behaves like an array. Use the `all()` method to convert it to a PHP array, for example, `pluck('name')->all()`.
:::

### Chunking Results

If you need to work with thousands of database records, consider using the `chunk` method. This method retrieves a small "chunk" of the results at a time, and feeds each chunk into a `Closure` for processing. This method is very useful for writing [console commands](../console-commands.md) that process thousands of records. For example, let's work with the entire `users` table in chunks of 100 records at a time.

```php
Db::table('users')->chunk(100, function($users) {
    foreach ($users as $user) {
        //
    }
});
```

You may stop further chunks from being processed by returning `false` from the `Closure`.

```php
Db::table('users')->chunk(100, function($users) {
    // Process the records...

    return false;
});
```

If you are updating database records while chunking results, your chunk results could change in unexpected ways. So, when updating records while chunking, it is always best to use the `chunkById` method instead. This method will automatically paginate the results based on the record's primary key.

```php
Db::table('users')->where('active', false)
    ->chunkById(100, function ($users) {
        foreach ($users as $user) {
            Db::table('users')
                ->where('id', $user->id)
                ->update(['active' => true]);
        }
    });
```

::: warning
When updating or deleting records inside the chunk callback, any changes to the primary key or foreign keys could affect the chunk query. This could potentially result in records not being included in the chunked results.
:::

### Aggregate Functions

The query builder also provides a variety of aggregate methods, such as `count`, `max`, `min`, `avg`, and `sum`. You may call any of these methods after constructing your query:

```php
$users = Db::table('users')->count();

$price = Db::table('orders')->max('price');
```

Of course, you may combine these methods with other clauses to build your query:

```php
$price = Db::table('orders')
    ->where('is_finalized', 1)
    ->avg('price');
```

Instead of using the `count` method to determine if any records exist that match your query's constraints, you may use the `exists` and `doesntExist` methods:

```php
return Db::table('orders')->where('finalized', 1)->exists();

return Db::table('orders')->where('finalized', 1)->doesntExist();
```

## Select Statements

In some cases you may not always want to select all columns from a database table. Using the `select` method, you can specify a custom `select` clause for the query.

```php
$users = Db::table('users')->select('name', 'email as user_email')->get();
```

The `distinct` method allows you to force the query to return distinct results.

```php
$users = Db::table('users')->distinct()->get();
```

If you already have a query builder instance and you wish to add a column to its existing select clause, you may use the `addSelect` method:

```php
$query = Db::table('users')->select('name');

$users = $query->addSelect('age')->get();
```

### Raw Expressions

Sometimes you may need to use a raw expression in a query. To create a raw expression, you may use the `Db::raw` method.

```php
$users = Db::table('users')
    ->select(Db::raw('count(*) as user_count, status'))
    ->where('status', '<>', 1)
    ->groupBy('status')
    ->get();
```

Another use might be to concatenate columns and/or strings together.

```php
Db::raw("(first_name, ' ', last_name) as full_name");
```

::: warning
Raw statements will be injected into the query as strings, so you should be extremely careful to not create SQL injection vulnerabilities.
:::

### Raw Methods

Instead of using `Db::raw`, you may also use the following methods to insert a raw expression into various parts of your query.

The `selectRaw` method can be used in place of `addSelect(Db::raw(...)).` This method accepts an optional array of bindings as its second argument:

```php
$orders = Db::table('orders')
    ->selectRaw('price * ? as price_with_tax', [1.0825])
    ->get();
```

The `whereRaw` and `orWhereRaw` methods can be used to inject a raw `where` clause into your query. These methods accept an optional array of bindings as their second argument:

```php
$orders = Db::table('orders')
    ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
    ->get();
```

The `havingRaw` and `orHavingRaw` methods may be used to set a raw string as the value of the `having` clause. These methods accept an optional array of bindings as their second argument:

```php
$orders = Db::table('orders')
    ->select('department', Db::raw('SUM(price) as total_sales'))
    ->groupBy('department')
    ->havingRaw('SUM(price) > ?', [2500])
    ->get();
```

The `orderByRaw` method may be used to set a raw string as the value of the order by clause:

```php
$orders = Db::table('orders')
    ->orderByRaw('updated_at - created_at DESC')
    ->get();
```

The `groupByRaw` method may be used to set a raw string as the value of the group by clause:

```php
$orders = Db::table('orders')
    ->select('city', 'state')
    ->groupByRaw('city, state')
    ->get();
```

## Joins

The query builder may also be used to write join statements. To perform a basic SQL "inner join", you may use the `join` method on a query builder instance. The first argument passed to the `join` method is the name of the table you need to join to, while the remaining arguments specify the column constraints for the join. Of course, as you can see, you can join to multiple tables in a single query.

```php
$users = Db::table('users')
    ->join('contacts', 'users.id', '=', 'contacts.user_id')
    ->join('orders', 'users.id', '=', 'orders.user_id')
    ->select('users.*', 'contacts.phone', 'orders.price')
    ->get();
```

If you would like to perform a "left join" or "right join" instead of an "inner join", use the `leftJoin` or `rightJoin` method. The `leftJoin` and `rightJoin` methods have the same signature as the `join` method.

```php
$users = Db::table('users')
    ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
    ->get();

$users = Db::table('users')
    ->rightJoin('posts', 'users.id', '=', 'posts.user_id')
    ->get();
```

To perform a "cross join" use the `crossJoin` method with the name of the table you wish to cross join to. Cross joins generate a cartesian product between the first table and the joined table.

```php
$users = Db::table('sizes')
    ->crossJoin('colors')
    ->get();
```

### Advanced Join Statements

You may also specify more advanced join clauses. To get started, pass a `Closure` as the second argument into the `join` method. The `Closure` will receive a `JoinClause` object which allows you to specify constraints on the `join` clause.

```php
Db::table('users')
    ->join('contacts', function ($join) {
        $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
    })
    ->get();
```

If you would like to use a "where" style clause on your joins, you may use the `where` and `orWhere` methods on a join. Instead of comparing two columns, these methods will compare the column against a value.

```php
Db::table('users')
    ->join('contacts', function ($join) {
        $join->on('users.id', '=', 'contacts.user_id')
            ->where('contacts.user_id', '>', 5);
    })
    ->get();
```

### Subquery Joins

You may use the `joinSub`, `leftJoinSub`, and `rightJoinSub` methods to join a query to a subquery. Each of these methods receive three arguments: the subquery, its table alias, and a Closure that defines the related columns:

```php
$latestPosts = Db::table('posts')
    ->select('user_id', Db::raw('MAX(created_at) as last_post_created_at'))
    ->where('is_published', true)
    ->groupBy('user_id');

$users = Db::table('users')
    ->joinSub($latestPosts, 'latest_posts', function ($join) {
        $join->on('users.id', '=', 'latest_posts.user_id');
    })->get();
```

### Unions

The query builder also provides a quick way to "union" two queries together. For example, you may create an initial query, and then use the `union` method to union it with a second query.

```php
$first = Db::table('users')
    ->whereNull('first_name');

$users = Db::table('users')
    ->whereNull('last_name')
    ->union($first)
    ->get();
```

The `unionAll` method is also available and has the same method signature as `union`.

## Where Clauses

To add `where` clauses to the query, use the `where` method on a query builder instance. The most basic call to `where` requires three arguments. The first argument is the name of the column. The second argument is an operator, which can be any of the database's supported operators. The third argument is the value to evaluate against the column.

For example, here is a query that verifies the value of the "votes" column is equal to 100.

```php
$users = Db::table('users')->where('votes', '=', 100)->get();
```

For convenience, if you simply want to verify that a column is equal to a given value, you may pass the value directly as the second argument to the `where` method.

```php
$users = Db::table('users')->where('votes', 100)->get();
```

You may use a variety of other operators when writing a `where` clause, for example, greater than `>`, doesn't equal `<>` and `like`.

```php
$users = Db::table('users')
    ->where('votes', '>=', 100)
    ->get();

$users = Db::table('users')
    ->where('votes', '<>', 100)
    ->get();

$users = Db::table('users')
    ->where('name', 'like', 'T%')
    ->get();
```

### Or Statements

You may chain where constraints together, as well as add `or` clauses to the query. The `orWhere` method accepts the same arguments as the `where` method:

```php
$users = Db::table('users')
    ->where('votes', '>', 100)
    ->orWhere('name', 'John')
    ->get();
```

::: tip
You can also prefix `or` to any of the where statements methods below, to make the condition an "OR" condition - for example, `orWhereBetween`, `orWhereIn`, etc.
:::

### More Where Statements

The `whereBetween` method verifies that a column's value is between two values.

```php
$users = Db::table('users')
    ->whereBetween('votes', [1, 100])->get();
```

The `whereNotBetween` method verifies that a column's value lies outside of two values.

```php
$users = Db::table('users')
    ->whereNotBetween('votes', [1, 100])
    ->get();
```

The `whereIn` method verifies that a given column's value is contained within the given array.

```php
$users = Db::table('users')
    ->whereIn('id', [1, 2, 3])
    ->get();
```

The `whereNotIn` method verifies that the given column's value is **not** contained in the given array.

```php
$users = Db::table('users')
    ->whereNotIn('id', [1, 2, 3])
    ->get();
```

The `whereNull` method verifies that the value of the given column is `NULL`.

```php
$users = Db::table('users')
    ->whereNull('updated_at')
    ->get();
```

The `whereNotNull` method verifies that the column's value is **not** `NULL`.

```php
$users = Db::table('users')
    ->whereNotNull('updated_at')
    ->get();
```

## Compound Where Clauses

Sometimes you may need to create more advanced where clauses such as "where exists" or nested parameter groupings. The Laravel query builder can handle these as well. To get started, let's look at an example of grouping constraints within parenthesis:

```php
Db::table('users')
    ->where('name', '=', 'John')
    ->orWhere(function ($query) {
        $query->where('votes', '>', 100)
            ->where('title', '<>', 'Admin');
    })
    ->get();
```

As you can see, passing `Closure` into the `orWhere` method instructs the query builder to begin a constraint group. The `Closure` will receive a query builder instance which you can use to set the constraints that should be contained within the parenthesis group. The example above will produce the following SQL:

```sql
select * from users where name = 'John' or (votes > 100 and title <> 'Admin')
```

### Exists Statements

The `whereExists` method allows you to write `where exist` SQL clauses. The `whereExists` method accepts a `Closure` argument, which will receive a query builder instance allowing you to define the query that should be placed inside of the "exists" clause:

```php
Db::table('users')
    ->whereExists(function ($query) {
        $query->select(Db::raw(1))
            ->from('orders')
            ->whereRaw('orders.user_id = users.id');
    })
    ->get();
```

The query above will produce the following SQL:

```sql
select * from users where exists (
    select 1 from orders where orders.user_id = users.id
)
```

### JSON Where Statements

October CMS also supports querying JSON column types on databases that provide support for JSON column types. To query a JSON column, use the `->` operator.

```php
$users = Db::table('users')
    ->where('options->language', 'en')
    ->get();

$users = Db::table('users')
    ->where('preferences->dining->meal', 'salad')
    ->get();
```

You may use `whereJsonContains` to query JSON arrays (not supported on SQLite).

```php
$users = Db::table('users')
    ->whereJsonContains('options->languages', 'en')
    ->get();
```

MySQL and PostgreSQL support `whereJsonContains` with multiple values.

```php
$users = Db::table('users')
    ->whereJsonContains('options->languages', ['en', 'de'])
    ->get();
```

You may use `whereJsonLength` to query JSON arrays by their length.

```php
$users = Db::table('users')
    ->whereJsonLength('options->languages', 0)
    ->get();

$users = Db::table('users')
    ->whereJsonLength('options->languages', '>', 1)
    ->get();
```

### Conditional Clauses

Sometimes you may want clauses to apply to a query only when something else is true. For instance you may only want to apply a `where` statement if a given input value is present on the incoming request. You may accomplish this using the `when` method:

```php
$role = $request->input('role');

$users = Db::table('users')
    ->when($role, function ($query, $role) {
        return $query->where('role_id', $role);
    })
    ->get();
```

The `when` method only executes the given Closure when the first parameter is `true`. If the first parameter is `false`, the Closure will not be executed.

You may pass another Closure as the third parameter to the `when` method. This Closure will execute if the first parameter evaluates as false. To illustrate how this feature may be used, we will use it to configure the default sorting of a query:

```php
$sortBy = null;

$users = Db::table('users')
    ->when($sortBy, function ($query, $sortBy) {
        return $query->orderBy($sortBy);
    }, function ($query) {
        return $query->orderBy('name');
    })
    ->get();
```

## Order, Group, Limit

### Ordering

The `orderBy` method allows you to sort the result of the query by a given column. The first argument to the `orderBy` method should be the column you wish to sort by, while the second argument controls the direction of the sort and may be either `asc` or `desc`.

```php
$users = Db::table('users')
    ->orderBy('name', 'desc')
    ->get();
```

The `latest` and `oldest` methods allow you to easily order results by date. By default, result will be ordered by the `created_at` column. Or, you may pass the column name that you wish to sort by.

```php
$user = Db::table('users')
    ->latest()
    ->first();
```

The `inRandomOrder` method may be used to sort the query results randomly. For example, you may use this method to fetch a random user.

```php
$randomUser = Db::table('users')
    ->inRandomOrder()
    ->first();
```

### Grouping

The `groupBy` and `having` methods may be used to group the query results. The `having` method's signature is similar to that of the `where` method.

```php
$users = Db::table('users')
    ->groupBy('account_id')
    ->having('account_id', '>', 100)
    ->get();
```

You may pass multiple arguments to the `groupBy` method to group by multiple columns.

```php
$users = Db::table('users')
    ->groupBy('first_name', 'status')
    ->having('account_id', '>', 100)
    ->get();
```

For more advanced `having` statements, you may wish to use the `havingRaw` method.

### Limit and Offset

To limit the number of results returned from the query, or to skip a given number of results in the query (`OFFSET`), you may use the `skip` and `take` methods.

```php
$users = Db::table('users')->skip(10)->take(5)->get();
```

## Inserts

The query builder also provides an `insert` method for inserting records into the database table. The `insert` method accepts an array of column names and values to insert:

```php
Db::table('users')->insert(
    ['email' => 'john@example.com', 'votes' => 0]
);
```

You may even insert several records into the table with a single call to `insert` by passing an array of arrays. Each array represents a row to be inserted into the table:

```php
Db::table('users')->insert([
    ['email' => 'taylor@example.com', 'votes' => 0],
    ['email' => 'dayle@example.com', 'votes' => 0]
]);
```

### Auto-Incrementing IDs

If the table has an auto-incrementing id, use the `insertGetId` method to insert a record and then retrieve the ID:

```php
$id = Db::table('users')->insertGetId(
    ['email' => 'john@example.com', 'votes' => 0]
);
```

::: tip
When using the PostgreSQL database driver, the insertGetId method expects the auto-incrementing column to be named `id`. If you would like to retrieve the ID from a different "sequence", you may pass the sequence name as the second parameter to the `insertGetId` method.
:::

## Updates

In addition to inserting records into the database, the query builder can also update existing records using the `update` method. The `update` method, like the `insert` method, accepts an array of column and value pairs containing the columns to be updated. You may constrain the `update` query using `where` clauses.

```php
Db::table('users')
    ->where('id', 1)
    ->update(['votes' => 1]);
```

### Update or Insert

Sometimes you may want to update an existing record in the database or create it if no matching record exists. In this scenario, the `updateOrInsert` method may be used. The `updateOrInsert` method accepts two arguments: an array of conditions by which to find the record, and an array of column and value pairs containing the columns to be updated.

The `updateOrInsert` method will first attempt to locate a matching database record using the first argument's column and value pairs. If the record exists, it will be updated with the values in the second argument. If the record can not be found, a new record will be inserted with the merged attributes of both arguments.

```php
Db::table('users')
    ->updateOrInsert(
        ['email' => 'john@example.com', 'name' => 'John'],
        ['votes' => '2']
    );
```

### Upsert

The `upsert` method will insert rows that do not exist and update the rows that already exist with the new values. The method's first argument consists of the values to insert or update, while the second argument lists the column(s) that uniquely identify records within the associated table. The method's third and final argument is an array of columns that should be updated if a matching record already exists in the database.

```php
Db::table('flights')->upsert([
    ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
    ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
], ['departure', 'destination'], ['price']);
```

::: tip
All databases except SQL Server require the columns in the second argument of the `upsert` method to have a "primary" or "unique" index.
:::

### Updating JSON columns

When updating a JSON column, you should use `->` syntax to access the appropriate key in the JSON object. This operation is supported on MySQL 5.7+ and PostgreSQL 9.5+:

```php
$affected = Db::table('users')
    ->where('id', 1)
    ->update(['options->enabled' => true]);
```

### Increment / Decrement

The query builder also provides convenient methods for incrementing or decrementing the value of a given column. This is simply a short-cut, providing a more expressive and terse interface compared to manually writing the `update` statement.

Both of these methods accept at least one argument: the column to modify. A second argument may optionally be passed to control the amount by which the column should be incremented / decremented.

```php
Db::table('users')->increment('votes');

Db::table('users')->increment('votes', 5);

Db::table('users')->decrement('votes');

Db::table('users')->decrement('votes', 5);
```

You may also specify additional columns to update during the operation:

```php
Db::table('users')->increment('votes', 1, ['name' => 'John']);
```

## Deletes

The query builder may also be used to delete records from the table via the `delete` method.

```php
Db::table('users')->delete();
```

You may constrain `delete` statements by adding `where` clauses before calling the `delete` method.

```php
Db::table('users')->where('votes', '<', 100)->delete();
```

If you wish to truncate the entire table, which will remove all rows and reset the auto-incrementing ID to zero, you may use the `truncate` method.

```php
Db::table('users')->truncate();
```

## Pessimistic Locking

The query builder also includes a few functions to help you do "pessimistic locking" on your `select` statements. To run the statement with a "shared lock", you may use the `sharedLock` method on a query. A shared lock prevents the selected rows from being modified until your transaction commits.

```php
Db::table('users')->where('votes', '>', 100)->sharedLock()->get();
```

Alternatively, you may use the `lockForUpdate` method. A "for update" lock prevents the rows from being modified or from being selected with another shared lock.

```php
Db::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
```

## Caching Queries

You may easily cache the results of a query using the [Cache service](../services/cache.md). Simply chain the `remember` or `rememberForever` method when preparing the query.

```php
$users = Db::table('users')->remember(10)->get();
```

In this example, the results of the query will be cached for ten minutes. While the results are cached, the query will not be run against the database, and the results will be loaded from the default cache driver specified for your application.

## Debugging

You may use the `dd` or `dump` methods while building a query to dump the query bindings and SQL. The `dd` method will display the debug information and then stop executing the request. The `dump` method will display the debug information but allow the request to keep executing:

```php
Db::table('users')->where('votes', '>', 100)->dd();

Db::table('users')->where('votes', '>', 100)->dump();
```
