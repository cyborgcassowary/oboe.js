
`Progressive.js` is a progressive json parser built on top of [clarinet](https://github.com/dscape/clarinet).
It provides an intuitive interface to use the a http response while the request is still
ongoing. The whole library gzips down to  `3.5k`.

## Purpse

Your web application probably spends more time waiting for io than anything else. However,
during much of the time it spends waiting there is already enough data loaded to start updating
the page.

In a browser progressive allows you to start updating your page earlier, creating a
more responsive user experience. Using $.ajax() etc is very simple but your ajax
call will wait for the whole response to return before your javascript gets given objects.

The aim of Progressive is to let you start using the response as early as possible with only
a slight increase in complexity for the programmer.

The resulting parser makese results ready as quickly as if they had been pased using a SAX
parser like Clarinet but the syntax is much simpler to use.

The downside compared to using clarinet directly is that because the whole json object
is stored in memory, progressive uses more memory.
However, the memory used is no more than if the json had been parsed with
JSON.parse() so for most real-world applications this is acceptable.

# Example

## listening to json over ajax

``` js
// we have things.json, to be fetched over ajax. In reality your json is probably bigger than this.
{
   foods: [
      {name:'aubergine', colour:'purple'},
      {name:'apple', colour:'red'},
      {name:'nuts', colour:'brown'}
   ],
   non_foods: [
      {name:'brick', colour:'red'},
      {name:'poison', colour:'pink'},
      {name:'broken_glass', colour:'green'}
   ]
}

// In our webapp we want to load the json and write the foods out
// but we don't want to wait for the non_foods to load before we show them
// since we're going to ignore them anyway:
progressive.fetch('/myapp/things.json')
   .onMatch('//foods/*', function( foodThing ){
      // this callback will be called everytime a new object is found in the foods array.
      // in this example we just use jQuery to set up some simple DOM:
      $('#foods')
         .append('<div>')
            .text('it is safe to eat', foodThing.name)
            .style('color', foodThing.colour)
   });



// the pattern can match strings as well as objcts, if we just want to make a list of names
// of the names we can do this:

progressive.fetch('/myapp/things.json')
   .onMatch('**/name', function( name ){
      $('#thing').append('<li>').text(name)
   });



// We probably want to provide some visual feedback that an area is still loading data, let's
// use progressive to notify us when we've loaded all the foods:

showSpinner('#foods');  //

progressive.fetch('/myapp/things.json')
   .onMatch('//foods/*', function( foodThing ){
      $('#foods')
         .append('<div>')
            .text('it is safe to eat', foodThing.name)
            .style('color', foodThing.colour)
   })
   .onMatch('//foods', function(){
      // Will be called when the whole foods array has loaded. We've already wrote
      // the DOM for each item in this array above so we don't need to use the items
      // anymore, just hide the spinner.

      hideSpinner('#foods');

      // note that even though we just hid the spinner, the json might have have completely
      // loaded. That's fine because we don't need the non-foods to update the #foods
   });
```

In the above example, progressive

# Pattern matching

Progressive's pattern matching recognises these special tokens:

* `//` root json object
* `/`  path separator
* `*`  any named node in the path
* `**` any number of intermediate nodes (non-greedy)

## Some Example patterns:

```
'//foods/colour'           // the colours of the foods
'**/person'                // all people in the json
'**/person/friends/*/name' // detecting links in social network
`**/person/**/email`       // email anywhere as descendent of a person object
`//`                       // the root object (once the whole document is ready, like JSON.parse())
`*`                        // every object, string, number etc in the json!
```


## Use as a stream in node.js


`Clarinet` supports use as a node stream. This hasn't been implemented in
progressive but it should be quite easy to do.

## Running the tests

Progressive is built using [grunt](http://gruntjs.com/) and
[require.js](http://requirejs.org/) and tested using
[jstestdriver](https://code.google.com/p/js-test-driver/). JSTD might
be chaged to jasmine later to make the tests easier to run from grunt
(there is a jstd plugin for Grunt but it doesn't work with Grunt v4).

The `runtests.sh` script combines several steps into one:

* Runs the unminified code against the tests
* build the code using grunt
* Run the minified code against the tests

If everything works you will see output like below:

```
no tests specified, will run all
Will run progressive json tests( all ) against unminified code
setting runnermode QUIET
............................................................
Total 60 tests (Passed: 60; Fails: 0; Errors: 0) (103.00 ms)
  Firefox 17.0 Mac OS: Run 20 tests (Passed: 20; Fails: 0; Errors 0) (103.00 ms)
  Safari 536.26.17 Mac OS: Run 20 tests (Passed: 20; Fails: 0; Errors 0) (31.00 ms)
  Chrome 27.0.1435.0 Mac OS: Run 20 tests (Passed: 20; Fails: 0; Errors 0) (38.00 ms)

Running "requirejs:compile" (requirejs) task
>> Tracing dependencies for: progressive
>> Uglifying file: /Users/jimhigson/Sites/progressivejson/progressive.min.js
>> /Users/jimhigson/Sites/progressivejson/progressive.min.js
>> ----------------
>> /Users/jimhigson/Sites/progressivejson/src/main/libs/clarinet.js
>> /Users/jimhigson/Sites/progressivejson/src/main/streamingXhr.js
>> /Users/jimhigson/Sites/progressivejson/src/main/progressive.js

Done, without errors.
Will run progressive json tests( all ) against minified code
setting runnermode QUIET
............................................................
Total 60 tests (Passed: 60; Fails: 0; Errors: 0) (80.00 ms)
  Firefox 17.0 Mac OS: Run 20 tests (Passed: 20; Fails: 0; Errors 0) (80.00 ms)
  Safari 536.26.17 Mac OS: Run 20 tests (Passed: 20; Fails: 0; Errors 0) (33.00 ms)
  Chrome 27.0.1435.0 Mac OS: Run 20 tests (Passed: 20; Fails: 0; Errors 0) (38.00 ms)

Process finished with exit code 0
```