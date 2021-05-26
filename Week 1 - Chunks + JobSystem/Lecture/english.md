# Chunks
Most voxel terrain systems split up all the voxels into "chunks". Make a new micro porject called "Chunk" in our Unity project

```
    Assets/
    |___Data.cs
    |___White (Material)
    |___Quad/
    |___Voxel/
    |___Chunk/
    |   |___Chunk.cs (CSharp)
    |   |___Chunk.unity (Scene)
```

We will be mostly copying last week's voxel mirco project with some changes. Change the code in Chunk.cs to look like this.

```
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

To make a chunk all we have to do is place voxel's on a 3D grid. To achieve this we can copy last weeks `DrawVoxel()` function with one minor change. We will pass in a position so that we can build the voxel at whatever position we want.

```
    private void DrawVoxel(Vector3 voxelPos)
    {
        for (int side = 0; side < 6; side++)
        {
            m_vertices[m_vertexIndex + 0] = Data.Vertices[Data.BuildOrder[side, 0]] + voxelPos;
            m_vertices[m_vertexIndex + 1] = Data.Vertices[Data.BuildOrder[side, 1]] + voxelPos;
            m_vertices[m_vertexIndex + 2] = Data.Vertices[Data.BuildOrder[side, 2]] + voxelPos;
            m_vertices[m_vertexIndex + 3] = Data.Vertices[Data.BuildOrder[side, 3]] + voxelPos;

            // ... snipping unchanged code ... //
        }
    }
```

Now we can use nested for loops to place voxel's in a 3D grid next to eachother. Before we do that we need to know the "size" of our chunk, how many voxel's to draw in the x, y, and z direction. We could add it manully but we will need to reference our chunk size all over our code, so add a line to Data.cs to have a centralized place for this

```
    public const int chunkSize = 8;
```

We make it a `const` to make sure that the number doesn't change. Also array initalizations require that a `const` is used for telling their size, arrays aren't dynamic in size. (A readonly wouldn't work since it could be modified by an enclosing class). 

Our vertex and triangle arrays will need to be larger, since we will be making a chunk and not a single voxel. We can calculate the amount by multipling 24 and 36 (the size of vertex / uv and triangle data for a single voxel) by the number of voxel we will be building in our chunk. This leads to the following code

```
    private void Start()
    {
        m_vertices = new NativeArray<Vector3>(24 * Data.chunkSize * Data.chunkSize * Data.chunkSize, Allocator.Temp);
        m_triangles = new NativeArray<int>(36 * Data.chunkSize * Data.chunkSize * Data.chunkSize, Allocator.Temp);
        m_uvs = new NativeArray<Vector2>(24 * Data.chunkSize * Data.chunkSize * Data.chunkSize, Allocator.Temp);

        for (int x = 0; x < Data.chunkSize; x++)
        {
            for (int y = 0; y < Data.chunkSize; y++)
            {
                for (int z = 0; z < Data.chunkSize; z++)
                {
                    Vector3 pos = new Vector3(x, y, z);
                    DrawVoxel(pos);
                }
            }
        }

        // ... snipping irrelevant code ... //
    }
```

Now the rest of the code in start is exactly the same as before! I'll let you copy paste that from Voxel.cs. 

Now make a new empty gameObject in the Chunk scene (Chunk.unity) and add Chunk.cs to it as a component. Also don't foregt to set the material. If you click play you'll see a chunk drawn!

# Optimized Chunk
If you look at the inside of our current chunk you might see something like this

![unoptimized chunk mesh](/Assets/unoptimized_chunk_mesh.png)

To make it so voxel's don't draw their side if that side has a neighboring voxel, we need an algorithm to calculate where we want and don't want voxel's. For terrains the most popular option is to use Noise. Perlin Noise, Complex Noise, Value Noise... Noise is a way of getting smoothed random numbers. Most noise algorithms take in an offset in either 2D or 3D and then return a number between 0 and 1. If you saved the values of a noise function on a texture fading between black (0) and white (1) it might look like this.

![TODO textured greyscale perlin noise image]()

We could initialize a 3D grid array of data storing bools (true = solid, false = not solid) using a 3D version of this noise at the beginning of our chunk draw and then reference that as grid of "voxel" when building voxels to check if they have neighbors. This saves us on processing power at the trade off of using memory. We would have to do this if we wanted editable terrain, so wecould save the current chunk data as a 3D array grid.

There is a noise library in the Resources of this repositiory called FastNoiseLite.zip. It is taken from [here](https://github.com/Auburn/FastNoiseLite/tree/master/CSharp) Download our repo and open the prepared zip and drag the folder "FastNoiseLite" into your project so that your project looks like this.

```
    Assets/
    |___FastNoiseLite/
    |   |___FastNoiseLite.cs
    |   |___LICENSE.md
    |___Data.cs
    |___White (Material)
    |___Quad/
    |___Voxel/
    |___Chunk/
