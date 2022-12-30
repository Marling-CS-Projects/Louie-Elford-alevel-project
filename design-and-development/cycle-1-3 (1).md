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

{% tabs %}
{% tab title="ship.ts" %}
{% code lineNumbers="true" %}
```typescript
export default class Ship extends Phaser.GameObjects.Sprite {

	public velocity = 0;
	public accelTime = 0;
	public limit = 0; //(Math.E-1)/this.accelcoef
	public maxV = 0;
	public accelCoef = 0;
	public moveOrderQueue = new Queue();
	public TurnR = 0;
	public TempDisplacement: number = 0;

	constructor(scene: Phaser.Scene, x: number, y: number, texture: string, frame?: string | number ) {
		super(scene, x, y, texture, frame);
		return Object.assign(this, scene.cache.json.get('TestShipJSON'));
	}
}
```
{% endcode %}
{% endtab %}

{% tab title="TestShip.JSON" %}
{% code lineNumbers="true" %}
```json
{
  "accelCoef": 0.5,
  "ArmorArray": [
    [ 0.36, 0.08, 0, 0.6 ], 
    [ 0.36, 0.68, 0.14, 0.32 ],
    [ 0.36, 0.68, 0.14, -0.08 ],
    [ 0.5, 0, 0.14, 0.08 ],
    [ 0.68, 0.08, 0, 0.08 ],
    [ 0.64, 0.68, -0.14, 0.68 ]
  ],
  "limit": 2.455,
  "maxV": 100,
  "TurnR": 50
}
```
{% endcode %}
{% endtab %}

{% tab title="GameRun.ts" %}
{% code lineNumbers="true" %}
```typescript
export default class GameRun extends Phaser.Scene {

	create() {

		console.log('gameRun active');
		let shipName = "testship";
		this.add[shipName](500, 500, "TestShipSpritesheet");
		
		console.log(this);
	}
	
	update() {
		graphics.clear();

		graphics.lineStyle(2, 0xffffff, 1);

		if (!this.shiplist[0].moveOrderQueue.isEmpty) {
			const lineToDraw = this.shiplist[0].moveOrderQueue.peek();
			lineToDraw.draw(graphics);
		}

		graphics.fillStyle(0xff0000, 1);
	}
}
```
{% endcode %}
{% endtab %}

{% tab title="Untitled" %}

{% endtab %}
{% endtabs %}

### Challenges

The first challenge was constructing the object, currently it's done in quite a patchwork way but the concept of using Object.assign() method to assign the properties from the JSON file to a ship object is something I'm happy with. I should note that it took me a while to realise that the phaser loader plugin was storing an already parsed JSON object instead of JSON string in the cache which was an unfortunate setback.

## Testing

Evidence for testing

### Tests

| Test | Instructions     | What I expect                                                                | What actually happens                             | Pass/Fail |
| ---- | ---------------- | ---------------------------------------------------------------------------- | ------------------------------------------------- | --------- |
| 1    | Run code         | Game doesn't freeze/crash or throw errors before reaching the end of content | The game runs without throwing errors or freezing | Pass      |
| 2    | Press Start Game | Test ship appears                                                            | As expected                                       | Pass      |

### Evidence

<figure><img src="../.gitbook/assets/image (3) (2).png" alt=""><figcaption></figcaption></figure>
