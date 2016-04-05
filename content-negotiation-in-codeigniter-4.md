# CodeIgniter 4 的内容协商

当我关注到 HTTP 层的时候，发现很多 CIer 对内容协商还不是很了解，下面我们一起来探讨一下什么是内容协商，以及如何在即将到来的 CodeIgniter 4 中使用他。

## 什么是内容协商？

简而言之，内容协商是指客户端和服务器端就响应的资源内容进行交涉，然后提供给客户端最为适合的资源。内容协商会以响应资源的语言、图片类型和编码方式等作为判断的基准（包含在请求头中的某些 `Accept` 字段就是判断的基准）。

举个例子，我用 Chrome 访问 Mozilla 的站点，可以看到下面的 HTTP 请求头信息：

*    accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,/;q=0.8
*    accept-encoding:gzip, deflate, sdch
*    accept-language:en-US,en;q=0.8

这些 `accept` 信息告诉我们浏览器所支持的格式，并提供这些格式的优先级信息（通过 q 的值来确定优先级）。以上信息说明浏览器在所有支持的内容类型中更希望接收 text/html 类型的内容。由于我的浏览器的语言设置是英语，所以 `accept-language` 请求头表示我更喜欢美式英语（en-US）的页面。

很显然，即使我们不提供任何内容协商信息，Web 站点还是可以照常运行，并且我们已经这样做了很多年。然而事实上 Web 服务器可以帮我们处理某些形式的内容协商，我们通常不太善于利用这一点，但并不意味着服务器不能处理这些信息。

内容协商有两个很吸引人的用处，一个是用于那些支持多国语言的站点，另一个是用于返回特定格式数据的 API 接口。

是不是必须要使用内容协商呢？可能不一定，他也许是把双刃剑，有些人提议不要使用他，也有些人认为他就像切片面包一样令人喜爱。但如果你想用，那在 CodeIgniter 中使用内容协商也是很容易的。

## 一个简单的例子

这里我不会对内容协商作过多详细的介绍（详细介绍将写到用户手册中），这个例子简单介绍了内容协商是如何确定输出语言的。

```php
class BaseController extends \CodeIgniter\Controller
{
    protected $language;

    public function __construct(...$params)
    {
        parent::__construct(...$params);

        $supportedLangs = ['en-US', 'en', 'fr'];

        $this->language = $this->request->negotiate('language', $supportedLangs);
    }
}
```

这个例子表示该站点可以支持英语和法语，我们将支持的语言赋值到 `$supportedLangs` 数组里，预示着默认语言是美式英语，但也支持普通英语和法语，然后简单调用 `$negotiate->language()` 方法，传递支持的语言类型，解析时就能识别正确的 HTTP 头，然后按照数组里定义的优先级顺序，返回最匹配的结果。如果两种语言都无法匹配，就会使用数组中的第一个语言。

Negotiate 类中的 4 个协商方法分别为：

*    **media()** 不同于通常的 `Accept` 请求头，他可以用来请求不同版本的 html/text，或者音频支持，图像支持，等等。
*    **charset()** 不同于 `Accept-Charset` 请求头，如果没有匹配的话，默认值为 UTF-8。
*    **encoding()** 不同于 `Accept-Encoding` 请求头，可以决定任何客户端支持使用的压缩类型。
*    **language()** 不同于 `Accept-Language` 请求头。

并不是所有场景都用得着内容协商，但他却是构建高质量 API 的一个有力工具，并且也能够创造性的应用于其他地方。

[原文地址](http://blog.newmythmedia.com/blog/show/2016-03-03_Content_Negotiation_in_CodeIgniter_4)
