# CoffeeScript notes

> To act as an aide-mémoire and to help explain to others why particular things work (or do not work) I have compiled a list of my coffeescript learnings.

## Class properties and instance properties

If you set a property outside of a method CS will treat it like a constant.  Take the following as an example, regardless passing a value to the `set` method, `@current` will continue to be null:

```coffeescript
class Servers
  @current = null
  constructor: ->
    
  set: (value) ->
    @current = value

  get: ->
    @current    
```

To correct this, change to the following:
```coffeescript
class Servers  
  constructor: ->
    @current = null

  set: (value) ->
    @current = value

  get: ->
    @current    
```

I believe any variable starting with @ outside of the scope of a method acts as an equivalent static type in `c#` 

So if you have a class like:

```coffeescript
class Servers
  @current = 123
  constructor: ->    
```
then without instantiation you'll get: 
```coffeescript
console.log Servers.current
```
> 123


## Chutzpuh & VS

The path to any references must be relative otherwise Chutzpuh will not find it.  For instance, the following will throw an `ReferenceError: Can't find variable: TimeOut in` error:

```coffeescript
## <reference path="/vendor/jasmine-1.3.1/jasmine.js" />
## <reference path="/scripts/models/timeout.js" />

describe "Timeout suite", ->
    it "initalise Timeout object", ->
        timeout = new TimeOut
```

When I add **../..** to the path all is good:
```coffeescript
## <reference path="../../vendor/jasmine-1.3.1/jasmine.js" />
## <reference path="../../scripts/models/timeout.js" />

describe "Timeout suite", ->
    it "initalise Timeout object", ->
        timeout = new TimeOut
```

## Callbacks

Whenever you pass a function to a class that is to act as a callback, that function much be called with parenthesis if no parameters are required.  For example:

```coffeescript
MyClass = class MyClass
	constructor: (@callback) ->
	doSomething: ->
		@callback()

myCallback = ->
	alert 'Hi'
	
myClass = new MyClass myCallback
``` 

You will note that the callback @**callback()** has '()'.  Without these, the callback function will not be called. 

## Arrays of objects

When using arrays, the `,` separator much be an indention back from the object itself.  This is illustrated below:
 
```coffeescript
myArray = [
		{id: "1", name: "abc"}
	,
		{id: "1", name: "xyz"}
]

```

## Call a method from within a class

To call an method you much either use `@` or `this.` notation. For example:

```coffeescript
MyClass = class MyClass
	constructor: ->
		@doSomething()
		this.doAnotherSomething()
	doSomething: ->
	doAnotherSomething: ->
		
```
If you omit the prefix the method will not get called.

## Lost scope via callback or attaching to different object

Here I have included a snippet from a class.  Please take note of the `_assess` method.  You will see a fat arrow `=>` being used.  I have used this here as the method is being attached to another / different object - in the `_start` method - and such the scope for the container class is lost. To avoid this, we used a fat arrow.  If we use a thin arrow `->` instead, the class properties `@minutes, @period, @interval & @timedOut` would not be accessible: 

```coffeescript
...
	_start: ->
		@internval = setInterval this._assess, @interval        

    _assess: =>                
        @minutes = @minutes + 1 if @minutes < @period
        if @minutes > @period            
            @internval = clearInterval @internval
            @timedOut() 
```


## Testing

Useful links:
    
	Jasmine - http://pivotal.github.io/jasmine/
	Jasmine-jquery - https://github.com/velesin/jasmine-jquery
	Chutzpah - http://chutzpah.codeplex.com/

### Testing where time is a factor

Whenever you need to test a feature that involves a time delay then you can use the Clock object in jasmine `jasmine.Clock.useMock()`

To illustrate this I have included an example below:

```coffeescript
describe "Timeout suite", ->   

 	flag  = false   
    restartFlag = false     
    timeout = null
    callback = ->
        flag = true
    restartCallback = ->
        restartFlag = true

    beforeEach ->
        timeout = new TimeOut 2, 2000, callback, restartCallback    

    it "restart called and callback triggered after a minute", ->   
        jasmine.Clock.useMock();
        timeout.start()
        timeout.restart()
        jasmine.Clock.tick(2100)
        timeout.restart()
                
        (expect restartFlag).toBeTruthy()        
        (expect timeout.internalBeenMoreThanAMinute).toBeFalsy()
```

> In the sample above a callback `restartCallback` - is called after a specific amount of time.  I know that 2+ seconds is enough time to wait and that the callback should have been called by then.  The `restartCallback` assigns `true` to the variable `restartFlag`.  It is this variable's value I am evaluating in the assertion. 
