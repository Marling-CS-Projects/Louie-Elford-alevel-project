# 2.3.6 Cycle 6 Complex move orders

## Design

Instead of a queue that only stores waypoints I'm going to modify the queue to store "Complex Move Orders" These complex move orders will dictate the exact path that a ship needs to follow for each move command issued by a player.

Phaser contains a Path class which allows developers to create "Tweens" to manage animations that require following a path or curve. In order to avoid creating several new functions to allow for curved paths to exist and to return a position on that path I will instead extend a new complex move order class from the Phaser Path class to suit my needs.

The reason why I am doing this is as a substitute for coding a complex set of rules for ship movement between coordinates and for the selection of waypoints by a player. Instead the player will directly create a move order and the process of communicating invalid movement commands to the player can also be simplified.

I want ships to be able to move in a very restricted manner. Requiring a turning circle to complete a turn and also requiring the ship to accellerate and decelerate. These complex move orders will restrict the turn rate of ships according to a turning circle of radius "TurnR" as specified in the ship's JSON file Whilst also allowing the player to navigate to a precise position and face a desired direction at the end of the move order. The functionality for managing acceleration can be added on later seeing as complex move order will be able to return a ship's relative position on the path and absolute distance down the path.\
\
The velocity and direction of ships will also be stored seperately and when a move order is cancelled (future development) The ship will retain its velocity and direction

### Objectives

To store complex move orders. These complex move orders need to dictate a path that moves a ship from any angle and position to any other angle and position

* [x] create a complex move order class
* [x] store complex move orders in the "waypoint queue" instead of waypoints (and rename it)

### Usability Features

* this is purely computational and won't see any direct interaction with the user as such I don't need to worry about Usability Features. The user interaction will later stem from the use of movement commands to add waypoints.

### Key Variables

| Variable Name  | Use                                                                         |
| -------------- | --------------------------------------------------------------------------- |
| moveOrderQueue | renamed Queue that used to store waypoints (now stores complex move orders) |
| MovePath       | This is it. The complex move order                                          |

### Pseudocode

{% code lineNumbers="true" %}
```
class MovePath extends Phaser.Curves.Path {
    constructor(args) {
        //code that generates a MovePath from the inputs (direction as an angle and position vector)
    }
    
    //any neccessary static methods
    
    getPointDistance(args) {
        //get a point on the MovePath at a certain distance and the tangent unit vector
    }
}

ship {
    moveOrderQueue = new queue();
}
```
{% endcode %}

## Development

I will define a complex move order as an arc lenght of one circle of radius turning radius, the tangent between this circle and a second circle with the same radius and finally an arc length along the second circle.

<figure><img src="../.gitbook/assets/image (5) (2).png" alt=""><figcaption><p>simple example representation of a new move order</p></figcaption></figure>

I quickly identified that this single arrangement would not be most efficient for every case. In an effort to find a computational way of determining how to generate/define a complex move order I drew up a list of expected scenarios and gave them temporary names: (a little o next to the vector denotes origin and a little e denotes exit. The Move path always finds a path from the origin to the exit respecting the angle of each)

<figure><img src="../.gitbook/assets/image (4) (2).png" alt=""><figcaption><p>left/right denotes the relative position of the exit vector from the origin vector and Trandsverse/Direct is a description of the tangent. The tangent is transverse when it crosses a line made between the centers of the two circles and direct when it does not.</p></figcaption></figure>

after some time spent theorising I realised that the 4 cases were created by the 2^2 (4) different combinations of positions for the turning circles.\
<img src="../.gitbook/assets/image (10) (1).png" alt="" data-size="original"> \
These positions are in turn decided by the angle between the vector tangent to the circle and the line between the two vectors.\
![](<../.gitbook/assets/image (6) (1).png>)\
demonstration using the origin vector, this works the same for the exit vector too.

If the turning circles are on the same side of the vectors the tangent will be direct, if they're on opposite sides of the vectors the tangent will be transverse.\
![](<../.gitbook/assets/image (8).png>)

This means that computationally we can simplify the problem down to two different cases.

I decided to attempt to solve the issue one case at a time, starting with direct tangent.

