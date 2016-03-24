# Routes in CodeIgniter 4

Routes in CodeIgniter have gone through a pretty big upgrade from version 3 to 4. This article will give a 100-foot view of some of the new changes, and give you something to look forward to.

## Route Basics

As a refresher, in version 3 routes were specified in a simple array, where each key was the "URI from" and the value of the element was where it should be routed to. It was simple, elegant, worked great, and looked something like this:

```php
$route['join']   = 'home/register';
$route['login']  = 'home/login';
$route['products/(:any)/details'] = 'products/show/$1';
```

The capability of routers in other frameworks has surpassed the simple elegance we have enjoyed for years. Even in the CodeIgniter community, there have been several router replacements people could use. So, it was time for an upgrade.

The first thing we had to do was to make it use a class, instead of a simple array. We tried to stick with using simple arrays to increase functionality, but it became too much of an complex beast. So, the new routes would look like this:

```php
$routes->add('join',   'Home::register');
$routes->add('login',  'Home::login');
$routes->add('products/(:segment)', 'Products::show/$1');
```

While the "to" portion of the route looks different, the functionality is much the same here. The `join` route is being directed to the `Home` controller, and its `register()` method. The `products` route is being directed to the `Products` controller, with the captured `(:segment)` being passed to the `show()` method. While it might appear that the controllers must now use static methods, that is not the case. The familiar syntax was used to specify the controller/method combination only, and methods are not allowed to be static.

## Module-like Functionality

Why the new format? Because we don't want to restrict you to controllers in the `/application/Controllers` directory. Instead, you can now route to any class/method that the system can find through its new PSR-4 compatible autoloader. This makes it a breeze to organize code in module-like directories.

For example, if you have a `Blog` "module" under the namespace `App\Blog`, you could create some routes like so:

```php
$routes->add('blog', 'App\Blog\Blog::index');
$routes->add('blog/category/(:segment)', 'App\Blog\Blog::byCategory/$1');
```

If the Blog controller lives under `application/Controllers`, great. But if you want to move it into it's own folder, say `application/Blog`, you can update the autoloader config file and everything still works.

## Closures

Routes no longer have to mapped to a controller. If you have a simple process you can route to an anonymous function, or Closure, that will be ran in place of any controller.

```php
$routes->add('pages/(:segment)', function($segment)
{
    if (file_exists(APPPATH.'views/'.$segment.'.php'))
    {
        echo view($segment);
    }
    else
    {
        throw new CodeIgniter\PageNotFoundException($segment);
    }
});
```

## Placeholders

I'm sure you've noticed a different placeholder than you're used to in the routes: `(:segment)`. This is one of a handful that come stock with CodeIgniter, and is used to replace the `(:any)` that is in v3 and clear up any confusion. Now, the system recognizes the following placeholders:

* **(:any)** will match all characters from that point to the end of the URI. This may include multiple URI segments.
* **(:segment)** will match any character except for a forward slash (/) restricting the result to a single segment.
* **(:num)** will match any integer.
* **(:alpha)** will match any string of alphabetic characters
* **(alphanum)** will match any string of alphabetic characters or integers, or any combination of the two.

It doesn't stop there, though. You can create your own at the top of the routes file by assigning a regular expression to it, and then it can be used in any of the routes, making your routes much more readable.

```php
$routes->addPlaceholder('uuid', '[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}');
$routes->add('users/(:uuid)', 'Users::show/$1');
```

## HTTP Verbs

So far, I've been using the generic `add` method to add a new route. Routes added this way will be accessible through any HTTP-verb, whether it's a GET request, POST, PATCH, or even from the command line. It's recommended, though, to restrict the route to only the type of access you need.

```php
$routes->get('products', 'Product::feature');
$routes->post('products', 'Product::feature');
$routes->put('products/(:num)', 'Product::feature');
$routes->delete('products/(:num)', 'Product::feature');
$routes->match(['get', 'post'], 'products/edit/(:num)', 'Product::edit/$1');
$routes->cli('maintenance/on', 'CLITools::maintenanceModeOn');
```

## Generating standard Resource routes

When working on API's it's best to keep a standard set of routes mapping to the same methods in each controller, just to make maintenance simpler. You can can easily do this with the `resources` method:

