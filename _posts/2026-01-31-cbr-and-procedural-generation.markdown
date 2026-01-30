---
layout: post
title:  "CBR and Procedural Generation"
date:   2026-01-31 07:03:00 +1000
categories: gamedev
---


Procedural generation is one of those things that just keeps popping up in gamedev no matter where you look. I don’t think that will change, and someday, someone will want to know how SCP - Containment Breach Reborn’s (CBR) procgen worked.

Now, Hyperity has played with a few versions of procedural generation. For a currently shelved Hyperity project, known only by its internal name of CW, a basic system that distributed rooms and connected them up was designed. It was written in Python and used cv2 to draw an image where white pixels were hallways, and black pixels were free space. It worked pretty decently, but never reached the Godot implementation stage, as CW was shelved.

Placing a bunch of squares on a grid is easy when the squares are all the same size, but CBR rooms aren’t all the same size, or even the same shape. There’s not even a set number of rooms in CBR. The next iteration of the CW procgen saw squares with different sizes being distributed across the image, although the positioning was still completely random. A simple check for existing rooms and a limit to the amount of tries the algorithm could try to place a room was all it needed.

The next step was to pack all the shapes into a smaller space. There’s a term for this stuff, called a Packing Algorithm, which are solutions to Packing Problems. The specific type we use (to my limited knowledge) would be called an online bin packing algorithm. Why ‘online’? Because an online algorithm doesn’t know the entire set of rooms, whereas an offline algorithm would know the entire set. By using an online algorithm, the order in which we provide the rooms to be placed changes the outcome of the distribution. The order is shuffled depending on the random number generator’s seed, creating a deterministic procedural generation algorithm.

However, there’s a little more to it than just that, so I’ll walk you through the pseudocode.

- Algorithm start, provided a list of rooms
- Create a small grid around the size of the first room in the list.
- For each room:
  - Find the best fitting position of the room on the grid by searching over every grid cell
  - If there’s not enough space, make the grid bigger and try again
  - When space has been found, keep track of the room’s size, and where it was placed
  - All rooms should check if a room of similar or smaller size has been placed. If it has, find where it stopped searching for space for that size of room, and start searching from there.
  - Make sure there’s at least a single grid cell space between all other place rooms
- Algorithm end.
That, when given a list of room shapes, will produce a relatively well-packed distribution of rooms with enough space to add hallways in.

And speaking of hallways, the hallways happen in three stages.
0. Create a weightmap The weightmap is a 2d grid where each cell has an integer value of steps from the origin, with the origin starting as the player’s spawn room. This is an initialisation stage, and I don’t count it as part of the hallway algorithm.
1. From each doorway (user-defined room entrance):
  - Backtrack along the lowest stepped cells to reach the origin, and mark each step as a hallway cell.
  - Regenerate the weightmap with each new hallway cell counting as the origin. This makes new hallways connect with existing hallways.
2. Perform an ‘orientation pass’ on the hallway cells, converting them from ‘alive or dead’ into hallway type (e.g. straight hallway, X intersection hallway, North-west corner hallway)
3. Each hallway is given a random number within a small range (0 to 5 at the time of writing). If two connected hallways have the same random number, a divider wall will not spawn between them.

And that’s it. It’s a fairly simple system in theory, and although the implementation tastes like spaghetti, it works like a charm and is easy to add new rooms to. Even better, each step of the overall generation system is its own system built off Godot’s Tilemaps, so modders could potentially swap out the generation algorithms for their own code.