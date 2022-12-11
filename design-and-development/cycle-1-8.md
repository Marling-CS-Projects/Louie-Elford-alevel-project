---
description: The Ship now needs to navigate using the complex move order queue
---

# 2.3.7 Cycle 7 Navigating using Complex Move Orders

## Design

now that Move orders have been implemented I'm going to aim to implement the actual movement of the ship in this cycle. This will set up the pieces for player controls. I'm going to use a method on the move path to return some commands that include position and a unit vector pointing in the direction of the tangent at that point given a distance along that move path.

### Objectives

* [x] The ship moves along the move order
* [x] the ship points in the direction of the tangent on the move order at the point it's on
* [x] the ship doesn't appear to move sideways or backwards

### Usability Features

* The direction of movement should be clear, this is partially achieved with the visible bow of the ship model and then finally by pointing the ship in the correct direction as per the objectives

### Key Variables

| Variable Name | Use                                                       |
| ------------- | --------------------------------------------------------- |
| TempDistance  | the temporary distance on the current move order          |
| commands      | stores the output of the MovePath getPointDistance method |

### Pseudocode

```
MovePath {
    getPointDistance() {
        //returns two Vector2s in an array
        //the first indicates the precise location of a point a certain distance
        //on the move path
        //the second is a unit vector in the direction tangent to the move
        //path at this distance on the move path
    }

}


ship {
    
    move() {
        //update acceleration time then velocity
        //increase TempDistance by velocity*time
        //call getPointDistance using TempDistance and store the resulting array in a variable
        //use index 0 of stored array to set position
        //call .angle method on index 1 of the array and use result to set the rotation
    }
    
    update() {
        move()
    }
}
```

## Development



### Outcome

{% tabs %}
{% tab title="GameRun.ts" %}
```
import 'phaser';
import './Ship';

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

	create() {

		console.log('gameRun active');
		let shipName = "testship";
		this.shiplist.push(this.add[shipName](500, 500, "TestShipSpritesheet"));
		console.log(this.shiplist[0]);
		
		const point1 = new Phaser.Math.Vector2(200,200);
		const ang1 = -50;
		const point2 = new Phaser.Math.Vector2(800,600);
		const ang2 = 90;
		
		this.shiplist[0].NewPath(point1,ang1,point2,ang2);

		const path4Point1 = new Phaser.Math.Vector2(800, 600);
		const path4Ang1 = 90;
		const path4Point2 = new Phaser.Math.Vector2(1300, 75);
		const path4Ang2 = -140;

		this.shiplist[0].NewPath(path4Point1, path4Ang1, path4Point2, path4Ang2);

		const path5Point1 = new Phaser.Math.Vector2(1300,75);
		const path5Ang1 = -140;
		const path5Point2 = new Phaser.Math.Vector2(1000,800);
		const path5Ang2 = 30;

		this.shiplist[0].NewPath(path5Point1, path5Ang1, path5Point2, path5Ang2);

		graphics = this.add.graphics();
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
{% endtab %}

{% tab title="ship.ts" %}
```
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
}

class MovePath extends Phaser.Curves.Path {

