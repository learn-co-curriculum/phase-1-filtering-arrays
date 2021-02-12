# Filtering Arrays

## Learning Goals

- Explain the concept of filtering an array.
- Write a `filter()` function that accepts a function as one of its arguments.
- Understand what a _callback function_ is.
- Define what makes a function _pure_ and explain why _pure functions_ are often
  preferable to _impure functions_.

## Introduction

In the world of programming, we often work with arrays, and there are a few
common actions you'll see repeated over and over. One of the most common is
transforming every element in an array to another value; for example, taking an
array of numbers and squaring each one (`[1, 2, 3]` -> `[1, 4, 9]`). Another
common action is searching through the array and only returning elements that
match a certain condition. For example, taking the same array of numbers and
only returning values greater than one (`[1, 2, 3]` -> `[2, 3]`).

In the JavaScript world, we refer to that search process as _filtering_ an
array, and in this lesson we're going to build our own `filter()` function.

## Filter

Imagine that we have a collection of Flatbook user objects in an array:

```js
const users = [
  {
    firstName: 'Niky',
    lastName: 'Morgan',
    favoriteColor: 'Blue',
    favoriteAnimal: 'Jaguar'
  },
  {
    firstName: 'Tracy',
    lastName: 'Lum',
    favoriteColor: 'Yellow',
    favoriteAnimal: 'Penguin'
  },
  {
    firstName: 'Josh',
    lastName: 'Rowley',
    favoriteColor: 'Blue',
    favoriteAnimal: 'Penguin'
  },
  {
    firstName: 'Kate',
    lastName: 'Travers',
    favoriteColor: 'Red',
    favoriteAnimal: 'Jaguar'
  },
  {
    firstName: 'Avidor',
    lastName: 'Turkewitz',
    favoriteColor: 'Blue',
    favoriteAnimal: 'Penguin'
  },
  {
    firstName: 'Drew',
    lastName: 'Price',
    favoriteColor: 'Yellow',
    favoriteAnimal: 'Elephant'
  }
];
```

It's easy enough to iterate over that collection and print out everyone's first
name:

```js
function firstNamePrinter (collection) {
  for (const user of collection) {
    console.log(user.firstName);
  }
}

firstNamePrinter(users);
// LOG: Niky
// LOG: Tracy
// LOG: Josh
// LOG: Kate
// LOG: Avidor
// LOG: Drew
```

It's also not too difficult to print out only users whose favorite color is
blue:

```js
function blueFilter (collection) {
  for (const user of collection) {
    if (user.favoriteColor === 'Blue') {
      console.log(user.firstName);
    }
  }
}

blueFilter(users);
// LOG: Niky
// LOG: Josh
// LOG: Avidor
```

Now what if we want to filter our collection of users for those whose favorite
color is red? We could define an entirely new function, `redFilter()`, but that
seems wasteful. Instead, let's just pass in the color that we want to filter
for:

```js
function colorFilter (collection, color) {
  for (const user of collection) {
    if (user.favoriteColor === color) {
      console.log(user.firstName);
    }
  }
}

colorFilter(users, 'Red');
// LOG: Kate
```

Nice, we've extracted some of the hard-coded logic out of the function, making
it more generic and reusable. However, now we want to filter our users based on
whose favorite animal is a jaguar, and our `colorFilter()` won't work. Let's
abstract the function a bit further:

```js
function filter (collection, attribute, value) {
  for (const user of collection) {
    if (user[attribute] === value) {
      console.log(user.firstName);
    }
  }
}

filter(users, 'favoriteAnimal', 'Jaguar');
// LOG: Niky
// LOG: Kate
```

This is getting slightly ridiculous by this point. That is **way** too much
logic to be putting on the shoulders of our poor little filter function. Let's
extract the comparison logic into a separate function:

```js
function filter (collection) {
  for (const user of collection) {
    if (likesElephants(user)) {
      console.log(user.firstName);
    }
  }
}

function likesElephants (user) {
  return user['favoriteAnimal'] === 'Elephant';
}

filter(users);
// LOG: Drew
```

That separation of concerns feels nice. `filter()` doesn't remotely care what
happens inside `likesElephants()`; it simply delegates the comparison and then
trusts that `likesElephants()` correctly returns `true` or `false`. We're almost
at the finish line, but there's one final abstraction we can make: right now,
our `filter()` function can only make comparisons using `likesElephants()`. If
we want to use a different comparison function, we'd have to rewrite `filter()`.
However, there is... another way.

