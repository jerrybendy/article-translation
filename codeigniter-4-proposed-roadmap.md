# CodeIgniter 4 Proposed Roadmap

After much consideration of the community's desires and opinions, and what we feel is best for the future, the CI Council has come to some decisions about the future of your favorite framework. Be warned, there are big changes ahead, but we believe they can be done in a way that not only prepares us for the future, but still feels like CodeIgniter - simple, fast, flexible. 

Here's a broad overview of what the future holds. Please remember that this is simply the initial plan. Any and all details are subject to change once development is underway.

## Core Changes

The PHP community has changed drastically since the time that CodeIgniter was originally built. Many of the core elements that exist in CodeIgniter do so because it was necessary at the time. When PHP5 came out, nothing in the core changed. That can't work if CodeIgniter is going to maintain its quality and place in the future of PHP frameworks. 

This means the system needs a complete rewrite.  It will be developed in a separate repository so they keep a clean separation. We imagine there are numerous chunks of code that will be reused, but the focus is on modern, clean code. 

We are targetting *PHP 7* since, at the anticipated release date, 5.6 will be in security maintenance mode and only have a few months until it becomes completely unsupported. Releasing with a target version that is almost dead would keep CI held back once more. We know this will not be a valid move to upgrade all apps to PHP 7 because of the variety of hosting situations, so the 3.x branch will be maintained for a while - much longer than the EOL window for the 2.x branch after the 3.x release.

The application and system directories will both work with PSR-4 autoloading. CodeIgniter will use our own autoloader, as well as integrate Composer for those times you need it. 

The individual components would be packaged in some way so that they can, for the most part, be used outside of CodeIgniter in your other projects. 

## Packages/Modules

We will lose the idea of Packages and modules. Don't panic! That's only because you can use name-spaced classes to handle most of the same things - at least for Controllers and Models. For the others, like views, config files, helpers, etc. we believe that we can make those work in your namespaces, also, so that all of the capabilities of packages, and the route-ability of modules, will exist in any directory that you can tell the autoloader how to find.

## Routing

Routing will be updated and feature the ability to turn off the "magic routing" that maps the URI directly to controller/method so that you can choose your development preferences. Either the "magic way" or by specifying each individual route in the routes file. 

## Improved Logging

Logging will be improved, though the specifics are yet to be determined. 

## Testing

We will be using PHPUnit for our tests like we do currently. This will also mean the tools you need to test your own applications will be there ready for you to use.

## Backwards Compatibility

As you can tell from the above, this is definitely a BC breaking version. We do think it is the best for the future of the framework and the developers that use it. By making a major change here, we set the base to work with for many years forward. We will attempt to ease the transition where possible, though how much we are able to do that and provide a modern codebase is yet to be seen. 

We will do our best to maintain what has made CodeIgniter as popular over the years as it has been. Namely the speed, simplicity, and the "feel". 

## Development Timeline

Development will be split into three phases, described below.

The following libraries will be removed from core and treated as optional downloads: Typography, FTP, ZIP, and XML-RPC. 

The Cart, Javascript, Unit_test, and Trackback libraries will be removed. 

We expect to have an alpha version of the core ready in less than a year. After that we would focus on improving the core and developing the rest of the packages to work with it. Exact timelines will vary, of course, depending on all of the things typical of open source projects like the quantity and quality of community contributions, as well as the available time and life events of the core contributors. 

##Phase 1 

The first phase would focus on nailing the essentials in the framework. This would ensure that all of the parts needed to make it work were in place and working well. This would include: 

* Autoloader
* Dependency Injection
* Logging
* Exception Handling
* HTTP Request/Response Layers (or Input/Output)
* Routing
* Controllers
* Models
* Database Layer
* Config
* Security

## Phase 2

This second phase focuses on providing and refining the existing classes and features that CodeIgniter users know and love. This would include: 

* The helpers
* Language/Localization features
* Caching
* Email
* Encryption
* Form Validation
* Image Library
* Pagination
* Uploader
* Sessions
* Views
* Debugging and Profiling Tools

## Phase 3 - Optional Libraries

The third phase includes fleshing out and working on the optional packages. At this point, the framework can be released and need not wait for these libraries to be brought up to date. 

* FTP
* XML-RPC
* Zip
* Typography
* Template Parser

We are excited by the opportunities ahead for the framework and looking forward to getting started on the framework very soon. We can't wait to see what this enables you to build in the future.
