# AI system to simulate combat behaviors in a FPS game

## Demo
See this youtube [playlist](https://www.youtube.com/playlist?list=PLeGS7otZ9mSexXzFVHJJb5Oaj1MCPA3rA) with many videos showing how the AI system behaves

## Introduction
This is my final project of my MSc. in Computer Science (2014-2016) at the University of Girona, Catalonia, Spain. Special  thanks to Dr. Gustavo Patow for helping me with this project.

The aim of this project is to build an AI system for an existing game. To do so, I replaced the AI system of the Shooter Game provided by the Library of Unreal Engine 4 with my own AI system. I didn't want to lose time developing a game. I wanted to focus only on the design and implementation of the AI system, and that's what I did.

### Why behaviors trees?
After reading about the different techniques used to implement AI system (FSM/HFSM, HTN, Utility Systems...) I decided to implement it using behaviors trees. Behaviors Trees are very simple and powerful. BT allow the designer to implement human-like behaviors in an easy way.

### Why Unreal Engine 4?
I choose UE4 (4.10) because it has a built-in implementation of behaviors trees, a pathfinding system, a perception system and a powerful query system (EQS).

### Goals
The main goal of this project was, obviusly, achieve a human-like intelligence for a shooter game. To do this I set many sub-goals:
- Tactical pathfinding. The NPC's should, not only move using the shortest path, but also move using the best path, tactically speaking. That is to say, avoid enemies characters or unsafe positions.
- Player prediction. The system should be able to predict the player position in a time tt given the position of the player in time t.
- Spatial dynamic reasoning. The NPC's should be able to reason about the space and identify the best positions to cover or attack.
- Group Behavior. Ideally, the NPC's should act as a group or squad, showing group behaviors like flanking or scorting. 

## The AI system
The AI system can be divided into three phases: Sense, Think and Act.

### Sense
To implement the sense phase I used the UE4 Perception System. Specifically I used the sight and hearing system to determine if the NPC had seen or heard the player.
The rest of the data that use the system to take decicions comes from his attributes (health, ammo...) and computations (distance to the player, player's visibility, cover positions...)

### Think
To implement the think phase I used several behaviors trees:
- Idle behavior
- Patrol behavior
- Search behavior
- Fight behavior

![Transtions of the different behavior trees](https://s13.postimg.org/6bgn3mjs7/fsm.png)

You can see the details of each behavior in the UE4 editor. Here is just the attack branch of the fight behavior, the most complex one:

![Attach branch of the Fight behavior](https://s17.postimg.org/4qv84zfwf/image.png)

### Act
The act phase is simply made up from the leaf nodes of the different behavior trees. In general, these were the tasks (with different parameters depending on the situation):
- Wait
- Reload
- Move to (shortest path  or safest path)
- Execute a tactical computation
  - Next patrol position
  - Next search position
  - Next cover position
  - Next attack position
  - Next suppression position

## Implementation (brief)

To see the full details about the implementation just download the source code. Here I am going to talk about very biefly of the most important things.

### Visibility algorithm
Thanks to the perception system of UE4 was easy to determine if an NPC was seeing the player or not. However, as I wanted to compute a tactical pathfinding, I needed the exact visibility of the player (when was being seen by any NPC). To do thos I implemented a visibility algorithm based on Ray tracing. This algorithm uses a very low number of Ray Tracer because it only does a RayTrace against the vertex of each obstacle. Read more about this technique [here](http://www.redblobgames.com/articles/visibility/)

![visibility_algorithm.png](https://s22.postimg.org/6tewtygcx/visibility.png)

### Tactical pathfinding
Once I implemented the visibility algorithm I managed to modify the A star algorithm of UE4 to do the pathfinding operations. I included weights in the computation of the cost to include also the safety of the path (if the player was seeing that area or not).

### Player prediction
To calculate the player prediction I used a technique called influence maps (checkout this [article](http://aigamedev.com/open/tutorial/influence-map-mechanics/). Every time the player is lost (his position is unknown by the AI system) a influence maps starts to run with its highest point at the last player's position. This way, the NPC's calculate the most probable position where the player might be and move towards that position to look for him.

### Group behavior
To create a feeling of group within the NPC's I simply created some shared data between them to simulate the communication and exchange of information. For example, to attack the player, the attack position of each bot is shared and thus known by all of the other bots. Hence, the other bots calculate new attack positions different from the attack positions of the other bots. This way we can maximize the spread of the bots when attacking.
I used this same technique to attack, flank, cover, search and suppress.

## EQS tests
To collect data from the environment I defined several custom tests to perform Environment Queries (EQS). This tests returned me some information about the environment that was very useful to make decisions:
- Pathfinding cost test
- Distance test
- Visibility test
- Behind obstacles test
- Near obstacles test
- Random test (add some noise)
- Influence test
- Spread test
- Flank test
- Avoid repeated positions tests

Using one of these tests or combining many of them I made all the queries used in the game.
![visibility test.png](https://s12.postimg.org/vca7d2ynx/visibility_test.png)

## Set up
To set up the project just download or clone the source files and compile them using Visual Studio with the UE4 plugins. Then the UE4 will open with the test map. There you can check the behavior trees of the bots or simplyadd more bots, change their positions or attributes.

## References
To do this project I looked at so many websites that makes no sense to post them all here. However, I think that is worth mention:

- [AIGameDev](http://aigamedev.com/)
- [Game AI PRO Books (I, II)](http://www.gameaipro.com/) 