<picture>
  <source srcset="https://curriculum-content.s3.amazonaws.com/web-development/js/looping-and-iteration/filter-readme/maybe_theres_another_way.webp" type="image/webp">
  <source srcset="https://curriculum-content.s3.amazonaws.com/web-development/js/looping-and-iteration/filter-readme/maybe_theres_another_way.gif" type="image/gif">
  <img src="https://curriculum-content.s3.amazonaws.com/web-development/js/looping-and-iteration/filter-readme/maybe_theres_another_way.gif" alt="Maybe there's another way.">
</picture>

## Passing functions

We know we can pass numbers, strings, objects, and arrays into a function as
arguments, but did you know we can also **pass functions into other functions**?
We'll go into this in much greater depth in an upcoming lesson, but it's
important to start thinking about this concept now: in JavaScript, **functions
are objects**. Specifically, they are objects with a special, hidden code
property that can be invoked. This is how we pass an object into a function:

```js
function iReturnThings (thing) {
  return thing;
}

iReturnThings({ firstName: 'Brendan', lastName: 'Eich' });
// => {firstName: "Brendan", lastName: "Eich"}
```

And this is how we pass a function into a function:

```js
iReturnThings(function () { return 4 + 5; });
// => ƒ () { return 4 + 5; }
```

Notice that a representation of the passed-in function was returned, but **it
was not invoked**. The `iReturnThings()` function accepted the passed-in
function as its lone argument, `thing`. As with all arguments, `thing` was then
available everywhere inside `iReturnThings()` as a local variable. When we
passed a function into `iReturnThings()`, the `thing` variable contained that
function. Currently, all `iReturnThings()` does is return whatever value is
stored inside `thing`. However, if we know `thing` contains a function, we can
do a piece of awesome, function-y magic to it: **we can invoke it** and return
the function's result:

```js
function iInvokeThings (thing) {
  return thing();
}

iInvokeThings(function () { return 4 + 5; });
// => 9

iInvokeThings(function () { return 'Hello, ' + 'world!'; });
// => "Hello, world!"
```

We pass in a function as the lone argument, store it inside the `thing`
variable, and then use the invocation operator (a pair of parentheses) to invoke
the stored function: `thing()`.

***NOTE***: As we dive deeper and deeper into functional programming in
JavaScript, it bears repeating: this is **very** complicated material!
JavaScript isn't the only language to treat functions as first-class objects,
but it's by far the most common and likely the first time you're encountering
any of this stuff. If you're struggling with the new concepts, don't sweat it!
Set aside some extra time to re-read and practice, and make sure you're coding
along with every example we cover in the lessons.

### Callback Functions

If you've done any outside reading on JavaScript, you've probably come across
the name of the pattern we just introduced: _callback functions_. When we pass a
function into another function wherein it might be invoked, we refer to the
passed function as a _callback_. The term derives from the fact that the
function isn't invoked immediately — instead it's _called back_, or invoked at a
later point.

You may have noticed, but all of our callback functions so far have been
_anonymous functions_; that is, we haven't assigned them an identifier. You're
welcome to name your callback functions if you'd like, but generally it just
clutters things up if you only use the callback function in one place. And,
anyway, we already have a way to refer to them: by the name of the parameter
into which they're passed! For example:

```js
function main (cb) {
  console.log(cb());
}

main(function () { return "After I get passed to the main() function as the only argument, I'm stored in the local 'cb' variable!"});
// LOG: After I get passed to the main() function as the only argument, I'm stored in the local 'cb' variable!
```

1. We passed an anonymous function, `function () { return "After I get passed...
   }`, as the lone argument to our invocation of `main()`.
2. `main()` stored the passed-in function in the local `cb` variable and then
   invoked the callback function.
3. The invoked callback returned its long string, which was `console.log()`-ed
   out in `main()`.

Because a callback function is invoked inside another function, we can forward
to it any arguments passed to the outer function. For example:

```js
function greet (name, cb) {
  return cb(name);
}

greet('Ada Lovelace', function (name) { return 'Hello there, ' + name; });
// => "Hello there, Ada Lovelace"

function doMath (num1, num2, cb) {
  return cb(num1, num2);
}

doMath(42, 8, function (num1, num2) { return num1 * num2; });
// => 336
```

