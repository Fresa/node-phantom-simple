Node-phantom-simple
---------------

This is a bridge between [PhantomJs](http://phantomjs.org/) and
[Node.js](http://nodejs.org/).

This module is API compatible with node-phantom but doesn't rely on WebSockets
or socket.io. In essence the communication between Node and Phantom has been
made significantly simpler. It has the following advantages over node-phantom:

  - Fewer dependencies/layers.
  - Just uses the HTTP server module built into Node.
  - Doesn't use the unreliable and huge socket.io.
  - Works under `cluster` (node-phantom does not due to how `server.listen(0)`
    works in cluster)

Requirements
------------
You will need to install PhantomJS first. The bridge assumes that the
"phantomjs" binary is available in the PATH, or you will need to pass its path
into the `phantom.create() method.

For running the tests you will need [Expresso](http://visionmedia.github.com/expresso/).
The tests require PhantomJS 1.6 or newer to pass.

Installing
----------

    npm install node-phantom-simple


Usage
-----
You can use it exactly like you would use Node-Phantom, and the entire API of
PhantomJS should work, with the exception that every method call takes a
callback (always as the last parameter) instead of returning values.

For example this is an adaptation of a
[web scraping example](http://net.tutsplus.com/tutorials/javascript-ajax/web-scraping-with-node-js/) :

```javascript
var phantom=require('node-phantom-simple');
phantom.create(function(err,ph) {
  return ph.createPage(function(err,page) {
    return page.open("http://tilomitra.com/repository/screenscrape/ajax.html", function(err,status) {
      console.log("opened site? ", status);
      page.includeJs('http://ajax.googleapis.com/ajax/libs/jquery/1.7.2/jquery.min.js', function(err) {
        //jQuery Loaded.
        //Wait for a bit for AJAX content to load on the page. Here, we are waiting 5 seconds.
        setTimeout(function() {
          return page.evaluate(function() {
            //Get what you want from the page using jQuery. A good way is to populate an object with all the jQuery commands that you need and then return the object.
            var h2Arr = [],
            pArr = [];
            $('h2').each(function() {
              h2Arr.push($(this).html());
            });
            $('p').each(function() {
              pArr.push($(this).html());
            });

            return {
              h2: h2Arr,
              p: pArr
            };
          }, function(err,result) {
            console.log(result);
            ph.exit();
          });
        }, 5000);
      });
	});
  });
});
```

### phantom.create(callback, options)

`options` is an optional object with options for how to start PhantomJS.
`options.parameters` is an array of parameters that will be passed to PhantomJS
on the commandline.

For example

```javascript
phantom.create(callback, {parameters: {'ignore-ssl-errors': 'yes'}})
```

will start phantom as:

```bash
phantomjs --ignore-ssl-errors=yes
```

You may also pass in a custom path if you need to select a specific instance
of PhantomJS or it is not present in PATH environment. This can for example
be used together with the [PhantomJS package](https://npmjs.org/package/phantomjs)
like so:

```javascript
phantom.create(callback, {phantomPath: require('phantomjs').path})
```

You can also have a look at the test folder to see some examples of using the
API, however the de-factor reference is the
[PhantomJS documentation](https://github.com/ariya/phantomjs/wiki/API-Reference).
Just change all return values into callbacks.

WebPage Callbacks
-----

All of the WebPage callbacks have been implemented including `onCallback`, and
are set the same way as with the core phantomjs library:

```javascript
page.onResourceReceived = function(response) {
    console.log('Response (#' + response.id + ', stage "' + response.stage + '"): ' + JSON.stringify(response));
};
```

This includes the `onPageCreated` callback which receives a new `page` object.

Properties
-----

Properties on the [WebPage](https://github.com/ariya/phantomjs/wiki/API-Reference-WebPage)
and [Phantom](https://github.com/ariya/phantomjs/wiki/API-Reference-phantom)
objects are accessed via the `get()/set()` method calls:

```javascript
page.get('content', function (html) {
  console.log("Page HTML is: " + html);
})
page.set('zoomfactor', 0.25, function () {
  page.render('capture.png');
})
```

License - MIT
-----

Copyright (c) 2013 Matt Sergeant

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.



Other
-----
Made by Matt Sergeant for Hubdoc Inc.

