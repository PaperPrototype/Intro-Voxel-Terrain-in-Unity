Pixel's make up 2D pictures and worlds. The idea of Voxel's is to make everything in a game out of 3D pixel's or "voxel". 

Minecraft, Astroneer, SpaceEngineers, and NoMansSky are all games that use voxel systems for their terrain
The biggest problem's facing voxel technology have been
- storing voxels takes a lot of space (picture formats are large enough)
- most graphis cards have hardware triangle rasterization (rasterization means filling in three points to make a triangle) which is extremely fast. On the other hand there is no (or little) support for hardware voxel rasterization. Instead we have to use software voxel rasterization or make the voxels out of triangles. Most voxel engines use triangles.

Worthy mentions of pioneers in voxel technology are
- VoxelBee - [Youtube (TODO)]()
- VoxelFarm - [Blog](https://procworld.blogspot.com/), [Website (TODO)]()
- Eucludean - Youtube TODO, Website TODO
- Vox the game

Most modern game engine rendering systems don't support 3D picture (voxel) formats. So we will be using many triangles put together to represent the voxel's. When we hold many triangles together this is often called a "Mesh". We will be learning about meshes in this lecture.

### Vertices
A mesh is really just contains information on how to represent a shape. Most meshes use a sets of three positions (called "points") in space to construct the triangles. Triangles can *approximately* make any shape. For example you can make a square using 2 triangles, and you can make a *pretty good* circle using triangles. We often refer to "points in space" (in the context of meshes) as vertices (singular is vertex). Let's make a triangle using 3 vertices.

(This isn't valid C# code, the techinical name would be "pseudo code" meaning its just to show a concept or express "to be" code)

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1 0) }
```

In Unity we can use a `Vector3` to represent any set of 3 decimal numbers. For vertices we use `Vector3`'s. And the points would look like this in 2D (ignoring z).

![three vertices](/Assets/three_vertices.png)


### Triangles ("connecting" our vertices)
Now this won't actually work because in a mesh we need to know what 3 vertices we should use to make a triangle, since there will be many vertices and triangles in a mesh.

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0) }
    mesh.triangles = { 0, 1, 2 }
```

In the list of "triangles" (they're just a set of 3 `int`s) we are saying, get the vertex at `0` in the array and make that the first vertex in our triangle. Then get vertex at `1` and use it for the second vertex of the triangle. Then get vertex at `2` for the third vertex. These "triangle" ints are called indexes. An index tells us where in a list (or array) to find something. Triangle ints (usually just called "triangles") have to be in sets of 3, if they are not we will get an error.

![triangle mesh](/Assets/triangle_mesh.png)

(If you think this is pointless... then yes, at this point it is. But when you want more than 1 triangle in a mesh it makes sense).

### A quad / square (two triangles)
To make a square shape (a "quad" in 3D jargon) we're going to add 1 more vertex so we have 4 vertices in the positions we need to make a square. We don't have to add 3 more vertices, since vertices can be referenced by any triangle, and therefore reused.

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0), Vector3(1, -1, 0) }
    mesh.triangles = { 0, 1, 2 }
```

![four vertices](/Assets/four_vertices.png)

We need to add another set of 3 ints (referred to as a triangle) to tell Unity which vertices to use to make the second triangle.

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0), Vector3(1, -1, 0) }
    mesh.triangles = { 0, 1, 2, 0, 2, 3 }
```

And voila!
![quad mesh](/Assets/quad_mesh.png)


### Normals (which side of the triangle to render)
We need a "direction" to tell Unity which side of the quad is up. These are called Normals. Normals tell the exact direction that is "upwards" from a surface or triangle. Here are normals telling us which direction is "upwards" on our triangle. (we are looking at it from the top = y).

![correct surface normal](/Assets/correct_normal.png)

But what if the normals face in a direction that is not 90 degrees angled from the surface? The renderer doesn't know anything, so it will just shade the surface's color as if it was at the angle the normal is saying its at.

![incorrrect surface normal](/Assets/incorrect_normal.png)

This could let us make surfaces, that are made of many triangles, that are not smooth look as if they were smooth.

The format for normals in a Unity mesh is each vertex needs a normal. We will assume z+ is facing toward us.

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0), Vector3(1, -1, 0) }
    mesh.normals = { Vector3(0, 0, 1), Vector3(0, 0, 1), Vector3(0, 0, 1), Vector3(0, 0, 1) }
    mesh.triangles = { 0, 1, 2, 0, 2, 3 }