This behavior makes callbacks the perfect companion for structuring our
`filter()` function in a slim, reusable way:

```js
const users = [
  { firstName: 'Niky',   lastName: 'Morgan',    favoriteColor: 'Blue',   favoriteAnimal: 'Jaguar' },
  { firstName: 'Tracy',  lastName: 'Lum',       favoriteColor: 'Yellow', favoriteAnimal: 'Penguin' },
  { firstName: 'Josh',   lastName: 'Rowley',    favoriteColor: 'Blue',   favoriteAnimal: 'Penguin' },
  { firstName: 'Kate',   lastName: 'Travers',   favoriteColor: 'Red',    favoriteAnimal: 'Jaguar' },
  { firstName: 'Avidor', lastName: 'Turkewitz', favoriteColor: 'Blue',   favoriteAnimal: 'Penguin' },
  { firstName: 'Drew',   lastName: 'Price',     favoriteColor: 'Yellow', favoriteAnimal: 'Elephant' }
];

function filter (collection, cb) {
  for (const user of collection) {
    if (cb(user)) {
      console.log(user.firstName);
    }
  }
}

filter(users, function (user) { return user.favoriteColor === 'Blue' && user.favoriteAnimal === 'Penguin'; });
// LOG: Josh
// LOG: Avidor

filter(users, function (user) { return user.favoriteColor === 'Yellow'; });
// LOG: Tracy
// LOG: Drew
```

Our `filter()` function doesn't know or care about any of the comparison logic
encapsulated in the callback function. All it does is take in a collection and a
callback and `console.log()` out the `firstName` of every `user` object that
makes the callback return `true`.

### Pure functions

One final note about `filter()` and manipulating objects in JavaScript. We
touched on this in the discussions of _destructive_ and _nondestructive_
operations, but there's some function-specific terminology that's important to
know. A function in JavaScript can be _pure_ or _impure_.

If a _pure function_ is repeatedly invoked with the same set of arguments, the
function will **always return the same result**. Its behavior is predictable.
Additionally, invoking the function has no external side-effects such as making
a network or database call or altering any object(s) passed to it as an
argument.

_Impure functions_ are the opposite: the return value is not predictable, and
invoking the function might make network or database calls or alter any objects
passed in as arguments.

This function is impure because the return value is not predictable:

```js
function randomMultiplyAndFloor () {
  return Math.floor(Math.random() * 100);
}

randomMultiplyAndFloor();
// => 53
randomMultiplyAndFloor();
// => 66
```

This one's impure because it alters the passed-in object:

```js
const ada = {
  name: 'Ada Lovelace',
  age: 202
};

function happyBirthday (person) {
  console.log(`Happy birthday, ${person.name}! You're ${++person.age} years old!`);
}

happyBirthday(ada);
// LOG: Happy birthday, Ada Lovelace! You're 203 years old!

happyBirthday(ada);
// LOG: Happy birthday, Ada Lovelace! You're 204 years old!

ada;
// => {name: "Ada Lovelace", age: 204}
```

When possible, it's generally good to avoid impure functions for the following
two reasons:

1. Predictable code is good. If you can be sure that a function will always
   return the same value when provided the same inputs, it makes writing tests
   for that function a cinch.
2. Because pure functions don't have side effects, it makes debugging a lot
   easier. Imagine that our code errors out due to an array that doesn't contain
   the correct properties.
    - If that array was returned from a pure function, our debugging process
      would be linear and well-scoped. We would first check what inputs were
      provided to the pure function. If the inputs are correct, that means the
      bug is inside our pure function. If the inputs aren't correct, then we
      figure out why they aren't correct. Case closed!
    - If, however, the array is modified by impure functions, we'd have to
      follow the data around on a wild goose chase, combing through each impure
      function to see where and how the array is modified.

***Top Tip***: The fewer places a particular object can be modified, the fewer
places we have to look when debugging.

Here's a pure take on our `randomMultiplyAndFloor()` function:

```js
function multiplyAndFloor (num) {
  return Math.floor(num * 100);
}

const randNum = Math.random();

randNum;
// => 0.9123939589869237

