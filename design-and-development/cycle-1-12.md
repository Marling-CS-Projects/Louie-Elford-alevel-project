# 2.3.12 Cycle 12 Player controls

## Design

The aim of this cycle is to create the movement controls that will let the player interact with the movement system

### Objectives

* [x] player can set a movement command with a target position and direction for the ship to face
* [x] commands can be chained
* [x] command is issued to the selected ship

### Usability Features

* The command is easy to see
* setting the rotation of the ship is straightforward and clear

### Pseudocode

I at first considered allowing the player to edit a move command before confirming it but this would overcomplicate the process and slow down the player so instead the command will be issued instantanously when the right mouse button is lifted. A short right mouse button click will send the ship to that position with no consideration for direction and a hold of the right mouse button will send the ship in the direction the player drags the mouse out to.

```
event on right mouse down {
    coordinates = get pointer position
    selected ship = get selected ship for player ID 0
    if mouse held down for less than 0.2s {
        //start shortened move path
    } else {
        //logic for selecting direction
    }
}

class movePath {
    constructor() {
        //modify so that it can take a shortened command with no second turning circle
    }
}
```

## Development

This was straightforward but there were a lot of things that had to be ironed out to make it smooth, the ship sprite is still misaligned by 90 degrees and that has to be factored in when a new move path is made (oops). Phaser passes a lot of useful information in an event call which I make use of here. I use the x and y coordinates of the mouse

because it's more straightforward in phaser for now movement simply uses the left mouse button. Some strange behaviour occurs when pressing right mouse button.

### Outcome

{% tabs %}
{% tab title="GameRun.ts" %}
{% code lineNumbers="true" %}
```typescript
import 'phaser';
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
	public MainPlayer;

	create() {

		this.playerlist.push(new Player(this.cache.json.get('PlayerJSON')));
		this.playerlist.push(new Player(this.cache.json.get("AIJSON")));



		for (let i = 0; i < this.playerlist.length; i++) {
			if (this.playerlist[i].ID == 0) {
				this.MainPlayer = this.playerlist[i];
			}
		}

		//this.shiplist[0].NewPath(path5Point1, path5Ang1, path5Point2, path5Ang2);

		graphics = this.add.graphics();


		//set up moust inputs
		//disable the context menu so that it doesn't interupt gameplay
		this.input.mouse.disableContextMenu();

		//mouse events
		this.input.on("pointerup", function (pointer) {
			if (typeof this.scene.MainPlayer.selectedShip === "undefined") return;
			if (!pointer.event.shiftKey) {
				this.scene.MainPlayer.selectedShip.clearPaths();
			}
			const endPoint = new Phaser.Math.Vector2(pointer.upX, pointer.upY)
			if (pointer.getDuration() < 150) {
				this.scene.MainPlayer.selectedShip.NewPath(endPoint);
				return;
            }
			const startPoint = new Phaser.Math.Vector2(pointer.downX, pointer.downY);
			const angle = Phaser.Math.Angle.BetweenPoints(startPoint, endPoint) * (180 / Math.PI);
			this.scene.MainPlayer.selectedShip.NewPath(startPoint, angle);
		});

		//generic keybinds
		this.input.keyboard.on("keydown", function (event) {
			switch (event.code) {
				case "KeyX":
					this.scene.MainPlayer.clearSelected();
            }
		});

		this.SpawnShips();

	}

	SpawnShips() {
		const SpawnLocations = this.cache.json.get("SpawnLocations")
		for (let i = 0; i < this.playerlist.length; i++) {
			const playerID = this.playerlist[i].ID;
			for (let n = 0; n < this.playerlist[i].pendingShips.length; n++) {
				const pendingShip = this.playerlist[i].pendingShips[n];

				const Spawnlocation = SpawnLocations[playerID][n];
				const createdShip = this.add[pendingShip](Spawnlocation[0], Spawnlocation[1], pendingShip.concat("Spritesheet"))

				this.shiplist.push(createdShip);
				this.playerlist[i].controledShips.push(createdShip);

				if (playerID == 0) {
					const shipnum = n + 1;
					const shipnumstring = shipnum.toString();
					this.input.keyboard.on("keydown", function (event) {
						switch (event.code) {
							case "Digit" + shipnumstring:
							case "Numpad" + shipnumstring:
								this.scene.playerlist[i].selectedShip = this.scene.playerlist[i].controledShips[n];
							break;
                        }
					});
				}
				delete this.playerlist[i].pendingShips[n];
			}
		}
    }
	
	update() {

		graphics.clear();

		graphics.lineStyle(2, 0xffffff, 1);

		if (typeof this.playerlist[0].selectedShip !== "undefined") {
			graphics.beginPath();
			graphics.arc(this.playerlist[0].selectedShip.x, this.playerlist[0].selectedShip.y, 40, 0.3927, 1.1781);
			graphics.strokePath();
			graphics.beginPath();
			graphics.arc(this.playerlist[0].selectedShip.x, this.playerlist[0].selectedShip.y, 40, 1.9635, 2.7489);
			graphics.strokePath();
			graphics.beginPath();
			graphics.arc(this.playerlist[0].selectedShip.x, this.playerlist[0].selectedShip.y, 40, 3.5343, 4.3197);
			graphics.strokePath();
			graphics.beginPath();
			graphics.arc(this.playerlist[0].selectedShip.x, this.playerlist[0].selectedShip.y, 40, 5.1051, 5.8905);
			graphics.strokePath();
			//yes all of this to produce a simple pattern...
		}

		if (!this.shiplist[0].moveOrderQueue.isEmpty) {
			const lineToDraw = this.shiplist[0].moveOrderQueue.peek();
			lineToDraw.draw(graphics);
		}
		if (!this.shiplist[1].moveOrderQueue.isEmpty) {
			const lineToDraw = this.shiplist[1].moveOrderQueue.peek();
			lineToDraw.draw(graphics);
		}
		if (!this.shiplist[2].moveOrderQueue.isEmpty) {
			const lineToDraw = this.shiplist[2].moveOrderQueue.peek();
			lineToDraw.draw(graphics);
		}
	}
}
```
{% endcode %}
{% endtab %}

