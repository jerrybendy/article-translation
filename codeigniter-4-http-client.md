# CodeIgniter 4 HTTP Client

> 原文: [CodeIgniter 4 HTTP Client](http://blog.newmythmedia.com/blog/show/2016-04-18_CodeIgniter_4_HTTP_Client)

It used to be that the majority of the websites were silos, not communicating with other websites much at all. That has changed a lot in the last few years, though, to the point where many sites consume information from external APIs, or communicate with third-party services, like payment gateways, social networks, and more, all on a day-to-day basis. For PHP developers, this typically involves the use of curl to make this happen. That means that any full-stack framework should provide some form of capabilities to help you out there. In the upcoming CodeIgniter 4, we've provided a lightweight, yet still very flexible, HTTP client built-in.

## The CURLRequest Class

The CURLRequest class extends our more generic Request class to provide most of the features that you'd need on a day-to-day basis. It is a synchronous-only HTTP client. Its syntax is modeled very closely after the excellent [Guzzle HTTP Client](http://docs.guzzlephp.org/en/latest/). This way if you find that the built-in client doesn't do everything you need it to, it is very easy to switch over to using Guzzle and take advantage of some of its more advanced features, including asynchronous requests, non-reliance on curl, and more.

Why build our own solution? For the last decade, many developers have looked to CodeIgniter as the framework that you can download and have 90% or more of the features you need to build your site at your fingertips. Bundling something like Guzzle into the framework doesn't make sense, when Guzzle provides its own HTTP layer, duplicating a large part of the core system code. If we wanted to provide a solution, we had to build our own, based around our own HTTP layer. Being a lightweight solution, it is primarily a wrapper around curl and so the only real trick was ensuring syntax compatibility with Guzzle to make your transitions, if you need to do them, as simple as possible.

> NOTE: This does require that the curl library be installed and usable.

## A Few Quick Examples

Let's walk through a few quick examples to see how easy and convenient it is to have a curl wrapper in our bundle of tricks. These examples are obviously very bare-bones and don't provide all of the details you would need in a finished application.

### A Single Call

Let's say you have another site that you need to communicate with, and that you only need to grab some information from it once, not as part of a larger conversation with the site. You can very simply grab the information you need something like this:

```php
$client = new \Config\Services::curlrequest();
$issues = $client->request('get', 'https://api.github.com/repos/bcit-ci/CodeIgniter/issues');
```

The `request()` method takes the HTTP verb and URI and returns an instance of CodeIgniter\HTTP\Response ready for you to use.

```php
if ($issues->getStatusCode() == 200)
{
    echo $issues->getBody();
}
```

### Consuming an API

If you're working with an API for more than a single call, you can pass the `base_uri` in as one of a number of available options. Then, all following request URIs are appended to that base_uri.

```php
$client = new \Config\Services::curlrequest(['base_uri' => 'https://example.com/api/v1/']);

// GET http://example.com/api/v1/photos
$client->get('photos');

// GET http://example.com/api/v1/photos/13
$client->delete('photos/13');
```

### Submitting A Form

Often, you will need to submit a form to an external site, sometimes even with file attachments. This is easily handled with the class.

```php
$client =  new \Config\Servies::curlrequest();
$response = $client->request('POST', 'http://example.com/form', [
    'multipart' => [
        'foo' => 'bar',
        'fuz' => ['bar', 'baz'],
        'userfile' => new CURLFile('/path/to/file.txt')
    ]
]);
```

### Multitude of Options

The library supports a wide array of options, allowing you to work with nearly any situation you might find yourself up against, including:

*   setting auth values for HTTP Basic or Digest authentication
*   setting arbitrary header values for more complex authentication (or any other needs)
*   specifying the location of PEM-formatted certificates,
*   restricting execution time
*   allowing redirects to be followed,
*   return content even when it encounters an error
*   easily upload JSON content
*   send query string or POST variables with the request
*   specify whether SSL certificates should be validated or not (helpful for development servers who don't have full SSL certificates in place)
*   and more.

___

Hopefully, this new addition to the framework will come in handy during development and help make using curl much more pleasant.
