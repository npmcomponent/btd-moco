*This repository is a mirror of the [component](http://component.io) module [btd/moco](http://github.com/btd/moco). It has been modified to work with NPM+Browserify. You can install it using the command `npm install npmcomponent/btd-moco`. Please do not open issues or send pull requests against this repo. If you have issues with this repo, report it to [npmcomponent](https://github.com/airportyh/npmcomponent).*
# Moco

It is a small library that create object-like and array-like models and collections. I do not want to use anything that already in es5. So this will work only in modern browsers (IE8 100% not supported).

## Installation

Node:

	npm install --save moco
    
Browser:

	bower install btd/moco
    
## Usage

Api exports just 2 functions: `model` and `collection`.

## Models

```js
var model = require('moco').model;

var User = model()
	.attr('_id', { primary: true }) // this attribute will be used as primary key
    .attr('name') // just simple attribute
    .attr('email', { get: function() {
 		return this.name + '@example.com';
    } }) // this is a getter and it called each time you try access this model
    .attr('title', { default: 'Unknow' })// this will be used to fill undefined attribute while creation
    
var u1 = new User({ _id: 1, name: 'den'});
console.log(u1._id); // 1
console.log(u1.primary); // 1

console.log(u1.email); // den@example.com
console.log(u1.title); // Unknow

u1.name = 'daniel';

console.log(u1.email); // daniel@example.com
```

When you call `model()` it return new predefined constructor function.

### .attr(name[, options])

Define new attribute on this model type. Only defined attributes will be saved to model (used `Object.seal`). By default created getter and setter for this attribute.

Supported options:

`get` - it is a function that will be used for getter, if it is defined for this attibute setter will not be defined.

`default` - it can be anything, that will be assigned to attribute, while creation if its values is undefined. If `default` is a function, it will be called, in context of model instance.

`primary` - it is a boolean field, when it is set to true you will can use this methods:

`#isNew()` - it will return true|false if primary attribute filled. If primary is false throw exception.

`#primary` - it is an alias for attribute with primary: true.

It is not possible to create 2 attributes with primary: true and all redefenitions of attributes ignored.

### .use(fn)

This method can be used to mix something general to model.

### .create(obj)

This method wrap obj to model if it is not and return it.

### #[attr]

For each defined with `.attr` attributes will be created getter and setter.

### Enumerable properties and defined attributes

Each defined attribute created enumerable, so you can use this model with any library that work with objects and it will be just works.

```js
Object.keys(u1) // ['_id', 'name', 'email', 'title']
```
If you will try to save anything that is not defined it will not be saved:
```js
u1.position = 'developer';
u1.position // undefined
```

### Events

When model attribute changed it emit event 'change' and 'change:'+attributeName with value and previous value. Also this event can be handled on Model level, as each model type also can emit events.

Model type emit event 'initialize' with create model.

See [component/emitter](https://github.com/component/emitter) about how to handle events.

## Collection

Collection mimic javascript native `Array` everywhere it is possible. It have all methods that `Array` has and they do everything that native method do. But also it wrap object to Model instance if it is not and emit events:

`add` emitted with instance when model added to collection.

`remove` emitter with instance when model removed from colleciton.

### .use(fn) 

It used in the same way as in model types.

### Example

```js
var collection = require('moco').collection;

var Users = collection(User).use(utils.byId);

var users = new Users([u1, { _id: 2, name: 'lisa' }]);
users.length; // 2
// utils.byId add 2 methods byId, removeById, of course Model should have primary attribute
users.byId(1).name // 'daniel'

users.byId(2).name // 'lisa'

users.forEach(function(user) {
	console.log(user.email);
})
// daniel@example.com
// lisa@example.com
```

## Collection plugins

There are 3 built in collection plugins:

### collection.byId

It create internal cache for primary attribute of models it contains and add methods `#byId(primary)` and `#removeById(primary)`.
Nothing in this plugin checks that collection contains 2 model with the same primary attribute.

### collection.byField(attr)

It create internal cache for specified attribute and add method `#by[Attr](attr)` [Attr] there it is name with capital letter.

### collection.modelsChanges

It add ability to collection emit `change` event when nested model changed.

### model.nestedObjects

It emit change event every time nested model or collection changed. It can works together with `collection.modelsChanges` and proxy these events too.

Small example:

```javascript
var NestedModel1 = moco.model()
    .attr('a');

var NestedCollection = moco.collection(NestedModel1)
    .use(moco.collection.modelsChanges);

var NestedModel2 = moco.model()
    .attr('a', { model: NestedModel1 })
    .attr('b', { collection: NestedCollection })
    .use(moco.model.nestedObjects);

var nm = new NestedModel1({ a: 'a' });

var c = new NestedCollection([ nm ]);

var m = new NestedModel2({ b: c });

/*
    this will emit 4 events on m:
    change:b.a with value 'b' and prev 'a'
    change:b with value === prev === c
    change with previous 2 events
*/
nm.a = 'b';
```

## License

The MIT License (MIT)

Copyright (c) 2013 Denis Bardadym

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