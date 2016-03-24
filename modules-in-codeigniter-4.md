# Modules in CodeIgniter 4

One of the big hot-buttons that came up during discussion about CodeIgniter 4 features a few months ago was that of HMVC. It seems that most of the comments fell in one of two uses: either for displaying "widgets" on the page, or for simply splitting code into basically modules. In this article, I wanted to look at how modules can work in the upcoming version of the framework.

> NOTE: These examples are all based on pre-release code and the specifics are subject to change at any time.

## Module/HMVC Support?

Let me get this out of the way up front: no, CodeIgniter 4 does not support either HMVC, or modules. At least, not in the traditional way that you might think about it. There's no formal definition of module structure, like you might find in a Yii Extension or Drupal plugin. And, there's no hierarchical loading of classes through a nest of different directories.

If that's the case, then how can we support any form of modules? Autoloading and Namespaces.

## Autoloading and Namespaces

The framework now ships with a built-in PSR-4 compliant autoloader, no Composer needed (though you're always free to use that in addition to the built-in one).

Why didn't we just use Composer as the core? We talked about it, and I was, at first, a big proponent for it. However, the more we talked and researched, the more it was clear that it wasn't the right thing for the framework. For one thing, it was having an external script at the core of our framework which left us at their mercy. Also, in different hosting environments, Composer can become problematic to update, especially on tightly-secured shared hosting. Finally, since we didn't have to support all of the flexibility and features that Composer does, we could make it a touch faster by default.

Both the system files and your application files can be namespaced. The system files live under the `CodeIgniter` namespace, while the application directory takes on the `App` namespace by default. You don't have to namespace your controllers and models if you don't want to. It's entirely optional and things will still work in the way that you're accustomed to working.

When you combine these two techniques, though, 90% of the work of supporting modules is already done for you. Let's look at a quick example, and then we'll cover the remaining 10% of the solution.

## A Quick Example

Imagine we are creating a Blog module. The first thing to do is to decide on a namespace and then create a home for all of the files to live. We'll use our company name, Standard, and Blog makes sense for the sub-namespace, since that describes the entire "module". While we could put it anywhere, let's create a new directory alongside the `/application` directory to hold all of our company's modules. The folder structure might look something like you're used to in HMVC:

```
/application
/standard
    /Blog
        /Config
        /Controllers
        /Helpers
        /Libraries
        /Models
        /Views
/system
```

Next, open up `/application/Config/Autoload.php` and let the system know where to find the files. In this example, we'll just create a namespace in the autoloader for the entire company namespace, though you could create additional ones if you want to create one for each module.

```php
$psr4 = [
        'Config'                     => APPPATH.'Config',
        APP_NAMESPACE.'\Controllers' => APPPATH.'Controllers',
        APP_NAMESPACE                => realpath(APPPATH),
        'Standard'                   => APPPATH.'../standard'
    ];
```

Now, as long as we namespace all of our classes, the system can find them and they can be used from anywhere.

```php
namespace Standard\Blog;

use Standard\Blog\Models\BlogModel;
use Standard\Blog\Libraries\BlogLibrary;
use Standard\Blog\Config\Blog as BlogConfig;

class BlogController extends \CodeIgniter\Controller
{
    public function index()
    {
        $model = new BlogModel();
        $blogLib = new BlogLibrary();
        $config = new BlogConfig();
    }
}
```

Simple stuff.

## What About Non-Class Files?

If you were paying attention, then you are probably saying, "Sure, buddy, but what about the non-class files, like helpers, and views? huh?!" And you're right. Straight PHP cannot load non-class-based files from namespaces. So, we built that functionality into CodeIgniter.

The way it works is that it will locate the folder based on the namespace of the file, and then look for it in the normal sub-directory. Some examples will clear this up.

## Loading Helpers

In our example, we might have a `blog_helper` file living at `/standard/Blog/Helpers/BlogHelper.php`. If this were a class, it might have a fully-qualified name like `Standard\Blog\Helpers\BlogHelper.php`. So we pretend that it is a class, and use the `load_helper()` function:

```php
load_helper('Standard\Blog\Helpers\BlogHelper');
```

And, voila!, it can locate the helper and load it.

## Loading Views

When using the module pattern, views can be loaded in the exact same way, except using the load_view() function.

```php
echo load_view('Standard\Blog\Views\index', $data);
```

The system will also look within the traditional CodeIgniter directories within that namespace so you don't have to include it in the name. The above examples could have also bee done like:

```php
load_helper('Standard\Blog\BlogHelper');
echo load_view('Standard\Blog\index', $data);
```

___

While this is not the only way that you can structure things in your application, I hope this gets you excited about the possibilities and flexibility that the framework will be bringing to your applications.
