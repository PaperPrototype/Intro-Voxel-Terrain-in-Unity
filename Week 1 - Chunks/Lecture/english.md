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

To make a chunk all we have to do is place voxel's on a 3D grid. To achieve this we can copy last weeks `DrawVoxel()` function with one minor change. We will pass in a position so that we can draw the voxel wherever we want.

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

Now we can use nested for loops to place voxel's next to eachother to make a chunk. Before we do that we need to know the "size" of our chunk, how many voxel's to draw in the x, y, and z direction. We could add it manully but we will need to know the size of our chunk in many places, so add a line to Data.cs to have a centralized place for this info

```
    public const int chunkSize = 8;
```

We make it a `const` to make sure that it doesn't change, and Array initalizations require that a `const` is used for telling their size.

SDFGHJSDFGHJKSDFGHJKDFGHJKSDFGHJKDFGHJKLFGHJFGHJKLFGHJFGHJKFGHJKDFGHJKDFGHJKFGHJKLDFGHJKFGHJKDFGHJKDFGHJKDFG ################# --> LEFT OFF HERE <-- ################

```
    private void Start()
    {
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
    }
```

Now since