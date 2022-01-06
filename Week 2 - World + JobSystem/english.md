# JobWorld
To make a large open world of voxels we can divide up the world into chunks. This makes it so that we can allocate the world in pieces rather than having to manage thousands of individual voxels. (Although if using an Octree data structure this might not be a bad idea (See notes for more info). We will not be using Octrees though). 

But now we have to manage the chunks. To have a player that can walk around the a terrain "forever" in any direction we have to keep making chunks appear in front of the player. Most systems delete old chunks and then instantiate new ones. But allocations and deletions are slow. We can make this significantly faster if we "recycle" chunks. 

A system like this is usually called a "pool", where we have a pool of objects, that when we want to an object "delete" we really just add it back to the pool for use later. If the pool is empty we just allocate a new object.

But pool's are complicated to write. Instead we will do something super simple yet fast. We will move old chunks into the position where new chunks are needed and then rebuild their meshes. Hehe, way easier.

## Recycle system in Unity
As an example we will use cubes. If the player's position moves past one of the cubes we will move the cube to the other side of the row.

![1D cube recycle](/Assets/1D_cube_recycle.png)

```
if (player.x > cube.x) {
    cube.x += numCubes
}
```

![1D cube recycled](/Assets/1D_cube_recycled.png)

and we can do this for every cube.

```
for (i = 0; i < numCubes; i++) {
    if (player.x > cube[i].x) {
        cube.x += numCubes
    }
}
```

and then we can check in the other direction by adding to the player's x position by `numCubes`, which is conveniently the right length.

```
for (i = 0; i < numCubes; i++) {
    if (player.x + numCubes < cube[i].x) {
        cube.x -= numCubes
    }
}
```

![1D cube inverse recycle](/Assets/1D_cube_inverse_recycle.png)

and if we do it in both directions the cubes will "follow" the player.

```
for (i = 0; i < numCubes; i++) {
    if (player.x > cube[i].x) {
        cube.x += numCubes
    }
    if (player.x + numCubes < cube[i].x) {
        cube.x -= numCubes
    }
}
```

to place the player in the center we can add / subtract an offset in the if statement that is half of `numCubes`

```
offset = numCubes / 2

for (i = 0; i < numCubes; i++) {
    if (player.x - offset > cube[i].x) {
        cube.x += numCubes
    }
    if (player.x + offset < cube[i].x) {
        cube.x -= numCubes
    }
}
```

Make a new micro project called "1D" in a folder called "Recycling".

```
    Assets/
    |___FastNoiseLite/
    |___DataDefs.cs
    |___White (Material)
    |___Quad/
    |___Voxel/
    |___Chunk/
    |___JobChunk/
    |___JobDefs.cs
    |___Recycling/
        |___1D/
            |___Recycle1D.cs
            |___1D.unity
```

in Recycle1D.cs we will instantiate a row of cubes, make the following boiler plate code.

```cs
using UnityEngine;

public class Recycle1D : MonoBehaviour
{
    public const int numCubes = 10;
    public const int offset = numCubes / 2;
    public GameObject[] cubes = new GameObject[numCubes];
    public Transform center;

    void Start()
    {
        for (int i = 0; i < numCubes; i++)
        {
            cubes[i] = GameObject.CreatePrimitive(PrimitiveType.Cube);
            cubes[i].GetComponent<Transform>().position = new Vector3(i - offset, 0, 0);
        }
    }
}
```