	constructor(point1: Phaser.Math.Vector2, ang1: number, point2: Phaser.Math.Vector2, ang2: number, TurnR: number) {
		//call the constructor from the Path class
		super(point1.x,point1.y);
		
		//find the angle between the two points
		const travelAngle: number = Phaser.Math.Angle.BetweenPoints(point1, point2) * (180/Math.PI);
		//find turn directions

		//if ang1-travelAngle < 0 clockwise
		let turn1Clockwise: boolean = (ang1 - travelAngle < 0 && ang1 - travelAngle > -180);
		//if travelAngle-ang2 < 0 clockwise
		const turn2Clockwise: boolean = (travelAngle - ang2 < 0 && travelAngle - ang2 > -180);

		//assigns the first turning circle center to point1 and the transformation from original point1 to the first turning circle center in CenterTranslation1
		const CenterTranslation1 = MovePath.FindTurningCircle(point1,ang1,TurnR,turn1Clockwise);	
		
		
		//assigns the second turning circle center to point2 and the transformation from original point2 to the second turning circle center in CenterTranslation2
		const CenterTranslation2 = MovePath.FindTurningCircle(point2,ang2,TurnR,turn2Clockwise);
		
		//find the translation between the two circle centers (point1 and point2)
		const interCenterTranslation = point2.clone();
		interCenterTranslation.subtract(point1);

		
		const TransitionRadiusAngle = interCenterTranslation.angle() * (180 / Math.PI) - 90;

		let rotation = 0

		if (!turn1Clockwise && !turn2Clockwise) {
			rotation = 180;
		}


		//this.ellipseTo(50,50,ang1+90,turn1,true,0);

		//this.ellipseTo(TurnR, TurnR, ang1 - 90, TransitionRadiusAngle, !turn1Clockwise, 0);
		this.ellipseTo(TurnR, TurnR, ang1 - 90, TransitionRadiusAngle, !turn1Clockwise, rotation);


		//create the straight section between the two turns
		const StraightTranslation = interCenterTranslation.clone();
		StraightTranslation.add(this.getEndPoint());
		this.lineTo(StraightTranslation);

		//create the second elipse curve representing turn2

		//this.ellipseTo(TurnR, TurnR, TransitionRadiusAngle, ang2 - 90, !turn2Clockwise, 0);
		this.ellipseTo(TurnR, TurnR, TransitionRadiusAngle, ang2 - 90, !turn1Clockwise, rotation);

		console.log(this.getEndPoint());
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
			console.log("current coordinates");
			console.log(this.x);
			console.log(this.y);
			this.Move(delta);
			return;
		}

		this.x = commands[0].x;
		this.y = commands[0].y;
		
		this.setRotation(commands[1].angle()+Math.PI/2);
	}
	NewPath(point1, ang1, point2, ang2): void{
		const PendingOrder = new MovePath(point1,ang1,point2,ang2,this.TurnR)
		let drawnPath = new Phaser.GameObjects.Graphics(this.scene);
		drawnPath.lineStyle(5, 0xFF00FF, 1.0);
		PendingOrder.draw(drawnPath);
		this.moveOrderQueue.enqueue(PendingOrder);
	}
	update(time, delta) {
		this.Move(delta);
	}
}

Phaser.GameObjects.GameObjectFactory.register('testship', function (this: Phaser.GameObjects.GameObjectFactory, x, y, key, frame) {
		console.log("making testship");
		const testship = new Ship(this.scene, x, y, key, frame);
		testship.setOrigin(0.5, 0.5);
		testship.setScale(0.2);
		console.log(testship);
		this.displayList.add(testship);
		this.updateList.add(testship);
		this.scene.events.on('update', testship.update, testship);
		return testship;
});
```
{% endtab %}
{% endtabs %}

for now I've hard coded in several movement commands by manually calling the NewPath() method on the testship by accessing the shiplist array on the GameRun instance and supplying fixed arguments.

The ship then follows the move commands in the array in order as with the movement queue in cycle 5.

### Challenges

the testship sprite appears to be misaligned seeing as I expected north to be 0 degrees when I made it so I compensated by rotating it an extra 90 degrees so that it appears to be facing the right direction.

functions only return one result by definition but I needed two different things from the move path to move the ship and they could both be found together simpler than seperately so I decided to use return an array containing both as a work around.

## Testing

test is simple, let it loose and hopefully the movement follows the move paths as in the objectives.

### Tests

| Test | Instructions     | What I expect                                                                                  | What actually happens                             | Pass/Fail |
| ---- | ---------------- | ---------------------------------------------------------------------------------------------- | ------------------------------------------------- | --------- |
| 1    | Run code         | Game doesn't freeze/crash or throw errors before reaching the end of content                   | The game runs without throwing errors or freezing | Pass      |
| 2    | Press Start Game | Test ship appears and follows the rendered move paths correctly as specified in the objectives | As expected                                       | Pass      |

### Evidence

{% file src="../.gitbook/assets/2022-12-11 20-31-27.mp4" %}
