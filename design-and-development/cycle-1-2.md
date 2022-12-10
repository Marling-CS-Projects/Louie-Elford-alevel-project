---
description: switching to Phaser was necessary but I like visual studio and typescript
---

# 2.3.1 Cycle 1 construct the project using Phaser and visual studio

## Design

### Objectives

simple objective is to create a running game using the typescript phaser 3 game template

* [x] get a window to display
* [x] create a loading screen
* [x] destroys the loading screen functionality when its task has completed
* [x] transitions scene successfully

### Usability Features

* loading screen informs the user of file being loaded
* loading screen informs the user of loading progress

### Pseudocode

this is mostly messing about with project files

#### game.ts

{% code lineNumbers="true" %}
```javascript
import phaser
import config
import LoadingScene

class Game extends Phaser.game {
    constructor
}

const game = new Game(config)
```
{% endcode %}

#### config.ts

{% code lineNumbers="true" %}
```javascript
const config = {
    //config options go here. This tells phaser how
    //to configure the game window and loads the 
};

export config
```
{% endcode %}

#### LoadingScreen.ts

{% code lineNumbers="true" %}
```javascript
class load extends phaser.scene {
    constructor()
    preload() {
        //load assetts
        //listen for loading events
        //update graphics based on loading events
        //destroy loading graphics
        //transition scene
    }
}

export load
```
{% endcode %}

## Development

### Outcome

I successfully created a window with a phaser game instance and a functioning loading screen. The process was simple: I used event listeners to pass loading progress to graphics objects displaying loading text and a progress bar. The built in phaser loader processes the assets and creates a variety of different events, I made use of the file\_load and the progress events.

{% embed url="https://newdocs.phaser.io/docs/3.55.2/Phaser.Loader.Events.FILE_LOAD" %}

{% embed url="https://newdocs.phaser.io/docs/3.55.2/Phaser.Loader.Events.PROGRESS" %}

### Challenges

this.load.on('fileprogress', listener) event listener didn't work correctly when testing using either chrome or edge with the phaser3 logo file. I replaced it with the this.load.on('load', listener) event listener

## Testing

I did some simple tests, the loading screen displayed correctly when launched, there's not much to go wrong here because user input is pretty much nonexistent.

### Tests

| Test | Instructions  | What I expect                                                                                                                   | What actually happens                                                                          | Pass/Fail |
| ---- | ------------- | ------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- | --------- |
| 1    | Run Code      | local web server hosts the game window which displays a loading screen then transitions to the currently blank main menu scene. | As expected. I used console.log to test if the menu scene loaded and the log message displayed | Pass      |
| 2    | Press buttons | nothing happens                                                                                                                 | As expected                                                                                    | Pass      |

### Evidence

<figure><img src="../.gitbook/assets/image (5) (1).png" alt=""><figcaption><p>loading screen in progress on local web server</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption><p>blank background and console logs showing successful start of the menu scene</p></figcaption></figure>
