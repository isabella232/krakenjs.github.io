---
Title: Getting Started With Kraken
layout: frontpage
---

## Getting started


#### 0. Install node.js and essential build tools

[Install node.js](http://nodejs.org), which will in turn install npm, which is used to manage all your dependencies.

In order for the `npm install` command to work, you need these essential build tools:

* Python 2.6 or 2.7
* C++ compiler (GCC 4.2 or newer)
* Make

The method of installing these pre-requisites will vary by operating system.

#### 1. Install the generator

Start by installing the generator globally using npm: `[sudo] npm install -g yo generator-kraken bower grunt-cli`



#### 2. Create a project

Once installed, you can create a basic project using the generator. Type `yo kraken` and follow the prompts:


{% highlight text %}
$ yo kraken

     ,'""`.
hh  / _  _ \
    |(@)(@)|   Release the Kraken!
    )  __  (
   /,'))((`.\
  (( ((  )) ))
   `\ `)(' /'

Tell me a bit about your application:

[?] Name: HelloWorld
[?] Description: A test kraken application
[?] Author: YourName GoesHere
...
{% endhighlight %}


The generator will create a new directory for your application, set up an empty project and download all the necessary dependencies.



#### 3. Start your server

To run your project, just go into the newly created directory and type `npm start`:


{% highlight bash %}
$ cd HelloWorld
$ npm start

> helloworld@0.1.0 start ~/HelloWorld
> node index.js

[development] Listening on http://localhost:8000
{% endhighlight %}


Your kraken application will start up on port 8000. You can visit it at [http://localhost:8000](http://localhost:8000). If all goes well, your very polite application will say hello.



### Structure of a Project

Kraken keeps your code organized by splitting up the configuration, content+templates and routing logic into different places so that it's easy to keep track of everything and to easily swap out components. Let's start by looking at the base structure of the project you just created:


{% highlight text %}
/config  
Application configuration including environment-specific configs

/controllers
Routes and logic

/locales
Language specific content bundles

/lib
Common libraries to be used across your app

/models
Models

/public
Web resources that are publicly available

/public/templates
Server and browser-side templates

/tasks
Grunt tasks to be automatically registered by [grunt-config-dir](https://github.com/logankoester/grunt-config-dir)

/tests
Unit and functional test cases

index.js
Application entry point
{% endhighlight %}


Let's say you want to create a simple application. As your application grows, this becomes unmanageable and messy. Kraken helps you stay organized by imposing a sound structure and strategy.

First let's look at our basic `index.js` entry point:


{% highlight javascript %}
'use strict';


var kraken = require('kraken-js'),
    app = require('express')(),
    options = {
        onconfig: function (config, next) {
            config.get('view engines:js:renderer:arguments').push(app);

            next(null, config);
        }
        /* more options are documented in the README */
    },
    port = process.env.PORT || 8000;


app.use(kraken(options));


app.listen(port, function (err) {
    console.log('[%s] Listening on http://localhost:%d', app.settings.env, port);
});
{% endhighlight %}



As you can see from the `index.js` entry point, kraken is just an express middleware. The configuration is just an object that defines an `onconfig`. That function is called when the application begins configuration.

There are a few other [options that can be set](https://github.com/krakenjs/kraken-js#options) as well as the `onconfig` handler.

So, where's all the configuration? Where are the routes?



### Configuration

Kraken's configuration can be found in the `config/config.json` file.

This JSON file contains key value pairs that are loaded at runtime. The advantage of this is that all your application configuration is in a single, well-known place; and you can swap it out without having to touch a single line of code.

This config file is also where you can define middleware and specify its load-order. To find out more, check out [meddleware](https://github.com/krakenjs/meddleware).



#### Development vs. Production environments

A common scenario is that development environments have slightly different parameters than production. Kraken allows you to define a second file `config/development.json` with alternate values.

You can control which file is loaded by defining an environment variable: `NODE_ENV` and setting its value to `production` or `development` as appropriate.



### Security

Security is provided out-of-the-box by the [Lusca](http://github.com/krakenjs/lusca) module. Lusca is middleware for express, and it follows [OWASP](http://www.owasp.org/) best practices by enabling the following request/response headers for all calls:

- [Cross Site Request Forgery](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_%28CSRF%29)
- [Content Security Policy](https://www.owasp.org/index.php/Content_Security_Policy)
- [Platform for Privacy Preferences Project](http://support.microsoft.com/kb/290333)
- [X-FRAME-OPTIONS](https://www.owasp.org/index.php/Clickjacking)
- And more!


If you want to disable or configure them, please see the [Lusca README](http://github.com/krakenjs/lusca).



### Routes

Kraken moves the routing logic into separate files in the `controllers` folder, allowing you to group routes by functionality.

For example, a route for your home page, would use a `controllers/index.js` file that looks as follows:


{% highlight javascript %}
'use strict';

var IndexModel = require('../models/index');

module.exports = function (router) {
    var model = new IndexModel();

    router.get('/', function (req, res) {
        res.render('index', model);
    });
};
{% endhighlight %}


This file would define the routes and the logic for the home page. The advantage of keeping routes and logic segregated in individual files starts to show as the application grows. If something fails, it's very easy to pinpoint where things went wrong.

Kraken is built on top of express, so the rest of the logic should be familiar to Node developers.

New to 1.x, your controllers are given an instance of your top-level `router` instead of the `app` instance, and routes are automatically determined for you based on folder-structure. For example, if you wanted to specify a handler for `/users`, simple drop this in `/controllers/users/index.js`:


{% highlight javascript %}
'use strict';

module.exports = function (router) {
    // notice that my route is '/' but I respond to '/users'
    router.get('/', function (req, res) {
        res.send('you can find me at /users');
    });
};
{% endhighlight %}

With 1.x, route registration is supercharged and very flexible. Check out [express-enrouten](https://github.com/krakenjs/express-enrouten) to learn more.


### Models

Kraken also separates data models from the controller logic, resulting in cleaner, more organized code. Data models live in the `models` folder.

When a new controller is created, the framework will also create a simple model for you.


{% highlight javascript %}
'use strict';

module.exports = function IndexModel() {
    return {
        name: 'myApp'
    };
};
{% endhighlight %}


While not very complex, this model serves as a base to build upon. See the [Kraken Shopping Cart](https://github.com/krakenjs/kraken-example-with-shoppingcart) example for more complex usage of models.


### Templates

Kraken uses [LinkedIn's Dust](http://linkedin.github.io/dustjs/) as the templating language of choice. [Adaro](http://github.com/krakenjs/adaro) is the module responsible for rendering and managing the templates.

Templates are loaded from the `public/templates` directory. Because they reside in the public folder, this allows kraken to use the same templates on the server side as well as the client side, allowing you to reuse code.

If we wanted to greet our customers, a basic template would be:


{% highlight html %}
<h1>Hello {name}!</h1>
{% endhighlight %}


### Localization

Thanks to [Makara](http://github.com/krakenjs/makara), kraken has the ability to load content bundles on the fly, based on the request context. If we wanted to greet a user in their native language (e.g.: Spanish), we can simply add this context to the response before rendering the template:


{% highlight javascript %}
res.locals.locale = { language: 'es', country: 'ES' };

var model = { name: 'Antonio Banderas' };
res.render('index',model);
{% endhighlight %}


We would also change our template as follows, using a `@pre type="content"` tag:


{% highlight html %}
<h1>{@pre type="content" key="index.greeting"/}</h1>
{% endhighlight %}

This instructs the framework to pick up the index.greeting string from one of the locale content bundles.

The `locales` directory holds these bundles, organized by country and language. The bundles are nothing more than simple `key=value` .property files. If our sample application caters to English and Spanish speakers, we would create two bundles:

`locales/US/en/index.properties` to hold `index.greeting=Hello {name}!`

and

`locales/ES/es/index.properties` to hold `index.greeting=Hola {name}!`

So, in the above example, since the language and country are set to `es` and `ES` respectively, the framework would pick the second bundle and display:


{% highlight text %}
Hola Antonio Banderas!
{% endhighlight %}

### Grunt tasks

Generated projects include the following grunt tasks by default:

* `$ grunt build` will localize and compile your templates to the `.build` directory, and also copy static assets there
* `$ grunt test` will run jshint as well as any mocha tests


### FAQ

#### How can I disable CSRF for specific routes?

If you need to disable the [CSRF](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)) protection provided by the [lusca module](https://github.com/krakenjs/lusca#luscacsrfoptions) for certain paths --a common use case for APIs-- you can accomplish this through the configuration file, on the middleware section.
The strategy is to first disable the default CSRF handling, and then re-enable it for the routes we want to protect. This can be done using a [negative lookahead regular expression](http://www.regular-expressions.info/lookaround.html).

> **Example:** Protect all routes, _except_ those starting with `/api` :

{% highlight javascript %}

{
    "middleware": {
        /**
         * Override the default lusca configuration to disable CSRF handling.
         */
        "appsec": {
            "module": {
                "arguments": [
                    {
                        "xframe": "SAMEORIGIN",
                        "p3p": false,
                        "csp": false
                    }
                ]
            }
        },

        /**
         * Enable *ONLY* CSRF filtered by route.
         * Note: The route "regex" needs the double parens
         * because of how express parses route strings.
         */
        "csrf": {
            "enabled": true,
            "priority": 111,
            "route": "/((?!api))*",
            "module": {
                "name": "lusca",
                "method": "csrf",
                "arguments": [ {} ]
            }
        }
    }
}
{% endhighlight %}

<hr>

#### How can I register middleware in config?

We use meddleware to facilitate the registration of middleware.

> **Example:** Let's define a middleware to let any origin access the resource. Say it's in './middlewares/cors'

{% highlight javascript %}

module.exports = function () {
    return function(req, res, next) {
        res.header("Access-Control-Allow-Origin", "*");
        res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
        next();
    };
};
{% endhighlight %}

I can include above middleware in my config like so:

{% highlight javascript %}

{
    // ...
    "middleware": {
        "cors": {
            "enabled": true,
            "priority": 119,
            "module": {
                "name": "path: ./middlewares/cors" //"path" is a shortstop-handler
            }
        }
    }
}
{% endhighlight %}

<hr>

#### I'm encountering `TypeError: Cannot call method 'replace' of undefined` when I run my app. Help!

If you're running in production mode, Kraken will look for pre-compiled templates in the `./.build` directory.
This error likely indicates that you have not built your project. Try running `grunt build`.

<hr>

#### How should I access configuration values in my application?

Since kraken compiles your configuration at runtime -- taking in to account the environment plus shortstop handlers -- you should not attempt to `require` the JSON directly.

You can access the configuration from the request: `req.app.kraken.get("my:config:property")`.

You can also access the configuration in an `onconfig` handler as discussed here: [https://github.com/krakenjs/kraken-js#options](https://github.com/krakenjs/kraken-js#options)

If you are using an onconfig handler, just be aware that startup is asynchronous. I.e. don't rely on the config value until onconfig has fired, or the app emits the start event.

<hr>

#### How can I contribute to this project?

Bugs and new features should be submitted using [GitHub issues](https://github.com/krakenjs/kraken-js/issues/new). Please include with a detailed description and the expected behavior. If you would like to submit a change yourself do the following steps.

- Fork it.
- Create a feature branch.
- Commit a test that fails due to the bug
- Commit a fix that makes the test pass
- Open a [pull request](https://github.com/krakenjs/kraken-js/pulls).


This page is built from the `krakenjs.github.io` repository. To refine the document even further, you can [create an issue](https://github.com/krakenjs/krakenjs.github.io/issues/new), or better yet, submit a [pull request](https://github.com/krakenjs/krakenjs.github.io/pulls).



### Examples

Here's a few examples to get you going with kraken:

- **[Kraken Shopping Cart](https://github.com/krakenjs/kraken-example-with-shoppingcart)**  
An end-to-end example showing how to build a shopping cart that integrates with PayPal

- **[Kraken Passport Integration](https://github.com/krakenjs/kraken-example-with-passport)**  
Authenticate and securely store user credentials using Passport, Mongo and bcrypt

- **[Localization and Internationalization](https://github.com/krakenjs/kraken-example-with-i18n)**  
Greet users in different languages. Shows support for localized content bundles.

- **[Specialization](https://github.com/krakenjs/kraken-example-with-specialization)**  
Show different layouts to different users based on a set of rules (locality, A/B testing, etc).

### Recommended patterns

- **[Environment specific routing](https://github.com/krakenjs/kraken-js/wiki/Using-kraken-js-middleware-config-to-setup-environment-specific-routes)**
Use `NODE_ENV` specific middleware to set up routing per-environment

- **[Subapp mounting](https://github.com/krakenjs/kraken-js/wiki/Using-krakenjs-middleware-config-for-express-subapps-mounting)**
Use kraken middleware for express subapp mounting

- **[Whitelisting and blacklisting routes](https://github.com/krakenjs/kraken-js/wiki/Using-krakenjs-middleware-config-for-whitelisting-and-blacklisting-routes)**
Use kraken middleware configuration to whitelist and blacklist routes