```

Normals are usually a Unit Vector. Unit Vector just means that the distance from the origin (`Vector3(0, 0, 0)`) to the "position" or "end" of our Normal needs to be 1. The distance from `Vector3(0, 0, 0)` to one of our normals `Vector3(0, 0, 1)` is obviously 1, so we're fine. 

![magnitude of normal](/Assets/normal_magnitude.png)

The distance of a vector is called it's Magnitude. Also Normals are always just giving us a direction, so they don't need to be positioned near the vertices.

Unity has a built in function for generating normals for us.

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0), Vector3(1, -1, 0) }
    mesh.triangles = { 0, 1, 2, 0, 2, 3 }
    mesh.RecalculateNormals()
```

`RecalculateNormals()` requires us to already have our `vertices` and `triangles` set. But there is one other thing. If we are using Unity's function it needs to know which side of the triangle/quad to render. This is because Unity only renders 1 side of a mesh. This is called backface culling. Unity uses the order we put the triangles in to determine which side to render. The way we can remember is clockwise order will make the normals face us

(taken from https://forum.unity.com/threads/unity-has-a-clockwise-winding-order.129923/#post-3198466)
![clockwise triangle order](/Assets/clockwise_triangle.png)

... and counter-clockwise will make it face away from us.

![counter clockwise triangle](/Assets/counter_clockwise_triangle.png)


# Making the quad in Unity
We will be making lots of micro Unity projects to test out what we learn. Make a new Unity project. Set it to 3D. Set it up like the diagram below with a folder for this micro project called Quad.

```
    Assets/
    |___Quad/
    |   |___Quad.unity (Scene)
    |   |___Quad.cs (Script)
```

In the scene make an empty GameObject. Now open the script in VS Code or whatever you're using to code. We are going to want a MeshRenderer component and a MeshFilter component. Lets force Unity to have these on our GameObject. Above the Quad class add

```cs
    [RequireComponent(typeof(MeshRenderer))]
    [RequireComponent(typeof(MeshFilter))]
```

Now go back to the scene and add the Quad script to the empty GameObject. It should automatically also add the needed components. Or you can just manually add them, but if you click play and it doesn't have them Unity will warn you.

Now we make our mesh as before. In `Start()` add

```cs
    Mesh mesh = new Mesh
    {
        vertices = new Vector3[] { new Vector3(-1, -1, 0), new Vector3(-1, 1, 0), new Vector3(1, 1, 0), new Vector3(1, -1, 0) },
        triangles = new int[] { 0, 1, 2, 0, 2, 3 }
    };
    mesh.RecalculateBounds();
    mesh.RecalculateNormals();
```

Also add a `RecalculateBounds()` this tells the renderer what 3D space the mesh takes up as if it were enclosed in a Box. (A box is easy to work with and very performant for volume calculations). Now add the mesh to our MeshFilter in `Start()`

```cs
    gameObject.GetComponent<MeshFilter>().mesh = mesh;
```

Now hit play. Voila! (If you get an error, make sure your triangles aren't trying to access vertices that don't exist).

The mesh should be colored pink. That just means we don't have a material attached. Add a material we can use for all our meshes from now on, put it in the root of the project.

```
    Assets/
    |___White (Material)
    |___Quad/
    |   |___Quad.unity
    |   |___Quad.cs
```

You can now add the material to the MeshRenderer component.


# UV's and textures
Almost all terrains have textures for grass, dirt, and rocks. A texture is just an image. The renderer will take care of 'warping' the textures onto our triangles, all we have to do is tell it that a specific part of our texture should "map" to a specific vertex. A textures "UV size" in Unity is always from 0 to 1, regardless of its size in pixels.

![2D uv texture](/Assets/2D_uv_texture.png)

A UV in Unity's mesh format is a `Vector2`, and each vertex in the mesh gets a UV. A Value of `Vector2(0, 0)` in a UV would get the bottom left corner of our texture.

![2D uv texture coordinate](/Assets/2D_uv_texture_coordinate.png)

Making a UV at 0 index of the meshes UV array would correspond to the first vertex in the mesh.

```
    mesh.vertices = { Vector3(-1, -1, 0) }
    mesh.uv = { Vector2(0, 0) }
```

Any subsequent UV's would reference the vertex at its same corresponding index.

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0) }
    mesh.uv = { Vector2(0, 0), Vector2(0, 1), Vector2(1, 1) }