```

We will make a function that takes in a voxel's position and returns `true` or `false` using the FastNoiseLite class. First we need to add a member variable to the Chunk class to hold an instance of FastNoiseLite (kind of how we hold an instance to our vertex arrays)

```
    private FastNoiseLite m_noise;
```

Then we need to initialize the FastNoiseLite instance in `Start()` and set the noise type we want

```
    private void Start()
    {
        m_noise = new FastNoiseLite();
        m_noise.SetNoiseType(FastNoiseLite.NoiseType.OpenSimplex2);

        // ... snipping irrelevant code... //
    }
```

Now we can make our function

```
    private bool IsSolid(Vector3 voxelPos)
    {
        float height = m_noise.GetNoise(voxelPos.x, voxelPos.z) * Data.chunkSize;

        if (voxelPos.y <= height)
        {
            return true;
        }
        else
        {
            return false;
        }
    }
```

We need to multiply the noise value by some height value so that our terrain has hills higher than 1 voxel. We chose to multiply by our chunk size so that the highest possible height is the same as the chunk height, and so that if we change our chunk size the max height changes to. We check if a voxel's y position is less than the noise value, if it is then return true.

Now when drawing voxel's in our for loop we can check if we should draw the voxel or not.

```
        for (int x = 0; x < Data.chunkSize; x++)
        {
            for (int y = 0; y < Data.chunkSize; y++)
            {
                for (int z = 0; z < Data.chunkSize; z++)
                {
                    Vector3 pos = new Vector3(x, y, z);
                    if (IsSolid(pos))
                    {
                        DrawVoxel(pos);
                    }
                }
            }
        }
```

And we have terrain! (Yours will have uv's this is an old picture (maybe take an updated screenshot and add it to the repo?))

![first noise chunk](/Assets/first_noise_chunk.png)

But we still aren't checking if a voxel has a neighbor! 

To do neigbor check's for each voxel we will make a lookup table of offset positions that we will use to check for an offset voxel from the current voxel position. Add the following to Data.cs

```
    public static readonly Vector3[] NeighborOffset = new Vector3[6]
    {
        new Vector3(1.0f, 0.0f, 0.0f),  // right
        new Vector3(-1.0f, 0.0f, 0.0f), // left
        new Vector3(0.0f, 1.0f, 0.0f),  // up
        new Vector3(0.0f, -1.0f, 0.0f), // down
        new Vector3(0.0f, 0.0f, 1.0f),  // front
        new Vector3(0.0f, 0.0f, -1.0f), // back
    };
```

Now change the `DrawVoxel()` function to this

```
    private void DrawVoxel(Vector3 voxelPos)
    {
        for (int side = 0; side < 6; side++)
        {
            if (!IsSolid(Data.NeighborOffset[side] + voxelPos))
            {
                // ... snipping irrelevant code ... //
            }
        }
    }
```

And now you should see this! Yay!

![optimized noise chunk](/Assets/optimized_noise_chunk.png)

One thing to note is that since we are not usually filling our vertex and triangle arrays all the way we can use the `m_vertexIndex` to find out how much of the array we are actually filling and make a "Slice" of the portion of the array that we are using.

```
    m_mesh = new Mesh
    {
        vertices = m_vertices.Slice<Vector3>(0, m_vertexIndex).ToArray(),
        triangles = m_triangles.Slice<int>(0, m_triangleIndex).ToArray()
    };
```

This can be visualized with the following diagram.

![slice of array](Assets/slice_of_array.png)

The last thing to note is that since we are checking for neighboring voxels will only ever use half of the `m_vertices` array at its max. So we can divide its allocation size by 2.

```
    private void Start()
    {
        m_vertices = new NativeArray<Vector3>(24 * Data.chunkSize * Data.chunkSize * Data.chunkSize / 2, Allocator.Temp);
        m_triangles = new NativeArray<int>(36 * Data.chunkSize * Data.chunkSize * Data.chunkSize / 2, Allocator.Temp);
    
        // ... snipping irrelevant code ... //
    }
```

If you're wondering why, then imagine a case where the most possible voxels are drawn (using neighbor checking), this would be perfectly alternating voxels, making a "checkered" pattern

![2D voxel terrain checkered](/Assets/2D_voxel_terrain_checkered.png)

# Jobified Chunk
Make a new micro project called JobChunk so that our project looks like this

```
    Assets/
    |___FastNoiseLite/
    |___Data.cs
    |___White (Material)
    |___Quad/
    |___Voxel/
    |___Chunk/
    |___JobChunk/
    |   |___JobChunk.unity
    |   |___JobChunk.cs
