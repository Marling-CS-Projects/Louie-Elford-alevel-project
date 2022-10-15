# Cycle 3.5 ensuring ship objects are creating within the scene scope

to ensure that ship objects work correctly in future cycles and for testing they need to be better integrated into the phaser framework. Currently they are created within the create() method of the scene by assigning a JSON object onto the ship class.

I plan to use the register method to do integrate the ship class into the object management system already in place in phaser 3

{% embed url="https://newdocs.phaser.io/docs/3.52.0/focus/Phaser.GameObjects.GameObjectFactory.register" %}

### Development

{% code lineNumbers="true" %}
```
Phaser.GameObjects.GameObjectFactory.register('testship', function (this: Phaser.GameObjects.GameObjectFactory, x, y, key, frame) {
		console.log("making testship:");
		const testship = new Ship(this.scene, x, y, key, frame);
		console.log(testship);
		this.displayList.add(testship);
		this.updateList.add(testship);
		this.scene.events.on('update', testship.update, testship);
		return testship;
});
```
{% endcode %}

I added this code to the bottom of my Ship.ts file containing the ship class and can now create new ships using this.add.\[ShipClass(in this case testship)]\(params) from within the scene object. This theoretically also makes it much easier for me to add the ability to make custom ships that aren't hard coded in further development.
