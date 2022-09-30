---
description: game needs ships
---

# 2.3.3 Cycle 3 making a ship class

## Design

Ships will each be represented by a folder containing a JSON file and a spritesheet representing the ship. The JSON file will contain details about the ship's properties.

### Objectives

Be able to load a ship from its folder and move it around the screen

* [x] create a ship class
* [x] JSON file stores paramaters and armor array
* [x] loads the spritesheet and displays the ship
* [x] move the ship using properties from the JSON file

### Usability Features

* the ship should be easy to see

### Key Variables

| Variable Name   | Use                                                                                 |
| --------------- | ----------------------------------------------------------------------------------- |
| maxSpeed        | max speed of the ship                                                               |
| acceleration    | acceleration of the ship                                                            |
| turnRate        | turn rate of the ship                                                               |
| directionVector | the direction of the ship in vector form                                            |
| armorArray      | an array of armor plating on the ship                                               |
| Ship (class)    | ship objects will be created from this class. It will contain a method for movement |

### Pseudocode

```
class ship extends game
```

## Development

### Outcome

The ship class functioned as intended, taking on properties from the JSON file. The ship moved wtih the desired parameters.

### Challenges

The first challenge was constructing the object, currently it's done in quite a patchwork way but the concept of using Object.assign() method to assign the properties from the JSON file to a ship object is something I'm happy with. I should note that it took me a while to realise that the phaser loader plugin was storing an already parsed JSON object instead of JSON string in the cache which was an unfortunate setback.

The last challenge which I ran into was integrating the new object into the gameloop. I'm still not entirely sure what the best method for doing this is but the current method of the constructor adding an event listner to the scene which calls the update method of the object works.

## Testing

Evidence for testing

### Tests

| Test | Instructions     | What I expect                                                                | What actually happens                             | Pass/Fail |
| ---- | ---------------- | ---------------------------------------------------------------------------- | ------------------------------------------------- | --------- |
| 1    | Run code         | Game doesn't freeze/crash or throw errors before reaching the end of content | The game runs without throwing errors or freezing | Pass      |
| 2    | Press Start Game | Test ship appears and moves up and off the screen                            | As expected                                       | Pass      |

### Evidence

{% file src="../.gitbook/assets/2022-09-29 18-16-33.mp4" %}
