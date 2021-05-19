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

We will be mostly copying last week's voxel mirco project. Change the code in Chunk.cs to look like this.

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

We make it a `const` to make sure that the number doesn't change. Also array initalizations require that a `const` is used for telling their size, arrays aren't dynamic in size. (A readonly wouldn't work since it could be modified by an enclosing (non static) class). 

Our vertex and triangle arrays will need to be larger, since we will be making a chunk and not a single voxel. We can calculate the amount by multipling 24 and 36 (the size of vertex and triangle data for a single voxel) by the number of voxel we will be building in our chunk. This leads to the following code

```
    private void Start()
    {
        m_vertices = new NativeArray<Vector3>(24 * Data.chunkSize * Data.chunkSize * Data.chunkSize, Allocator.Temp);
        m_triangles = new NativeArray<int>(36 * Data.chunkSize * Data.chunkSize * Data.chunkSize, Allocator.Temp);

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

        // ... snip ... //
    }
```

Now the rest of the code in start is exactly the same as before! I'll let you copy paste that from Voxel.cs. 

Now make a new empty gameObject in the Chunk scene (Chunk.unity) and add Chunk.cs to it as a component. Also don't foregt to set the material. If you click play you'll see a chunk drawn!

# Optimized Chunk
If you look at the inside of our current chunk you might see something like this

![unoptimized chunk mesh](/Assets/unoptimized_chunk_mesh.png)

To make it so voxel's don't draw their side if that side has a neighboring voxel, we need an algorithm to know where we want voxel's and where we don't. For terrains the most popular option is to use Noise. Perlin Noise, Complex Noise, Value Noise... Noise is a way of getting smoothed random numbers. Most noise algorithms take in an offset in either 2D or 3D and then return a number between 0 and 1. If you saved the values of a noise function on a texure fading between black (meaning 0) and white (meaning 1) it might look like this.

![TODO texture image]()

We could initialize a 3D array with our data once at the beginning and then reference that when building voxels to check if they have neighbors. This saves us on processing power at the trade off of using memory. We would have to do this if we wanted editable terrain.

There is a noise library in the Resources of this repositiory called FastNoiseLite.zip. It is taken from [here](https://github.com/Auburn/FastNoiseLite/tree/master/CSharp) Download our repo and open the prepared zip and drag the folder "FastNoiseLite" into your project so that your project looks like this.

```
    Assets/
    |___FastNoiseLite/
    |   |___FastNoiseLite.cs
    |___Data.cs
    |___White (Material)
    |___Quad/
    |___Voxel/
    |___Chunk/
```

We will make a function that takes in a voxel's position and returns `true` or `false` using the FastNoiseLite class. First we need to add a member variable to hold an instance of FastNoiseLite

```
    private FastNoiseLite m_noise;
```

Then we need to initialize the FastNoiseLite instance in `Start()` and set the nosie type we want

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

We need to multiply the noise value by some height value so that our terrain has hills higher than 1 block. We chose to multiply by our chunk size so that the highest possible height is the same as the chunk height, and so that if we change our chunk size the max height changes to. We check if a voxel's y position is less than the noise value, if it is then return true.

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

And we have terrain!

![first noise chunk](/Assets/first_noise_chunk.png)

But we still aren't checking if a voxel has a neighbor! 

To do neigbor check for each voxel we will make a lookup table of offset positions that we will use in our for loop in the DrawVoxel function. Add the following to Data.cs

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

# Jobified Chunk