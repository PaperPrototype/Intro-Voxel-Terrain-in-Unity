# Chunks
Most voxel terrain systems split up all the voxels into "chunks". You may find this familiar with how Minecraft is loaded in "Chunks". Make a new micro porject called "Chunk" in our Unity project

```
    Assets/
    |___DataDefs.cs
    |___White (Material)
    |___Quad/
    |___Voxel/
    |___Chunk/
    |   |___Chunk.cs (CSharp)
    |   |___Chunk.unity (Scene)
```

We will be mostly copying last week's voxel mirco project with some changes. Change the code in Chunk.cs to look like this.

```cs
using UnityEngine;
using Unity.Collections;

[RequireComponent(typeof(MeshFilter))]
[RequireComponent(typeof(MeshRenderer))]
public class Chunk : MonoBehaviour
{
    private Mesh m_mesh;
    private NativeArray<Vector3> m_vertices;
    private NativeArray<int> m_triangles;
    private NativeArray<Vector2> m_uvs;
    private int m_vertexIndex = 0;
    private int m_triangleIndex = 0;

    private void Start()
    {

    }
}
```

To make a chunk all we have to do is place voxel's on a 3D grid. To achieve this we can copy last weeks `DrawVoxel()` function with one minor change. We will pass in a grid positon so that we can place the voxel at whatever positon we want. This position is added to the voxel's vertices which are used to position the mesh.

```cs
    private void DrawVoxel(int x, int y, int z)
    {
        Vector3 pos = new Vector3(x, y, z);

        for (int side = 0; side < 6; side++)
        {
            m_vertices[m_vertexIndex + 0] = DataDefs.Vertices[DataDefs.BuildOrder[side, 0]] + pos;
            m_vertices[m_vertexIndex + 1] = DataDefs.Vertices[DataDefs.BuildOrder[side, 1]] + pos;
            m_vertices[m_vertexIndex + 2] = DataDefs.Vertices[DataDefs.BuildOrder[side, 2]] + pos;
            m_vertices[m_vertexIndex + 3] = DataDefs.Vertices[DataDefs.BuildOrder[side, 3]] + pos;

            // ... snipping unchanged code ...
        }
    }
```

Now we can use nested for loops to place voxel's in a 3D grid next to eachother. Before we do that we need to know the "size" of our chunk, how many voxel's to draw in the x, y, and z direction. We could add it manully but we will need to reference our chunk size all over our code, so add a line to DataDefs.cs to have a centralized place for this

```cs
    public const int chunkSize = 16;
```

We make it a `const` to make sure that the number doesn't change. Also array initalizations require that a `const` is used for telling their size. Arrays, once created, cannot have anything "added" to them. (A readonly wouldn't work since it could be modified by its enclosing class). 

Our vertex and triangle arrays will need to be larger, since we will be making a chunk and not a single voxel, and a chunks's mesh has more data. We can calculate the amount by multipling 24 and 36 (the size of vertex / uv and triangle data for a single voxel) by the number of voxels we will be building in our chunk. This leads to the following code for generationg a chunk.

```cs
    private void Start()
    {
        m_vertices = new NativeArray<Vector3>(24 * DataDefs.chunkSize * DataDefs.chunkSize * DataDefs.chunkSize, Allocator.Temp);
        m_triangles = new NativeArray<int>(36 * DataDefs.chunkSize * DataDefs.chunkSize * DataDefs.chunkSize, Allocator.Temp);
        m_uvs = new NativeArray<Vector2>(24 * DataDefs.chunkSize * DataDefs.chunkSize * DataDefs.chunkSize, Allocator.Temp);

        for (int x = 0; x < DataDefs.chunkSize; x++)
        {
            for (int y = 0; y < DataDefs.chunkSize; y++)
            {
                for (int z = 0; z < DataDefs.chunkSize; z++)
                {
                    DrawVoxel(x, y, z);
                }
            }
        }

        // ... snipping irrelevant code ... //
    }
```

