---
description: creating parameters and an acceleration function
---

# 2.3.something Cycle something acceleration and parameters

## Design

I want the ship to accelerate in a non linear fascion with decreasing rate of acceleration (Jerk). To do this I've decided to use a function to return velocity for a given time input. Time is then increased for time spent accelerating and decreased for time spent decelerating.

The function needs to include several parameters that allow me to model acceleration so that each potential ship can have different properties.

### Objectives

The ship accelerates according to parameters set in its JSON file.

* [x] create a velocity time function
* [x] use parameters from the JSON file
* [x] the ship accelerates in a decreasing rate

### Usability Features

* acceleration is visible and isn't locked to taking an excessive amount of time

### Key Variables

| Variable Name | Use                                                                                                                   |
| ------------- | --------------------------------------------------------------------------------------------------------------------- |
| maxV          | maximum velocity for the ship                                                                                         |
| accelCoef     | a coeficient which determines the rate of acceleration                                                                |
| limit         | variable that stores the limit of acceleration time, this is the time at which the ship reaches its maximum velocity. |

### Pseudocode

```
ship {
    
    updateAccelTime() {
        //updates the time spent accelerating
    }
    
    accelerate() {
        //acceleration function
    }
    
    move() {
        //call updateAccelTime
        //call accelerate
        //move through velocity*time
    }
    
    update() {
        move()
    }
}
```

## Development

as the psuedocode indicates, first we call UpdateAccelTime to update the time spent accelerating (here it's fixed to true meaning that the ship is accelerating) then we call accelerate to get the velocity and multiply that by the change in time (to get change in displacement) and finally add that to the x position of the ship.

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
		this.UpdateAccelTime(true,delta);
		this.x = this.x + this.Accelerate() * (delta/1000);
	}
	
	update(time, delta) {
		this.Move(delta);
	}
}

```
{% endcode %}

### Outcome

acceleration!

{% file src="../.gitbook/assets/2022-12-11 18-03-13.mp4" %}

### Challenges

decreasing acceleration is relatively complex to model in a way that's natural and this method of using logarithms runs the danger of being computationally expensive but I'm ignoring the issue right now because it's unlikely to effect performance.

## Testing

I've coded this to be straightforward to test by simply changing the x displacement. What we should see is that the ship accelerates rapidly at first then at a decreasing rate and soon reaches maximum velocity all whilst moving only in the x direction.

### Tests

| Test | Instructions     | What I expect                                                                            | What actually happens                             | Pass/Fail |
| ---- | ---------------- | ---------------------------------------------------------------------------------------- | ------------------------------------------------- | --------- |
| 1    | Run code         | Game doesn't freeze/crash or throw errors before reaching the end of content.            | The game runs without throwing errors or freezing | Pass      |
| 2    | Press Start Game | Test ship appears and moves to the right with decreasing acceleration until hitting maxV | As expected                                       | Pass      |

### Evidence

