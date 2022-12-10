---
description: >-
  need a way to manage movement of the ships that will allow for an RTS control
  scheme
---

# 2.3.5 Cycle 5 Movement queue and managing ship movement

## Design

Each ship object will contain a queue with coordinates, when the player uses a move command coordinates for the waypoint will be added to the queue. The ships will move toward the next coordinate on the queue until they reach the coordinate then remove it from the queue and repeat.

### Objectives

The ship moves from one point to another.

* [x] create a queue class
* [x] add some test move waypoints
* [x] move the ship towards the waypoint
* [x] when the ship reaches a waypoint it stops navigating to it and instead navigates to the next waypoint

### Usability Features

* this is purely computational and won't see any direct interaction with the user as such I don't need to worry about Usability Features. The user interaction will later stem from the use of movement commands to add waypoints.

### Key Variables

| Variable Name | Use                                                                                                                     |
| ------------- | ----------------------------------------------------------------------------------------------------------------------- |
| WayPointQueue | Queue of waypoints for the ship to navigate to implemented with a custom queue class                                    |
| queue (class) | a queue class that will enque and deque variables/objects correctly and allow for peeking the first in the queue order. |

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

I implemented a queue class at the top of the ship.ts file and then proceeded to add code to navigate to the next waypoint in the queue

### Outcome

The ship can now be fed orders from a queue allowing for RTS style queueing of movement orders.

However there are now issues to do with controlling deceleration and the ship isn't able to turn yet.

### Challenges

not particularly challenging, the queue worked surprisingly well.

## Testing

### Tests

| Test | Instructions     | What I expect                                                                 | What actually happens                             | Pass/Fail |
| ---- | ---------------- | ----------------------------------------------------------------------------- | ------------------------------------------------- | --------- |
| 1    | Run code         | Game doesn't freeze/crash or throw errors before reaching the end of content. | The game runs without throwing errors or freezing | Pass      |
| 2    | Press Start Game | Test ship appears and moves between hard coded waypoints.                     | As expected                                       | Pass      |

### Evidence

{% file src="../.gitbook/assets/2022-10-05 15-34-14.mp4" %}

The ship can be seen moving from waypoitn to waypoint
