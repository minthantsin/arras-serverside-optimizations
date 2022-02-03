# Controllers
## What are controllers?
Controllers (also known as `ioTypes`) are basic AI for entities.
This is used in things like bosses and bots, as well as things like shape movement and turret rotation.
## Why are they slow?
When a controller is declared in the code as a constructor object, it is defined as follows:
```js
class io_example extends IO {
    constructor(body) {
        super(body);
    }
    think(input) {
        // process stuff and return a new modified input object
    }
}
```
And when a controller is applied to an entitiy, it is done as follows:
```js
this.controllers.push(eval('new ' + ioName + '(self)'));
```
The code above is "evaling" a new controller onto the entity. This is **very** slow, and it is best to be changed as described below:
## Optimizing the usage of controllers
As you see above, controllers are declared `class io_<name> extends IO {`. And I already explained how this is slow, so we're going to fix this by **making an object of controllers**
### Step 1
Above `class IO {`, add `const ioTypes = {};`.
For **all** controllers (`class IO {` is **not** an IO, just the basic parent Constructor Object), replace their declaration with `ioTypes.<name> = class extends IO {`.
A full example of this is:
```js
ioTypes.example = class extends IO {
    constructor(body) {
        super(body);
    }
    think(input) {
        // process stuff and return a new modified input object
    }
}
```
### Step 2
Make sure to do this for every controller, then find `info.PROPERTIES.GUN_CONTROLLERS.forEach` in server.js, and replace `toAdd.push(eval('new ' + ioName + '(self)'));` with `toAdd.push(new ioTypes[ioName](self));`
Next, find `set.CONTROLLERS.forEach` in server.js, and replace `toAdd.push(eval('new io_' + ioName + '(this)'));` with `toAdd.push(new ioTypes[ioName](this));`
### Step 3
Make sure to do that for any instance you have of manually adding controllers!!