The MovePath can be expressed as 3 different Phaser Path components and the Path class already has methods for adding these components to a path. Two ellipse curves (those arc lengths) and one straight line. The phaser ellipse curves are defined with an x radius length; a y radius length; a starting angle; an end angle;the direction to take the angles in as a boolean (true = clockwise, false = anticlockwise); and finally, an angle to rotate the ellipse through. The Phaser definition of a straight line is simply the end coordinates of the straight line. The straight line is then drawn from the end point of the current path to those coordinates

### Outcome

{% code lineNumbers="true" %}
```typescript
class MovePath extends Phaser.Curves.Path {

	constructor(point1: Phaser.Math.Vector2, ang1: number, point2: Phaser.Math.Vector2, ang2: number, TurnR: number) {
		//call the constructor from the Path class
		super(point1.x,point1.y);
		
		//find the angle between the two points
		const travelAngle: number = Phaser.Math.Angle.BetweenPoints(point1, point2) * (180/Math.PI);
		
		//find turn directions

		//if ang1-travelAngle < 0 clockwise
		let turn1DIR: boolean = (ang1 - travelAngle < 0);
		//if travelAngle-ang2 < 0 clockwise
		const turn2DIR: boolean = (travelAngle - ang2 < 0);

		console.log(turn1DIR);
		console.log(turn2DIR);

		//assigns the first turning circle center to point1 and the transformation from original point1 to the first turning circle center in CenterTranslation1
		const CenterTranslation1 = MovePath.FindTurningCircle(point1,ang1,TurnR,turn1DIR);	
		
		
		//assigns the second turning circle center to point2 and the transformation from original point2 to the second turning circle center in CenterTranslation2
		const CenterTranslation2 = MovePath.FindTurningCircle(point2,ang2,TurnR,turn2DIR);
		
		//find the translation between the two circle centers (point1 and point2)
		const interCenterTranslation = point2.clone();
		interCenterTranslation.subtract(point1);

		console.log("straight angle before");
		console.log(interCenterTranslation.angle()*(180/Math.PI));
		this.ellipseTo(50,50,0,interCenterTranslation.angle()*(180/Math.PI)-ang1,false,ang1-90);


		//create the straight section between the two turns
		const StraightTranslation = interCenterTranslation.clone();
		StraightTranslation.add(this.getEndPoint());
		console.log(point2);
		console.log(StraightTranslation);
		this.lineTo(StraightTranslation);

		console.log("straight angle after");
		console.log(interCenterTranslation.angle() * (180 / Math.PI));
		//create the second elipse curve representing turn2
		this.ellipseTo(TurnR, TurnR, 0, ang2 - (interCenterTranslation.angle() * (180/Math.PI)), false, (interCenterTranslation.angle() * (180 / Math.PI)) - 90);

		console.log(this.getEndPoint());
		const TempTangent1 = this.getTangent(0);
		console.log(TempTangent1.angle()*180/Math.PI);
		const TempTangent = this.getTangent(1);
		console.log(TempTangent.angle()*180/Math.PI);
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
```
{% endcode %}



### Challenges

It proved very challenging to familirize myself with Phaser Paths. The ellipse curve generating method took me a very long time to understand but by rendering the complex move order I was able to understand where I was going wrong and finally get the MovePath construction to work.

The documentation was very unclear as to the input arguments of the ellipseTo method.

## Testing

I tested the Move path constructer by hard coding several different coordinates and angles to move between and running the game to see if the result was a reasonable path. The move path renders each time due to cycle 6.5 and a bunch of console logs indicate the parameters of the move path.

I found an issue when the turn required for a direct turn is over 180 degrees due to the constructor misidentifying the position of one of the turning circles and becoming quite innacurate. I will fix this in a later cycle after the ship can navigate to the move orders

### Tests

| Test | Instructions     | What I expect                                                                                              | What actually happens                             | Pass/Fail |
| ---- | ---------------- | ---------------------------------------------------------------------------------------------------------- | ------------------------------------------------- | --------- |
| 1    | Run code         | Game doesn't freeze/crash or throw errors before reaching the end of content                               | The game runs without throwing errors or freezing | Pass      |
| 2    | Press Start Game | Test ship appears and a MovePath is rendered correctly to the hard coded point and direction for that test | As expected                                       | Pass      |

### Evidence

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>
