---
description: need a way to control the ships that will allow for an RTS control scheme
---

# 2.3.4 Cycle 4 controlling ship movement

## Design

Each ship object will contain a queue with coordinates, when the player uses a move command coordinates for the waypoint will be added to the queue. The ships will move toward the next coordinate on the queue until they reach the coordinate then remove it from the queue and repeat.

### Objectives

The ship moves naturally from one point to another.

* [x] create a queue class
* [x] add some test move waypoints
* [x] move the ship towards the waypoint
* [x] turn the ship if it needs to turn
* [x] the ship only moves forwards and turns (no strafing and no reversing for now.)

### Usability Features

* this is purely computational and won't see any direct interaction with the user as such I don't need to worry about Usability Features. The user interaction will later stem from the use of movement commands to add waypoints.

### Key Variables

| Variable Name   | Use                                                                                                                     |
| --------------- | ----------------------------------------------------------------------------------------------------------------------- |
| WayPointQueue   | Queue of waypoints for the ship to navigate to implemented with a custom queue class                                    |
| directionVector | the direction of the ship in vector form                                                                                |
| queue (class)   | a queue class that will enque and deque variables/objects correctly and allow for peeking the first in the queue order. |

### Pseudocode

```
class queue {
    array = [];
    enque() {
        //code here
    }
    dequeue() {
        //code here
    }
    peek() {
        //code here
    }
}

ship {
    WayPointQueue = new queue();
    
    move() {
        //code to navigate to waypoint
    }
    
    update() {
        move()
    }
}
```

## Development

ship moveming directly to waypoints in a queue from early development of the cycle:

{% file src="../.gitbook/assets/2022-10-05 15-34-14.mp4" %}

During development I stumbled across the fact that the ships would have to start decelerating early when travelling short distances. This and the fact that turning would further complicate this and would probably be clumsy with a simple waypoint system has led me to decide to rework the idea of waypoints. Instead of simply navigating to waypoints I have decided to instead make the move orders much more complex in order to allow for precise and pre calculated movement (this will still allow for cancelling the move order and decelerating in a straight line). The more complex move orders will allow me to accelerate and decelerate with great precision and find the most efficient path to end up at the desired point with the desired rotation.

To achieve this more complex movement I will define one move order as an arc lenght of one circle radius turning radius of the moving ship, the tangent between this circle and a second circle with the same radius and an arc length along the second circle

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption><p>simple representation of a new move order</p></figcaption></figure>

I quickly identified that this single arrangement would not work for every case. In an effort to find a computational way of determining which case to use and how they related and could be implemented I drew up a list and gave them temporary names:

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption><p>left/right denotes the relative position of the exit vector from the origin vector and Trandsverse/Direct is a description of the tangent. The tangent is transverse when it crosses a line made between the centers of the two circles and direct when it does not.</p></figcaption></figure>

after some time spent theorising I realised that the 4 cases were created by the 2^2 (4) different combinations of positions for the turning circles.\
<img src="../.gitbook/assets/image (10).png" alt="" data-size="original"> \
These positions are in turn decided by the angle between the vector tangent to the circle and the line between the two vectors.\
![](<../.gitbook/assets/image (6).png>)\
demonstration using the origin vector, this works the same for the exit vector too.

If the turning circles are on the same side of the vectors the tangent will be direct, if they're on opposite sides of the vectors the tangent will be transverse.\
![](<../.gitbook/assets/image (8).png>)

The last theory challenge is to determine when to start decelerating. I've decided to use a variable to store the distance moved for each move order for this "stopDistance".

### Complex move order Objectives

The ship needs to move along the best path to end up at the desired angle

* [ ] calculate the move order.
* [ ] store the move order effectively in the queue.
* [ ] ensure that the move order contains information about when to start decelerating if neccesary.
* [ ] move according to the next move order in the queue
* [ ] if there's another move order after the current move order don't decelerate

### Complex move order Key Variables

| Key Variable    | Use                                                                                                                                                         |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CompMove        | the class of object that will be used to represent move orders, these will be queued onto the list after being calculated by a MoveOrder function           |
| stopDistance    | variable that stores the distance required to stop, incremented if < maxStopDistance and accelerating and decreased if deccelerating. Reset when stationary |
| maxStopDistance | a constant stored in the ship.JSON file which is the stoping distance of the ship                                                                           |

### Pseudocode

```
class MoveOrder {
    //variables
    MoveOrder(ship, params) {
        //constructor which computes the move order
    }
}


// add a move order to the queue

update() {
    MoveOrder(ship, params);
}



//move using the move order
move(queue.peek) {
    //movement code
}
```

## Development

I have decided to create the curve using a phaser path gameobject, then add it to the WayPointQueue, which I will rename moveOrderQueue. This will allow me to display the move in the ui before creation using a path gameobject then simply add it to the moveOrderQueue when the player confirms it later.

since there are methods missing from the Phaser path object that would be useful and to simplify the creation of move orders I've decided to create a MovePath class. The constructor generates a move order based off two pairs of one point and an angle. these are the starting position and angle and end position and angle.

```
```

### Outcome



### Challenges



## Testing



### Tests

| Test | Instructions     | What I expect                                                                | What actually happens                             | Pass/Fail |
| ---- | ---------------- | ---------------------------------------------------------------------------- | ------------------------------------------------- | --------- |
| 1    | Run code         | Game doesn't freeze/crash or throw errors before reaching the end of content | The game runs without throwing errors or freezing | Pass      |
| 2    | Press Start Game | Test ship appears and moves up and off the screen                            | As expected                                       | Pass      |

### Evidence

{% file src="../.gitbook/assets/2022-09-29 18-16-33.mp4" %}
