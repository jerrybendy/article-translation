# First Glimpse at CodeIgniter 4 Database Layer

> 原文: [First Glimpse at CodeIgniter 4 Database Layer](http://blog.newmythmedia.com/blog/show/2016-04-01_FirstGlimpseatCodeIgniter4DatabaseLayer)

While work on the database layer is still under heavy construction, I think we're far enough along to be able to give you a glimpse into how it works, how it's the same, and how it's different from what you're accustomed to in previous versions.

First things first: how far along is it? At the moment we can connect to a MySQL database and run both raw queries, and use the Query Builder to run queries. I just wrapped up tests on the existing Query Builder features, I believe, so it should be fairly solid at the moment. What's left? The Query Caching layer needs built, as does the Forge, and the utility methods, as well as getting the drivers in place and in shape.

## What's the Same?

While the underlying structure of the database engine has been changed a fair amount, what you'll see while using it will be fairly familiar. The biggest cosmetic difference is in method names using CamelCasing instead of snake_casing. The query builder still largely works like you're used to, so there won't be much to relearn. You should be able to dive right in and use your years of experience with just the tiniest amount of time getting accustomed to it.

## What's different?

I won't go into all of the details here, just the big items. Instead of a boring little list, let's take a look at a few examples of it in action.

### Configuration

The config files are still mostly like the old ones. There was no need to reinvent the wheel here since it worked great already. They have been changed to be a simple class, like the rest of them but the fields are the same.

```php
class Database extends \CodeIgniter\Database\Config
{
    /**
     * Lets you choose which connection group to
     * use if no other is specified.
     *
     * @var string
     */
    public $defaultGroup = 'default';

    /**
     * The default database connection.
     *
     * @var array
     */
    public $default = [
        'dsn'          => '',
        'hostname'     => 'localhost',
        'username'     => '',
        'password'     => '',
        'database'     => '',
        'dbdriver'     => 'MySQLi',
        'dbprefix'     => '',
        'pconnect'     => false,
        'db_debug'     => (ENVIRONMENT !== 'production'),
        'cache_on'     => false,
        'cachedir'     => '',
        'charset'      => 'utf8',
        'dbcollat'     => 'utf8_general_ci',
        'swapPre'      => '',
        'encrypt'      => false,
        'compress'     => false,
        'stricton'     => false,
        'failover'     => [],
        'saveQueries' => true,
    ];

    //--------------------------------------------------------------------

}
```

### Raw Queries

Making queries without using the Query Builder is simple. Get a database instance, run the `query()` method and get back a result object.

```php
// Connects to the default connection, 'default' in this example.
$db = Config\Database::connect();

$query = $db->query('SELECT * FROM users');

// Get results as objects.
$results = $query->getResultArray();
// Get results as arrays
$results = $query->getResultObject();
// Get result as custom class instances
$result = $query->getResult('\My\Class');
```

The first thing to note is that `num_rows()` has been removed. For the last few years it's use has been discouraged, and written out of examples, since some drivers have horrible performance and/or memory issues when using it. Instead, all `result*()` methods return empty arrays if no results, while all `row*()` methods return `null`.

Parameter binding still exists:

```php
$query = $db->query('SELECT * FROM users WHERE id > ? AND role = ?', [3, 'Admin']);
```

Parameter binding has gotten a new trick, though, with named parameters, for more readable (and flexible) queries:

```php
$query = $db->query('SELECT * FROM users WHERE id > :id AND role = :role',
    [ 'id'   => 3,
      'role' => Admin'
    ]
);
```

All values are automatically escaped, of course, to keep your queries safe.

### Saved Queries

One of the big changes in the database layer is that all queries are saved in history as Query objects, instead of strings in an array. This is partially to clean up the code and remove some resposibilities from other classes. But it will also allow for more flexibility in the Query Caching layer, and other places. Just be aware that if you need to get `$db->getLastQuery()` you're getting a Query object back, not a string.

The query objects hold the query string, which can be retrieved with and without the parameters bound to it, as well as any error information that query might have, and performance data (when it started, how long it took).

### Query Builder

The Query builder operates mostly as you're used to, with one big change. The Query Builder is now it's own class, not part of the driver files. This helps keep the code cleaner, and works nicely with the new Query objects and named paramater binding, which is used throughout the builder.

One of the biggest benefits of having it as a completely separate class is that it allows us to keep each query completely seperate. There is no more "query bleeding" where you're in the middle of building a query and make a call out to another method to retrieve some values, only to have the other method build it's own query, and incorrectly using the values from the original query. That's a thing of the past.

The primary visible change in the Query Builder is how you access the builder object. Since it's no longer part of the driver, we need a way to get an instance of it that is setup to work with the current driver. That's where the new `table()` method comes into play.

```php
$db = Config\Database::connect();

$result = $db->table('users')
             ->select('id, role')
             ->where('active', 1)
             ->get();
```

Basically, the main table is now specified in the `table()` method instead of the `get()` method. Everything else works just like you're used to.

## What's Still Coming?

Aside from the previously mentioned parts that need implementing, there are some nice additions potentially coming down the pike. There's no guarantee all of these items will make it in, but these are a handful of the ideas I'd currently like to see make it in the database layer.

**Native Read/Write Connection Support** is likely to make it in the configuration of your database. Once the connections have been defined, using them is automatic. The Connection will determine if your query is a write query or read query and choose the correct connection to use based on that. So, if you have a master/slave setup, this should make things a breeze.

**New QueryBuilder Methods** will likely be added. I'm going to scout out the other frameworks a little more, to see if there's features that are useful enough to warrant looking into. The following are my short list, though:

*    **first()** is a convenience method to retrieve the first result itself.
*    **increment()** and **decrement()** methods would be nice to have.
*    **chunk()** would loop over all matching results in chunks of 100 (or whatever). This allows you to process potentially thousands or even millions of rows without completely killing your server or running out of memory.

**Enhanced Model** The only reason the CI_Model class exists in v3 is to provide easy access to the rest of the framework by using magic methods to access the loaded libraries, etc. That's not really necessary anymore, since there is no singleton object. So, it only makes sense to take this opportunity to actually create a Model class that is useful. The details of this haven't been discussed much in the Council, yet, so I can't say what will make it in. Over the years, though, creating base MY_Model classes with a a fair amount of convenience features has become fairly common. Time to build it into the core, I think.

**Simpler Pagination** This idea is ripped straight from Laravel, but the first project I worked on in Laravel, it was the pagination that blew me away. This would work hand-in-hand with the Enhanced Model, allowing you to simply call `$model->paginate(20)` and it would be smart enough to figure out the rest of the details. Obviously, there's more involved than that, but if you've ever used Laravel's version, you'll know how much of a breath of fresh air it is compared to CodeIgniter's. Now, there's is built into their ORM, so it might turn out to be not very feasible for our system, but it's definitely something I want to look into.

___

I hope that gets you excited about the future of the framework, and maybe calms down some fears that things are going to change too much. One of my big goals while rewriting is to keep the system familiar, while bringing more modern code practices and flexibility into the system.

Are there any features from other systems that you love and miss when you work in CodeIgniter that you'd like us to consider? I won't say that everything (or even any of it) will make its way into the system, but we'll definitely consider it.
