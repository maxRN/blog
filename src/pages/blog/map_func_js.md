---
layout: "../../layouts/BlogPost.astro"
title: "Map with mutating function in JavaScript"
description: "Watchout when using `.map` in JavaScript"
pubDate: "Dec 12 2022"
---

The [map][map] Array function in JavaScript applies a function to each element
in the array and _returns_ a new array.
It is a [copying method][copy_func] which means it returns a new copy of the array,
which in turn means that usually you have to do something like this

```javascript
myArray = myArray.map((el) => el++);
```

in order to update `myArray`. If you just write

```javascript
myArray.map((el) => el++);
```

then the call to map will create a copy of the array, apply the function to each element
and then return this copy.
In this case however it goes into the void.

Usually I notice errors like this immediately, however recently I wanted to parse the
date strings on a JSON object into actual Date objects.
My first instinct was to reach for a map:

```javascript
myArray.map((el) => {
  const newEl = el
  newEl.date = new Date(Date.parse(el.date))
}
```

There are multiple errors in this small snippet of code:

1. I forgot to assign the returned array clone to `myArray`.
1. I forgot to return the "updated" `newEl` inside the callback function.
1. I wanted to leave the underlying array alone so I tried creating `newEl`, but this actually does not create a new object,
   but `newEl` is not pointing to `el` so the call `newEl.date = ...` actually transforms the underlying array.

The code above actually solved my problem but the problem is that it's not working as I intented it to.
The code is ineffiecient because the clone created by `.map` is never used and
it is also confusing that I'm creating a reference to each element even though that does not do anything.

I saw two options of improving this:

1. Make `newEl` an actual deep copy of `el` and then returning this real copy
   as well as assigning the new array to `myArray`.
1. Use `.forEach` instead to avoid creating a new clone and also to make the intent more clear.

The way I initially discovered the error was by accident.
I was looking over the diff in the merge request and when I read the snippet above,
I was very confused how my code worked.
This is when I realized that I had made multiple mistakes and I also had really wrong assumptions
about how assignments in JavaScript work (they don't create a copy but just hold a reference to the underlying object).

[map]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map
[copy_func]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array#copying_methods_and_mutating_methods