```

To texture our triangle we could change the pseudo code to the following.

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0) }
    mesh.uv = { Vector2(0, 0), Vector2(0, 1), Vector2(1, 1) }
    mesh.triangles = { 0, 1, 2 }
```

This would yield the following

![2D textured triangle](/Assets/2D_textured_triangle.png)

To texture the quad we only have to add one more UV, since there is one UV per vertex

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0), Vector3(1, -1, 0) }
    mesh.uv = { Vector2(0, 0), Vector2(0, 1), Vector2(1, 1), Vector2(1, 0) }
    mesh.triangles = { 0, 1, 2, 0, 2, 3 }
```

![2D textured quad](/Assets/2D_textured_quad.png)


# Adding uv's to the quad in Unity
From the Resources folder of this course get the file called "Texture.png.zip" unzip it and place it in the root of the Project.

```
    Assets/
    |___Texture.png
    |___White (Material)
    |___Quad/
```

Now in the "White" material set its albedo (color) to be Texture.png. You can drag Texture.png onto the square next to the word "albedo" on the material. Also set the materials smoothness to be 0. This will make it less shiny. 

Change the mesh code in Quad.cs to the following

```cs
    Mesh mesh = new Mesh
    {
        vertices = new Vector3[] { new Vector3(-1, -1, 0), new Vector3(-1, 1, 0), new Vector3(1, 1, 0), new Vector3(1, -1, 0) },
        uv = new Vector2[] { new Vector2(0, 0), new Vector2(0, 1), new Vector2(1, 1), new Vector2(1, 0) },
        triangles = new int[] { 0, 1, 2, 0, 2, 3 }
    };
```

Run this new code and you should see the following

![textured quad screenshot](/Assets/textured_quad_screenshot.png)


# Making a voxel in Unity
Voxel terrains are made of the surface of thousands of small voxels. If we were to make a terrain out of voxels and then slice it, it would look like this

![2D voxel terrain unoptimized](/Assets/2D_voxel_terrain_unoptimized.png)

You may notice the inefficiency here. The side of a voxel that is not visible is still being generated and rendered. This leads to horrible performance in a large world. To fix this we can generate the individual sides of a voxel separately and then check if the voxel has a neighbor, if it does then we just don't generate that side.

![2D voxel terrain optimized](/Assets/2D_voxel_terrain_optimized.png)

Add a new micro project to our Unity project so that looks like this

```
    Assets/
    |___White (Material)
    |___Quad/
    |___Voxel/
    |   |___Voxel.unity
    |   |___Voxel.cs
```

Now open up the Voxel scene and add an empty GameObject. Open up Voxel.cs and force Unity to add our components as before. Now instead of hard coding our triangle we are going to be smart. We will make an array of all 8 possible vertices for a cube, that we can always reference. We'll make a new script to contain all this called DataDefs.cs

```
    Assets/
    |___DataDefs.cs
    |___White (Material)
    |___Quad/
    |___Voxel/
```

Change DataDefs.cs to the following.

```cs
using UnityEngine;

public static class DataDefs
{
    public static readonly Vector3[] Vertices = new Vector3[8]
    {
        new Vector3(-0.5f, -0.5f, -0.5f),
        new Vector3(0.5f, -0.5f, -0.5f),
        new Vector3(0.5f, 0.5f, -0.5f),
        new Vector3(-0.5f, 0.5f, -0.5f),
        new Vector3(-0.5f, -0.5f, 0.5f),
        new Vector3(0.5f, -0.5f, 0.5f),
        new Vector3(0.5f, 0.5f, 0.5f),
        new Vector3(-0.5f, 0.5f, 0.5f)
    };
}
```

This is all 8 possible vertices for the corners of a voxel.

Now we need a way to find all the vertices per each side of the voxel. For this we'll make a lookup table (a 2D array) that gets 4 vertices based on the side of the cube we want.

```cs
    public static readonly int[,] BuildOrder = new int[6, 4]
    {
        // 0 1 2 2 1 3 <- triangle order

        // right, left, up, down, front, back
        
        {1, 2, 5, 6},  // right face
        {4, 7, 0, 3}, // left face
        
        {3, 7, 2, 6}, // up face
        {1, 5, 0, 4}, // down face
        
        {5, 6, 4, 7}, // front face
        {0, 3, 1, 2} // back face
    };
