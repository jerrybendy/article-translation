# CI 4 的内容协商

内容协商可能对于很多CI开发者来说是很陌生的，当我关注到Http层的时候才了解到这一点。下面我们一起来探讨一下什么是内容协商，以及它是如何提现在即将到来的CI4。

## 什么是内容协商？

最简单的情景，内容协商就是你的网站和用户的浏览器通过浏览器发送的一些特定`Accept`头(这些头可以确定返回页面的语言，页面图片的类型，浏览器支持的编码类型等等)来确定最终返回的数据。

举个例子，我用chrome访问Mozilla的站点，可以看到下面的头信息：

*    accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,/;q=0.8
*    accept-encoding:gzip, deflate, sdch
*    accept-language:en-US,en;q=0.8

这些`Accept`头信息告诉我们浏览器支持的格式，并且提供给我们所支持的这些格式的优先级信息，上面的信息告诉我们浏览器在支持的文件类型中更希望接收`text/html`文件。因为浏览器设置的是英文，`accept-language`预示着我更喜欢美式英语的页面。

很显然，即使我们不提供任何内容协商信息，web站点还是可以照常运行，并且我们已经这样做了很多年。然而事实上web服务器是可以帮助我们处理更多的信息，我们通常不太善于利用这一点，并不意味着我们不能处理这些信息。

内容协商很吸引人的两个用处是对于那些多语言支持的站点，或者是可以返回特定格式的数据的API接口，等等。

是不是必须要使用内容协商呢？可能不一定，它也许是把双刃剑，有些人提议不要使用它，也有些人认为它就像切片面包一样令人喜爱。如果你需要使用内容协商，在Codeigniter中将可以很简单的使用。

## 一个简单例子

这里我不会对内容协商作过多详细的介绍(详细介绍将在文档中),这个例子简单介绍了内容协商是如何确定输出语言的。

```
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

这个例子表示该站点可以支持英文和法语，我们将支持的语言赋值到`$supportedLangs`数组里，预示着默认语言是美式英语，但也支持普通英文和法文，然后简单调用`$negotiate->language()`函数，传递支持的语言类型，解析时就能够识别正确的头部，按照数组里定义的优先级顺序，返回最匹配的结果。如果两种语言都无法匹配，就会使用数组中定义的第一个参数定义的语言。

内容协商类中的四个协商方法分别为：

*  `media()`不同于通常的`Accept`头，它可以用来请求不同版本的html/text,或者音频支持，图像支持，等等。
*  `charset()`不同于`Accept-Charset`头，如果没有匹配的话，默认值为UTF-8
*  `encoding()`不同于`Accept-Encoding`头，可以决定任何客户端支持使用的压缩类型
*  `language()`不同于`Accept-Language`头

并不是所有的场景都用的着内容协商，但它却是构建高质量API的一个有力工具，并且也能够创造性的应用于其他地方。

[原文地址](http://blog.newmythmedia.com/blog/show/2016-03-03_Content_Negotiation_in_CodeIgniter_4)



































