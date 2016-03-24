# Dependency Injection in CodeIgniter 4

I remember reading a forum thread during the time that we were originally asking for community input on the future of the framework. In it, they ridiculed the community for even considering whether or not we would be using Dependency Injection. At the time, I believe the council was pretty set on having it, but we were letting the discussions and suggestions arise naturally within the community. I read another forum thread the other day on a different site that was looking at our features and wondering why we were bothering since it just read like Laravel, due in large part to the DI, the namespacing, the PSR4 autoloading, etc. I guess you just can't please everyone, right?

## Why Is DI Important?

Dependency Injection decouples your code from other, specific, classes. When used correctly, it allows you to easily replace dependencies with mock classes during testing, or replace the dependency with a completely different class that handles the task better. In short, it makes the code more resilient to change. It makes it more portable. It's a good thing, without a doubt. If you've spent most of your PHP career using CodeIgniter, you might not have run across Dependency Injection before, so a short example will help clear things up.

> Note: The database layer is still under early development. This is purely an example.

Let's say you have a model for managing Users. You will, naturally need the database class to work with, so without DI you might do something like:

```php
class UserModel
{
    protected $db;

    public function __construct()
    {
        $this->db = new Database();
    }
}
```

But there's a problem here. If you ever need to use a different database library, or switch from MySQL to MongoDB, you have to change code in every class. If you're running tests, you can never separate the logic in your UserModel from the Database class. Long term maintenance can become a problem, too.

To fix this, the next step is to pass the Database class into the constructor as a dependency. This solves all of those problems, especially when you're requiring the a class that implements an interface, instead of any specific class.

```php
class UserModel
{
    protected $db;

    public function __construct(DatabaseInterface $db)
    {
        $this->db = $db;
    }
}
```

This is the purest form of Dependency Injection. Any external classes needed are injected into the class, either through the constructor, as shown here, or through a setter method.

Within the core of CodeIgniter 4, constructor-based DI is used everywhere. While this has advantages for the developers of the framework, it has huge implications for you. No longer do you need `MY_*` classes to extend or replace core files. Now, you can simply create a new class that conforms to the Interface we're expecting, and ensure that class is passed in it's place. How you make sure it gets used instead of the original file requires a bit more story.

## The Rise and Fall of the Container

If you spend any time at all reading up on "modern PHP" best practices, you'll always see a DI Container (sometimes called an Inversion of Control Container) used. Most of the major frameworks use one, including Symfony, Laravel, Nette, Zend, and most others. Because of this, my natural first reaction was to [create one](https://github.com/newmythmedia/di) for the framework. I thought it turned it pretty sweet. You could configure classes with their alias, and, through the magic of Reflection, it would examine the constructor and automatically insert a shared instance of any configured classes, or parameters. It worked great, and was pretty fast.

