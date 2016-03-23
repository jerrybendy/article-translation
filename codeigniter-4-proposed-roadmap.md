# CodeIgniter 4 建议路线图

After much consideration of the community's desires and opinions, and what we feel is best for the future, the CI Council has come to some decisions about the future of your favorite framework. Be warned, there are big changes ahead, but we believe they can be done in a way that not only prepares us for the future, but still feels like CodeIgniter - simple, fast, flexible.

我们综合考虑了社区的愿望和意见后，也对什么样的未来对 CI 是最合适的做了一些思考，然后，CI 理事会对框架的未来做出了一些决策。预告一下，未来将会有重大变化，但我们相信他们不仅为我们的未来做准备，也会保持 CodeIgniter 的一贯特点 - 简洁、快速和灵活。

Here's a broad overview of what the future holds. Please remember that this is simply the initial plan. Any and all details are subject to change once development is underway.

这篇文章是对未来的一个简单概述。请记住，这仅仅是最初的计划。任何信息都可能会随着开发的进行而发生变化。

## 核心变化

The PHP community has changed drastically since the time that CodeIgniter was originally built. Many of the core elements that exist in CodeIgniter do so because it was necessary at the time. When PHP5 came out, nothing in the core changed. That can't work if CodeIgniter is going to maintain its quality and place in the future of PHP frameworks.

从 CodeIgniter 第一版发布以来，PHP 社区发生了巨大变化。CodeIgniter 的许多核心要素在当时是必须的，但当 PHP5 发布后，CodeIgniter 的核心几乎没有任何变化。如果未来 CodeIgniter 想要继续保持高水准并巩固其在 PHP 框架中的地位，则必须进行大改变。

This means the system needs a complete rewrite.  It will be developed in a separate repository so they keep a clean separation. We imagine there are numerous chunks of code that will be reused, but the focus is on modern, clean code.

这意味着系统必须全部重写。新的 CI 将在独立的代码库中开发以保持代码的清晰。我们设想会复用一些以前的代码，但重点是 *现代的* 清晰的代码。

We are targeting *PHP 7* since, at the anticipated release date, 5.6 will be in security maintenance mode and only have a few months until it becomes completely unsupported. Releasing with a target version that is almost dead would keep CI held back once more. We know this will not be a valid move to upgrade all apps to PHP 7 because of the variety of hosting situations, so the 3.x branch will be maintained for a while - much longer than the EOL window for the 2.x branch after the 3.x release.

自从我们以 *PHP 7* 为目标以来，PHP 5.6 已进入安全维护模式，并将在几个月后完全停止支持。CI 暂时不会为一个即将停止支持的 PHP 版本发布新版本。我们知道主机的环境千差万别，那么一些程序可能就无法完全支持 PHP 7，所以 3.x 分支将继续维护一段时间 - 将远远长于 3.x 发布后的 2.x 分支的 EOL 时限。

The application and system directories will both work with PSR-4 autoloading. CodeIgniter will use our own autoloader, as well as integrate Composer for those times you need it.

application 和 system 目录都将支持 PSR-4 自动加载。CodeIgniter 将使用自己的自动加载器，并将会整合 Composer。

The individual components would be packaged in some way so that they can, for the most part, be used outside of CodeIgniter in your other projects.

我们将封装一些组件，以便在大多数情况下可以用于 CodeIgniter 以外的项目。

## 包/模块

We will lose the idea of Packages and modules. Don't panic! That's only because you can use name-spaced classes to handle most of the same things - at least for Controllers and Models. For the others, like views, config files, helpers, etc. we believe that we can make those work in your namespaces, also, so that all of the capabilities of packages, and the route-ability of modules, will exist in any directory that you can tell the autoloader how to find.

我们将丢弃应用程序包和模块的概念。不必惊慌！因为你可以用名字空间来处理大多数情况 - 至少控制器和模型是这样的。对于其他的例如视图、配置文件和 Helper 等，我们相信可以让这些东东支持名字空间。你也可以把所有包的功能和模块的路由能力放到任意目录中，只需告诉自动加载器如何找到他们。

## 路由