Tada! Now the rest of the code is exactly the same as before! I'll let you copy paste that from Voxel.cs. 

Make a new empty gameObject in the Chunk scene (Chunk.unity) and add Chunk.cs to it as a component. Also don't forget to set the material. If you click play you'll see a chunk drawn!

# Optimized Chunk
If you look at the inside of the current chunk you will see something like this

![unoptimized chunk mesh](/Assets/unoptimized_chunk_mesh.png)

To make it so voxel's don't draw their side if that side has a neighboring voxel, we need an algorithm to calculate where we want and don't want voxel's. For terrains the most popular option is to use Noise. Perlin Noise, Complex Noise, Value Noise... Noise is a way of getting smoothed random numbers. Most noise algorithms take in an offset in either 2D or 3D and then return a number between -1 to 1 (sometimes 0 to +1). If you saved the values of a noise function on a texture fading between black (-1) and white (1) it might look like this.

![TODO textured greyscale perlin noise image]()

We could initialize a 3D grid array of data storing bools (true = solid, false = not solid) using a 3D version of this noise at the beginning of our chunk draw and then reference that as grid of "voxel" when building voxels to check if they have neighbors. This saves us on processing power at the trade off of using memory. We would have to do this if we wanted editable terrain, so wecould save the current chunk data as a 3D array grid.

