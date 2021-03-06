# API Reference

## components schema

```javascript
{
	// lifecycle methods
	shouldComponentUpdate:     (props, state, this) => {}
	componentWillReceiveProps: (props, state, this) => {}
	componentWillUpdate:       (props, state, this) => {}
	componentDidUpdate:        (props, state, this) => {}
	componentWillMount:        (props, state, this) => {}
	componentDidMount:         (props, state, this) => {}
	componentWillUnmount:      (node) => {}
	
	// this.methods
	this.withAttr              ({String|String[]}, {Function|Function[]})
	this.forceUpdate:          (this?: {Object})
	this.setState:             ({Object})
	this.setProps:             ({Object})

	// render method
	render:                    (props, state, this) => {}
}
```

---

## creating hyperscript objects

```javascript
h(
	tag: {String}, 
	props?|children?: {Object} | {Array|Object}...,
	children?: {Array|Object}...
)

h('.wrap')
// <div class=wrap></div>

h('input[type=checkbox]')
// <input type=checkbox>

h('input', {value: 'text', onInput: ()=>{}})
// <input value=text>

h('div', 'Text')
// <div>Text</div>

h('div', h('span', 'Text'))
// <div><span>Text</span></div>

h('div', [h('h1', '1st'), h('h1', '2nd'), ...])
// <div><h1>1st</h1><h1>2nd</h1></div>

h('div', {innerHTML: '<script>alert('hello')</script>'});
// <div><script>alert('hello')</script></div>
```

---

## dio.createComponent

```
dio.createComponent({Function|Object})

// for example
var myComponent = dio.createComponent({
	render: () => { return h('div') }
});

// or with a function
var myComponent = dio.createComponent(function () {
	return {
		render: () => { return h('div') }
	};
});

```

---

## dio.createRender

```javascript
dio.createRender(component: {Function|Object}, mount?: {String|Element})
// where component is either a component
// created with dio.createComponent, 
// a hyperscript object 
// an object with a render method that 
// returns a hyperscript object 
// or a function that returns one of the above.

// While mount is either a selector or element reference
// if left blank this defaults to document.body
```

Components that do not return an object with a render function
or are itself an object with a render function will not feature
the component lifecycles and methods as presented above in the schema

Now lets go over some component examples.

```javascript
// function that returns a hyperscript object
function () {
	return h('div', 'Hello World')
}

// function that returns an object
function () {
	return {
		render: function () {
			return h('div', 'Hello World')
		}
	}
}

// hyperscript
h('div', 'Hello World')

// object with render method 
{
	render: function () {
		return h('div', 'Hello World')
	}
}

dio.createRender(componentPlaceholder)
```

Now lets go over some mount examples

```
document || document.querySelector('.myapp')
'.myapp'

dio.createRender(__, mountPlaceholder)

// note that the default for mount is document.body
```

---

## dio.createRouter

```
dio.createRouter(
	routes: {Object}, 
	rootAddress?: {String}, 
	onInitNavTo?: {String}
)
```

example router
```
dio.createRouter({
	'/': function () {
		dio.createRender(home)()
	},
	'/user/:id': function (data) {
		dio.createRender(user)(data)
	}
}, '/backend', '/user/sultan')

// The above firstly defines a set of routes
// '/' & '/user/:id' the second of which feature s data attribute
// that is passed to the callback function (data) in the form
// {id: value} i.e /user/sultan will output {id: 'sultan'}
// '/backend' specifies the root address to be used
// i.e your app lives in url.com/backend rather than on the root /
// and the third argument '/user/sultan' specifies
// an initial route to address that is navigated to
// initially. The last two arguments are optional.
```

You can then assign this route to a variable and use it to navigate across views

```
var myrouter = ...

myrouter.nav('/user/sultan')
myrouter.go(-2) // like calling myrouter.back() twice
myrouter.back()
myrouter.forward()
```

---

## dio.createStore

Alot like redux createStore, inface it's exactly like redux createStore
with the addition of `.connect` that accepts a render insance of component
and mount with which to update everytime the store is updated.
Which is mostly a short hand for creating a listerner with `.subscribe`
that updates your component on state changes.

```
var store = dio.createStore(reducer: {Function})
// or
var store = dio.createStore(object of reducers: {Object})


store.dispatch({type: '' ...})
// dispatch an action

store.getState()
// returns the current state

store.subscribe(listener: {Function})
// called everytime the state is updated

store.connect(render: {Function})
store.connect(render: {Function|Object}, element: '.myapp')
// if you provide an element to .connect it assumes the render
// passed is not a render instance but a component and will then
// proceed to create a render instance.
```

---

## dio.createHTML

Like the name suggests this methods outputs the html 
of a specific component/render instance with the props passed
to it

```
dio.createHTML(component, props, children)
// you can also use this on plain hyperscript objects
```

---

## dio.request

make ajax requests

```
// returns a stream
dio.request(
	url: {String}, 
	payload?: {Object},
	enctype?: {String}, 
	// 'file' | 'json' | 'text', default: 'application/x-www-form-urlencoded'
	callback?: {Function}, 
	withCredentials?: {Boolean} 
	// true/false for CORS requests 
)

// example

dio.request('/url')
	.then((res)=>{return res})
	.then((res)=>{'do something'})
	.catch((err)=>{throw err});

```

---

## dio.createStream

```
var foo = dio.createStream('initial value')
foo('changed value')
foo() // => 'changed value'

// map
var bar = foo.map(function(fooValue){
	return fooValue + ' and bar';
});

bar() // => 'changed value and bar';
foo('hello world')
bar() // => 'hello world and bar'

// combine two or more streams
var faz = dio.createStream.combine(function(fooValue, barValue){
	return fooValue() + barValue();
}, foo, bar);

foo(1)
bar(2)

faz() // => 3

// listen for changes to a value
// note: this behaves kind of like a promise but it is not a promise
// ergo no polyfill needed.
faz.then(function(fazValue){
	console.log(fazValue)
});

faz('changed') // => 'changed'

// or chained
faz.then(fn).then(fn)....
```

---

## dio.createFunction

```
var foo = dio.createFunction((a, b)=>{}, ['a', 'b'], event: {Boolean})
// setting the event argument does a preventDefault if the caller is an event
// for example:

onClick: dio.createFunction((a) => {'look no e.preventDefault()'}, ['a'], true)

```

---

## dio.animate

```javascript
dio.animate.flip(className, duration, transform, transformOrigin, easing)(Element)
dio.animate.transition(className)(node, callback)
```

for example `dio.animate.flip` can be used within a render as follows

```javascript
render: function () {
	return h('.card', 
		{onclick: dio.animate.flip('active-state', 200)}, 
		''
	)
}
```
since `dio.animate.flip(...)` returns a function this is the same as

```javascript
dio.animate('active-state', 200)(Element) // returns duration
``` 

another animation helper is `animate.transition`

```javascript
// within a method
handleDelete: function (e) {
	var 
	node = e.currentTarget,
	self = this
	
	// animate node out then update state
	dio.animate.transition('slideUp')(node, function(){
		store.dispatch({type: 'DELETE', id: 1234})
	})
}
```