Routing will be updated and feature the ability to turn off the "magic routing" that maps the URI directly to controller/method so that you can choose your development preferences. Either the "magic way" or by specifying each individual route in the routes file.

路由功能将被更新。URI 直接映射到控制器/方法这个『魔术路由』功能将可关闭，以便让你选择自己喜欢的路由方式。在路由配置文件中你可以选择使用『魔术路由』或者单独指定每个路由。

## 改进的日志系统

Logging will be improved, though the specifics are yet to be determined.

日志系统将被改进，但具体细节尚未确定。

## 测试

We will be using PHPUnit for our tests like we do currently. This will also mean the tools you need to test your own applications will be there ready for you to use.

我们将继续使用 PHPUnit 做测试。这也意味着你需要自己测试应用程序，但我们将为你准备好所需的工具。

## 向后兼容性

As you can tell from the above, this is definitely a BC breaking version. We do think it is the best for the future of the framework and the developers that use it. By making a major change here, we set the base to work with for many years forward. We will attempt to ease the transition where possible, though how much we are able to do that and provide a modern codebase is yet to be seen.

正如上面讲到的那样，这一定是一个和老版本不兼容的版本。我们认为这应该是框架最好的未来。对于这次的重大变化，我们已经做了很多年的基础工作，我们将尽可能的使过渡更平滑，但是对于我们能提供一个怎样的现代化的基础代码仍有待观察。

We will do our best to maintain what has made CodeIgniter as popular over the years as it has been. Namely the speed, simplicity, and the "feel".

我们将尽最大努力保持让 CodeIgniter 多年来流行的特性，即快速、简洁和『优雅』。

## 开发时间表

Development will be split into three phases, described below.

整个开发过程将分为三个阶段。

The following libraries will be removed from core and treated as optional downloads: Typography, FTP, ZIP, and XML-RPC.

下列类库将从内核删除，改为按需下载：排版类、FTP 类、ZIP 类和 XML-RPC 类。

The Cart, Javascript, Unit_test, and Trackback libraries will be removed.

购物车类、Javascript 类、单元测试类和 Trackback 类将被删除。

We expect to have an alpha version of the core ready in less than a year. After that we would focus on improving the core and developing the rest of the packages to work with it. Exact timelines will vary, of course, depending on all of the things typical of open source projects like the quantity and quality of community contributions, as well as the available time and life events of the core contributors.

我们期望在一年内能完成一个内核基本成型的 Alpha 版本。之后，我们将把重点放在改进内核与开发剩余软件包的工作中。确切的时间表可能会有所不同，因为，与大多数开源项目一样，这取决于社区贡献的数量与质量，以及核心开发者的时间和精力。

## 第一阶段

The first phase would focus on nailing the essentials in the framework. This would ensure that all of the parts needed to make it work were in place and working well. This would include:

第一阶段将侧重于抓住框架最重要的部分。这将是框架其他部分的基础。他们包括：

*   自动加载器
*   依赖注入
*   日志
*   异常处理
*   HTTP 请求/应答层（或输入/输出）
*   路由
*   控制器
*   模型
*   数据库层
*   配置
*   安全

## 第二阶段

This second phase focuses on providing and refining the existing classes and features that CodeIgniter users know and love. This would include:

第二阶段的重点是完善一些 CodeIgniter 用户喜闻乐见的类和特性。他们包括：

*   Helper
*   语言/国际化特性
*   缓存
*   Email
*   加密
*   表单验证
*   图像处理类库
*   分页
*   上传类
*   Session
*   视图
*   调试和分析工具

## 第三阶段 - 可选类库

The third phase includes fleshing out and working on the optional packages. At this point, the framework can be released and need not wait for these libraries to be brought up to date.

第三阶段将主要用来扩充可选软件包。在这个时间点，框架应该可以发布了，不需要等到所有类库都准备就绪。

*   FTP
*   XML-RPC
*   Zip
*   排版类
*   模板解析器

We are excited by the opportunities ahead for the framework and looking forward to getting started on the framework very soon. We can't wait to see what this enables you to build in the future.

框架迎来了新的机遇使我们很激动，并迫不及待的希望框架迈出第一步。不要在一旁围观了，赶快来构筑框架的未来吧。