```php
$routes->resources('photos');
```

This will create the 5 standard routes for a resource:


HTTP Verb | Path | Action | Used for...
--------- | ---- | ------ | -----------
GET	| /photos | listAll | display a list of photos
GET	| /photos/{id} | show | display a specific photo
POST | /photos | create | create a new photo
PUT	| /photos/{id} | update | update an existing photo
DELETE | /photos/{id} | delete | deletes an existing photo

The routes can have a fair amount of customization to them through by passing an array of options in as the second parameter, but we'll leave those for the docs.

## No More Magic

By default, the URI will attempt to be matched up to a controller/method if no route exists for it. This is very convenient and, for those familiar with it, makes it a breeze to find where the code is that you're trying to use. Sometimes, though, you don't want this functionality.

For example, you might be building an API, and want a single location to serve as documentation for the API. This can be easily handled by turning off the `autoRoute` feature:

```php
$routes->setAutoRoute(false);
```

Now, only routes that have been defined can be served up by your application.

## Groups

Routes can be grouped together under a common prefix, reducing the amount of typing needed and helping to organize the routes.

```php
$routes->group('admin', function($routes) {
    $routes->add('users', 'Admin\Users::index');
    $routes->add('blog',  'Admin\Blog::index');
});
```

These routes would now all be available under an 'admin' segment in the URI, like:

* example.com/admin/users
* example.com/admin/blog

## Environment Groups

Another form of grouping, `environment()` allows you to restrict some routes to only work in a specific environment. This can be great for building some tools that only work on develoment machines, but not on the production server.

```php
$routes->environment('development', function($routes)
{
    $routes->add('builder', 'Tools\Builder::index');
});
```

## Redirect Old Routes

If your site has some pages that have been moved, you can assign redirect routes that will send a 302 (Temporary) Redirect status and send the user to the correct page.

```php
$routes->addRedirect('users/about', 'users/profile');
```

This will redirect any routes that match `users/about` to the new location at `users/profile`.

## Using Routes In Views

One of the more fragile things when building links within views is having your URIs change, which forces you to edit the links throughout your system. CodeIgniter now provides a couple of different tools to help get around this.

## Named Routes

Anytime you create a route, a name is made for it. By default, this is the same as the "from" portion of the route definition. However, this doesn't help, so you can assign a custom name to the route. This can then be used with the `route_to()` function that is always available to return the correct relative URI.

```php
// Create the route
$route->add('auth/login', 'Users::login', ['as' => 'login']);

// Use it in a view
<a href="<?= route_to('login') ?>">Login</a>
```

Named routes used in this way can also accept parameters:

```php
// The route is defined as:
$routes->add('users/(:id)/gallery(:any)', 'Galleries::showUserGallery/$1/$2', ['as' => 'user_gallery');

// Generate the relative URL to link to user ID 15, gallery 12
// Generates: /users/15/gallery/12
<a href="<?= route_to('user_gallery', 15, 12) ?>">View Gallery</a>
```

## Reverse Routing

For even more fine-grained control, you can use the `route_to()` function to locate the route that corresponds to the controller/method and parameters that you know won't change.

```php
// The route is defined as:
$routes->add('users/(:id)/gallery(:any)', 'Galleries::showUserGallery/$1/$2');

// Generate the relative URL to link to user ID 15, gallery 12
// Generates: /users/15/gallery/12
<a href="<?= route_to('Galleries::showUserGallery', 15, 12) ?>">View Gallery</a>
```

## Global Options

Any of the route creation methods can be passed an array of options that can help further refine the route, doing things like:

* assign a namespace to the controllers, reducing typing
* restrict the route to a specific hostname, or sub-domain
* offset the matched parameters to ignore one or more (that might have been used for language, version, etc)

## Need More? Customize it

If you find that you need something different from the router, it's simple to replace the RouteCollection class with your own, if you want a custom solution. The RouteCollection class is only responsible for reading and parsing the routes, not for doing the actual routing, so everything will still work with your custom solutions

Just be sure to share what you create with the rest of us! :)

___

Whew! There's the goodness that you get to look forward to. At least, I think I mentioned it all.
