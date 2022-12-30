# cycle whatever, Selecting ships and player ownership of ships

## Design

The aim of this cycle is that players have a certain number of ships that they own/control and they can select the ship which they want to move by pressing the corresponding number key.

### Objectives

* [x] a ship can be selected and is deselected when another is selected
* [x] non player controlled ships exist
* [x] there is a visible indication that the ship is selected.
* [x] Selected ship is tracked adequately by the game

### Usability Features

* selected ship has a graphical indication that it is selected

### Key Variables

| Variable Name  | Use                                            |
| -------------- | ---------------------------------------------- |
| selectionArray | an array which tracks which ships are selected |

### Pseudocode

```
class GameRun {
    SpawnShips() {
	const SpawnLocations = this.cache.json.get("SpawnLocations")
	for (let i = 0; i < this.playerlist.length; i++) {
		const playerID = this.playerlist[i].ID;
		for (let n = 0; n < this.playerlist[i].pendingShips.length; n++) {
			const pendingShip = this.playerlist[i].pendingShips[n];

			const Spawnlocation = SpawnLocations[playerID][n];
			this.shiplist.push(this.add[pendingShip](Spawnlocation[0], Spawnlocation[1], pendingShip.concat("Spritesheet")));
			//add to the correct player ship list
			//add an event for playerID 0 ships with the correct number keybinding
		}
	}
    }
	
}



class player {
    selectedShip = ...;
}
```

## Development

for now since it's unnecessary to have an array for single ship selection and selecting multiple ships is not required for any commands I'm planning to implement soon I'm going to replace the selected ships array with a single pointer to the selected ship.

I decided to fiulfil the third objective I would add an ellipse around the ship using the graphics object I had previously used to render the movePaths. To fulfil the usability features I decided to make it more satisfying to look at by using four 45 degree arcs instead of a full circle.

this ended up in a surprisingly large block of code in the update() method of the GameRun scene

### Outcome

{% tabs %}
{% tab title="Player.ts" %}
I substituted selectedShips for selectedShip, which is now no longer an array.

<pre class="language-typescript" data-line-numbers><code class="lang-typescript">export default class Player {
    public ID: number;
    public name: string;
    public team: number;
    public faction: string;
    public score: number = 0;

    public pendingShips: string[];
    public controledShips = [];
<strong>    public selectedShip;
</strong>
    constructor(JSONconfig) {
        return Object.assign(this,JSONconfig);
    }
}
</code></pre>
{% endtab %}

{% tab title="GameRun.ts" %}
<pre class="language-typescript" data-line-numbers><code class="lang-typescript">import 'phaser';
import './Ship';
import './Player';
import Player from './Player';

const config = {
  key: "GameRun",
  // active: false,
  // visible: true,
  // pack: false,
  // cameras: null,
  // map: {},
  // physics: {},
  // loader: {},
  // plugins: false,
  // input: {}
};


let graphics: Phaser.GameObjects.Graphics;

export default class GameRun extends Phaser.Scene {

	public shiplist = [];
	public playerlist = [];

	create() {

		this.playerlist.push(new Player(this.cache.json.get('PlayerJSON')));
		this.playerlist.push(new Player(this.cache.json.get("AIJSON")));
		console.log('gameRun active');

		this.SpawnShips();
		const point1 = new Phaser.Math.Vector2(200,200);
		const ang1 = -50;
		const point2 = new Phaser.Math.Vector2(800,600);
		const ang2 = -30;

		console.log(this.shiplist);
		this.shiplist[0].NewPath(point1,ang1,point2,ang2);

		const path4Point1 = new Phaser.Math.Vector2(800, 600);
		const path4Ang1 = -30;
		const path4Point2 = new Phaser.Math.Vector2(200, 200);
		const path4Ang2 = -50;

		const apath4Point1 = new Phaser.Math.Vector2(300, 760);
		const apath4Ang1 = -60;
		const apath4Point2 = new Phaser.Math.Vector2(200, 350);
		const apath4Ang2 = -40;

		this.shiplist[1].NewPath(apath4Point1, apath4Ang1, apath4Point2, apath4Ang1);

		this.shiplist[0].NewPath(path4Point1, path4Ang1, path4Point2, path4Ang2);

		const path5Point1 = new Phaser.Math.Vector2(200,200);
		const path5Ang1 = -50;
		const path5Point2 = new Phaser.Math.Vector2(220,210);
		const path5Ang2 = -50;

		//this.shiplist[0].NewPath(path5Point1, path5Ang1, path5Point2, path5Ang2);

		graphics = this.add.graphics();
		console.log(this);
		console.log(this.playerlist);

	}