```

Now open up JobChunk.cs and add this boiler plate code. It's different from our previous code

```
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

You might notice that the all our member variables have been changed to be stored in a `NativeArray`. This is because after a Job finishes running we will want access to the `m_vertexIndex`, if we didn't use a NativeArray we would get zero back from the vertexIndex of our Job. Remember how C# makes copies of everything? NativeArrays allow us to access the same memory / instance from different places, rather than making copies of it every time. So to access any data outside a Job we need to use a NativeArray. Make a new Job (which are just struct's) that inherits from `IJob` below the `JobChunk` class

```
public struct ChunkJob : IJob
{
    public Vector3 chunkPos;
    public NativeArray<Vector3> vertices;
    public NativeArray<int> triangles;
    public NativeArray<Vector2> uvs;
    public NativeArray<int> vertexIndex;
    public NativeArray<int> triangleIndex;
}
```

`IJob` is an interface. Interfaces are just a bunch of functions that is a class or struct that uses that interface it has to implement those functions. In this case that function is `Execute()`. The execute function will be called when the job runs, so we can treat it the same way we've treated the `Start()` function.

```
public struct ChunkJob : IJob
{
    // ... snipping irrelevant code ... //

    public void Execute()
    {
        FastNoiseLite noise = new FastNoiseLite();
        noise.SetNoiseType(FastNoiseLite.NoiseType.OpenSimplex2);

        vertexIndex[0] = 0;
        triangleIndex[0] = 0;

        for (int x = 0; x < Data.chunkSize; x++)
        {
            for (int y = 0; y < Data.chunkSize; y++)
            {
                for (int z = 0; z < Data.chunkSize; z++)
                {
                    Vector3 position = new Vector3(x, y, z);

                    if (IsSolid(noise, position))
                    {
                        DrawVoxel(noise, position);
                    }
                }
            }
        }
    }
}
```

One note is that Jobs are not allowed to hold reference types (classes are always a reference to the same memeory and not a copy), so the FastNoiseLite class instance cannot live as a member variable. Instead we will have to declare and initialize it in the `Execute()` function. Paste in the `DrawVoxel()` and `IsSolid()` functions from before, and change them so that they take in the FastNoiseLite instance as a parameter. Now we can initialize our Job in the `Start()` function of our `JobChunk` class.

Since the `vertexIndex` is stored in a `NativeArray` we have to use `[0]` to access the first element in the array. This will result in the `DrawVoxel()` function changing slightly.

```
    private void DrawVoxel(FastNoiseLite noise, Vector3 position)
    {
        for (int face = 0; face < 6; face++)
        {
            if (!IsSolid(noise, Data.NeighborOffset[face] + position))
            {
                vertices[vertexIndex[0] + 0] = position + Data.Vertices[Data.BuildOrder[face, 0]];
                vertices[vertexIndex[0] + 1] = position + Data.Vertices[Data.BuildOrder[face, 1]];
                vertices[vertexIndex[0] + 2] = position + Data.Vertices[Data.BuildOrder[face, 2]];
                vertices[vertexIndex[0] + 3] = position + Data.Vertices[Data.BuildOrder[face, 3]];

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

```

When initializing a job we set up pointers to the memory it will use (`NativeArray`'s), as well as copies of data it will need, like our Chunk position.

```
    private void Start()
    {
        m_vertices = new NativeArray<Vector3>(24 * Data.chunkSize * Data.chunkSize * Data.chunkSize, Allocator.TempJob);
        m_triangles = new NativeArray<int>(36 * Data.chunkSize * Data.chunkSize * Data.chunkSize, Allocator.TempJob);
        m_vertexIndex = new NativeArray<int>(1, Allocator.TempJob);
        m_triangleIndex = new NativeArray<int>(1, Allocator.TempJob);

        ChunkJob job = new ChunkJob();
        job.chunkPos = gameObject.transform.position;
        job.vertices = m_vertices;
        job.triangles = m_triangles;
        job.vertexIndex = m_vertexIndex;
        job.triangleIndex = m_triangleIndex;
    }
```

When we run the job it will return a `JobHandle`. This lets us have a "handle" to hold on to our Job. Since the Job will be running on a differrent thread, we can't know when it will finish running, so we just have to wait until it completes, and that is exaclty what the `Complete()` function does.

```
    private void Start()
    {
        // ... snipping irrelevant code ... //

        JobHandle handle = job.Schedule();
        handle.Complete();
    }
```

Now the rest of the code is much the same as past times

```
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

If you run this you should see a chunk as before! YESSSSS!!!!!!!!!! Now even though we haven't actually benefited from using the JobSystem in this example, it lays the groundwork for week 4, where we will be drawing a lot of chunks using the job system.

# NOTE TO SELF
thinking of just doing week 4 right here.