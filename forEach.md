# Array.prototype.forEach
## What's the deal with Array.prototype.forEach loops?
`Array.prototype.forEach` is a method on the `Array` Object in JavaScript. Here's a simple example:
```js
const list = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
list.forEach(function(element, index) {
    console.log("Position", index, "-", element);
});
```
The example above will loop through the array `list`, and for every element in the array, it will print the position and the value to the console.
However this is slow in comparison to a for loop on most machines.
## So what is a for loop, and how does it work?
I'm so glad you asked! Here's the same example in two different ways:
```js
const list = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
for (let element of list) {
    console.log(element);
}
```
```js
const list = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
for (let index = 0, length = list.length; index < length; index ++) {
    console.log("Position", index, "-", element);
}
```
So you may notice there's no `index` in the `for (... of ...)` loop, this is a limitation that can be fixed simply:
```js
const list = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
let index = 0;
for (let element of list) {
    console.log("Position", index, "-", element);
    index ++;
}
```
Either of these approaches are much faster than a `forEach` loop.
## So how can we apply this knowledge?
### Step 1
Find every instance of `forEach` in `server.js`.
### Step 2
Detemine if each step can work with `for (... of ...)` or `for (let i = 0, l = <array>.length; i < l; i ++)` (Most work with `for (... of ...)`, as they do not require an index)
### Step 3
Replace the `forEach` with the faster for loop!
## Example
I found `entities.forEach(e => entitiesliveloop(e));`, and here's how I replaced it:
```js
for (let e of entities) {
    entitiesliveloop(e);
}
```
Good luck, this is a bit time consuming, but it is very much worth it!