The function `GameObject.CreatePrimitive(Primitive.Cube)` is a shortcut for instantiating (creating) a built-in cube (Called a primitive). (This makes it so we don't have to manually add a renderer component, and mesh filter, and material, and mesh).

We offset the postion of the cube by `offset` to place the cubes so that the player is in the center of them. 

Also the numCubes has to be an even number that is divisible by 2. Otherwise we will end up with a decimal `.5` that will just get chopped off. Say we set numCubes to `5` and then divide by `2`. This gives us back `2` which is wrong. This is because we chopped off the `.5`, so the answer should have been `2.5` but we are storing it in an `int` and int's don't have decimals.

Why not use a `float` since they can have decimals? Because then the cubes will not be in a grid and will have positions like `2.5`. You could use floats but we will be using `int`'s.

Now we can write the code to move the cubes.

```cs
    void Update()
    {
        for (int i = 0; i < numCubes; i++)
        {
            if (center.position.x + offset < cubes[i].transform.position.x)
            {
                cubes[i].transform.position -= new Vector3(numCubes, 0, 0);
            }
            if (center.position.x - offset > cubes[i].transform.position.x)
            {
                cubes[i].transform.position += new Vector3(numCubes, 0, 0);
            }
        }
    }
```

Go into the 1D.unity scene and add the Recycle1D component (script) to a gameObject called "cubes". Create a gameObject called "center", then click on the cubes gameObject and drag the center gameObject onto the Recycle1D slot called "center".

Now hit play and move the center gameObject around in the x axis. (You may need to reset the "center" gameObject's position to (0, 0, 0)).

Yay! We've made a recycling system for the chunks!

Now make another micro project called 2D in the Recycling folder

```
    Assets/
    |___FastNoiseLite/
    |___DataDefs.cs
    |___White (Material)
    |___Quad/
    |___Voxel/
    |___Chunk/
    |___JobChunk/
    |___JobDefs.cs
    |___Recycling/
    |   |___1D/
    |   |___2D/
            |___Recycle2D.unity
            |___2D.cs
```

The code will be the same except that we will use a 2D array to hold our cubes, and we will instantiate them in a grid, much like we will for the terrain later. We use a 2D array so that the for loop's are easier to write, and we don't have to convert from 1D array indexing to 2D indexing, which is not that hard but will make this example a bit involved (complex). (There is a tutorial for this in the tutorials of this week)

paste this boiler plate code in.

```cs
using UnityEngine;

public class Recycle2D : MonoBehaviour
{
    const int numCubes = 10;
    public const int offset = numCubes / 2;
    public GameObject[,] cubes = new GameObject[numCubes, numCubes];
    public Transform center;
}

```

Instantiating the cubes in a grid changes, now the grid is 2D.

```cs
    private void Start()
    {
        for (int x = 0; x < numCubes; x++)
        {
            for (int z = 0; z < numCubes; z++)
            {
                cubes[x, z] = GameObject.CreatePrimitive(PrimitiveType.Cube);
                cubes[x, z].GetComponent<Transform>().position = new Vector3(x - offset, 0, z - offset);
            }
        }
    }
```

We center the the creating of the cubes by using the subtracting the `offset` in the position for the cubes.

The recycling code changes with an additional for loop and, includes code to check the z coordinate, which is the exactly the same code as the x coordinate.

```cs
    private void Update()
    {
        for (int x = 0; x < numCubes; x++)
        {
            for (int z = 0; z < numCubes; z++)
            {
                // x
                if (center.position.x + offset < cubes[x, z].transform.position.x)
                {
                    cubes[x, z].transform.position += new Vector3(-numCubes, 0, 0);
                }
                else
                if (center.position.x - offset > cubes[x, z].transform.position.x)
                {
                    cubes[x, z].transform.position += new Vector3(numCubes, 0, 0);
                }

                // z
                if (center.position.z + offset < cubes[x, z].transform.position.z)
                {
                    cubes[x, z].transform.position += new Vector3(0, 0, -numCubes);
                }
                else
                if (center.position.z - offset > cubes[x, z].transform.position.z)
                {
                    cubes[x, z].transform.position += new Vector3(0, 0, numCubes);
                }
            }
        }
    }
```

Now add a gameObject called cubes to the scene and drag Recycle2D onto it. Then make a gameObject to act as the center and drag it onto the slot on the Recycle2D component called "center". Now hit play and you should see a grid of cubes that recycle!

## World chunk system in Unity
To have a chunk handling system for chunks we need a "handle" for each chunk that the chunk system can use. Create a micro project called "JobWorld" (yep thats right!).

```
    Assets/
    |___FastNoiseLite/
    |___DataDefs.cs
    |___White (Material)
    |___Quad/
    |___Voxel/
    |___Chunk/
    |___JobChunk/
    |___JobDefs.cs
    |___Recycling/
    |___JobWorld/
        |___JobWorld.unity
        |___JobWorldChunk.cs
        |___JobWorld.cs
```

We will do the instantiation of the chunk gameObject manually through the `JobWorldChunk`. Paste this boiler plate code into `JobWorldChunk`.

```cs
using UnityEngine;
using Unity.Jobs;
using Unity.Collections;

public class JobWorldChunk
{
    public GameObject gameObject;

    private MeshFilter m_meshFilter;
    private MeshRenderer m_meshRenderer;
    private Mesh m_mesh;

    private NativeArray<Vector3> m_vertices;
    private NativeArray<int> m_triangles;
    private NativeArray<Vector2> m_uvs;
    private NativeArray<int> m_vertexIndex;
    private NativeArray<int> m_triangleIndex;

    private JobHandle m_handle;
    private JobDefs.ChunkJob m_chunkJob;
}
```

The `gameObject` variable will hold a reference to the actual gameObject, the `m_meshFilter` and `m_meshRenderer` hold references to the gameObjects Meshfilter and MeshRenderer components. The `m_handle` is the chunks JobHandle and the `m_chunkJob` is the chunks Job.

Also notice that the `JobWorldChunk` does not inherit from `MonoBehaviour` since it will not be a component of the chunk gameObject. Inheritince is denoted by the `:` followed by a class. The example `OurClass : AnotherClass` can be read as, `OurClass` inherits from `AnotherClass`. 

We will create a constructor. Constructors are functions that are special. They get called when we put the `new` keyword in front of a class or struct. For example if we say 

```cs
JobWorldChunk chunk = new JobWorldChunk()
``` 

the `JobWorldChunk()` with the parenthesis is the constructor. Constructor function have to have the same name as the class that holds them. Make a constructor function for `JobWorldChunk` with all of the initalization code.

```cs
public class JobWorldChunk
{
    // ... snipping unchanged code ... //

    public JobWorldChunk(Material m_material, Vector3 m_position)
    {
        m_mesh = new Mesh();

        gameObject = new GameObject();
        gameObject.transform.position = m_position;

        m_meshFilter = gameObject.AddComponent<MeshFilter>();
        m_meshFilter.mesh = m_mesh;

        m_meshRenderer = gameObject.AddComponent<MeshRenderer>();
        m_meshRenderer.material = m_material;
    }
}
```

The constructor takes in a Material and a position (Vector3) to place the chunk at. We then instantiate a gameObject and set the `gameObject` refernce to it. Then we add a meshFilter and meshRenderer to the gameObject, then set up the member variable references to the meshFilter and meshRenderer.

Now we will create a function to schedule the Job for drawing the chunk's mesh. This uses the same Job created in last weeks lecture.

```cs
public class JobWorldChunk
{
    // ... snipping unchanged code ... //

    public void ScheduleDraw()
    {
        Debug.Log("Starting draw: " + gameObject.transform.position);

        m_vertices = new NativeArray<Vector3>(24 * Data.chunkSize * Data.chunkSize * Data.chunkSize / 2, Allocator.TempJob);
        m_triangles = new NativeArray<int>(36 * Data.chunkSize * Data.chunkSize * Data.chunkSize / 2, Allocator.TempJob);
        m_uvs = new NativeArray<Vector2>(24 * Data.chunkSize * Data.chunkSize * Data.chunkSize / 2, Allocator.TempJob);
        m_vertexIndex = new NativeArray<int>(1, Allocator.TempJob);
        m_triangleIndex = new NativeArray<int>(1, Allocator.TempJob);

        m_chunkJob = new JobDefs.ChunkJob();
        m_chunkJob.chunkPos = gameObject.transform.position;
        m_chunkJob.vertices = m_vertices;
        m_chunkJob.triangles = m_triangles;
        m_chunkJob.uvs = m_uvs;
        m_chunkJob.vertexIndex = m_vertexIndex;
        m_chunkJob.triangleIndex = m_triangleIndex;

        m_handle = m_chunkJob.Schedule();
    }
}
```

We make a log to the console to help us with possible debugging... and satifaction. The most important part to note is the `m_handle = m_chunkJob.Schedule()` the `m_handle` is a memeber variable we created. This will let us call the `m_handle.Complete()` function in the chunks `CompleteDraw()` function. Remember that the `Complete()` function waits for the job to complete.

```cs
public class JobWorldChunk
{
    // ... snipping unchanged code ... //

    public void CompleteDraw()
    {
        Debug.Log("Completing draw: " + gameObject.transform.position);

        m_handle.Complete();

        m_mesh = new Mesh
        {
            vertices = m_vertices.Slice<Vector3>(0, m_chunkJob.vertexIndex[0]).ToArray(),
            triangles = m_triangles.Slice<int>(0, m_chunkJob.triangleIndex[0]).ToArray(),
            uv = m_uvs.Slice<Vector2>(0, m_chunkJob.vertexIndex[0]).ToArray()
        };

        m_mesh.RecalculateBounds();
        m_mesh.RecalculateNormals();

        m_meshFilter.mesh = m_mesh;

        m_vertices.Dispose();
        m_triangles.Dispose();
        m_uvs.Dispose();
        m_vertexIndex.Dispose();
        m_triangleIndex.Dispose();
    }   
}
```

We purposefully make 2 separate functions for Schedule and Complete. This allows us to Schedule many chunks to draw and then Complete them all at once. Once the job is completed we know that the mesh data arrays are ready and we set the `m_mesh`'s data. 

Now we will instantiate the chunk through the `JobWorld` class, which will be the chunk manager. Paste in this boiler plate code.

```cs
using UnityEngine;

public class JobWorld : MonoBehaviour
{
    public Material material;

    private void Start()
    {

    }
}
```

We can instantiate the chunk using the constructor and draw it in start.

```cs
    private void Start()
    {
        JobWorldChunk chunk = new JobWorldChunk(material, new Vector3(0, 0, 0));
        chunk.ScheduleDraw();
        chunk.CompleteDraw();
    }
```

Now make a new gameObject in the JobWorld.unity scene and add the `JobWorld` script as a component to the gameObject. Make sure to set the material property in the JobWorld component.

and... Yay! 

![job chunk](/Assets/optimized_noise_chunk.png)

If we use a timeline diagram to visualize what is happening on threads we can see that we have gained very little by multi-threading.

![threading timeline single](/Assets/threading_timeline_single.png)

Why is multithreading so important for video games? In a game that needs to run at 60 fps, we end up with only 16 ms to do run all of our code each frame! Multithreading, when used correctly, can increase the amount of work we can get done in a small amount of time. Read on to see how!

## World generation
Now we will be instantiating chunks in a grid much like the 1DRecycle code. Change the code to hold a 2D array of `JobWorldChunk`'s. We multiply the `x` and `z` by the `Data.chunkSize` to make sure the chunks are positioned apart by their size.

```cs
public class JobWorld : MonoBehaviour
{
    public Material material;
    public JobWorldChunk[,] chunks = new JobWorldChunk[Data.chunkNum, Data.chunkNum];

    private void Start()
    {
        for (int x = 0; x < Data.chunkNum; x++) 
        {
            for (int z = 0; z < Data.chunkNum; z++) 
            {
                Vector3 position = new Vector3(x * Data.chunkSize, 0, z * Data.chunkSize);
                chunks[x, z] = new JobWorldChunk(material, position);
            }
        }
    }
}
```

We will also need to add a new constant in DataDefs.cs that tells the number of chunks we will want in the x and z directions. We use this constant in the above code.

```cs
public static class DataDefs
{
    public const int chunkNum = 8;

    // ... snipping unchanged code ... //
}
```

In the same `for` loop as the instantiation we could schedule and complete the drawing. This would not be taking advantage of multithreading though.

```cs
    private void Start()
    {
        for (int x = 0; x < Data.chunkNum; x++) 
        {
            for (int z = 0; z < Data.chunkNum; z++) 
            {
                Vector3 position = new Vector3(x * Data.chunkSize, 0, z * Data.chunkSize);
                chunks[x, z] = new JobWorldChunk(material, position);
                chunks[x, z].ScheduleDraw();
                chunks[x, z].CompleteDraw();
            }
        }
    }
```

If we use a timeline diagram we can see that we aren't gaining anything (time wise) from using Jobs.

![threading timeline inefficient](/Assets/threading_timeline_inefficient.png)

But if we have a separate loop for scheduling and then another loop for completion we gain a lot of time!

```cs
    private void Start()
    {
        // ... snipping unchanged code ... //

        for (int x = 0; x < Data.chunkNum; x++) 
        {
            for (int z = 0; z < Data.chunkNum; z++) 
            {
                chunks[x, z].ScheduleDraw();
            }
        }

        for (int x = 0; x < Data.chunkNum; x++) 
        {
            for (int z = 0; z < Data.chunkNum; z++) 
            {
                chunks[x, z].CompleteDraw();
            }
        }
    }

```

![threading timeline efficient](/Assets/thrading_timeline_efficient.png)

And it turns out that Unity provides us with a live timeline viewer! To open it go to Window -> Analysis -> Profiler

![screenshot threading timeline](/Assets/screenshot_threading_timeline.png)

You will notice the record button at the top, this is for recording data for the profiler to use. The other highlighted thing is a dropdown, by default you will be in Hierarchy mode click on the dropdown to change it to Timeline mode. In timeline mode you should be able to see worker threads under "Jobs". But first you will have to hit play and record some data.

How does the JobSystem work, and what happens when we call the `Schedule()` function? 

When we call the Schedule function our Job (a struct) gets added to a large queue of all the Jobs that need to run. Worker threads then constantly grab a Job off of the queue (eventually our Job)  and run its `Execute()` function. Pretty simple.

Some job systems will stop running a Job's `execute` function if has to wait for Input from the OS. The worker will then start working on another Job, once it finishes it will go back and finish its previously paused Job.

# Recycling chunks
Add the recycling code from the cubes example. We move chunks over to the other side of the world as in the cubes example.

```cs
public class JobWorld : MonoBehaviour
{
    // ... snipping unchanged code ... //
    private int offset = Data.worldSize * Data.chunkSize / 2;

    // ... snipping unchanged code ... //

    private void RecycleChunks()
    {
        for (int x = 0; x < Data.chunkNum; x++)
        {
            for (int z = 0; z < Data.chunkNum; z++)
            {
                // x
                if (center.position.x + offset < chunks[x, z].gameObject.transform.position.x)
                {
                    chunks[x, z].gameObject.transform.position -= new Vector3(Data.chunkNum * Data.chunkSize, 0, 0);
                }
                if (center.position.x - offset > chunks[x, z].gameObject.transform.position.x)
                {
                    chunks[x, z].gameObject.transform.position += new Vector3(Data.chunkNum * Data.chunkSize, 0, 0);
                }

                // z
                if (center.position.z + offset < chunks[x, z].gameObject.transform.position.z)
                {
                    chunks[x, z].gameObject.transform.position -= new Vector3(0, 0, Data.chunkNum * Data.chunkSize);
                }
                if (center.position.z - offset > chunks[x, z].gameObject.transform.position.z)
                {
                    chunks[x, z].gameObject.transform.position += new Vector3(0, 0, Data.chunkNum * Data.chunkSize);
                }
            }
        }
    }
}
```

The offset in this case is half of the total world size, now you may think just `Data.worldSize / 2` will work but remember `Data.worldSize` is the number of chunks, not the actual world's size.

If you make a new gameObject and set it as the "center" property for the `JobWorld` script then hit play you should be able to have a working recycling chunk system! Move the center around to see the chunks follow!

![screenshot chunk recycling](/Assets/screenshot_chunk_recycling.png)

You will see the chunks being recycled byt their meshes will not be redrawn.

Move all the recycling code into its own function, and then call that function in `Update()`.

```cs
public class JobWorld : MonoBehaviour
{
    // ... snipping unchanged code ... //

    private void Update()
    {
        RecycleChunks();
    }

    private void RecycleChunks()
    {
        for (int x = 0; x < Data.chunkNum; x++)
        {
            for (int z = 0; z < Data.chunkNum; z++)
            {
                // x
                if (center.position.x + offset < chunks[x, z].gameObject.transform.position.x)
                {
                    chunks[x, z].gameObject.transform.position -= new Vector3(Data.chunkNum * Data.chunkSize, 0, 0);
                    chunks[x, z].needsDrawn = true;
                }
                if (center.position.x - offset > chunks[x, z].gameObject.transform.position.x)
                {
                    chunks[x, z].gameObject.transform.position += new Vector3(Data.chunkNum * Data.chunkSize, 0, 0);
                    chunks[x, z].needsDrawn = true;
                }

                // z
                if (center.position.z + offset < chunks[x, z].gameObject.transform.position.z)
                {
                    chunks[x, z].gameObject.transform.position -= new Vector3(0, 0, Data.chunkNum * Data.chunkSize);
                    chunks[x, z].needsDrawn = true;
                }
                if (center.position.z - offset > chunks[x, z].gameObject.transform.position.z)
                {
                    chunks[x, z].gameObject.transform.position += new Vector3(0, 0, Data.chunkNum * Data.chunkSize);
                    chunks[x, z].needsDrawn = true;
                }
            }
        }
    }

    // ... snipping unchanged code ... //
}
```

# Redrawing
To ensure that a chunk get's drawn after it is moved we can just redraw all the chunks each frame.

```cs
    private void Update()
    {
        RecycleChunks();

        for (int x = 0; x < Data.chunkNum; x++)
        {
            for (int z = 0; z < Data.chunkNum; z++)
            {
                chunks[x, z].ScheduleDraw();
            }
        }

        for (int x = 0; x < Data.chunkNum; x++)
        {
            for (int z = 0; z < Data.chunkNum; z++)
            {
                chunks[x, z].CompleteDraw();
            }
        }
    }
```

And this will actually work pretty well! (It ran at 20 fps for me). How is this possible? To redraw all the chunks each frame with decent frame rates? Multithreading. If you remember from the timeline diagrams we can get a lot of work done in a small amount of time by using multithreadng, in this case we are multi-threading through the C# JobSystem.

How could we go about only redrawing the chunks that have been recycled? We could add all the chunks that have been moved to a queue. A queue is like a list that lets us add items to the bottom and when getting items gets them from the top and removes them, the same way "waiting in line" works. This was what I have done in the past, but this is not as performant and gets complicated. 

Thankfully there is a simpler solution. We can add a variable to the chunk class called `needsDrawn`, that we set to true whenever we move a chunk. And then in the `ScheduleDraw()` and `CompleteDraw()` functions have an if statement that check's if `needsDrawn` is true.

```cs
public class JobWorldChunk
{
    // .. snip .. //

    public bool needsDrawn;

    // .. snip .. //

    public JobWorldChunk(Material m_material, Vector3 m_position)
    {
        m_mesh = new Mesh();
        needsDrawn = true; // this makes sure we draw the chunk at the beginning

        // .. snip .. //
    }

    public void ScheduleDraw()
    {
        if (needsDrawn == true)
        {
            // scheduling code goes here
        }
    }

    public void CompleteDraw()
    {
        if (needsDrawn == true)
        {
            // completion code goes here

            needsDrawn = false;
        }
    }
}
```

If `needsDrawn` was true we schedule a draw then complete a draw and then set `needsDrawn` to false so that we don't draw the mesh over and over.

Now in the `JobWorld` class in the `RecycleChunks()` function we can simply set `needsDrawn` to true if the chunk was recycled.

```cs
    private void RecycleChunks()
    {
        for (int x = 0; x < Data.chunkNum; x++)
        {
            for (int z = 0; z < Data.chunkNum; z++)
            {
                // x
                if (center.position.x + offset < chunks[x, z].gameObject.transform.position.x)
                {
                    chunks[x, z].gameObject.transform.position -= new Vector3(Data.chunkNum * Data.chunkSize, 0, 0);
                    chunks[x, z].needsDrawn = true;
                }
                if (center.position.x - offset > chunks[x, z].gameObject.transform.position.x)
                {
                    chunks[x, z].gameObject.transform.position += new Vector3(Data.chunkNum * Data.chunkSize, 0, 0);
                    chunks[x, z].needsDrawn = true;
                }

                // z
                if (center.position.z + offset < chunks[x, z].gameObject.transform.position.z)
                {
                    chunks[x, z].gameObject.transform.position -= new Vector3(0, 0, Data.chunkNum * Data.chunkSize);
                    chunks[x, z].needsDrawn = true;
                }
                if (center.position.z - offset > chunks[x, z].gameObject.transform.position.z)
                {
                    chunks[x, z].gameObject.transform.position += new Vector3(0, 0, Data.chunkNum * Data.chunkSize);
                    chunks[x, z].needsDrawn = true;
                }
            }
        }
    }
```

And if you hit play you should only have the chunks redrawing when they are recycled! Try setting the `chunkNum` in DataDefs.cs as high as you can for stress testing!

And thats week 2! See you next week! Make sure to check out the Tutorials folder!