# JavaScript Reduce

## Overview

In this lesson, we'll look at how to reduce a list to a single value.

## Objectives

1. Explain the concept of reduce() in programming
2. Write a function that accepts a function as one of its arguments
3. Build small pieces of code that can be used later

## Introduction
In the world of programming, we often work with lists. Sometimes we want to transform elements in that list to another value — but other times, we want to **aggregate** a result. In other words, we want to _reduce_ a list to a single value — it could be, for example, a string, a number, a boolean.

![Amazing parking job](http://www.carcrushing.com/wp-content/uploads/2014/11/Bad-Parking-Jobs-Car-Crushing-004.jpg)

For example, let's say our friend is really bad at parking. We have an entire stack of his parking fees in front of us. Wouldn't it be great if we could reduce all of those parking fees to a single total value? Then we can tell him how much he owes the city for his obnoxious behavior. Shame on him.

## Coupon Queen
![Honey Boo Boo](https://media.giphy.com/media/ponvUa3urCwJq/giphy.gif)

Let's pretend we're Honey Boo Boo. Bored from parading around in beauty pageants, we decide to take up another hobby — programming. This might even help out Mama June, the Coupon Queen! Pulling up some store data, we're interested in getting a total price of all products that have a discount of 50% or more. The store data looks like this:

```js
const products = [
  { name: 'Head & Shoulders Shampoo', price: 4.99, discount: .6 },
  { name: 'Twinkies', price: 7.99, discount: .45 },
  { name: 'Oreos', price: 6.49, discount: .8 },
  { name: 'Jasmine-scented bath pearls', price: 13.99, discount: .7 },
];
```

This means that we're essentially going to reduce the array of products to a _single value_ — in this case the total price of all products that qualify. Let's create a function that has an initial value, then iterates the given products and adds their price to the total price if its discount is higher than 50% (represented as `.5` in our code). When the loop has finished, we return our `totalPrice` result:

```js
function getTotalAmountForProducts(products) {
  let totalPrice = 0;

  products.forEach(product => {
    if (product.discount >= .5) {
      totalPrice += product.price;
    }
  });

  return totalPrice;
}

console.log(getTotalAmountForProducts(products)); // prints 25.5
```

Great! We now have a way to add together the prices of the products we're interested in. But what if the condition was not to have a discount of 50% or more, but rather that the product has to cost 7 dollars or less? That means we'd have to rewrite our logic in the loop!

Instead, maybe we can pass in a function as a second argument. That would save us having to rewrite our `getTotalAmountForProducts()` function if our logic changes. We'll add a callback that handles this logic for us, so that part is abstracted away. The callback has to return a value that is then set to the `totalPrice`. Its arguments are:

- the `totalPrice` variable (to add the price of the product to it)
- the `product` currently in the loop so we can access its price

```js
function getTotalAmountForProducts(products, callback) {
  let totalPrice = 0;

  products.forEach(product => {
    totalPrice = callback(totalPrice, product);
  });

  return totalPrice;
}

function callback(totalPrice, product) {
  if (product.discount >= .5) {
    return totalPrice + product.price;
  }
  return totalPrice;
}

console.log(getTotalAmountForProducts(products, callback)); // prints 25.5
```

We could have passed in an anonymous function too, but declaring the function like this makes our code a little more readable. Totally optional, though!

As you can see, we can easily change the callback to tweak things accordingly. Going back to our earlier problem, now we're interested in the total price for products that cost less than 7 dollars. Luckily, since we're now completely in control of the logic, it should be straight-forward to change:

```js
function callback(totalPrice, product) {
  if (product.price < 7) {
    return totalPrice + product.price;
  }
  return totalPrice;
}

console.log(getTotalAmountForProducts(products, callback)); // prints 11.5
```

Nice! This illustrates how powerful our abstraction is becoming.

Time to take things a step further. Let's pretend we already have some items in our shopping basket (that, when combined, also have a total price). We'll make things easy for Mama June — we don't want her to add two numbers together. We can do that for her by passing in an *initial value* to our function, instead of automatically setting it to `0`.

```js
function getTotalAmountForProducts(products, callback, initialValue) {
  let totalPrice = initialValue;

  products.forEach(product => {
    totalPrice = callback(totalPrice, product);
  });

  return totalPrice;
}

function callback(totalPrice, product) {
  if (product.price < 7) {
    return totalPrice + product.price;
  }
  return totalPrice;
}

console.log(getTotalAmountForProducts(products, callback, 0)); // still prints 11.5. Yeah!
```

To further illustrate how powerful our abstraction is becoming, we'll make our function more generic. Going back to the introduction of this lesson, we talked about *reducing* an array to another value. So, let's call our function `reduce` and change some other variable names to make it more generic:

```js
function reduce(collection, callback, initialValue) {
  let result = initialValue;

  collection.forEach(product => {
    result = callback(result, product);
  });

  return result;
}
```

Finally, as the finishing touch, we'll also pass the index of the loop, as well as the entire collection as an argument to the callback. The callback doesn't *have* to use these arguments, but they're there if we need them:

```js
function reduce(collection, callback, initialValue) {
  let result = initialValue;

  collection.forEach((product, index) => {
    result = callback(result, product, index, collection);
  });

  return result;
}
```

## Hard work pays off
![Congrats!](https://media.giphy.com/media/b7oW9sR0wcr2U/giphy.gif)

To prove that our implementation is sufficiently abstract, we'll use it for a completely different purpose. Let's count the number of coupons Mama June has lying around the house:

```js
const couponLocations = [
  { room: 'Living room', amount: 5 },
  { room: 'Kitchen', amount: 2 },
  { room: 'Bathroom', amount: 1 },
  { room: 'Master bedroom', amount: 7 },
];

function couponCounter(totalAmount, location) {
  return totalAmount + location.amount;
}

console.log(reduce(couponLocations, couponCounter, 0)); // prints 15
```

What if we already have three coupons in our hand? We can easily account for that by adjusting the initial value:

```js
console.log(reduce(couponLocations, couponCounter, 3)); // prints 18
```

## On the shoulders of giants

![A programmer in its natural habitat.](https://media.giphy.com/media/129fSchexp3aPC/giphy.gif)

Fortunately, all programmers are inherently lazy. Who wants to write this stuff over and over again? Don't despair — it seems someone has already done the hard work for us! `Array.prototype.reduce()` is an array method which does _exactly_ the same thing as our own `reduce()` function. So, no need to port over our `reduce()` function to all of our projects — **it's already in the standard JS library**. Don't feel misled, now you know exactly how reduce works!

The proof is in the pudding, so let's take the native implementation for a spin:

```js
console.log(couponLocations.reduce(couponCounter, 0)); // also prints 15!
```

Same result, but without using our custom `reduce()` function. Feel free to throw it in the bin!

## Resources

* [MDN: Array.prototype.reduce()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)

<p class='util--hide'>View <a href='https://learn.co/lessons/javascript-reduce'>Reduce</a> on Learn.co and start learning to code for free.</p>
