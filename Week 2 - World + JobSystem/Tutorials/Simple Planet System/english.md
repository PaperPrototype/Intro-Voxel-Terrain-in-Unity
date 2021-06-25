This tutorial assumes you've finished week 2's lecture. You can view the final code (aka source) fo this project in the "Intro-to-Voxel-Terrain-in-Unity" repo linked in the README file.

To make a planet first we have to figure out how to calculate where a voxel should be, and where it shouldn't be.

Make a new micro project in you project (don't make anothe Unity project because we will be using previous code from other lectures).

```
Assets/
    |___...other micro projects from course...
    |___SimplePlanet
        |___SimplePlanet.cs
        |___SimplePlanetChunk.cs
        |___SimplePlanet.unity (scene)
```

Now paste all the code from the `JobWorldChunk` class into the `SimplePlanetChunk` class. You will need to add some `using` statements at the top to include support for the `NativeArray`'s and the Jobs.

```cs
using UnityEngine;
using Unity.Collections;
using Unity.Jobs;

public class SimplePlanetChunk : MonoBehaviour
{
    // ... snipping pasted code
}
```

Also you will need to change the constructor function to be the same as the class name (since it was pasted from another class it is different).

```cs
public class SimplePlanetChunk : MonoBehaviour
{
    // ... snip

    public SimplePlanetChunk(Material m_material, Vector3 m_position)
    {
        // ... snip
    }

    // ... snip
}
```

Now past all the code from the JobWorld class (week 2) into the SimplePlanet class.

The current noise algorithm the `SimplePlanetChunk` is using right now is not going to work for making a planet. We could just edit the current job from the `JobWorld` project but that would break our previous projects.

Lets make a new Job in the `JobDefs` class (we made the `JobDefs` class earlier in the course to hold all of the Job's we would make, since there will be many in the course).

Copy our old `ChunkJob` paste it and rename it to `PlanetChunkJob`.

(For now we are not using the `FastNoiseLite` instance in the Job, but leave it there because we will use it later)

Now we have to change the `IsSolid` function to do planet generation. We will calculate the distance from the center of the planet to the current voxel position. If the distance is greater than the planet radius we want we will return false, meaning a voxel should not be made there.

Change `PlanetChunkJob` to take in a planetRadius, and use the planetRadius variable to calculate the distance between a voxel's position and the center of the planet.

```cs
    public struct PlanetChunkJob : IJob
    {
        public float planetRadius;

        // ... snipping unchanged code

        private bool IsSolid(FastNoiseLite noise, int x, int y, int z)
        {
            float distance = Vector3.Distance(new Vector3(x, y, z) + chunkPos, Vector3.zero);

            if (distance <= planetRadius)
            {
                return true;
            }
            else
            {
                return false;
            }
        }
    }
```

Right now we assume the planet is at the center of the world so we just use `Vector3.zero` (make sure the planet gameObject you make later is actually at the center of the world). If you don't want the planet to be in the center of the world then feel free to add a `Vector3 planetPos` to the job struct (make sure to set this variable later tho!).

Change `SimplePlanetChunk`'s `ScheduleDraw` function use the new job we created. Also add a member variable to hold a reference of the `SimplePlanet` parent class, so that we can access a planetRadius variable that we will add to the `SimplePlanet` class.

In the following code I highlight the changes 

```cs
public class SimplePlanetChunk
{
    // ... snipping unchanged code

    private SimplePlanet m_owner; // added

    // ... snipping unchanged code

    private JobDefs.PlanetChunkJob m_chunkJob; // changed

    public SimplePlanetChunk(SimplePlanet owner, Material m_material, Vector3 m_position) // changed
    {
        m_owner = owner; // added

        // ... snipping unchaged code
    }

    public void ScheduleDraw()
    {
        if (needsDrawn == true)
        {
            // ... snipping unchanged code

            m_chunkJob = new JobDefs.PlanetChunkJob(); // changed
            m_chunkJob.planetRadius = m_owner.planetRadius; // added
            // ... snipping unchanged code
        }
    }

    // ... snipping unchanged code
}
```

Now lets add the planetRadius variable, and actually generate this planet! Open up SimplePlanet.cs and add the following boiler plate code.

```cs
using UnityEngine;

public class SimplePlanet : MonoBehaviour
{
    public Material material;
    public JobWorldChunk1[,,] chunks = new JobWorldChunk1[DataDefs.chunkNum, DataDefs.chunkNum, DataDefs.chunkNum];
    public Transform player;
    public float planetRadius = 100;
}
```

We take in a reference to a material for the chunk class to assign to its gameObject's `MeshRenderer`. 

Then we have a 3D array of `SimplePlanetChunk`'s which will hold each chunk. It is a 3D array becuase we will now be generating chunks in all three directions, whereas in the `JobWorld` from week 2 we only generate in directions (namely `x` and `z`).

Then we take in a reference to the player so that we can generate the world around the player. And finally we have a paramter for setting the planet radius.

Now paste in some familiar chunk recycling code.

```cs
public class SimplePlanet : MonoBehaviour
{
    // ... snipping unchanged code
    
    private void RecycleChunks()
    {
        for (int x = 0; x < DataDefs.chunkNum; x++)
        {
            for (int y = 0; y < DataDefs.chunkNum; y++)
            {
                for (int z = 0; z < DataDefs.chunkNum; z++)
                {
                    // x
                    if (player.position.x + offset < chunks[x, y, z].gameObject.transform.position.x)
                    {
                        chunks[x, y, z].gameObject.transform.position -= new Vector3(DataDefs.chunkNum * DataDefs.chunkSize, 0, 0);
                        chunks[x, y, z].needsDrawn = true;
                    }
                    if (player.position.x - offset > chunks[x, y, z].gameObject.transform.position.x)
                    {
                        chunks[x, y, z].gameObject.transform.position += new Vector3(DataDefs.chunkNum * DataDefs.chunkSize, 0, 0);
                        chunks[x, y, z].needsDrawn = true;
                    }

                    // y
                    if (player.position.y + offset < chunks[x, y, z].gameObject.transform.position.y)
                    {
                        chunks[x, y, z].gameObject.transform.position -= new Vector3(0, DataDefs.chunkNum * DataDefs.chunkSize, 0);
                        chunks[x, y, z].needsDrawn = true;
                    }
                    if (player.position.y - offset > chunks[x, y, z].gameObject.transform.position.y)
                    {
                        chunks[x, y, z].gameObject.transform.position += new Vector3(0, DataDefs.chunkNum * DataDefs.chunkSize, 0);
                        chunks[x, y, z].needsDrawn = true;
                    }

                    // z
                    if (player.position.z + offset < chunks[x, y, z].gameObject.transform.position.z)
                    {
                        chunks[x, y, z].gameObject.transform.position -= new Vector3(0, 0, DataDefs.chunkNum * DataDefs.chunkSize);
                        chunks[x, y, z].needsDrawn = true;
                    }
                    if (player.position.z - offset > chunks[x, y, z].gameObject.transform.position.z)
                    {
                        chunks[x, y, z].gameObject.transform.position += new Vector3(0, 0, DataDefs.chunkNum * DataDefs.chunkSize);
                        chunks[x, y, z].needsDrawn = true;
                    }
                }
            }
        }
    }
}
```

This is exactly the same code as the `RecycleChunks` function from week 2's `JobWorld` the only difference is that we are now recycling in the `y` axis.

As a refresher the code goes through each chunk and checks it's position, if say the `x` position of the chunk is farther away than the offset the chunk gets moved to the other side of the world, and its `needsDrawn` variable is set to true.

Make a function that schedules all the draw jobs for each chunk.

```cs
    private void ScheduleChunks()
    {
        for (int x = 0; x < DataDefs.chunkNum; x++)
        {
            for (int y = 0; y < DataDefs.chunkNum; y++)
            {
                for (int z = 0; z < DataDefs.chunkNum; z++)
                {
                    chunks[x, y, z].ScheduleDraw();
                }
            }
        }
    }
```

Now make another function that Completes all the draw jobs each chunk. When complete is called it waits for the job to finish and then sets the chunks mesh.

```cs
    private void CompleteChunks()
    {
        for (int x = 0; x < DataDefs.chunkNum; x++)
        {
            for (int y = 0; y < DataDefs.chunkNum; y++)
            {
                for (int z = 0; z < DataDefs.chunkNum; z++)
                {
                    chunks[x, y, z].CompleteDraw();
                }
            }
        }
    }
```

Now finally in start we can generate each chunk in a grid. Then make sure they are in the correct position relative to the player, before we spend a long time drawing them, by calling the `RecycleChunks` function.

```cs
    private void Start()
    {
        for (int x = 0; x < DataDefs.chunkNum; x++)
        {
            for (int y = 0; y < DataDefs.chunkNum; y++)
            {
                for (int z = 0; z < DataDefs.chunkNum; z++)
                {
                    Vector3 position = new Vector3(x * DataDefs.chunkSize, y * DataDefs.chunkSize, z * DataDefs.chunkSize);
                    chunks[x, y, z] = new SimplePlanetChunk(this, material, position);
                }
            }
        }

        RecycleChunks(); // "recycle" chunks into the correct position to prevent from drawing them and then having to move them again (although if the player is super far away then this problem will still happen)
    }
```

In `Update` call the `ScheduleChunks` and `CompleteChunks` functions to generate all the chunks meshes. We also recycle all the chunks right before we draw them.

```cs
    private void Update()
    {
        RecycleChunks();

        ScheduleChunks();
        CompleteChunks();
    }
```

Open the SimplePlanets.unity scene and add a gameObject called "Planet" (Make sure it is in the center of the world!) then attach the `SimplePlanet` component script and assign the material. Also create a gameObject to act as the player, so the terrain can generate around it.

Now if you hit play you should see a planet!

EDIT: the planet is broken I am working on fixing this now!

TODO finish Mock tutorial in The-Teaching-Handbook repo
TODO add planetary gravity and orientation
TODO show player controller code change for making the planet

You can join my discord for help from me! https://discord.gg/QhqTE4t2tR

Here is the player controller code change for making the planet gravity script work, for those wonderful poeple who don't want to wait for me to finish this tutorial

```cs
    private void UpdateMovement()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            m_rb.velocity += transform.up * jumpVelocity; // the jump code was originally "m_rb.velocity += new Vector3(0, jumpVelocity, 0);
        }

        // snipping unchanged code
    }
```

(it was getting late and I needed to go to bed, I have been coding and writing all day)

If you don't want to wait for me to add this you can visit the