{% tab title="Ship.ts" %}
{% code lineNumbers="true" %}
```typescript
import 'phaser';

class Queue {
	
	private elements = {};
	private head = 0;
	private tail = 0;

	enqueue(element) {
		this.elements[this.tail] = element;
		this.tail++;
	}
	dequeue() {
		const item = this.elements[this.head];
		this.elements[this.head].destroy();
		delete this.elements[this.head];
		this.head++;
		return item;
	}
	peek() {
		return this.elements[this.head];
	}
	get length() {
		return this.tail - this.head;
	}
	get isEmpty() {
		return this.length === 0;
	}
	peekLast() {
		return this.elements[this.tail-1];
	}
	clear() {
		const n = this.length
		for (let i = 0; i < n; i++) {
			this.dequeue();
        }
    }
}

class MovePath extends Phaser.Curves.Path {

	constructor(TurnR: number, point1: Phaser.Math.Vector2, ang1: number, point2: Phaser.Math.Vector2, ang2: number) {
		//call the constructor from the Path class
		super(point1.x, point1.y);

		//find the angle between the two points
		const travelAngle: number = Phaser.Math.Angle.BetweenPoints(point1, point2) * (180 / Math.PI);
		//find turn directions



		//if ang1-travelAngle < 0 clockwise
		let turn1Clockwise: boolean = !((travelAngle + 180) - (ang1 + 180) < 0 || (travelAngle + 180) - (ang1 + 180) > 180);

		//assigns the first turning circle center to point1 and the transformation from original point1 to the first turning circle center in CenterTranslation1
		const CenterTranslation1 = MovePath.FindTurningCircle(point1, ang1, TurnR, turn1Clockwise);

		//if travelAngle-ang2 < 0 clockwise
		let turn2Clockwise: boolean = ((travelAngle + 180) - (ang2 + 180) < 0 || (travelAngle + 180) - (ang2 + 180) > 180);

		//assigns the second turning circle center to point2 and the transformation from original point2 to the second turning circle center in CenterTranslation2
		const CenterTranslation2 = MovePath.FindTurningCircle(point2, ang2, TurnR, turn2Clockwise);

		//find the translation between the two circle centers (point1 and point2)


		const interCenterTranslation = point2.clone();
		interCenterTranslation.subtract(point1);


		let rotation1 = 0
		let rotation2 = 0

		if (turn1Clockwise) {
			if (!turn2Clockwise) {
				//transverse with a right first turn
				rotation2 = 180;
				const length = interCenterTranslation.length();
				const angle = Math.atan((2 * TurnR) / length);

				interCenterTranslation.rotate(angle);
				interCenterTranslation.scale(Math.cos(angle));
			}
		}
		else {
			if (turn2Clockwise) {
				//transverse with a left first turn
				rotation1 = 180;
				const length = interCenterTranslation.length();
				const angle = Math.atan((2 * TurnR) / length);

				interCenterTranslation.rotate(-angle);
				interCenterTranslation.scale(Math.cos(angle));
			}
			else {
				rotation1 = 180;
				rotation2 = 180;
			}
		}




		const TransitionRadiusAngle = interCenterTranslation.angle() * (180 / Math.PI) - 90;

		//create the first ellipse curve
		const PathDeviance1 = Math.abs((ang1 - 90) - (TransitionRadiusAngle));
		const PathDeviance2 = Math.abs(Math.abs((ang1 - 90) - (TransitionRadiusAngle)) - 360);
		if (PathDeviance1 > 10 && PathDeviance2 > 10) {
			this.ellipseTo(TurnR, TurnR, ang1 - 90, TransitionRadiusAngle, !turn1Clockwise, rotation1);
		}

		//create the straight section between the two turns
		const StraightTranslation = interCenterTranslation.clone();
		StraightTranslation.add(this.getEndPoint());
		this.lineTo(StraightTranslation);

		//create the second elipse curve representing turn2
		const Path2Deviance1 = Math.abs((TransitionRadiusAngle) - (ang2 - 90));
		const Path2Deviance2 = Math.abs(Math.abs((TransitionRadiusAngle) - (ang2 - 90)) - 360);
		if (Path2Deviance1 > 10 && Path2Deviance2 > 10) {
			this.ellipseTo(TurnR, TurnR, TransitionRadiusAngle, ang2 - 90, !turn2Clockwise, rotation2);
		}

	}//phaser.math.angle.BetweenPoints(interCenterTranslation,)

	//determines the location of a turning circle
	static FindTurningCircle(point: Phaser.Math.Vector2, ang: number, TurnR: number, TurnDir: boolean) {
		
		const Translation = new Phaser.Math.Vector2(TurnR,0);
		
		const tempAngle = (TurnDir ? ang + 90 : ang - 90)*(Math.PI/180);
		
		Translation.setAngle(tempAngle);
		
		point.add(Translation);
		
		return Translation;
	}

	getPointDistance(d: number) {
		let outPoint = new Phaser.Math.Vector2();
		let outDirection = new Phaser.Math.Vector2();

        const curveLengths = this.getCurveLengths();
        let i = 0;

        while (i < curveLengths.length)
        {
            if (curveLengths[i] >= d)
            {
                var diff = curveLengths[i] - d;
                var curve = this.curves[i];

                var segmentLength = curve.getLength();
                var u = (segmentLength === 0) ? 0 : 1 - diff / segmentLength;

				return [curve.getPointAt(u, outPoint), curve.getTangentAt(u, outDirection)];
            }

            i++;
        }

        // loop where sum != 0, sum > d , sum+1 <d
        return null;
	}

}

export default class Ship extends Phaser.GameObjects.Sprite {

	public velocity = 0;
	public accelTime = 0;
	public limit = 0; //(Math.E-1)/this.accelcoef
	public maxV = 0;
	public accelCoef = 0;
	public moveOrderQueue = new Queue();
	public TurnR = 0;
	public TempDistance: number = 0;

	constructor(scene: Phaser.Scene, x: number, y: number, texture: string, frame?: string | number ) {
		super(scene, x, y, texture, frame);
		return Object.assign(this, scene.cache.json.get('TestShipJSON'));
	}

	Accelerate(): number{
		return this.velocity = this.maxV*(Math.E*Math.log1p(this.accelCoef*this.accelTime)-this.accelCoef*this.accelTime);
	}	// velocity = v*(e*Math.log1p(a*x)-a*x)

	UpdateAccelTime(acc: boolean, delta: number): void {
		
		let incTime: number;

		if (acc) {
			incTime = this.accelTime + delta/1000;
		} 
		else {
			incTime = this.accelTime - delta/1000;
		}

		if (incTime < this.limit && incTime > 0) {
			this.accelTime = incTime;
		}
	}

	Move(delta: number): void {
		if (this.moveOrderQueue.isEmpty) return;
		const moveOrder = this.moveOrderQueue.peek();
		this.UpdateAccelTime(true,delta);
		this.TempDistance = this.TempDistance + this.Accelerate() * (delta/1000);
		const commands = moveOrder.getPointDistance(this.TempDistance);
		if (!commands) {
			this.moveOrderQueue.dequeue();
			this.TempDistance = 0
			this.Move(delta);
			return;
		}

		this.x = commands[0].x;
		this.y = commands[0].y;
		
		this.setRotation(commands[1].angle()+Math.PI/2);
	}
	NewPath(point2, ang2?): void{

		let startPos: Phaser.Math.Vector2;
		let startAngle: number;

		if (this.moveOrderQueue.isEmpty) {
			startPos = new Phaser.Math.Vector2(this.x, this.y);
			startAngle = this.angle - 90;
		} else {
			const lastMove = this.moveOrderQueue.peekLast()
			startPos = lastMove.getEndPoint();
			startAngle = lastMove.getTangent(1).angle() * (180/Math.PI);
        }

		if (typeof ang2 == "undefined") {
			ang2 = Phaser.Math.Angle.BetweenPoints(startPos, point2) * (180 / Math.PI);
        }

		const PendingOrder = new MovePath(this.TurnR, startPos, startAngle, point2, ang2);
		this.moveOrderQueue.enqueue(PendingOrder);
	}
	clearPaths(): void {
		this.moveOrderQueue.clear();
		this.TempDistance = 0;
    }
	update(time, delta) {
		this.Move(delta);
	}
}

Phaser.GameObjects.GameObjectFactory.register('TestShip', function (this: Phaser.GameObjects.GameObjectFactory, x, y, key, frame) {
		const testship = new Ship(this.scene, x, y, key, frame);
		testship.setOrigin(0.5, 0.5);
		testship.setScale(0.2);
		this.displayList.add(testship);
		this.updateList.add(testship);
		this.scene.events.on('update', testship.update, testship);
		return testship;
});
```
{% endcode %}
{% endtab %}
{% endtabs %}