Then I read a [blog post by Anthony Ferrara](http://blog.ircmaxell.com/2015/11/simple-easy-risk-and-change.html) that was discussing the differences between simple and easy when it comes to programming, and recomended optimizing for simplicty. One section in particular hit a chord: "A simple example is the way many PHP frameworks have adopted 'Dependency Injection' . . . But what if instead of using this complex system, we just created a series of functions? Real code that you can debug and understand." Bam. This was shortly after I had written the container, thinking the problem solved. This comment gnawed inside of me for a couple of weeks.

At first, I tried to make excuses about why we needed that container. But as I looked back on the things that keep bringing me back to CodeIgniter over the last 10 years, I realized a big part of it was the simplicity of the framework. It was simple code, that made things simple to understand and trust. You didn't have to wade through 6 different abstractions to understand what was going on. So, I ripped the container out, replacing it with a simple class that was just "a series of functions".

And you know what? It works great. It even came with a few unintended benefits, the biggest being that backtraces during errors are MUCH more understandable now. I'm currently working on a project that's using Laravel and get so frustrated by the backtrace being full of 15 calls through Laravel's sub-structure before I can find my code, if I even can.

## Services

At the core of the way this whole thing ties together is the Services config file. While it's possible that the name may change, Services are simply other classes, and the config file tells how to call them. Almost all of the core has an entry here. So, a quick look at a couple of the Services methods to see how they work, and then we'll move on to a quick example of Services and DI as you'd use it in your application.

```php
/**
 * The Logger class is a PSR-3 compatible Logging class that supports
 * multiple handlers that process the actual logging.
 */
public static function logger($getShared = false)
{
    if (! $getShared)
    {
        return new \CodeIgniter\Log\Logger(new \Config\Logger());
    }

    return self::getSharedInstance('logger');
}
```

Here's a service at it's simplest. All of the methods allow you to get either a brand new instance of the class, or one that's shared among all other uses, which is a great option for things like this logger, where there's no real reason to waste memory on multiple instances. Assuming that you did want the shared instance, you'd simply call `$logger = Config\Servies::logger(true)`.

Because they are just simple methods, some of them support parameters to customize how the class works. For instance, the `renderer()` method, which handles displaying views, can take the path to the views folder as the first parameter.

```php
/**
 * The Renderer class is the class that actually displays a file to the user.
 * The default View class within CodeIgniter is intentionally simple, but this
 * service could easily be replaced by a template engine if the user needed to.
 */
public static function renderer($viewPath = APPPATH.'Views/', $getShared = false)
{
    if (! $getShared)
    {
        return new \CodeIgniter\View\View($viewPath, self::loader(true), CI_DEBUG, self::logger(true));
    }

    return self::getSharedInstance('renderer');
}
```

If you want to replace the renderer with a simple solution to use a template engine like Twig, you'd create a small adapter class that implented the `CodeIgniter\View\RenderableInterface` and modify the Services::renderer() method to return an instance of your new adapter class. Then, you could call the `view()` command that is always available, and it would use Twig instead of the simple view-based solution that CodeIgniter provides. Couldn't be simpler.

Alright, you've seen how to define the services, and we've talked about why DI is a great thing, so it's time to take a look at how to use the two together in your own application.

## A Quick Example

> Using Dependency Injection in your applications is not required, though it's recommended. The framework itself uses these files, and they provide a simple way to modify the core, but any other use by yourself is optional.

With that disclaimer out of the way, let's look at our `UserModel` again. Assume that we're in the `Users` controller and you need to pull all active users. Earlier, we showed the UserModel taking a Database object in it's constructor. Ignoring the exact class names for now, getting a new instance of the model would be done something like this:

```php
class Users extends \CodeIgniter\Controller
{
    public function index()
    {
        $model = new UserModel( Config\Services::database() );

        $data = [
            'users' => $model->findAll()
        ];

        echo view('users/list_all', $data);
    }
}
```

This way, if you ever change the database solution, you don't have to hunt around trying to find every location. Simply change it in the Services config file, and you're golden. If you're only using a single database connection, you could modify the `database()` service to insert the correct config, etc. Since there is actually a couple of parameters needed to create a database connection, you could make things even simpler and create a new service for your model if you needed to.

> Remember - all database stuff shown here is for example only, and doesn't reflect the end product in any fashion!

## Coupling?

As a keen observer, you might be yelling at me that this added services stuff just moves the coupling of these classes from the libraries, models, etc, to the controller. And you'd be correct. At some point, though, you have to be able to specify which classes you want to use.

And the truth is that is what a Controller's job is. It glues the other pieces together. It is the one piece of your application that should be tightly coupled with your framework. If you design the rest of your application correctly, it's about the only place that even knows about the framework you're using. And that's great. If, some years down the road, you need to switch frameworks for some reason (it happens, unfortunately), you will mostly just have to change the controllers.

Even better - this simple Services class further reduces your dependency on any specific framework. It's just a simple class, and could be used with any framework you wanted to use.