```

We've made the `BuildOrder` lookup table for getting the vertices so that the triangles for each side of a voxel will always be the same `0, 1, 2, 2, 1, 3`.

We mark the class as well as the arrays `static` for a reason. Anything marked as `static` gets saved to a special part of our programs for data that will be accessed a lot and doesn't change. A result of this is there is only one copy of that data and data can be accessed really fast. Also the arrays are marked as `readonly`. This tells the compiler that the data can only be read and not modified. The correct word is "not mutated" or "immutable". This allows any Thread or Job to access the arrays since there is only one copy of them and they aren't allowed to be modified. A result of nothing being allowed to change is that the C# Job system won't complain when we use the lookup tables in a Job.

We will start simple and use the lookup tables to make one side of a voxel (AKA a quad). Open up the Quad.cs code and change it to use the lookup tables to make the quad.

```cs
// ... snipping header stuff ... //
    void Start()
    {
        Mesh mesh = new Mesh
        {
            vertices = new Vector3[]
            {
                DataDefs.Vertices[DataDefs.BuildOrder[0, 0]],
                DataDefs.Vertices[DataDefs.BuildOrder[0, 1]],
                DataDefs.Vertices[DataDefs.BuildOrder[0, 2]],
                DataDefs.Vertices[DataDefs.BuildOrder[0, 3]],
            },
            uv = new Vector2[]
            {
                new Vector2(0, 0),
                new Vector2(0, 1),
                new Vector2(1, 0),
                new Vector2(1, 1)
            },
            triangles = new int[]
            {
                0, 1, 2, 2, 1, 3
            }
        };
        mesh.RecalculateBounds();
        mesh.RecalculateNormals();

        gameObject.GetComponent<MeshFilter>().mesh = mesh;
    }

```

The vertices array uses the `DataDefs.BuildOrder` lookup table to get the vertices out of the `Vertices` table for side 0 of a voxel. You will notice all the first array indexes are 0, this gets the first dimension of the array...

```
// 1D array
BuildOrder = int[]
{
    first dimension,
    first dimension,
};
```

which contains the indexes to use to lookup the correct vertices to use in the `DataDefs.Vertices` table. Then the `0, 1, 2, 3` numbers get the individual indexes from the second dimension in the lookup table.

```
// 2D array      \/ notice the `,`
BuildOrder = int[,]
{
    {
        second_dimension, 
        second_dimension, 
        second_dimension, 
        second_dimension
    },
    // or
    { second_dimension, second_dimension, second_dimension, second_dimension },
};
```

Then the triangles have to change from `0, 1, 2, 0, 2, 3` to `0, 1, 2, 2, 1, 3`, which will get the vertices from and build the triangles. The reason for this is that the `BuildOrder` table gets vertices from the `Vertices` table in a specific order, and the triangles have to be able to access the vertices in a pattern that can work with the order that the `BuildOrder` table gives us vertices.

Also the uv's change from `(0, 0), (0, 1), (1, 1), (1, 0)` to `(0, 0), (0, 1), (1, 0), (1, 1)` since the positions of the vertices that they refer to have changed.

Now if you run this it should still work!

# Actually making a voxel
Open Voxel.cs and create a new mesh except we are going to use our lookup tables this time. We use a NativeArray to store the vertices and triangles. This is to get you used to using them since they are JobSystem friendly and you will need to know how to use them to use the JobSystem.

```cs
using UnityEngine;
using Unity.Collections;

[RequireComponent(typeof(MeshRenderer))]
[RequireComponent(typeof(MeshFilter))]
public class Voxel : MonoBehaviour
{
    private Mesh m_mesh;
    private NativeArray<Vector3> m_vertices;
    private NativeArray<int> m_triangles;
    private NativeArray<Vector2> m_uvs;
    private int m_vertexIndex = 0;
    private int m_triangleIndex = 0;

}
```

(We use the naming convention `m_variable` to represent private member class variables, this helps to distiniguish them from the meshes variable names. We recommend you copy us for an optimal course experience.)

The m_vertexIndex is used to keep track of the current newest vertex. That way new vertices being added don't overwrite eachother, but get added on after the already added vertices. You can visualize this as

```
vertexIndex = 4

                            \/- vertexIndex
