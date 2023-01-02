# 2.3.10 Cycle 10 Player Objects

## Design

The aim of this cycle is to create an object to use to centralize the player's interaction with the game also potentially allowing for multiple players/AIs

### Objectives

* [x] each player object has an ID, an array that will be used for selected ships and an array of controled ships, a name, a faction, a team and a points score
* [x] can switch and identify which player object belongs to the user
* [x] player objects are created with JSON objects just like the ships and contain a list of ships to assign to the player

### Usability Features

nope

### Key Variables

| Variable Name | Use                               |
| ------------- | --------------------------------- |
| Player        | a class for making player objects |

### Pseudocode

```
class player

    public ID: //for now 0 for the user and 1 for the AI player
    public team:
    public name:
    public faction:
    public score:
    
    public controled ships: [];
    public selected ships: [];
    
```

## Development

for now an ID of 0 will relate to the user controled player object and an ID of 1 will be AI. They will be kept in an array similar to the ships.

### Outcome

{% tabs %}
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
<strong>	public playerlist = [];
</strong>
	create() {

<strong>		this.playerlist.push(new Player(this.cache.json.get('PlayerJSON')));
</strong><strong>		this.playerlist.push(new Player(this.cache.json.get("AIJSON")));
</strong>		console.log('gameRun active');

<strong>		this.SpawnShips();
</strong>		const point1 = new Phaser.Math.Vector2(200,200);
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

		this.shiplist[0].NewPath(path5Point1, path5Ang1, path5Point2, path5Ang2);

		graphics = this.add.graphics();
		console.log(this);
		console.log(this.playerlist);
	}

<strong>	SpawnShips() {
</strong><strong>		const SpawnLocations = this.cache.json.get("SpawnLocations")
</strong><strong>		for (let i = 0; i &#x3C; this.playerlist.length; i++) {
</strong><strong>			const playerID = this.playerlist[i].ID;
</strong><strong>			for (let n = 0; n &#x3C; this.playerlist[i].pendingShips.length; n++) {
</strong><strong>				const pendingShip = this.playerlist[i].pendingShips[n];
</strong><strong>
</strong><strong>				const Spawnlocation = SpawnLocations[playerID][n];
</strong><strong>				this.shiplist.push(this.add[pendingShip](Spawnlocation[0], Spawnlocation[1], pendingShip.concat("Spritesheet")));
</strong><strong>			}
</strong><strong>		}
</strong>    }
	
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
</code></pre>
{% endtab %}

{% tab title="Second Tab" %}

{% endtab %}
{% endtabs %}

### Challenges

I realised that spawning the ships would have to invovle spawning them in specific locations, I decided to use another JSON file listing spawn locations for the different players for now.

## Testing

Quite simple really I just need to check that the objects generate correctly and that the game runs without crashing, during development I tested and found that I'd accidentaly used a variable name that was involved with the HTML path and was causing weird behaviour but I fixed that before the final tests by changing the name.

### Tests

| Test | Instructions | What I expect                                                                                                                 | What actually happens                             | Pass/Fail |
| ---- | ------------ | ----------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- | --------- |
| 1    | Run code     | Game doesn't freeze/crash or throw errors and a the console log shows that the player list array contains the correct objects | The game runs without throwing errors or freezing | Pass      |

### Evidence

<figure><img src="../.gitbook/assets/image (6) (2).png" alt=""><figcaption><p>console log of the player list</p></figcaption></figure>
