---
title: Non-enumerable properties in JavaScript
layout: post
categories: javascript
description: "JavaScript ES5 supports non-enumerable and read-only properties. Non-enumerable properties will be ignored by JSON.stringify(). I spent way too long figuring this out tonight."
redirect_to: http://joshua.poehls.me/2013/non-enumerable-properties-in-javascript
---

It isn't very common in the wild but JavaScript (as of ES5) does support non-enumerable properties.
That is to say, objects can have properties that don't show up when you do a [`for...in`][3] loop over the object or use [`Object.keys()`][2] to get an array of property names.

I spent way too long tonight figuring out why a property was not being
included in the [`JSON.stringify()`][4] output because I didn't remember that this was possible. You see, `stringify()` works by enumerating the object's properties.
So it makes sense that non-enumerable properties would be ignored.

Let me explain.

<pre data-language="javascript">
var person = { age: 18 };
person.name = 'Joshua';
person['color'] = 'Red';

Object.keys(person); // ['age', 'name', 'color']
</pre>

That is how you typically create and assign properties. We know that they are enumerable because they show up when you call `Object.keys(person)`.

To create a non-enumerable property we have to use [`Object.defineProperty()`][1]. A special method for creating properties on an object.

<pre data-language="javascript">
var person = { age: 18 };
Object.defineProperty(person, 'name', { value: 'Joshua', enumerable: false });

person.name; // 'Joshua'
Object.keys(person); // ['age']
</pre>

See there? Our `name` property didn't show up because it wasn't enumerable. Consequently, doing `JSON.stringify(person)` would not include the `name` property either.

Hopefully this will help you the next time a property is missing from your stringified object and you can't figure out why. Good luck!

### Bonus!

`Object.defineProperty()` is also how you create **read-only** properties. Didn't know you could do that in JavaScript? Time to [study up][1]!

[1]: https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Object/defineProperty
[2]: https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Object/keys
[3]: https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Statements/for...in
[4]: https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/JSON/stringify