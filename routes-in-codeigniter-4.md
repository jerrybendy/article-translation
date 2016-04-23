# Routes in CodeIgniter 4
# CodeIgniter 4 的路由

> 原文: [Routes in CodeIgniter 4](http://blog.newmythmedia.com/blog/show/2016-03-08_Routes_in_CodeIgniter_4)

Routes in CodeIgniter have gone through a pretty big upgrade from version 3 to 4. This article will give a 100-foot view of some of the new changes, and give you something to look forward to.

CodeIgniter 的路由在 3 和 4 的版本中有了很大的变更，在本文中将会从零开始带你了解 CI4 的路由。

## Route Basics
## 路由基础

As a refresher, in version 3 routes were specified in a simple array, where each key was the "URI from" and the value of the element was where it should be routed to. It was simple, elegant, worked great, and looked something like this:

先来看一下 CI3 的路由。在 CI3 中路由使用一个简单的数组来指定，数组的键代表原始的URI，值代表最终被路由到的URI。非常简单、优雅的一种方式，并且工作良好，如下：

```php
$route['join']   = 'home/register';
$route['login']  = 'home/login';
$route['products/(:any)/details'] = 'products/show/$1';
```

The capability of routers in other frameworks has surpassed the simple elegance we have enjoyed for years. Even in the CodeIgniter community, there have been several router replacements people could use. So, it was time for an upgrade.

其它很多框架中的路由功能已经变得越来越强大，而不仅仅是简单优雅了，甚至在 CodeIgniter 社区也有人提出了很多可用的替代路由方案。所以，是时候改进 CodeIgniter 的路由了。

The first thing we had to do was to make it use a class, instead of a simple array. We tried to stick with using simple arrays to increase functionality, but it became too much of an complex beast. So, the new routes would look like this:

我们要做的第一件事是改为使用一个类来定义路由，而不再使用简单的数组。我们试图使用简单的数组来增加功能，但这会使路由变得太过于复杂。所以新的路由应该看起来像下面这样：

```php
$routes->add('join',   'Home::register');
$routes->add('login',  'Home::login');
$routes->add('products/(:segment)', 'Products::show/$1');
```

While the "to" portion of the route looks different, the functionality is much the same here. The `join` route is being directed to the `Home` controller, and its `register()` method. The `products` route is being directed to the `Products` controller, with the captured `(:segment)` being passed to the `show()` method. While it might appear that the controllers must now use static methods, that is not the case. The familiar syntax was used to specify the controller/method combination only, and methods are not allowed to be static.

与 CodeIgniter 3 相比，“路由到”的部分看起来会有一些不同，但功能是基本相同的。`join`路由会被定向到`Home`控制器的`register`方法；`products`路由将会被定向到`Products`控制器，并将`(:segment)`参数传递给`show()`方法。从路由上看起来控制器似乎必须要使用静态方法，并非如此。使用`::`只是方便用更熟悉更直观的方法来指定控制器与方法的组合，而方法是不允许使用静态方式声明的。

## Module-like Functionality
## 模块化的功能

Why the new format? Because we don't want to restrict you to controllers in the `/application/Controllers` directory. Instead, you can now route to any class/method that the system can find through its new PSR-4 compatible autoloader. This makes it a breeze to organize code in module-like directories.

为什么要使用新的语法？因为我们不想限制你必须把控制器放置在`/application/Controllers`目录下。相反，现在你可以路由到任意的类/方法，只要这个类/方法可以在系统中可以通过新的 PSR-4 兼容的自动加载机制加载到。这可以使以模块化的方式组织目录变得轻而易举。

For example, if you have a `Blog` "module" under the namespace `App\Blog`, you could create some routes like so:

例如，如果你有一个博客模块，模块放置在`App\Blog`命名空间下，你可以像下面这样创建路由：

```php
$routes->add('blog', 'App\Blog\Blog::index');
$routes->add('blog/category/(:segment)', 'App\Blog\Blog::byCategory/$1');
```

If the Blog controller lives under `application/Controllers`, great. But if you want to move it into it's own folder, say `application/Blog`, you can update the autoloader config file and everything still works.

如果你的`Blog`控制器是放在`application/Controllers`目录下，很好。但是如果你想把它移动到其它目录下，如`application/Blog`，你只需要更新你的`autoloader`配置，一切将会运行如初。

## Closures
## 闭包

Routes no longer have to mapped to a controller. If you have a simple process you can route to an anonymous function, or Closure, that will be ran in place of any controller.

路由不再需要和控制器对应。如果你只是需要做一个简单的处理，你甚至可以不用写控制器。现在你可以把路由指向一个匿名函数或者一个闭包来代替控制器。

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
## 占位符

I'm sure you've noticed a different placeholder than you're used to in the routes: `(:segment)`. This is one of a handful that come stock with CodeIgniter, and is used to replace the `(:any)` that is in v3 and clear up any confusion. Now, the system recognizes the following placeholders:

我确定你一定注意到了我们使用了不同于 CodeIgniter 3 中的路由占位符：`(:segment)`。This is one of a handful that come stock with CodeIgniter, and is used to replace the `(:any)` that is in v3 and clear up any confusion.现在，系统可以识别以下的占位符：

* **(:any)** will match all characters from that point to the end of the URI. This may include multiple URI segments.
* **(:segment)** will match any character except for a forward slash (/) restricting the result to a single segment.
* **(:num)** will match any integer.
* **(:alpha)** will match any string of alphabetic characters
* **(alphanum)** will match any string of alphabetic characters or integers, or any combination of the two.

* **(:any)** 会匹配所有字符直到URI的结尾。可能会包含多个URI段。
* **(:segment)** 会匹配除了正斜线（/）外的所有字符。限制结果只包含一个URI段。
* **(:num)** 会匹配所有整数。
* **(:alpha)** 会匹配所有英文字母。
* **(:alphanum)** 会匹配所有英文字母或数字，或两者混合。

It doesn't stop there, though. You can create your own at the top of the routes file by assigning a regular expression to it, and then it can be used in any of the routes, making your routes much more readable.

不止如此，你可以在路由配置文件的顶部通过正则表达式创建你自己的规则，然后你就可以在所有路由配置中使用你的规则，使你的路由可读性更强。

```php
$routes->addPlaceholder('uuid', '[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}');
$routes->add('users/(:uuid)', 'Users::show/$1');
```

## HTTP Verbs
## HTTP Verbs（HTTP动作）

So far, I've been using the generic `add` method to add a new route. Routes added this way will be accessible through any HTTP-verb, whether it's a GET request, POST, PATCH, or even from the command line. It's recommended, though, to restrict the route to only the type of access you need.

到目前为止，我们仅仅使用了普通的`add`方法添加一个新的路由。用这种方式添加的路由可以通过任意的HTTP-verb访问，不管是GET、POST、PATCH甚至使用命令行方式。尽管如此，我们更推荐你限制你的路由只能通过你需要的方式被访问。

```php
$routes->get('products', 'Product::feature');
$routes->post('products', 'Product::feature');
$routes->put('products/(:num)', 'Product::feature');
$routes->delete('products/(:num)', 'Product::feature');
$routes->match(['get', 'post'], 'products/edit/(:num)', 'Product::edit/$1');
$routes->cli('maintenance/on', 'CLITools::maintenanceModeOn');
```

## Generating standard Resource routes
## 生成标准的 Resource 路由

When working on API's it's best to keep a standard set of routes mapping to the same methods in each controller, just to make maintenance simpler. You can can easily do this with the `resources` method:

当你在写API时，最好的方式是保持一个标准的路由集合，这些路由对应到每个控制器中的相同的方法，这样可以使代码更易于维护。你可以使用`resources`方法轻松实现这些方式：


```php
$routes->resources('photos');
```

This will create the 5 standard routes for a resource:

这会为同一个资源创建5个标准的路由：

HTTP Verb | Path | Action | Used for...
--------- | ---- | ------ | -----------
GET	| /photos | listAll | display a list of photos
GET	| /photos/{id} | show | display a specific photo
POST | /photos | create | create a new photo
PUT	| /photos/{id} | update | update an existing photo
DELETE | /photos/{id} | delete | deletes an existing photo


HTTP Verb | 路径      | 动作 | 用途
--------- | ----     | ------ | -----------
GET	       | /photos | listAll | 显示照片列表
GET      	| /photos/{id} | show | 显示指定的照片
POST      | /photos | create | 创建新的照片
PUT	       | /photos/{id} | update | 更新已经存在的照片
DELETE    | /photos/{id} | delete | 删除已经存在的照片

The routes can have a fair amount of customization to them through by passing an array of options in as the second parameter, but we'll leave those for the docs.

路由可以通过传递可选的第二个数组参数来定义具体的行为，这些将会留在文档里面具体说明。

## No More Magic
## 没有魔法

By default, the URI will attempt to be matched up to a controller/method if no route exists for it. This is very convenient and, for those familiar with it, makes it a breeze to find where the code is that you're trying to use. Sometimes, though, you don't want this functionality.

默认情况下，URI会尝试匹配

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
