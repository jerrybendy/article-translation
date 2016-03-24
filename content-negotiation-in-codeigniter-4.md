# Content Negotiation in CodeIgniter 4

For many CodeIgniter developers, the idea of Content Negotiation is probably a unfamiliar one. I know it was for me when I started working on the HTTP layer. So, let’s take a look at what it is, and then how it can be used in the upcoming CodeIgniter 4.

## What Is Content Negotiation?

In its simplest terms, Content Negotiation is your website and the user’s browser working together to decide the best type of data to return. This is done via several `Accept` headers the browser can send, that can specify the language to return the page in, the types of images it likes, encodings that it supports, and more.

As an example, when I visit Mozilla’s site in Chrome, I see these headers:

* accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,/;q=0.8
* accept-encoding:gzip, deflate, sdch
* accept-language:en-US,en;q=0.8

This tells us that the browser can support the formats in the `accept` header and provides us information (via the q score) about how the preferences are ordered. In this case, it would prefer the response as text/html over all of the other formats. Because of settings in my browser, the `accept-language` header says that I would like to read the page in American English (en-US).

Obviously, the web still works even if we don’t perform any form of content negotiation. We’ve done it for years just fine without worrying about it. To be fair, the web server itself can do some forms of conneg for us, but we don’t typically take advantage of that, either. That doesn’t mean it’s not handy, though.

The two times that having this ability is really appealing is for sites that support multiple languages, or for API’s that can use it to return the data in specific formats, and more.

Should you always use it? Probably not. There are pros and cons, and some people who claim it should never be used, with others thinking it’s the greatest thing since sliced bread. But if you need it, it’s simple in CodeIgniter.

## A Quick Example

I won’t get into all of the details here (we’ll save that for the docs), but here’s a simple example of how it might be used to determine language.

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

In this example, our site can display the content in either English and French. We assign that to the `$supportedLangs` array, which says that our default language is American English, but we can support generic English, and also French. Then, we simply call `$negotiate->language()`, passing it the values that we support, and it will parse the correct header, sort by it's order of priority, and return the best match. If there isn't a match between the two, the first element in our supported values array is returned.

The four negotiation methods in the class are:

* **media()** which matches values against the generic `Accept` header, and can be used to ask for different versions of html/text, or audio support, image support, and more.
* **charset()** matches against the `Accept-Charset` header. The default value, if no match is made, is UTF-8.
* **encoding()** matches against the `Accept-Encoding` header, and helps to determine the type of compression, if any, that the client supports.
* **language()** matches against the `Accept-Language` header.

While this is not something that will be used all of the time, it is a tool that could prove itself extremely helpful for building out quailty API's, and can probably be used creatively in other areas, also.
