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

{% tabs %}
{% tab title="LoadingScene.ts" %}
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

export default class LoadingScene extends Phaser.Scene {
	constructor(config) {
		super(config);
	}

	preload() {

		let width = this.cameras.main.width;
		let height = this.cameras.main.height;
		
		//load menu graphics
		this.load.image('StartButton', 'assets/start-button.png');
		this.load.image('SettingsButton', 'assets/settings-button.png');
		this.load.image('ControlsButton', 'assets/controls-button.png');

		//load controls graphics
		this.load.image('ControlsHelp', 'assets/controls-help.png');
		this.load.image('x', 'assets/x.png');

		//load the phaser3 logo a bunch of times to simulate loading
		this.load.image('test', 'assets/libs.png');
		this.load.image('logo', 'assets/phaser3-logo.png');
		for (let i = 0; i < 100; i++) {
			this.load.image('logo'+i, 'assets/phaser3-logo.png');
		}

		//create the loading progress bar
		let progressBar = this.add.graphics();
		let progressBox = this.add.graphics();
		progressBox.fillStyle(0x222222, 0.8);
		progressBox.fillRect((width/2) -160, (height/2) - 30, 320, 50);

		//add loading... text
		let loadingText = this.make.text({
			x: width / 2,
			y: height / 2 - 50,
			text: 'Loading...',
			style: {
				font: '20px monospace',
				color: '#ffffff'
			}
		});
		loadingText.setOrigin(0.5, 0.5);

		//add loading percentage text
		let percentText = this.make.text({
			x: width / 2,
			y: height / 2 - 5,
			text: '0%',
			style: {
				font: '18px monospace',
				color: '#ffffff'
			}
		});
		percentText.setOrigin(0.5, 0.5);

		//add asset loading text which will display the asset currently loading
		let assetText = this.make.text({
			x: width / 2,
			y: height / 2 + 50,
			text: '',
			style: {
				font: '18px monospace',
				color: '#ffffff'
			}
		});
		assetText.setOrigin(0.5, 0.5);
		
		//listen for loading progress
		this.load.on('progress', function (value) {
			//update the loading bar
			//console.log(value);
			progressBar.clear();
			progressBar.fillStyle(0xffffff, 1);
			progressBar.fillRect((width/2) -150, (height/2) - 20, 300 * value, 30);
			//update the percentage text
			percentText.setText(Math.floor(value * 100) + '%');
		});
		
		//listen for file progress
		this.load.on('load', function (file) {
			//update the loading file text with file being loaded
			assetText.setText('Loading asset: ' + file.key);
		});

		//listen for loading complete
		this.load.on('complete', function () {
			//destroy loading graphics
			console.log('complete');
			progressBar.destroy();
			progressBox.destroy();
			loadingText.destroy();
			percentText.destroy();
			assetText.destroy();
		});
	}

	create() {
		//create the logo graphic
		console.log('Starting Menu');
		this.scene.start('Menu');
	}
}
```
{% endcode %}
{% endtab %}

{% tab title="config.ts" %}
{% code lineNumbers="true" %}
```typescript
const GameConfig = {
    type: Phaser.AUTO,
    backgroundColor: '#125555',
    width: 1920,
    height: 1080,
	scale: {
        mode: Phaser.Scale.FIT,
        autoCenter: Phaser.Scale.CENTER_BOTH
    },
	scene: null
};
export default GameConfig;
```
{% endcode %}
{% endtab %}

{% tab title="game.ts" %}
{% code lineNumbers="true" %}
```typescript
import 'phaser';
import GameConfig from './config';
import LoadingScene from './LoadingScene';
import MenuScene from './MenuScene';
import ControlsScene from './ControlsScene';
import GameRun from './GameRun';

class Game extends Phaser.Game {
	constructor(GameConfig) {
		super(GameConfig);
		//add scenes
		this.scene.add('Loading', LoadingScene);
		this.scene.add('Menu', MenuScene);
		this.scene.add('Controls', ControlsScene);
		this.scene.add('GameRun', GameRun);

		//start loading the game
		this.scene.start('Loading');
	}
}

let game = new Game(GameConfig);
```
{% endcode %}
{% endtab %}
{% endtabs %}

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
