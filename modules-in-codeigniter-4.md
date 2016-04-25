# CodeIgniter 4 的模块

> 原文: [Modules in CodeIgniter 4](http://blog.newmythmedia.com/blog/show/2016-03-15_Modules_in_CodeIgniter_4)

几个月前当我们讨论 CodeIgniter 4 的功能时，HMVC 成为了焦点之一。总结下来，大多数观点把 HMVC 归类成两种用途：一种是显示在页面中的『Widget』，或者是简单的把代码拆分成一个一个的模块。在本篇文章中，我想探讨一下『模块』是如何应用在即将发布的新版框架中的。

> 注意：下面提到的所有例子都是基于预发行版本的，这些内容可能会随时发生变化。

## 模块/HMVC 支持？

在开始前，先让我来澄清一下：CodeIgniter 4 不支持 HMVC 和模块，至少，不是你想象的那种传统的支持方式。在 CodeIgniter 4 中没有一个正式的模块结构的定义，就像你可能在其他框架中看到的一样，诸如 Yii 的扩展或 Drupal 的插件。*并且，在 CodeIgniter 4 中也没有通过不同的目录来完成不同类的优先级加载。*

## 自动加载和命名空间

CodeIgniter 4 自带兼容 PSR-4 的自动加载器，所以不需要 Composer（但你随时都可以添加任何你喜欢的加载器）。

有可能你会想知道为什么我们不直接使用Composer作为框架默认的加载器？在开发团队讨论之初，作为核心开发人员的我曾经一度大力支持使用Composer。然而，通过我们越来越多的讨论与研究， 我们也已经越来越清晰的发现Composer对于CodeIgniter 4 来说，并不是一个正确的选择。 其中一个主要的原因是，之前框架的核心部分就曾经使用过一个第三方的脚本，然而每当它改变的时候，框架也必须做出相应的改动来适应它的需求。 还有，在不同的服务器环境中，特别是在为安全问题而实施严格限制的共享主机（shared hosting）的情况下，使用Composer更新时，将非常有可能会产生一系列的问题。 最后，因为Composer 不是框架的核心部分，我们就不必在为让框架支持Composer各种特性而花费大量精力，这让的我们可以更专注于框架本身。

在框架中，不仅框架自带的文件支持命名空间，而且你自己的项目应用程序也可以。框架本身自带的文件使用的命名空间是 `CodeIgniter`， application目录默认使用的是 `App` 命名空间。
如果你不想改变默认的命名空间，你可以在控制器和模型中完全使用框架自带的命名空间。 使用你自己的命名空间亦或是框架自带的命名空间都不会影响框架本身正常的工作。

当你结合这两种技术时，支持项目模块的工作，框架已经为你完成了90%。 接着，让我们来看一个简单的实例，然后我们会了解那剩下的10%。


## 一个简单的实例

设想一下我们现在正为我们的Blog创建一个模块。第一件事我们需要去决定的就是命名空间的名称。然后让我们创建所有即将用到的类。我们会用到Comapny name，Standard， Blog 这些有意义的子命名空间，因为这些命名空间可以很好的描述这个“模块”。接着我们创建了一个和 `/application` 同级的目录并且这个目录将会用来存放所有创建的模块。现在，这个项目文件结构看起来像你已经使用过的HMVC：

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
接着打开文件 `/application/Config/Autoload.php` 并且用它来让框架知道如何路由到那些我们刚刚创建的类。 在这个例子中，我们只在加载器中创建了一个命名空间，尽管你可以为每一个模块创建一个命名空间。

```php
$psr4 = [
        'Config'                     => APPPATH.'Config',
        APP_NAMESPACE.'\Controllers' => APPPATH.'Controllers',
        APP_NAMESPACE                => realpath(APPPATH),
        'Standard'                   => APPPATH.'../standard'
    ];
```
现在，只需要为我们的类创建相对应的命名空间，那么框架就会知道如何去找到这些类并且在框架的任何地方我们都可以使用它们。

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

真的是非常的方便，不是嘛？


## 那些非类文件框架是如何路由的呢？

如果你刚刚仔细看过上面的例子，也许会感到疑惑， “那些没有任何类的文件，就像助手类，视图类，框架是如何路由的呢？！” 正如你所说的，PHP不能使用命名空间规则路由到那些没有任何类的文件。 所以，现在就让我们来创建这样一个功能到CodeIgniter中来解决这个问题。

这个功能的工作原理是基于通过文件使用的命名空间来定位到我们想要的位置，然后在其子目录中找到它。让我们创建一些例子来帮助你理解这个知识点。

## 加载助手类

在本例中，我们假设有一个 `blog_helper` 的文件位于 `/standard/Blog/Helpers/BlogHelper.php`。 如果假设这个是类文件，那么它的命名空间看起来就像 `Standard\Blog\Helpers\BlogHelper.php`。 我们可以使用框架自带的 `load_helper()` 方法加载：

```php
load_helper('Standard\Blog\Helpers\BlogHelper');
```

哇！就是这么简单。现在你可以找到助手类并且加载它了。

## 加载视图类

视图类的加载模式与助手类完全相同，可以用load_view()方法进行加载。

```php
echo load_view('Standard\Blog\Views\index', $data);
```
神奇的是，框架会自动在指定搜索的命名空间下的框架本身自带目录中去搜索目标文件，所以你不必在路径中包含那些框架本身自帶的目录名字。前面的两个例子也可以这样来写：

```php
load_helper('Standard\Blog\BlogHelper');
echo load_view('Standard\Blog\index', $data);
```


___

然而，这不是唯一的项目文件结构你必须在你的项目中遵守的，你完全可以创建属于你自己的结构。 我希望你会为框架带给你项目的灵活性而感到高兴。
