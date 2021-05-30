# JobWorld
When we want to make a large open world of voxels we tend to divide up a world into chunks. This makes it so that we allocating the world in pieces rather than having to manage thousands of individual voxels. (Although if using an Octree data structure this might not be a bad idea (See notes for more info). We will not be using Octrees though). 

But now we have to manage the chunks. To have a player that can walk around the a terrain "forever" in any direction we have to keep making chunks appear in the direction the player is going. To solve this we will check for chunks that have gone outside of a certain area and move them to the opposite side of the terrain, then redraw their meshes.

We can start illustrate the concept by making a single "row" of cubes and then checking if the player has moved passed a cube, if the player has then move the cube to the other side of the row of cubes.

```
    foreach chunk in Chunks {
        if (player.x > chunk.x) {
            chunk.x += numCubes;
        }
    }
```