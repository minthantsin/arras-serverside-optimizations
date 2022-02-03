# Gameloop
## The problem
Take a look at the `gameloop` function's code
```js
const gameLoop = (function() {
    ...
    return () => {
        logs.loops.tally();
        logs.master.set();
        logs.activation.set();
            entities.forEach(e => entitiesactivationloop(e));
        logs.activation.mark();
        // Do collisions
        logs.collide.set();
        if (entities.length > 1) {
            // Load the grid
            grid.update();
            // Run collisions in each grid
            grid.queryForCollisionPairs().forEach(collision => collide(collision));
        }
        logs.collide.mark();
        // Do entities life
        logs.entities.set();
            entities.forEach(e => entitiesliveloop(e));
        logs.entities.mark();
        logs.master.mark();
        // Remove dead entities
        purgeEntities();
        room.lastCycle = util.time();
    };
})();
```
We see some interesting things, first off, there are three major loops in this:\n
1) The loop that calls the `entitiesactivationloop` function on each entity. This checks if entities will be able to collide and use AI, to help improve performance.
2) The `collide` loop, this loops through all possible collisions and handles them.
3) The `entitiesliveloop` loop, this loops through all entities and updates their physics and life things.
### So why is this bad?
Well, we're looping through entities twice in a tick just here. That's slow. Instead, look at the function `entitiesliveloop` (above it). We see that it does check if an entitiy dies, so we can modify it to look something like this:
```js
    function entitiesliveloop(my) {
        // Consider death.
        if (my.contemplationOfMortality()) my.destroy();
        else {
            if (my.bond == null) {
                // Resolve the physical behavior from the last collision cycle.
                logs.physics.set();
                my.physics();
                logs.physics.mark();
            }
            if (my.activation.check()) {
                logs.entities.tally();
                // Think about my actions.
                logs.life.set();
                my.life();
                logs.life.mark();
                // Apply friction.
                my.friction();
                my.confinementToTheseEarthlyShackles();
                logs.selfie.set();
                my.takeSelfie();
                logs.selfie.mark();
            }
            entitiesactivationloop(my); // Activate it for the next loop if we are gonna live...
        }
        // Update collisions.
        my.collisionArray = [];
    }
```
What this does is call the activation of the entity for the next tick if it should be activated, and save a lot of performance speed.
## Important, to get the above to work you must modify activation code!
Replace `this.activation = ...` with
```js
        this.activation = (() => {
            let active = true;
            let timer = ran.irandom(15);
            return {
                update: () => {
                    if (this.isDead()) {
                        return 0;
                    }
                    if (!active) {
                        this.removeFromGrid();
                        if (this.settings.diesAtRange) {
                            this.kill();
                        }
                        if (!(timer--)) {
                            active = true;
                        }
                    } else {
                        this.addToGrid();
                        timer = 15;
                        active = views.some(v => v.check(this, 0.6)) || this.alwaysActive;
                    }
                },
                check: () => {
                    return active;
                }
            };
        })();
```
In the entity constructor, under `views.forEach(v => v.add(this));`, add `this.activation.update();`
You should be all set!