	SpawnShips() {
		const SpawnLocations = this.cache.json.get("SpawnLocations")
		for (let i = 0; i &#x3C; this.playerlist.length; i++) {
			const playerID = this.playerlist[i].ID;
			for (let n = 0; n &#x3C; this.playerlist[i].pendingShips.length; n++) {
				const pendingShip = this.playerlist[i].pendingShips[n];

				const Spawnlocation = SpawnLocations[playerID][n];
<strong>				const createdShip = this.add[pendingShip](Spawnlocation[0], Spawnlocation[1], pendingShip.concat("Spritesheet"))
</strong><strong>
</strong><strong>				this.shiplist.push(createdShip);
</strong><strong>				this.playerlist[i].controledShips.push(createdShip);
</strong><strong>
</strong><strong>				if (playerID == 0) {
</strong><strong>					const shipnum = n + 1;
</strong><strong>					const shipnumstring = shipnum.toString();
</strong><strong>					this.input.keyboard.on("keydown", function (event) {
</strong><strong>						switch (event.code) {
</strong><strong>							case "Digit" + shipnumstring:
</strong><strong>							case "Numpad" + shipnumstring:
</strong><strong>								console.log(shipnumstring + " pressed");
</strong><strong>								this.scene.playerlist[0].selectedShip = this.scene.playerlist[0].controledShips[n];
</strong><strong>							break;
</strong><strong>                        			}
</strong><strong>					});
</strong><strong>                		}
</strong>			}
		}
    }
	
	update() {

		graphics.clear();

		graphics.lineStyle(2, 0xffffff, 1);

<strong>		if (typeof this.playerlist[0].selectedShip !== "undefined") {
</strong><strong>			console.log("trying");
</strong><strong>			graphics.beginPath();
</strong><strong>			graphics.arc(this.playerlist[0].selectedShip.x, this.playerlist[0].selectedShip.y, 40, 0.3927, 1.1781);
</strong><strong>			graphics.strokePath();
</strong><strong>			graphics.beginPath();
</strong><strong>			graphics.arc(this.playerlist[0].selectedShip.x, this.playerlist[0].selectedShip.y, 40, 1.9635, 2.7489);
</strong><strong>			graphics.strokePath();
</strong><strong>			graphics.beginPath();
</strong><strong>			graphics.arc(this.playerlist[0].selectedShip.x, this.playerlist[0].selectedShip.y, 40, 3.5343, 4.3197);
</strong><strong>			graphics.strokePath();
</strong><strong>			graphics.beginPath();
</strong><strong>			graphics.arc(this.playerlist[0].selectedShip.x, this.playerlist[0].selectedShip.y, 40, 5.1051, 5.8905);
</strong><strong>			graphics.strokePath();
</strong><strong>			//yes all of this to produce a simple pattern...
</strong><strong>		}
</strong>
		if (!this.shiplist[0].moveOrderQueue.isEmpty) {
			const lineToDraw = this.shiplist[0].moveOrderQueue.peek();
			lineToDraw.draw(graphics);
		}
	}
}
</code></pre>
{% endtab %}
{% endtabs %}

### Challenges

creating the graphics for ship selection presented a simple challenge in the fact that before a ship is selected the selectedShip property is undefined and causes an error if that isn't checked for first

## Testing

This is exciting. The first test with user input. I don't just have to watch the screen!

I pressed 1,2 and 3.

### Tests

| Test                      | Instructions             | What I expect                                       | What actually happens      | Pass/Fail |
| ------------------------- | ------------------------ | --------------------------------------------------- | -------------------------- | --------- |
| 1: run the game           | run game and press start | game runs without issue                             | The game ran without issue | Pass      |
| 2: select the first ship  | press 1                  | ship 1 is highlighted (the first ship from the top) | ship 1 is highlighted      | pass      |
| 3: select the second ship | press 2                  | ship 2 is highlighted (second ship from the top)    | ship 2 is highlighted      | pass      |
| 4: select the third ship  | press 3                  | ship 3 is highlighted (third ship from the top)     | ship 3 is highlighted      | pass      |

### Evidence

{% file src="../.gitbook/assets/2022-12-29 23-32-14.mp4" %}

### Conclusions

the selection highlight looks pretty cool but it doesn't rotate with the ship and doesn't give any indication of the ship's rotation. This could be corrected to further highlight more useful information for the player.