### Challenges

The ship would often do a full 360 degree turn when the required turn is quite small due to small natural errors cropping up in the calculations. This meant that I had to include an if statement that lead to the EllipseTo method not running if the change in angle is too small. This worked relatively well at preventing these eratic turns whilst not impacting the precision too much.

I found it easier to test the movement of multiple ships after I modified my code so that all of the ships have their next move order rendered

the code for generating move paths needed a bit of correcting. The code for finding the direction of the two turns has been corrected and simplified.

## Testing

I performed several tests giving any number of the 3 ships movement commands to do. Here's a snapshot of the ships excecuting movement commands correctly in testing

### Tests

| test                                               | conditions                                                                                                                                                                               | success state | notes                                                                                                                  |
| -------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------- | ---------------------------------------------------------------------------------------------------------------------- |
| game runs                                          | no crahses or freezes                                                                                                                                                                    | success       |                                                                                                                        |
| the ships move along the move command correctly    | the ships follow the move command with the correct heading and stay on the move command                                                                                                  | success       | this took accounting for the rotation of the sprite                                                                    |
| short click movment                                | ship moves to position clicked without changing heading once there                                                                                                                       | success\*\*   | one thing to note is that to keep this simple I made a slight concession in that the ship's heading changes slightly\* |
| long click movements                               | holding down the mouse button then releasing it moves the ship to the position where the mouse button was first pressed pointing in the direction of where the mouse button was released | success\*\*   |                                                                                                                        |
| shift click adds a waypoint for the ship to follow | ship follows any moves prior to the shift click command then excecutes the correct command just given                                                                                    | success       | queued commands aren't rendered, only the current one but the ship can be seen to excecute them                        |

\*Implementing this properly would require me to essentially create a new way of generating move orders for this case so I opted to take an angle that would be relatively correct (angle from starting position to end position)

\*\*there can be unfortunately still be occasional issues where a command isn't generated quite right.

### Evidence

{% file src="../.gitbook/assets/2023-01-01 19-49-41.mp4" %}