There is a noise library in the Resources of this repositiory called FastNoiseLite.zip. It is taken from [here](https://github.com/Auburn/FastNoiseLite/tree/master/CSharp) Download our repo and open the prepared zip and drag the folder "FastNoiseLite" into your project so that your project looks like this.

```
    Assets/
    |___FastNoiseLite/
    |   |___FastNoiseLite.cs
    |   |___LICENSE.md
    |___DataDefs.cs
    |___White (Material)
    |___Quad/
    |___Voxel/
    |___Chunk/
```

We will make a function that takes in a voxel's position and returns `true` or `false` using the FastNoiseLite class. First we need to add a member variable to the Chunk class to hold an instance of a FastNoiseLite.

```cs
    private FastNoiseLite m_noise;
```

Then we need to initialize the FastNoiseLite instance in `Start()` and set the noise type we want

```cs
    private void Start()
    {
        m_noise = new FastNoiseLite();
        m_noise.SetNoiseType(FastNoiseLite.NoiseType.OpenSimplex2);

        // ... snipping irrelevant code... //
    }
```

Now we can make our function for checking if a voxel should exist or not exist

```cs
    private bool IsSolid(int x, int y, int z)
    {
        float height = (m_noise.GetNoise(x, z) + 1) / 2 * DataDefs.chunkSize;

        if (y <= height)
        {
            return true;
        }
        else
        {
            return false;
        }
    }
```

The `FastNoiseLite`'s `GetNoise()` function gives us values between -1 and 1 depending on what position we use to get the noise. We add 1 to shift us up to get values between 0 and 2. Then divide it by 2 to get values between 0 and 1.

We multiply the noise value by a height value so the terrain doesn't stay within the height of 0 and 1. We chose to multiply by our chunk size so that the highest possible height is the same as the chunks height. 

Then to check if a voxel should exist check if the y position is below (less) than the noise value. If it is then there should be voxel's there.

Now when drawing voxel's in the for loop we can check if we should draw it or not.

```cs
        for (int x = 0; x < DataDefs.chunkSize; x++)
        {
            for (int y = 0; y < DataDefs.chunkSize; y++)
            {
                for (int z = 0; z < DataDefs.chunkSize; z++)
                {
                    if (IsSolid(x, y, z))
                    {
                        DrawVoxel(x, y, z);
                    }
                }
            }
        }
```

And we have terrain!

![first noise chunk](/Assets/first_noise_chunk.png)

But we still aren't checking if a voxel has a neighbor! 

To check if a voxel has a neighbor we will make a lookup table of offset grid positions. We will then check each voxel's neighbor using this. Add the following to DataDefs.cs

```cs
using Unity.Mathematics;

public static class DataDefs
{
    // ... snipping unchanged code ... //

    public static readonly int3[] NeighborOffset = new int3[6]
    {
        new int3(1, 0, 0),  // right
        new int3(-1, 0, 0), // left
        new int3(0, 1, 0),  // up
        new int3(0, -1, 0), // down
        new int3(0, 0, 1),  // front
        new int3(0, 0, -1), // back
    };
}
```

Now change the `DrawVoxel()` function to this

```cs
    private void DrawVoxel(Vector3 voxelPos)
    {
        for (int side = 0; side < 6; side++)
        {
            if (!IsSolid(x + DataDefs.NeighborOffset[side].x, y + DataDefs.NeighborOffset[side].y, z + DataDefs.NeighborOffset[side].z))
            {
                // ... snipping irrelevant code ... //
            }
        }
    }
```

The lookup table uses the `side` variable index to look up the offset for the side of the voxel we are building, to, check if that side of the voxel should be built. We made sure to put the offsets in the same order that we build the sides.

And now you should see this! Yay!

![optimized noise chunk](/Assets/optimized_noise_chunk.png)

One thing to note is that since we are not usually filling our vertex and triangle arrays all the way we can use the `m_vertexIndex` to find out how much of the array we are actually filling, since it stores the current number of vertices, and make a "Slice" of the portion of the array that we are using.

Making a "slice" of an array can be visualized with the following diagram.

![slice of array](Assets/slice_of_array.png)

```cs
    m_mesh = new Mesh
    {
        vertices = m_vertices.Slice<Vector3>(0, m_vertexIndex).ToArray(),
        triangles = m_triangles.Slice<int>(0, m_triangleIndex).ToArray()
    };
```

The last thing to note is that since we are checking for neighboring voxels, we will never use more than half of the `m_vertices` array. Imagine a case where the most possible number of voxels are drawn (using neighbor checking), this would be perfectly alternating voxels in a checkered pattern.

![2D voxel terrain checkered](/Assets/2D_voxel_terrain_checkered.png)

So, we can divide its allocation size by 2.

```cs
    private void Start()
    {
        m_vertices = new NativeArray<Vector3>(24 * DataDefs.chunkSize * DataDefs.chunkSize * DataDefs.chunkSize / 2, Allocator.Temp);
        m_triangles = new NativeArray<int>(36 * DataDefs.chunkSize * DataDefs.chunkSize * DataDefs.chunkSize / 2, Allocator.Temp);
    
        // ... snipping irrelevant code ... //
    }
```

# Jobified Chunk
Make a new micro project called JobChunk so that our project looks like this

```
    Assets/
    |___FastNoiseLite/
    |___DataDefs.cs
    |___White (Material)
    |___Quad/
    |___Voxel/
    |___Chunk/
    |___JobChunk/
    |   |___JobChunk.unity
    |   |___JobChunk.cs
    |___JobDefs.cs
```

Now open up JobChunk.cs and add this boiler plate code. It's different from our previous code

```cs
using UnityEngine;
using Unity.Jobs;
using Unity.Collections;

[RequireComponent(typeof(MeshFilter))]
[RequireComponent(typeof(MeshRenderer))]
public class JobChunk : MonoBehaviour
{
    private NativeArray<Vector3> m_vertices;
    private NativeArray<int> m_triangles;
    private NativeArray<Vector2> m_uvs;
    private NativeArray<int> m_vertexIndex;
    private NativeArray<int> m_triangleIndex;

    private void Start()
    {

    }
}
```

You might notice that the all our member variables have been changed to be stored in a `NativeArray`. Remember how C# makes copies of everything? NativeArrays allow us to access the same memory / instance from different places, rather than making copies of it every time. So to access any data outside a Job we need to use a NativeArray. 

Now open up JobDefs.cs and change the code to make the class static and add a new Job (which is just a struct).

```cs
using UnityEngine;
using Unity.Jobs;
using Unity.Collections;

public static class JobDefs
{
    public struct ChunkJob : IJob
    {
        public Vector3 chunkPos;
        public NativeArray<Vector3> vertices;
        public NativeArray<int> triangles;
        public NativeArray<Vector2> uvs;
        public NativeArray<int> vertexIndex;
        public NativeArray<int> triangleIndex;
    }
}
```

Jobs are structs that hold the data they will need and they inherit from the `IJob` interface. An interface is a bunch functions (or in our case just one function) that a class or struct that inherits from the interface has to implement. In this case `IJob` makes us implement the `Execute()` function, so that the JobSYstem can have a function to call when it should "run" or execute the job. We can treat the `Execute()` function it similar to the way we've treated the `Start()` function in the past.

```cs
public static class JobDefs
{
    public struct ChunkJob : IJob
    {
        // ... snipping irrelevant code ... //

        public void Execute()
        {
            FastNoiseLite noise = new FastNoiseLite();
            noise.SetNoiseType(FastNoiseLite.NoiseType.OpenSimplex2);

            vertexIndex[0] = 0;
            triangleIndex[0] = 0;

            for (int x = 0; x < DataDefs.chunkSize; x++)
            {
                for (int y = 0; y < DataDefs.chunkSize; y++)
                {
                    for (int z = 0; z < DataDefs.chunkSize; z++)
                    {
                        if (IsSolid(noise, x, y, z))
                        {
                            DrawVoxel(noise, x, y, z);
                        }
                    }
                }
            }
        }
    }
}
```

Because the `vertexIndex` is stored in a `NativeArray` we have to use `[0]` to access the first element in the array which is the `vertexIndex`.

One note is that Jobs are not allowed to hold "reference types" (classes are always a reference to the same memory and not a copy, much like a NativeArray), so the FastNoiseLite reference cannot live as a member variable of the Job. Instead we will have to declare and initialize it in the `Execute()` function of the Job.

You might notice that the `DrawVoxel()` and `IsSolid()` functions have changed. Paste them in from before, and change them so that they take in the `FastNoiseLite` instance as a parameter. This is because we cannot hol a reference memebr variable, and so we have to keep the FastNoiseLiteInstance within our functions.

```cs
    public struct ChunkJob : IJob
    {
        // ... snipping irrelevant code ... //

        private void DrawVoxel(FastNoiseLite noise, int x, int y, int z)
        {
            Vector3 pos = new Vector3(x, y, z);

            for (int face = 0; face < 6; face++)
            {
                if (!IsSolid(noise, DataDefs.NeighborOffset[face].x + x, DataDefs.NeighborOffset[face].y + y, DataDefs.NeighborOffset[face].z + z))
                {
                    vertices[vertexIndex[0] + 0] = pos + DataDefs.Vertices[DataDefs.BuildOrder[face, 0]];
                    vertices[vertexIndex[0] + 1] = pos + DataDefs.Vertices[DataDefs.BuildOrder[face, 1]];
                    vertices[vertexIndex[0] + 2] = pos + DataDefs.Vertices[DataDefs.BuildOrder[face, 2]];
                    vertices[vertexIndex[0] + 3] = pos + DataDefs.Vertices[DataDefs.BuildOrder[face, 3]];

                    // get the correct triangle index
                    triangles[triangleIndex[0] + 0] = vertexIndex[0] + 0;
                    triangles[triangleIndex[0] + 1] = vertexIndex[0] + 1;
                    triangles[triangleIndex[0] + 2] = vertexIndex[0] + 2;
                    triangles[triangleIndex[0] + 3] = vertexIndex[0] + 2;
                    triangles[triangleIndex[0] + 4] = vertexIndex[0] + 1;
                    triangles[triangleIndex[0] + 5] = vertexIndex[0] + 3;

                    uvs[vertexIndex[0] + 0] = new Vector2(0, 0);
                    uvs[vertexIndex[0] + 1] = new Vector2(0, 1);
                    uvs[vertexIndex[0] + 2] = new Vector2(1, 0);
                    uvs[vertexIndex[0] + 3] = new Vector2(1, 1);

                    // increment by 4 because we only added 4 vertices
                    vertexIndex[0] += 4;

                    // increment by 6 because we only added 6 ints (6 / 3 = 2 triangles)
                    triangleIndex[0] += 6;
                }
            }
        }
    }
```

We also make sure to add the current chunks position to the `GetNoise` function...

```cs
    public struct ChunkJob : IJob
    {
        // ... snipping irrelevant code ... //

        private bool IsSolid(FastNoiseLite noise, int x, int y, int z)
        {
            float height = (noise.GetNoise(x + chunkPos.x, z + chunkPos.z) + 1) / 2 * DataDefs.chunkSize;

            if (y <= height)
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

otherwise you will get this horrifying monstrocity

![repetitive chunks same noise](/Assets/repetitive_chunks_same_noise.png)

In the start of ChunkJob allocate our mesh's data arrays and then setup the `ChunkJob` variables.

```cs
    private void Start() {

        m_vertices = new NativeArray<Vector3>(24 * Data.chunkSize * Data.chunkSize * Data.chunkSize / 2, Allocator.TempJob);
        m_triangles = new NativeArray<int>(36 * Data.chunkSize * Data.chunkSize * Data.chunkSize / 2, Allocator.TempJob);
        m_uvs = new NativeArray<Vector2>(24 * Data.chunkSize * Data.chunkSize * Data.chunkSize / 2, Allocator.TempJob);
        m_vertexIndex = new NativeArray<int>(1, Allocator.TempJob);
        m_triangleIndex = new NativeArray<int>(1, Allocator.TempJob);

        JobDefs.ChunkJob job = new JobDefs.ChunkJob();
        job.chunkPos = gameObject.transform.position;
        job.vertices = m_vertices;
        job.triangles = m_triangles;
        job.uvs = m_uvs;
        job.vertexIndex = m_vertexIndex;
        job.triangleIndex = m_triangleIndex;

        JobHandle handle = job.Schedule();

        // ... to be continued
    }
```

When we schedule the job it returns a `JobHandle`. This lets us have a "handle" to hold on to our Job. Since the Job will be running on a differrent thread, we can't know when it will finish running, so we just have to wait until it completes, and that is exactly what the `Complete()` function does. It makes the code wait until 

```cs
    private void Start()
    {
        // ... snipping irrelevant code ... //

        JobHandle handle = job.Schedule();
        handle.Complete();

        // ... to be continued
    }
```

Now the rest of the code is much the same as past times

```cs
    private void Start()
    {
        // ... snipping irrelevant code ... //

        Mesh m_mesh = new Mesh
        {
            vertices = m_vertices.Slice<Vector3>(0, job.vertexIndex[0]).ToArray(),
            triangles = m_triangles.Slice<int>(0, job.triangleIndex[0]).ToArray(),
            uv = m_uvs.Slice<Vector2>(0, job.vertexIndex[0]).ToArray()
        };

        m_mesh.RecalculateBounds();
        m_mesh.RecalculateNormals();

        MeshFilter filter = gameObject.GetComponent<MeshFilter>();
        filter.mesh = m_mesh;

        // free memory
        m_vertices.Dispose();
        m_triangles.Dispose();
        m_uvs.Dispose();
        m_vertexIndex.Dispose();
        m_triangleIndex.Dispose();
    }
```

If you run this you should see a chunk as before! YESSSSS!!!!!!!!!! Now even though we haven't actually benefited from using the JobSystem in this example, it lays the groundwork for the next weeks lecture, where we will be drawing an entire terrain with chunks using the JobSystem!