# Filtering Arrays

## Learning Goals

- Explain the concept of filtering an array
- Build our own version of JavaScript's `Array.prototype.filter()` method
- Define what makes a function _pure_ and explain why _pure functions_ are often
  preferable to _impure functions_
- Use `Array.prototype.filter()`

## Introduction

| Use Case                                                          | Method                |
| ----------------------------------------------------------------- | --------------------- |
| Executing a provided callback on each element                     | `forEach()`           |
| Finding a single element that meets a condition                   | `find()`, `indexOf()` |
| ** Finding and returning a list of elements that meet a condition | `filter()`            |
| Modifying each element and returning the modified array           | `map()`               |
| Creating a summary or aggregation of values in an array           | `reduce()`            |

We've seen the `Array` methods available in JavaScript to find a _single_
element, but sometimes we want to return _all_ elements that match a certain
condition. For example, we might want to search through an array and return
values greater than one (`[1, 2, 3]` -> `[2, 3]`). In the JavaScript world, we
refer to that search process as _filtering_ an array. In this lesson we're going
to build our own `filter()` function.

**Note:** In this lesson, we'll be creating our own `filter()` function and
showing how JavaScript's built-in `filter()` _method_ can save us a lot of
work. The built-in `filter()` method belongs to JavaScript's `Array` prototype.
As we've learned, this means that it is _called on_ an instance of an array. For
example, we can call the `Array.prototype.push()` method as follows:

```js
[1, 2, 3].push(4);
```

The version of the `filter()` method that we'll be creating, on the other hand,
is a _function_. Instead of calling it on an array, we will pass the array to it
as an argument: `filter(arr)`. Because our version will share the same name as
the built-in method, paying attention to this difference will help avoid
confusion.

## Filter

Let's revisit the example we looked at in an earlier lesson, using the array of
Flatbook user objects:

```js
const users = [
  {
    firstName: "Niky",
    lastName: "Morgan",
    favoriteColor: "Blue",
    favoriteAnimal: "Jaguar",
  },
  {
    firstName: "Tracy",
    lastName: "Lum",
    favoriteColor: "Yellow",
    favoriteAnimal: "Penguin",
  },
  {
    firstName: "Josh",
    lastName: "Rowley",
    favoriteColor: "Blue",
    favoriteAnimal: "Penguin",
  },
  {
    firstName: "Kate",
    lastName: "Travers",
    favoriteColor: "Red",
    favoriteAnimal: "Jaguar",
  },
  {
    firstName: "Avidor",
    lastName: "Turkewitz",
    favoriteColor: "Blue",
    favoriteAnimal: "Penguin",
  },
  {
    firstName: "Drew",
    lastName: "Price",
    favoriteColor: "Yellow",
    favoriteAnimal: "Elephant",
  },
];
```

We know we can iterate over that collection using `for...of` and print out
everyone's first name:

```js
function firstNamePrinter(collection) {
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

We also know how to print out only users whose favorite color is blue:

```js
function blueFilter(collection) {
  for (const user of collection) {
    if (user.favoriteColor === "Blue") {
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
seems wasteful. Instead, let's just pass in the color that we want to filter for
as an argument:

```js
function colorFilter(collection, color) {
  for (const user of collection) {
    if (user.favoriteColor === color) {
      console.log(user.firstName);
    }
  }
}

colorFilter(users, "Red");
// LOG: Kate
```

Nice! We've extracted some of the hard-coded logic out of the function, making
it more generic and reusable. However, now we want to filter our users based on
whose favorite animal is a jaguar, and our `colorFilter()` function won't work.
Let's abstract the function a bit further:

```js
function filter(collection, attribute, value) {
  for (const user of collection) {
    if (user[attribute] === value) {
      console.log(user.firstName);
    }
  }
}

filter(users, "favoriteAnimal", "Jaguar");
// LOG: Niky
// LOG: Kate
```

So our function is definitely getting more abstract, but what if we wanted to
filter by two attributes? We'd have to do something like this:

```js
function filter(collection, attribute1, value1, attribute2, value2) {
  for (const user of collection) {
    if (user[attribute1] === value1 && user[attribute2] === value2) {
      console.log(user.firstName);
    }
  }
}

filter(users, "favoriteAnimal", "Jaguar", "favoriteColor", "Blue");
// LOG: Niky
```

This is getting slightly ridiculous by this point. That is **way** too much
logic to be putting on the shoulders of our poor little filter function. Plus,
now our filter will only work if we're filtering by two attributes. To fix this,
we can extract the comparison logic into a separate function:

```js
function filter(collection) {
  for (const user of collection) {
    if (likesElephants(user)) {
      console.log(user.firstName);
    }
  }
}

function likesElephants(user) {
  return user["favoriteAnimal"] === "Elephant";
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
However, there is another way: we can use a callback function!

Let's refactor our filter function to take a callback:

```js
const users = [
  {
    firstName: "Niky",
    lastName: "Morgan",
    favoriteColor: "Blue",
    favoriteAnimal: "Jaguar",
  },
  {
    firstName: "Tracy",
    lastName: "Lum",
    favoriteColor: "Yellow",
    favoriteAnimal: "Penguin",
  },
  {
    firstName: "Josh",
    lastName: "Rowley",
    favoriteColor: "Blue",
    favoriteAnimal: "Penguin",
  },
  {
    firstName: "Kate",
    lastName: "Travers",
    favoriteColor: "Red",
    favoriteAnimal: "Jaguar",
  },
  {
    firstName: "Avidor",
    lastName: "Turkewitz",
    favoriteColor: "Blue",
    favoriteAnimal: "Penguin",
  },
  {
    firstName: "Drew",
    lastName: "Price",
    favoriteColor: "Yellow",
    favoriteAnimal: "Elephant",
  },
];

function filter(collection, cb) {
  for (const user of collection) {
    if (cb(user)) {
      console.log(user.firstName);
    }
  }
}

filter(users, function (user) {
  return user.favoriteColor === "Blue" && user.favoriteAnimal === "Penguin";
});
// LOG: Josh
// LOG: Avidor

filter(users, function (user) {
  return user.favoriteColor === "Yellow";
});
// LOG: Tracy
// LOG: Drew
```

Our `filter()` function doesn't know or care about any of the comparison logic
encapsulated in the callback function. All it does is take in a collection and a
callback and `console.log()` out the `firstName` of every `user` object that
makes the callback return `true`. And because we've extracted the logic into a
separate function, our `filter` now works regardless of how many conditions we
want to filter on.

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
function randomMultiplyAndFloor() {
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
  name: "Ada Lovelace",
  age: 202,
};

function happyBirthday(person) {
  console.log(
    `Happy birthday, ${person.name}! You're ${++person.age} years old!`
  );
}

happyBirthday(ada);
// LOG: Happy birthday, Ada Lovelace! You're 203 years old!

happyBirthday(ada);
// LOG: Happy birthday, Ada Lovelace! You're 204 years old!

ada;
// => {name: "Ada Lovelace", age: 204}
```

By using the `++` operator on the `age` property, we're changing the passed-in
object in our function.

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
   - If, however, the array is modified by impure functions, we'd have to follow
     the data around on a wild goose chase, combing through each impure function
     to see where and how the array is modified.

> **Top Tip**: The fewer places a particular object can be modified, the fewer
> places we have to look when debugging.

Here's a pure take on our `randomMultiplyAndFloor()` function:

```js
function multiplyAndFloor(num) {
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

The randomization is now happening _outside_ our function, so the return value
of `multiplyAndFloor` will always be the same for a given input.

For our `happyBirthday()` function, we could make it pure by first cloning the
passed-in object then modifying the clone instead. We'll learn how to do that in
a later lesson.

## Tying it all together

As a final challenge, let's rewrite our `filter()` function so it returns an
array containing the filtered elements. We'll make it a pure function by
creating a new array to contain the filtered elements and returning that array
at the end:

```js
const users = [
  {
    firstName: "Niky",
    lastName: "Morgan",
    favoriteColor: "Blue",
    favoriteAnimal: "Jaguar",
  },
  {
    firstName: "Tracy",
    lastName: "Lum",
    favoriteColor: "Yellow",
    favoriteAnimal: "Penguin",
  },
  {
    firstName: "Josh",
    lastName: "Rowley",
    favoriteColor: "Blue",
    favoriteAnimal: "Penguin",
  },
  {
    firstName: "Kate",
    lastName: "Travers",
    favoriteColor: "Red",
    favoriteAnimal: "Jaguar",
  },
  {
    firstName: "Avidor",
    lastName: "Turkewitz",
    favoriteColor: "Blue",
    favoriteAnimal: "Penguin",
  },
  {
    firstName: "Drew",
    lastName: "Price",
    favoriteColor: "Yellow",
    favoriteAnimal: "Elephant",
  },
];

function filter(collection, cb) {
  const newCollection = [];

  for (const user of collection) {
    if (cb(user)) {
      newCollection.push(user);
    }
  }

  return newCollection;
}

const bluePenguinUsers = filter(users, function (user) {
  return user.favoriteColor === "Blue" && user.favoriteAnimal === "Penguin";
});

bluePenguinUsers;
// => [{ firstName: "Josh", lastName: "Rowley", favoriteColor: "Blue", favoriteAnimal: "Penguin" }, { firstName: "Avidor", lastName: "Turkewitz", favoriteColor: "Blue", favoriteAnimal: "Penguin" }]

const yellowUsers = filter(users, function (user) {
  return user.favoriteColor === "Yellow";
});

yellowUsers;
// => [{ firstName: "Tracy", lastName: "Lum", favoriteColor: "Yellow", favoriteAnimal: "Penguin" }, { firstName: "Drew", lastName: "Price", favoriteColor: "Yellow", favoriteAnimal: "Elephant" }]

users.length;
// => 6
```

Woohoo! We successfully built a clone of JavaScript's built-in `filter()` array
method!

## Using `Array.prototype.filter()`

Now that we've built our own version of `filter()`, we have a better
understanding of what JavaScript's built-in `filter()` method is doing for us
and how it works under the hood. Here's an example of what a call to `filter()`
might look like:

```js
[1, 2, 3, 4, 5].filter(function (num) {
  return num > 3;
});
// => [4, 5]
```

The method accepts one argument, a callback function that it will invoke with
each element in the array. For each element passed to the callback, if the
callback's return value is `true`, that element is copied into a new array. If
the callback's return value is `false`, the element is filtered out. After
iterating over every element in the collection, `filter()` returns the new
array.

Again, notice how similar `filter()` is to `find()`. Both methods are called on
an array and take a callback as an argument. Both automatically iterate through
the passed-in array and call the callback on each element. Both automatically
pass the element, the element's index, and the array itself to the callback
(we're only using the element itself in this example).

The only difference between them is what is returned: `find()` returns a
_single value_, the first element in the array that meets the search condition
(or `undefined` if no matching element is found), while `filter()` returns an
array containing _all_ the elements that meet the search condition.

## Conclusion

As we've learned in this lesson, using JavaScript's built-in `filter()` method
enables us to write more efficient, less repetitive code. Specifically:

- We no longer need to create a `for` or `for...of` loop.
- In each iteration through the array, the current element is stored in a
  variable for us.
- A new array is automatically created and returned after the iterations are
  complete, so we no longer need to create an empty array and push elements into
  it.

Finally, `Array` methods like `find()`, `filter()` and the other methods we will
learn about in this section are _expressive_. As soon as we (or other
developers) see that `filter()` is being called, we know that the code is
looking for elements in an array that meet a certain condition and returning a
new array containing those elements. Or if we see that `map()` is being called,
we immediately know that the code is modifying each of the elements in an array
and returning an array containing the modified values. (We'll learn about
`map()` in the next lesson.) This expressiveness makes our code easier to read
and understand than if we use a generic looping construct.

## Resources

- [MDN — `Array.prototype.filter()`][filter]
- [Tutorial Horizon — Pure vs. Impure Functions][pure]

[filter]:
  https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter
[pure]: https://tutorialhorizon.com/javascript/pure-vs-impure-functions/
