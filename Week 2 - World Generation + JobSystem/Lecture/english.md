# JobWorld
To make a large open world of voxels we can divide up the world into chunks. This makes it so that we can allocate the world in pieces rather than having to manage thousands of individual voxels. (Although if using an Octree data structure this might not be a bad idea (See notes for more info). We will not be using Octrees though). 

But now we have to manage the chunks. To have a player that can walk around the a terrain "forever" in any direction we have to keep making chunks appear in front of the player. Most systems delete old chunks and then instantiate new ones. But allocations and deletions are slow. We can make this significantly faster if we "recycle" chunks. 

A system like this is usually called a "pool", where we have a pool of objects, that when we "delete" we really jsut add it to the pool, and when we instantiate we just get one from the pool. If the pool is empty we just allocate a new one.

But pool's are complicated to write. Instead we will do something super simple yet fast. We will move old chunks into the position where new chunks are needed and then rebuild their meshes.

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
    |___Data.cs
    |___White (Material)
    |___Quad/
    |___Voxel/
    |___Chunk/
    |___JobChunk/
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

the function `GameObject.CreatePrimitive(Primitive.Cube)` is a shortcut for instantiating (creating) a built-in cube (Called a primitive). (This makes it so we don't have to manually add a renderer component, and mesh filter, and material, and mesh).

We will round the center position (the player's position).

```cs
public class Recycle1D : MonoBehaviour
{
    // ... snipping unchanged code ... //

    private Vector3 GetRoundedPos()
    {
        return new Vector3(Mathf.Round(center.position.x), 0, 0);
    }
}
```

and now we can write the code to move the cubes.

```cs
    void Update()
    {
        for (int i = 0; i < numCubes; i++)
        {
            if (GetRoundedPos().x + offset < cubes[i].transform.position.x)
            {
                cubes[i].transform.position -= new Vector3(numCubes, 0, 0);
            }
            if (GetRoundedPos().x - offset > cubes[i].transform.position.x)
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
    |___Data.cs
    |___White (Material)
    |___Quad/
    |___Voxel/
    |___Chunk/
    |___JobChunk/
    |___Recycling/
    |   |___1D/
    |   |___2D/
            |___Recycle2D.unity
            |___2D.cs
```

The code will be the same except that we will use a 2D array to hold our cubes, and we will instantiate them in a grid, much like we will for the terrain later. We use a 2D array so that the for loop's are easier to write, and we don't have to convert from 1D array indexing to 2D indexing, which is not that hard but will make this example a bit involved (complex). (There is a tutorial for this in the tutorials of this lecture)

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

The round function has changed slightly to include z.

```cs
    private Vector3 GetRoundedPos()
    {
        return new Vector3(Mathf.Round(center.position.x), 0, Mathf.Round(center.position.z));
    }
```

Instantiating the cubes in a grid changes as well. We also change the name of each cube according to its index and position. We provide the method for indexing a 1D array using 2D coordinates `(x * numCubes) + z`. (There is a tutorial explaining how to index a 1D array as a 2D or 3D array in this week's tutorials).

```cs
    private void Start()
    {
        int i = 0;
        for (int x = (int)GetRoundedPos().x; x < numCubes + (int)GetRoundedPos().x; x++)
        {
            for (int z = (int)GetRoundedPos().z; z < numCubes + (int)GetRoundedPos().z; z++)
            {
                cubes[x, z] = GameObject.CreatePrimitive(PrimitiveType.Cube);
                cubes[x, z].GetComponent<Transform>().position = new Vector3(x, 0, z);
                cubes[x, z].gameObject.name = "(" + x + " " + z + ") " + "2D index conversion: " + ((x * numCubes) + z) + " actual index: " + i;
                i++;
            }
        }
    }
```

We also center the the creating of the cubes by using the GetRoundedPos() in the for loop declaration.

The recycling code changes with an additional for loop and, includes code checking for the z coordinate, which is the exactly the same code as the x coordinate.

```cs
    private void Update()
    {
        for (int x = 0; x < numCubes; x++)
        {
            for (int z = 0; z < numCubes; z++)
            {
                // x
                if (GetRoundedPos().x + offset < cubes[x, z].transform.position.x)
                {
                    cubes[x, z].transform.position += new Vector3(-numCubes, 0, 0);
                }
                if (GetRoundedPos().x - offset > cubes[x, z].transform.position.x)
                {
                    cubes[x, z].transform.position += new Vector3(numCubes, 0, 0);
                }

                // z
                if (GetRoundedPos().z + offset < cubes[x, z].transform.position.z)
                {
                    cubes[x, z].transform.position += new Vector3(0, 0, -numCubes);
                }
                if (GetRoundedPos().z - offset > cubes[x, z].transform.position.z)
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
    |___Data.cs
    |___White (Material)
    |___Quad/
    |___Voxel/
    |___Chunk/
    |___JobChunk/
    |___Recycling/
    |___JobWorld/
        |___JobWorld.unity
        |___JobWorldChunk.cs
        |___JobWorld.cs
```

We will manually do the instantiation of the chunk gameObject through the `JobWorldChunk` class. Paste this boiler plate code into `JobWorldChunk`.

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
    private ChunkJob m_chunkJob;
}
```

Don't worry you should be able to understand it. The `gameObject` variable will hold a reference to the actual gameObject, the `m_meshFilter` and `m_meshRenderer` hold references to the gameObjects Meshfilter and MeshRenderer. The `m_handle` is our chunks JobHandle and the `m_chunkJob` is the chunks Job.

Also notice that the `JobWorldChunk` does not inherit from `MonoBehaviour` since it will not be a component of the chunk gameObject. Inheritince is denoted by the `:` followed by a class. The example `OurClass : AnotherClass` can be read as, `OurClass` inherits from `AnotherClass`. 

We will create an initialiozer function (? a little help here, I forgot what the correct word was for "initalizer functions", an Issue or pull request will suffice). These functions are special. They get called when we put the `new` keyword in front of a class or struct. For example if we say 

```
JobWorldChunk chunk = new JobWorldChunk()
``` 

the `JobWorldChunk()` with the parenthesis is an intializer function.

Make an initializer function for `JobWorldChunk` with all of our initalization code. (Notice how the type the function returns is not there, since, the type it returns is itself!)

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

The initializer function takes in a Material and a position (Vector3) to place the chunk at. Then we instantiate a gameObject and add a meshFilter and meshRenderer to it, then set up the member variable references to the meshFilter and meshRenderer, and set the meshFilters mesh reference to our `m_mesh` member variable.

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

        m_chunkJob = new ChunkJob();
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

Once the job is completed we know that the mesh data arrays are ready and we set the `m_mesh`'s data. Now we will instantiate the chunk through the `JobWorld` class. Paste in this boiler plate code.

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

and we can instantiate the chunk and draw it in start. You will notice the initializer function is being used.

```cs
    private void Start()
    {
        JobWorldChunk chunk = new JobWorldChunk(material, new Vector3(0, 0, 0));
        chunk.ScheduleDraw();
        chunk.CompleteDraw();
    }
```

now make a new gameObject in JobWorld.unity and add `JobWorld` as a component to the gameObject. Make sure to set the material property in the JobWorld component.

and... Yay! 

![job chunk](/Assets/optimized_noise_chunk.png)

If we use a timeline to see what is happening on threads we can see that time wise nothing was gained.

![threading timeline single](/Assets/threading_timeline_single.png)

Why is multithreading so important for video games? In a game that needs to run at 60 fps, we end up with only 16 ms for each frame! Multithreading when used correctly can increase the amount of work we can get done in a small amount of time. Read on to see how!

## World generation
Now we will be instantiating chunks in a grid much like the 1DRecycle code. Change the code to hold a 2D array of `JobWorldChunk`'s. We multiply the x and z values for the position by the chunkSize to make sure that chunks are positioned apart from eachother by their size, otherwise they would be overlapping eachother (only being spaced apart by 1 voxel).

```cs
public class JobWorld : MonoBehaviour
{
    public Material material;
    public JobWorldChunk[,] chunks = new JobWorldChunk[Data.worldSize, Data.worldSize];

    private void Start()
    {
        for (int x = 0; x < Data.worldSize; x++) 
        {
            for (int z = 0; z < Data.worldSize; z++) 
            {
                Vector3 position = new Vector3(x * Data.chunkSize, 0, z * Data.chunkSize);
                chunks[x, z] = new JobWorldChunk(material, position);
            }
        }
    }
}
```

We will also need to add a new constant in Data.cs that tells the number of chunks we will want in the x and z directions.

```cs
public class Data
{
    public const int worldSize = 8;

    // ... snipping unchanged code ... //
}
```

In the same `for` loop as the instantiation we could schedule and complete the drawing. This would not be taking advantage of multithreading though.

```cs
    private void Start()
    {
        for (int x = 0; x < Data.worldSize; x++) 
        {
            for (int z = 0; z < Data.worldSize; z++) 
            {
                Vector3 position = new Vector3(x * Data.chunkSize, 0, z * Data.chunkSize);
                chunks[x, z] = new JobWorldChunk(material, position);
                chunks[x, z].ScheduleDraw();
                chunks[x, z].CompleteDraw();
            }
        }
    }
```

If we use a timeline diagram we can see that we aren't gaining anything time wise from using the jobs.

![threading timeline inefficient](/Assets/threading_timeline_inefficient.png)

But if we have a separate loop for scheduling and then another loop for completion we gain a lot of time!

```cs
    private void Start()
    {
        // ... snipping unchanged code ... //

        for (int x = 0; x < Data.worldSize; x++) 
        {
            for (int z = 0; z < Data.worldSize; z++) 
            {
                chunks[x, z].ScheduleDraw();
            }
        }

        for (int x = 0; x < Data.worldSize; x++) 
        {
            for (int z = 0; z < Data.worldSize; z++) 
            {
                chunks[x, z].CompleteDraw();
            }
        }
    }

```

![threading timeline efficient](/Assets/thrading_timeline_efficient.png)

And it turns out that Unity provides us with a live timeline viewer! To open it go to Window -> Analysis -> Profiler

![screenshot threading timeline](/Assets/screenshot_threading_timeline.png)

You will notice the record button at the top, this is for recording data for the profiler to use, the thing is a dropdown, by default you will be in Hierarchy mode click on the dropdown to change it to Timeline mode. In timeline mode you shoul dbe able to see worker threads under "Jobs".

# Recycling chunks
Before we can make the recycling systrem we used in the cubes example we have to address the `GetRoundedPos()` function. Before we were rounding to the 1's place (since the cubes were only 1 x 1 x 1), but now we need to round to whatever `Data.chunkSize` is, so that `GetRoundedPos()` will return multiples of 16 (assuming `Data.chunkSize` was 16). 

The `Mathf.Round()` function will only round to the nearest 1's place. We can reduce the players chunk position to become multiples of 1 by simply dividing the players position by 16 (therefore reducing the players chunk position to multiples of 1) rounding it, and then multiplying again by 16 to get back to a multiple of 16.

```cs
public class JobWorld : MonoBehaviour
{
    // ... snipping unchanged code ... //
    public Transform center;

    // ... snipping unchanged code ... //

    private Vector3 GetRoundedPos()
    {
        return new Vector3(Mathf.Round(center.position.x / Data.chunkSize), 0, Mathf.Round(center.position.z / Data.chunkSize)) * Data.chunkSize;
    }
}
```

Now add the recycling code from the cubes example adjusted to work with chunk sizes (Don't worry about redrawing the chunks just yet).

```cs
public class JobWorld : MonoBehaviour
{
    // ... snipping unchanged code ... //
    private int offset = Data.worldSize * Data.chunkSize / 2;

    // ... snipping unchanged code ... //

    private void Update()
    {
        for (int x = 0; x < Data.worldSize; x++)
        {
            for (int z = 0; z < Data.worldSize; z++)
            {
                // x
                if (GetRoundedPos().x + offset < chunks[x, z].gameObject.transform.position.x)
                {
                    chunks[x, z].gameObject.transform.position -= new Vector3(Data.worldSize * Data.chunkSize, 0, 0);
                }
                if (GetRoundedPos().x - offset > chunks[x, z].gameObject.transform.position.x)
                {
                    chunks[x, z].gameObject.transform.position += new Vector3(Data.worldSize * Data.chunkSize, 0, 0);
                }

                // z
                if (GetRoundedPos().z + offset < chunks[x, z].gameObject.transform.position.z)
                {
                    chunks[x, z].gameObject.transform.position -= new Vector3(0, 0, Data.worldSize * Data.chunkSize);
                }
                if (GetRoundedPos().z - offset > chunks[x, z].gameObject.transform.position.z)
                {
                    chunks[x, z].gameObject.transform.position += new Vector3(0, 0, Data.worldSize * Data.chunkSize);
                }
            }
        }
    }
}
```

The offset is half of the world size, now you may think just `Data.worldSize / 2` will work but remember `Data.worldSize` is the number of chunks, not the actual world's size. Change the name to `Data.chunkNum` for clarity from now on.

And if you make a new gameObject and set it as the "center" for the JobWorld script then hit play you should be able to have a working recycling chunk system! Move the centetr around to see the chunks follow!

![screenshot chunk recycling](/Assets/screenshot_chunk_recycling.png)

You will not see choppy mismatched meshes because we aren't redrawing them.