[![NPM version](https://badge.fury.io/js/browser-http.png)](http://badge.fury.io/js/browser-http)
[![Dependency Status](https://gemnasium.com/sakren/node-browser-http.png)](https://gemnasium.com/sakren/node-browser-http)
[![Build Status](https://travis-ci.org/sakren/node-browser-http.png?branch=master)](https://travis-ci.org/sakren/node-browser-http)

[![Donate](http://b.repl.ca/v1/donate-PayPal-brightgreen.png)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=YJCJ4FWVRDT4A)

# http-browser

Simple (but advanced) library for working with http in browser (like for example jQuery.ajax).

http-browser uses [q](https://github.com/kriskowal/q) promise pattern and is instance of [EventEmitter](http://nodejs.org/api/events.html).

You can use it for example with [simq](https://github.com/sakren/node-simq) as you are used to from node environment or use
standalone build with expanded package into window:

## Installation


```
$ npm install browser-http
```

or for standalone version just choose desired version and include it.
* [Development version](https://github.com/sakren/node-browser-http/blob/master/build/http.js)
* [Minified version](https://github.com/sakren/node-browser-http/blob/master/build/http.min.js)


## Usage

```
var http = require('browser-http');

// or standalone version:
var http = window.http;		// you can of course just call http directly without window at the beginning

http.request('http://www.google.com', {type: 'GET'}).then(function(response) {
	console.log(response.text);
}, function(e) {
	throw e;		// some error occurred
});
```

In then function, you will get response object with data from server.

## Shorthands

```
var http = require('browser-http');

http.get('http://www.google.com');
http.post('http://www.google.com');
http.put('http://www.google.com');
http.delete('http://www.google.com');
http.getJson('http://www.google.com');
http.postJson('http://www.google.com');
http.jsonp('http://www.google.com');
```

## Options

In every http function, you can set other options. Now it is just type and data.

* `type`: GET, POST, PUT or DELETE. This is always replaced in shorthand methods
* `data`: literal object of data which needs to be send to server
* `jsonp`: name of callback for jsonp requests, when true is given `callback` name is used. Default is false
* `jsonPrefix`: prefix for json requests
* `parallel`: can disable parallel sending of GET requests, see issue [#4](https://github.com/sakren/node-browser-http/issues/4)

### Rewriting default options

Some options can be rewritten globally for all requests.

* `http.options.type`: default is `GET`
* `http.options.jsonPrefix`: default is `null`, see section `json hijacking` below
* `http.options.parallel`: default is `true`, see issue [#4](https://github.com/sakren/node-browser-http/issues/4)

## Response object

Basically it is just wrapper for some data from XMLHttpRequest.

* state
* status
* statusText
* rawData: same like responseText
* data: same like responseText or literal object (json)
* xml: same like responseXML

## Load JSON

If content-type in response header is `application/json` then your data will be automatically transformed into js object.

If you can not set this header on your server, than you can use `*Json` methods.

```
http.getJson('http://www.google.com/some.json').then(function(response) {
	console.log(response.data);		// output will be object
});

http.postJson('http://www.google.com/some.json');
```

## Requests queue

By default all your requests are called from queue one by one, so there is always just one request running (or zero).
Inspiration from this article [http://blog.alexmaccaw.com/queuing-ajax-requests](http://blog.alexmaccaw.com/queuing-ajax-requests).

You can of course disable this behavior:
```
http.useQueue = false;
```

## JSONP

It is very easy to work with jsonp requests.

```
http.jsonp('http://some.url.com').then(function(response) {
	console.log(response.data);
});
```

## Json hijacking

First, please read [this](http://stackoverflow.com/questions/2669690/why-does-google-prepend-while1-to-their-json-responses)
discussion on stackoverflow.

Now if you want to use same technique just like Google or eg. Facebook do, you only need to set your own prefix in requests.

```
http.get('http://some.url.com', {
	jsonPrefix: 'while(1);'
}).then(function(response) {
	console.log(response.data);
});
```

String `while(1);` will be removed from the beginning of received data before parsing into json object.

## Events

You can listen for all http events with your own functions.

```
http.on('send', function(response, request) {
	console.log('In any moment, new http request will be send to server');
});

http.on('afterSend', function(response, request) {
	console.log('I just sent some request to server, but there is still no response');
});

http.on('complete', function(error, response, request) {
	console.log('I just finished some request, but there may be some error');
});

http.on('success', function(response, request) {
	console.log('I have got response from server without any error :-)');
});

http.on('error', function(err, response, request) {
	console.log('Sorry, there was some error with this response');
});
```

## Extensions

Sometimes it will be better to register whole group of events and this group is called extension.

```
http.addExtension('nameOfMyExtension', {
	send: function(response, request) {},
	complete: function(response, request) {},
	success: function(response, request) {},
	error: function(err, response, request) {},
});
```

You can also remove other extensions.

```
http.removeExtension('nameOfMyExtension');
```

### Build in extensions

browser-http already comes with few extensions. Originally they were created for projects build on [Nette](http://nette.org/en/)
framework, but can be used on any other project.

#### Loading cursor

```
http.addExtension('loading', new http.Extensions.Loading);
```

Every time new request is send, your cursor is changed into `progress` cursor. After receiving response from server, cursor
is changed into `auto`.

#### Redirect

```
http.addExtension('redirect', new http.Extensions.Redirect);
```

If your server sends json data with `redirect` variable, then you will be redirected to address in this variable.

#### Snippets

```
http.addExtension(new http.Extensions.Snippets(window.jQuery));
```

If in response data is `snippets` object with html id and content pairs, then browser-http will iterate throw this object,
find element in page with given id and change content of this element into the one from given data.

This extension depends on jquery.

#### Ajax links

```
http.addExtension(new http.Extensions.Links(window.jQuery));
```

This is not true extension for browser-http. It listen for all click events on `a` links with class `ajax` but not with
class `not-ajax` and after this click, it creates ajax request.

Depends on jquery.

#### Ajax forms

This is the same like the previous one, but apply for all forms with `ajax` class.
This extension can not handle forms with file uploads.
Depends on jquery.

```
http.addExtension(new http.Extensions.Forms(window.jQuery));
```

## Tests

```
$ npm test
```

### Test mocks

```
var http = require('browser-http/Mocks/Http');

// or for standalone version: var http = http.Mocks.Http;

afterEach(function() {		// restore original http and remove all attached listeners to events
	http.restore();
	http.removeAllListeners();
    http.queue.removeAllListeners();
});

it('should load some data', function(done) {
	http.receive('some data', {'content-type': 'text/plain'}, 200);

	http.get('localhost').then(function(response) {
		expect(response.data).to.be.equal('some data');
		done();
	});
});

// text/plain in headers list is default content-type, so you don't have to set it. Also status 200 is default.

it('should load some data and check received data', function(done) {
	http.receive('some data', {'content-type': 'application/json'});

	http.once('send', function(response, request) {
		expect(request.xhr.url).to.be.equal('localhost?greeting=hello')			// now we can test eg. url with parsed data
	});

	http.get('localhost', {data: {greeting: 'hello'}}).then(function(response) {
		expect(response.data).to.be.eql({greeting: 'hello'});
		done()
	});
});
```

### Resending sent data back in response

```
http.receiveDataFromRequestAndSendBack({'content-type': 'application/json'});

http.get('localhost', {data: {greeting: 'hello'}}).then(function(response) {
	expect(response.data).to.be.eql({greeting: 'hello'});
});

http.get('localhost', {data: {greeting: 'good day'}}).then(function(response) {
	expect(response.data).to.be.eql({greeting: 'good day'});
});
```

### Timeout

Response will be send after 400 ms:
```
http.receive('some data', {'content-type': 'text/plain'}, 200, 400);

// or simple

http.receive('some data', null, null, 400);
```

Response will be send between 100 and 300 ms:
```
http.receive('some data', {'content-type': 'text/plain'}, 200, {min: 100, max: 300});
```

## Changelog

* 2.2.0
	+ Added support for environments without `require` (like with [simq](https://github.com/sakren/node-simq))
	+ Tests uses minified standalone version of browser-http

* 2.1.1
	+ Removed forgotten tests building

* 2.1.0
	+ Added support for json prefixes ([discussion on stackoverflow](http://stackoverflow.com/questions/2669690/why-does-google-prepend-while1-to-their-json-responses))

* 2.0.0
	+ jQuery must be passed in constructor into extensions which depends on it
	+ Updated dependencies
	+ Test frameworks are in devDependencies (not globally installed)
	+ Large refactoring (better for testing)
	+ Tests does not need real server (using mock from [philikon/MockHttpRequest](https://github.com/philikon/MockHttpRequest))
	+ Many optimizations
	+ Added some badges + travis

* 1.8.0
	+ Requests are added into queue
	+ Added support for jsonp

* 1.7.1
	+ Added some tests
	+ Bug with responses without content-type header

* 1.7.0
	+ Refactoring
	+ `buildQuery` and `urlencode` moved to `browser-http/Helpers`
	+ Instance of EventEmitter

* 1.6.4
	+ Optimizations + bug with sending data

* 1.6.3
	+ Bug with `buildQuery` - replaced with the real one from jQuery

* 1.6.2
	+ Just removed some useless code

* 1.6.1
	+ Forgot to add Extensions/Links shortcut

* 1.6.0
	+ `buildQuery` should got the same output like jQuery.param
	+ Added some extensions

* 1.5.2
	+ Bug with sending data via POST method

* 1.5.1
	+ Bug with X-Requested-With header

* 1.5.0
	+ Added method isHistoryApiSupported

* 1.4.0
	+ Sending X-Requested-With header

* 1.3.1 - 1.3.5
	+ Bugs

* 1.3.0
	+ Added `urlencode` and `buildQuery` methods
	+ Prepared changelog
	+ Added some tests
	+ Some bugs and optimizations
	+ Transforming response data into json if mime type is application/json