vertices = [v0, v1, v2, v3, null, null, null, null]
```

The m_triangleIndex serves the same purpose except for the triangles.

Make a new function that uses the lookup tables to make a voxel. The `side` variable looks up each side of the voxel in the lookup table. Also a voxel (AKA cube) has 6 sides.

```cs
    private void DrawVoxel()
    {
        for (int side = 0; side < 6; side++)
        {
            m_vertices[m_vertexIndex + 0] = DataDefs.Vertices[DataDefs.BuildOrder[side, 0]];
            m_vertices[m_vertexIndex + 1] = DataDefs.Vertices[DataDefs.BuildOrder[side, 1]];
            m_vertices[m_vertexIndex + 2] = DataDefs.Vertices[DataDefs.BuildOrder[side, 2]];
            m_vertices[m_vertexIndex + 3] = DataDefs.Vertices[DataDefs.BuildOrder[side, 3]];

            // get the correct triangle index
            m_triangles[m_triangleIndex + 0] = m_vertexIndex + 0;
            m_triangles[m_triangleIndex + 1] = m_vertexIndex + 1;
            m_triangles[m_triangleIndex + 2] = m_vertexIndex + 2;
            m_triangles[m_triangleIndex + 3] = m_vertexIndex + 2;
            m_triangles[m_triangleIndex + 4] = m_vertexIndex + 1;
            m_triangles[m_triangleIndex + 5] = m_vertexIndex + 3;

            // set the uv's (different than the quad uv's due to the order of the lookup tables in DataDefs.cs)
            m_uvs[m_vertexIndex + 0] = new Vector2(0, 0);
            m_uvs[m_vertexIndex + 1] = new Vector2(0, 1);
            m_uvs[m_vertexIndex + 2] = new Vector2(1, 0);
            m_uvs[m_vertexIndex + 3] = new Vector2(1, 1);

            // increment by 4 because we only added 4 vertices
            m_vertexIndex += 4;

            // increment by 6 because we added 6 int's to our triangles array
            m_triangleIndex += 6;
        }
    }
```

You should notice that when setting the triangles we set them to 

```cs
= m_vertexIndex + 0;
= m_vertexIndex + 1;
= m_vertexIndex + 2;
= m_vertexIndex + 2;
= m_vertexIndex + 1;
= m_vertexIndex + 3;
```

which is the order that corresponds to the comment in the `BuildOrder` lookup table

```cs
    // 0 1 2 2 1 3 <- triangle order per side
```

We add the vertexIndex to the triangles when setting the triangles to make sure that the triangles are refering to their set of vertices and not the previous set.

Now we can initialize all of the member variables in `Start()` and Draw the voxel using the `DrawVoxel()` function. Then set the meshes data, and convert our NativeArray to an Array (since the mesh takes in a regular array) using `.ToArray()`. We set the MeshFilters mesh to `m_mesh`. Also make sure to Calculate the normals and Bounds.

```cs
    private void Start()
    {
        m_vertices = new NativeArray<Vector3>(24, Allocator.Temp);
        m_triangles = new NativeArray<int>(36, Allocator.Temp);
        m_uvs = new NativeArray<Vector2>(24, Allocator.Temp);

        DrawVoxel();

        m_mesh = new Mesh
        {
            vertices = m_vertices.ToArray(),
            triangles = m_triangles.ToArray(),
            uv = m_uvs.ToArray()
        };

        m_mesh.RecalculateBounds();
        m_mesh.RecalculateNormals();

        gameObject.GetComponent<MeshFilter>().mesh = m_mesh;

        m_vertices.Dispose();
        m_triangles.Dispose();
        m_uvs.Dispose();
    }
```

You should notice the line `Allocator.Temp`. This gets us the memory we need realy fast, but it can only exist for 4 frames (ever heard of fps? (frames per second)). The other `Allocator.Persistent` allows us to have the memroy for as long as we want, but it takes longer to find that memory. `Allocator.TempJob` is a special allocator for Job's and multithreading.

Also since we are using NativeCollections (Native meaning it uses actual pointers and not copies of everything. C# normaly makes copies of everything to make our code "safe") we have to manually free our memory, much like in C or C++.

```cs
    private void Start()
    {
        //... snipping out previous code ...//

        m_vertices.Dispose();
        m_triangles.Dispose();
    }
```

And now if you hit play you should have a voxel! (Make sure to set your Material) See you next week!