# CodeIgniter 4 的请求和响应

CI4的最大的一个变化是输入输出将被改变，在之前的版本，包括最新的V3版本，输入和输出是作为两个包含有相关的处理函数的类来分别处理，那样虽然不是很好的模式，但是处理事务却很高效。所以，在CI4中，我们将更加模块化HTTP层，并且构建你一个新的类来处理HTTP的将来请求和响应。

## 初看之下

在开发web应用时（不同于命令行程序），你只需要考虑两个类，`将来请求类`，和`响应类`。

## 将来请求类

将来请求类包含了HTTP请求和该请求附带的数据，包括：

*  GET, POST, SERVER, and ENV 变量
*  HEADERS(请求头)
*  Cookies
*  当前请求的URL对象
*  上传的文件

并且包且还包括常见请求信息比如：

*  请求客户端IP地址
*  该请求是否为Ajax请求
*  是否是命令行请求
*  是否是Https传输

如果你想知道该命名的不同之处，或者说认为将来请求就是简单的一个请求嘛，答案是否定的，已经有一个包含有像GET和POS变量的另一个更为普遍但是没有详细HTTP请求信息的普通请求类。一个请求通常只要满足两个条件其中的一个：要么是浏览器客户端向服务器作出请求(进来)，或者是一个发送给外部服务器的请求(出去)。

## 响应

响应类是将处理的结果返回给客户端，可以设置响应头，直接发送结果输出信息，等等。响应类提供了一些便捷方法比如：

*  设置适当的无缓存头信息(no-cache headers)
*  处理HTTP缓存头信息
*  重定向页面

## 一个简单的例子

上面的说的这些处理起来好像很有科技含量，但事实上其实很简单。每一个控制器已经有一个类的实例作为属性，但是为了简单，你并没有必要去使用它。控制器的输出会被捕获，然后自动被设置为响应的主体。一个简单的Hello World看起来像这样：

```
class Home extends \CodeIgniter\Controller
{
    public function index()
    {
        echo "Hello World!";
    }
}
```
一个简单的例子。

在这个例子中，实现了一个基本的请求的响应信息。你还可以创建更复杂的HTTP缓存策略,通过内容协商裁剪部分响应信息来处理将来请求，或者处理更多的东西。

下面是个稍微复杂一点的例子，你会发现代码很容易看明白，并且处理的很简单

```
class Home extends \CodeIgniter\Controller
{
    public function __construct(...$params)
    {
        parent::__construct(...$params);

        // This controller is only accessible via HTTPS
        if (! $this->request->isSecure())
        {
            // Redirect the user to this page via HTTPS, and set the Strict-Transport-Security
            // header so the browser will automatically convert all links to this page to HTTPS
            // for the next year.
            force_https();
        }
    }

    public function index()
    {
        $data = [
            ...
        ];

        // Set some HTTP cache rules for this page.
        $this->response->setCache([
            'max-age' => 300,
            's-max-age' => 900,
            'etag' => 'foo'
        ]);

        // Return JSON
        $this->response->setContentType('application/json')
                       ->setOutput(json_encode($data));
    }
}
```

在这个例子中，我们做了很简单的三件事。第一，通过将当前的请求地址重定向到Https请求和设置一个严格的安全传输头(Strict-Transport-Security，这种方式已经被很多主流浏览器供应商所支持，在发送请求前通过浏览器自动将Http请求地址转换到Https请求),强制这个页面通过Https请求；第二，我们将通过设置一些Http缓存规则来帮助浏览器识别什么时候缓存的数据是可以重用的，什么时候不可以重用，这意味着能减少Http请求量，减轻服务器负担，增加服务端性能；最后，我们将输出`JSON`数据给用户，保证输出的是正确的内容类型。

希望这篇文章能简要展望一下未来的Codeigniter,并且让人意识到变化不总是令人害怕的，将来还会有更多的文章来讲述框架的概念，更多的文章将会出现在一个相对固定的地方。

原文链接地址：http://blog.newmythmedia.com/blog/show/2016-03-02_Requests_and_Responses_In_CodeIgniter_4
