multiplyAndFloor(randNum);
// => 91
multiplyAndFloor(randNum);
// => 91
```

And one that returns a new object instead of mutating the passed-in object:

```js
const adaAge202 = {
  name: 'Ada Lovelace',
  age: 202
};

function happyBirthday (person) {
  const newPerson = Object.assign({}, person, { age: person.age + 1 });

  console.log(`Happy birthday, ${newPerson.name}! You're ${newPerson.age} years old!`);

  return newPerson;
}

const adaAge203 = happyBirthday(adaAge202);
// LOG: Happy birthday, Ada Lovelace! You're 203 years old!

adaAge202;
// => {name: "Ada Lovelace", age: 202}

adaAge203;
// => {name: "Ada Lovelace", age: 203}
```

## Tying it all together

As a final challenge, let's rewrite our `filter()` function as a pure function
that returns a new array containing the filtered elements:

```js
const users = [
  { firstName: 'Niky',   lastName: 'Morgan',    favoriteColor: 'Blue',   favoriteAnimal: 'Jaguar' },
  { firstName: 'Tracy',  lastName: 'Lum',       favoriteColor: 'Yellow', favoriteAnimal: 'Penguin' },
  { firstName: 'Josh',   lastName: 'Rowley',    favoriteColor: 'Blue',   favoriteAnimal: 'Penguin' },
  { firstName: 'Kate',   lastName: 'Travers',   favoriteColor: 'Red',    favoriteAnimal: 'Jaguar' },
  { firstName: 'Avidor', lastName: 'Turkewitz', favoriteColor: 'Blue',   favoriteAnimal: 'Penguin' },
  { firstName: 'Drew',   lastName: 'Price',     favoriteColor: 'Yellow', favoriteAnimal: 'Elephant' }
];

function filter (collection, cb) {
  const newCollection = [];

  for (const user of collection) {
    if (cb(user)) {
      newCollection.push(user);
    }
  }

  return newCollection;
}

const bluePenguinUsers = filter(users, function (user) { return user.favoriteColor === 'Blue' && user.favoriteAnimal === 'Penguin'; });

bluePenguinUsers;
// => [{ firstName: "Josh", lastName: "Rowley", favoriteColor: "Blue", favoriteAnimal: "Penguin" }, { firstName: "Avidor", lastName: "Turkewitz", favoriteColor: "Blue", favoriteAnimal: "Penguin" }]

const yellowUsers = filter(users, function (user) { return user.favoriteColor === 'Yellow'; });

yellowUsers;
// => [{ firstName: "Tracy", lastName: "Lum", favoriteColor: "Yellow", favoriteAnimal: "Penguin" }, { firstName: "Drew", lastName: "Price", favoriteColor: "Yellow", favoriteAnimal: "Elephant" }]

users.length;
// => 6
```

Woohoo! We successfully built a clone of JavaScript's built-in `.filter()` array
method!

Wait... **a** ***clone?!***

<picture>
  <source srcset="https://curriculum-content.s3.amazonaws.com/web-development/js/looping-and-iteration/filter-readme/no_shortcuts.webp" type="image/webp">
  <source srcset="https://curriculum-content.s3.amazonaws.com/web-development/js/looping-and-iteration/filter-readme/no_shortcuts.gif" type="image/gif">
  <img src="https://curriculum-content.s3.amazonaws.com/web-development/js/looping-and-iteration/filter-readme/no_shortcuts.gif" alt="Our journey has never been one of shortcuts or settling for less.">
</picture>

Yep, sorry about that. All JavaScript arrays come with their own `.filter()`
method:

```js
[1, 2, 3, 4, 5].filter(function (num) { return num > 3; });
// => [4, 5]
```

The method accepts one argument, a callback function that it will invoke with
each element in the array. For each element passed to the callback, if the
callback's return value is truthy, that element is copied into a new array. If
the callback's return value is falsy, the element is filtered out. After
iterating over every element in the collection, `.filter()` returns the new
array.

Now that you've built your own `filter()` method, you have a much deeper grasp
of how JavaScript's built-in `Array.prototype.filter()` method works.

## Resources

- [MDN — `Array.prototype.filter()`][filter]
- [Tutorial Horizon — Pure vs. Impure Functions][pure]

[filter]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter
[pure]: http://javascript.tutorialhorizon.com/2016/04/24/pure-vs-impure-functions/
