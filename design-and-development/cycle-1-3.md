---
description: game needs a menu
---

# 2.3.2 Cycle 2 making the main menu

## Design

### Objectives

The game needs a main menu Scene for the player to navigate and act as a point to launch the GameRun Scene from

* [x] main menu GUI
* [x] main menu GUI interactivity
* [x] launches GameRun Scene successfully
* [x] controls button to display game controls

### Usability Features

* it needs to be clear to the user what each element of the GUI means
* user needs feedback for when they interact with a button (i.e when hovering over buttons)

### Key Objects

| Object Name    | Use                                                                                             |
| -------------- | ----------------------------------------------------------------------------------------------- |
| MenuScene      | contains code that constructs the menu using assets from the assets folder and the input plugin |
| ControlsScene  | has an image with guidance on the controls (placeholder right now)                              |
| StartButton    | starts the GameRun scene and stops the MenuScene                                                |
| ControlsButton | launches the ControlsScene                                                                      |

### Pseudocode

```
MenuScene extends phaser.scene {

    let StartButton = image(properties...)
    .on(interaction), ()=> {
        //start the GameRun scene and stop this scene
    }
    
    let ControlsButton = image(properties...)
    .on(interaction), ()=> {
        //open the controls scene parralel to this one
    }
}
```

```
ControlsScene extends phaser.scene {

    image(properties...) // control help image
    
    let CloseMenuButton = image(properties...)
    .on(interaction), ()=> {
        //close ControlsScene
    }
}
```

## Development

### Outcome

I decided to switch from 800 by 600 to full HD resolution whilst coding, fortunately only one of the GUI elements used absolute values for its position and the change was straightforward.

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption><p>old resolution during early development of the menu</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (4) (1).png" alt=""><figcaption><p>new resolution with completed menu (placeholder graphics)</p></figcaption></figure>

the menu works well and it's possible to move onto the game now.

{% code lineNumbers="true" %}
```typescript
const config = {
  key: "LoadingScene",
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

export default class MenuScene extends Phaser.Scene {
	constructor(config) {
		super(config);
	}

	create() {
		
		//startButton that logs 'Start Game' when pressed (add Game Scene)
		let startButton = this.add.image(this.game.renderer.width/2, this.game.renderer.height/2, 'StartButton')
			.setDepth(1)
			.setInteractive()
			.on('pointerover', ()=> {
				startButton.setScale(1.2);
			})
			.on('pointerout', ()=> {
				startButton.setScale(1);
			})
			.on('pointerup', ()=> {
				console.log('Start Game');
				this.scene.start('GameRun');
			});
		
		//settingsButton that logs 'Settings' when pressed (add settings scene)
		let settingsButton = this.add.image(this.game.renderer.width/2, (this.game.renderer.height/2) +100, 'SettingsButton')
			.setDepth(1)
			.setInteractive()
			.on('pointerover', ()=> {
				settingsButton.setScale(1.2);
			})
			.on('pointerout', ()=> {
				settingsButton.setScale(1);
			})
			.on('pointerup', ()=> {
				console.log('Settings');
			});

		//controlsButton, currently opens a placeholder controls Scene
		let controlsButton = this.add.image(this.game.renderer.width/2, (this.game.renderer.height/2) +200, 'ControlsButton')
			.setDepth(1)
			.setInteractive()
			.on('pointerover', ()=> {
				controlsButton.setScale(1.2);
			})
			.on('pointerout', ()=> {
				controlsButton.setScale(1);
			})
			.on('pointerup', ()=> {
				console.log('Controls');
				this.scene.launch('Controls');
			});
	}
}
```
{% endcode %}

### Challenges

finding the correct syntax for utilizing the phaser input plugin was relatively difficult provided that the phaser documentation is not very user friendly.

## Testing

There's still minimal user input and there aren't any calculations happening so testing was simple. testing involved running the game and ensuring the GUI elements displayed in the right places and that the console logs for Scene launching were appearing.

### Tests

| Test | Instructions     | What I expect                                                                     | What actually happens | Pass/Fail |
| ---- | ---------------- | --------------------------------------------------------------------------------- | --------------------- | --------- |
| 1    | Run code         | Menu Scene opens after the loading completes                                      | As expected           | Pass      |
| 2    | Press Settings   | console log                                                                       | As expected           | Pass      |
| 3    | press controls   | opens controls scen                                                               | As expected           | Pass      |
| 4    | press Start game | stops the Menu Scene and starts the GameRun Scene which logs its start to console | As expected           | Pass      |

### Evidence

{% file src="../.gitbook/assets/2022-09-22 11-51-07.mp4" %}
