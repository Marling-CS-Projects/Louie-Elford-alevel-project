# 2.1 Design Frame

## Systems Diagram

![](<../.gitbook/assets/Untitled (1).jpg>)

This diagram shows the different parts of the game that I will focus on creating. I have split each section into smaller sub-sections. Throughout the development stage, I will pick one or two of these sections to focus on at a time to gradually build up and piece together the game. I have broken the project down this way as it roughly corresponds to the success criteria. Elements with a question mark in brackets next to them "(?)" are further development only.

## Usability Features

Usability is an important aspect to my game as I want it to be accessible to all. There are 5 key points of usability to create the best user experience that I will be focusing on when developing my project. These are:

### Effective

Users can achieve the goal with completeness and accuracy. To do this, I will make it easy for the players to realise what they need to do in order to win a game.

#### Aims

* Keep win conditions for games straightforward
* give the player a prompt explaining how the win conditions work and an indicator of their progress towards it i.e either a list of the remaining ships each player has or an indicator of how many points each player has.

### Efficiency

The speed and accuracy to which a user can complete the goal. To do this, I will create a menu system which is easy to navigate through in order for to find what you are looking for. The information which is more important can be found with less clicks.

#### Aims

* Create a menu system that is quick and easy to navigate through
* Create a controls system that isn't too complicated but allows the player to do multiple actions

### Engaging

The solution is engaging for the user to use. To do this, I will create 2d assets for the ships and ideally a background for the first map so that the player doesn't spend all of their time looking at polygons on empty space. Further development could include making the solution more engaging for the players by adding additional ships or maps to play with.

#### Aims

* Create 2d assets for the ships
* Create a background for the first map

### Error Tolerant

The solution should have as few errors as possible and if one does occur, it should be able to correct itself. To do this, I will write my code to manage as many different game scenarios as possible so that it will not crash when someone is playing it.

#### Aims

* The game doesn't crash
* The game does not contain any bugs that damage the user experience

### Easy To Learn

The solution should be easy to use and not be over complicated. To do this, I will create simple controls for the game. I will make sure that no more controls are added than are needed in order to keep them as simple as possible for the players.

#### Aims

* Create a list of controls for the game.
* Create an in-game guide that helps players learn how to play the game like a help window.
* allow access to all the basic functionality through simple controls i.e movement and attacking should be simple but any more advanced controls need to be optimized to be easy to use for the experienced player instead.

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
        create players
        create platforms
        create puzzle elements
        create enemies
        create obstacles
        create finishing position
        create key bindings
    end procedure
    
    procedure update
        handle key presses
        move player
        move interactable objects
        update animations
        check if player at exit
    end procedure
    
end class
```
