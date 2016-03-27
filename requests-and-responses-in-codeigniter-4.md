# CodeIgniter 4 的请求和响应

CodeIgniter 4 对输入和输出的处理方式做了一个较大的改变。在前一个版本中，包括最新的 V3 版，输入和输出分别是用两个包含相关函数的类来处理的。这样的处理方式背后虽然没有什么高深的技术，但却能简单直接的实现功能。在 V4 版，我们将更加模块化 HTTP 层，并构建全新的类结构来同时处理 HTTP 的请求和响应。

## 概览

在开发 Web 应用时（不同于 CLI 程序），你只需关心两个类：`IncomingRequest` 和 `Response`。

### IncomingRequest 类

IncomingRequest 类包含了 HTTP 请求和该请求附带的数据，包括：

*  GET、POST、SERVER 和 ENV 等环境变量
*  HTTP 请求头
*  Cookie
*  当前请求的 URL 对象
*  上传的文件

并且还包括常见的请求信息比如：

*  客户端的 IP 地址
*  是否为 Ajax 请求
*  是否为 CLI 请求
*  是否为 HTTPS

如果你对 IncomingRequest 这个类名感到奇怪，或者说 IncomingRequest 是不是可以简单的称为 Request？答案是否定的，因为已经有另一个包含 GET 和 POST 等变量的更为通用的 Request 类，但这个类不包括详细的 HTTP 请求信息。一个请求通常只做两件事：一个是浏览器客户端发送请求到服务器（连入），或者是当前服务器发送请求到外部服务器（连出）。

### Response 类

Response 类用于把程序的执行结果返回给客户端。你可以设置 HTTP 响应头，或直接发送内容到客户端，等等。Response 类提供了一些便捷方法比如：

*  设置适当的 no-cache 头信息
*  处理 HTTP 缓存头信息
*  重定向页面

## 一个简单的例子

上面说的这些看起来好像很有科技含量，但其实很简单。这些类的实例作为属性已放到每个控制器中，如果你觉得很麻烦，则无需直接使用这些属性。Response 类会捕获控制器的输出，并自动设置为响应的主体。一个简单的 Hello World 看起来像这样：

```php
class Home extends \CodeIgniter\Controller
{
    public function index()
    {
        echo "Hello World!";
    }
}
```

易如反掌。

在需要的时候，框架为你提供了精确控制响应的能力。你可以创建复杂的 HTTP 缓存策略，并与 IncomingRequest 类一起通过内容协商定制响应内容。

下面是个稍微复杂一点的例子，你会发现代码很容易看明白，并且处理的很简单。

```php
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

在这个例子中，我们主要做了三件事。首先，通过将当前 URL 重定向到 HTTPS URL，并设置一个 Strict-Transport-Security 响应头（这种方式已被很多主流浏览器所支持，在发送请求前通过浏览器自动将 HTTP 请求转换成 HTTPS 请求），来强制这个页面以 HTTPS 的方式访问；其次，我们通过设置一些 HTTP 缓存规则来帮助浏览器正确处理缓存，这意味着能减少 HTTP 请求量，减轻服务器负担，提高性能；最后，我们输出 JSON 数据给用户，并确保内容类型是正确的。

希望这篇文章能有助于大家粗略的了解 CodeIgniter 的未来，让大家意识到改变并不可怕。:) 未来将敲定框架越来越多的细节，直到形成一个相对稳定的架构，并且会撰写更多的文章来讲述这些内容。
