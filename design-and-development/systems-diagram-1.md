# 2.3 Revised Design Frame

## Reasons for Revision

I've decided to move to a much simpler approach by using a template for Phaser 3, a game framework which will make creating the game much simpler. using C++ and making the game with OpenGL and SLD(or equivalent) libraries would be far too time consuming.

The switch to phaser will lead to me focusing more on the game components as all the groundwork is in place with the library.

## Systems Diagram

<figure><img src="../.gitbook/assets/Louie Elford project ideation.jpg" alt=""><figcaption><p>systems diagram</p></figcaption></figure>

This diagram is a simple representation of the baseline requirements of the game. I will work to develop the loading and main menu scenes initially and will likely update the logo and menu graphics, Game.ts and config.ts files as needed.

## Usability Features

Usability is an important aspect to my game as I want it to be accessible to all. There are 5 key points of usability to create the best user experience that I will be focusing on when developing my project. These are:

### Effective

Users can achieve the goal with completeness and accuracy. To do this, I will make it easy for the players to understand what they need to do in order to win a game.

#### Aims

* Keep win conditions for games straightforward
* give the player a prompt explaining how the win conditions work and an indicator of their progress towards it i.e either a list of the remaining ships each player has or an indicator of how many points each player has.

### Efficiency

The speed and accuracy to which a user can complete the goal. To do this, I will create a menu system which is easy to navigate through in order for to find what you are looking for. The information which is more important can be found with less clicks.

#### Aims

* Create a menu system that is quick and easy to navigate through
* Create a controls system that isn't too complicated but allows the player to queue actions or control multiple ships

### Engaging

The solution is engaging for the user to use. To do this, I will create 2d assets for the ships.

#### Aims

* Create 2d assets for the ships

### Error Tolerant

The solution should have as few errors as possible and if one does occur, it should be able to correct itself. To do this, I will write my code to manage as many different game scenarios as possible so that it will not crash when someone is playing it.

#### Aims

* The game doesn't crash
* The game does not contain any bugs that damage the user experience

### Easy To Learn

The solution should be easy to use and not be over complicated. To do this, I will create simple controls for the game. I will make sure that no more controls are added than are needed in order to keep them as simple as possible for the players.

#### Aims

* Create a list of controls for the game.
* allow access to all the basic functionality through simple controls

## Pseudocode for the Game

### Pseudocode for game

This is the basic layout of the object to store the details of the game. This will be what is rendered as it will inherit all important code for the scenes.

```
object Game
    type: Phaser
    parent: id of HTML element
    width: width
    height: height
    physics: set up for physics
    scenes: add all menus, levels and other scenes
end object

render Game to HTML web page
```

### Pseudocode for a level

This shows the basic layout of code for a Phaser scene. It shows where each task will be executed.

```
class Level extends Phaser Scene

    procedure preload
        load all sprites and music
    end procedure
    
    procedure create
        start music
        draw background
        create ships
        create key bindings
    end procedure
    
    procedure update
        interpret player commands
        move ships and projectiles
        detect projectile hits
        update animations
        check for win conditions
    end procedure
    
end class
```
