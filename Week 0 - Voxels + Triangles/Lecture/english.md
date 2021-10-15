TODOS
- reduce ambiguity between the word voxel and cube

Pixels make up 2D pictures and worlds. The idea of Voxels is to make everything in a game out of 3D pixels or "voxels". 

Minecraft, Astroneer, SpaceEngineers, and NoMansSky are all games that use voxel systems for their terrain.
The biggest problems facing voxel technology:
- Storing voxels takes a lot of space (picture formats are large enough).
- Most graphics cards have hardware triangle rasterization (rasterization means filling in three points to make a triangle) which is extremely fast. On the other hand, there is no (or little) support for hardware voxel rasterization. Instead, we have to use software voxel rasterization or make the voxels out of triangles. Most voxel engines use triangles.

Worthy mentions of pioneers in voxel technology are
- VoxelBee - [Youtube (TODO)]()
- VoxelFarm - [Blog](https://procworld.blogspot.com/), [Website (TODO)]()
- Eucludean - Youtube TODO, Website TODO
- Vox the game

Most game engines' rendering systems don't support 3D picture (voxel) formats. So we will be using many triangles put together to represent the voxels (aka Cubes). When we hold many triangles together this is often called a "Mesh". We will be learning about meshes in this lecture, and using them to make a single cube.

By the end of the lecture, you will have made this.

![final_voxel](/Assets/final_voxel.png)

### Vertices
(For really good definitions of the vocabulary and words used in this lecture open up the notes for this lecture. [Here's a quick link](https://github.com/PaperPrototype/Intro-Voxel-Terrain-in-Unity/blob/main/Week%200%20-%20Voxels%20%2B%20Triangles/Notes/english.md))

A "mesh" contains information on how to represent a shape. Most meshes use a set of three positions, called a vertex, to make triangles. Why triangles? Well, since triangles are the simplest shape, we can make just about any other shape using triangles.

For example, you can make a square using 2 triangles, and you can make a *pretty good* circle using a whole lot of triangles put together. 

Let's make a triangle using 3 *vertices* in some fake code. (Funny thing is, we also have another name for "position", called a "point").

(This code is just an example no need to make a script in unity)
```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1 0) }
```

In Unity we use a `Vector3` to represent a set of 3 decimal numbers, vertices are a position, and have 3 decimal numbers, so a `Vector3` works pretty well.

If we make a simple graph with an `x` and `y` axis (much like the one you use for math) and put the vertices from the fake code onto it, you would see something like this.

![three vertices](/Assets/three_vertices.png)


### Triangles ("connecting" our vertices)
Now, if everything is a triangle how do we tell the computer to put them together? In code, we can say get the first vertex, the second one, and the third one, then fill in the shape of a triangle using those vertices.

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0) }
    mesh.triangles = { 0, 1, 2 }
```

If you look at the code that is exactly what we did! Now as a note, these "triangles" are just numbers telling us, get the first vertex in the list of vertices, but since computers start counting at 0 rather than 1, the "first" vertex is at `0` in the list.

These "triangle" numbers are rendered in sets of 3. So if had say `0, 1, 2, 1` the last number wouldn't belong to any "triangle", since it has to be part of a set of other numbers. We would also get an error from Unity.

Now if we used the "triangles" in our code to build a triangle it would look like this.

![triangle mesh](/Assets/triangle_mesh.png)

### A square (AKA quad)
To make a square shape (a "quad" in 3D jargon) we need one more vertex in the upper right corner.

![four vertices](/Assets/four_vertices.png)

Then to use the new vertex, and some of the old ones, to make another "triangle". For this, we can add another 3 numbers to the triangles list.

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0), Vector3(1, -1, 0) }
    mesh.triangles = { 0, 1, 2,  0, 2, 3 }
                                 \__second set of "triangle" numbers
```

We will call these "set of 3 numbers" a "triangle" from now on.

If we rendered the resulting "mesh" it would look like this 

![quad mesh](/Assets/quad_mesh.png)

Remember a "mesh" is just the name for the idea of making shapes with triangles.

### Normals (which side of the triangle to render)
We need a "direction" to tell the renderer which side of the quad is up and should be rendered. Also, the opposite side that is not "upwards" doesn't get rendered. This is for performance reasons and is called "backface culling". 

The direction that is upwards from a mesh is called a "normal." Usually a normal is angled at 90 degrees from a surface.

The renderer uses normals to know the exact direction that is upwards from a surface and it uses that to apply lighting and shadows.

Here is what normals would look like on our quad.

![correct surface normal](/Assets/correct_normal.png)

But what if the normals were aiming in a direction that is not a 90-degree angle from the surface?

(We usually refer to the aim direction of a normal as the direction a normal is "facing") 

The renderer will just smooth the surface's color so that the "angle" the surface is facing is consistent with the normals.

![incorrrect surface normal](/Assets/incorrect_normal.png)

This could let us make surfaces that are not smooth look as if they were smooth. This is used a lot in games because it gives better performance to use normals to smooth a surface rather than a million detailed triangles.

Each vertex on a mesh needs a normal.

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0), Vector3(1, -1, 0) }
    mesh.normals = { Vector3(0, 0, 1), Vector3(0, 0, 1), Vector3(0, 0, 1), Vector3(0, 0, 1) }
    mesh.triangles = { 0, 1, 2, 0, 2, 3 }
```

If the distance a normal covers is 1, then it is called a "Unit" normal.

![]() TODO add a pic of Unit normal being measured

Normals are always just giving us a "direction", so they are not positioned near their vertex. Instead since all we care about is the direction they give us, they are positioned in the center of the world.

The distance from the beginning to the end of any normal (or any Vector3) is called its Magnitude. 

![magnitude of normal](/Assets/normal_magnitude.png)

So we could say that a "Unit" normal, has a "magnitude" of 1 (I'm telling you all this because someone will probably use these words and you might want to know what they are talking about).

Now calculating all the normals for a mesh can get complicated so Unity has a built-in function for generating normals for us called `RecalculateNormals`.

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0), Vector3(1, -1, 0) }
    mesh.triangles = { 0, 1, 2, 0, 2, 3 }
    mesh.RecalculateNormals()
```

Don't worry this is still fake code so you don't have to worry about adding this to a script (yet). Although the `RecalculateNormals` function is a real function in Unity.

To use the `RecalculateNormals` function it requires us to already have the `vertices` and `triangles` set in the mesh.

The `RecalculateNormals` function uses the "order" we put the triangles in to determine which side of a triangle the normals should face.

Clockwise will make the normals face us

(taken from https://forum.unity.com/threads/unity-has-a-clockwise-winding-order.129923/#post-3198466)
![clockwise triangle order](/Assets/clockwise_triangle.png)

... and counter-clockwise will make them face away from us.

![counter clockwise triangle](/Assets/counter_clockwise_triangle.png)


# Making the quad in Unity
Make a new Unity project for this course. Make the project a 3D project using the Universal Render Pipeline.

Once that loads, make a new folder under the Assets folder called "Quad". In the "Quad" folder put a new scene called Quad, as well as a new script called `Quad`. The project should then look like the following.

```
    Assets/
    |___Quad/
    |   |___Quad.unity (Scene)
    |   |___Quad.cs (Script)
```

Open the scene we made by double-clicking it. In the "Quad" scene add an empty GameObject. 

Open the script by double-clicking it. The gameObject will need a MeshRenderer component and a MeshFilter component. We can force Unity to have these on the GameObject by adding a `RequireComponent` attribute above the `Quad` class.

```cs
using UnityEngine;

[RequireComponent(typeof(MeshRenderer))] // added this
[RequireComponent(typeof(MeshFilter))] // added this
public class Quad : Monobehaviour
// ... irrelevant code
```

Now make sure to save the script by clicking `cmd + s` on mac or `ctrl + s` on windows (you can also go `file -> save` and click save).

Go back to the scene and drag the Quad script onto the empty GameObject. It should automatically also add the needed components since we added the `RequireComponent` attribute.

Now in the `Quad` script, we can make our mesh as before. In the `Start()` function add

```cs
    void Start()
    {
        Mesh mesh = new Mesh();
    }
```

Now we make a new `Mesh` object (a "class" is an object), and then as we did before in the fake code we can add the vertices and the triangles to the mesh.

```cs
    void Start()
    {
        Mesh mesh = new Mesh();
        mesh.vertices = new Vector3[] { new Vector3(-1, -1, 0), new Vector3(-1, 1, 0), new Vector3(1, 1, 0), new Vector3(1, -1, 0) };
        mesh.triangles = new int[] { 0, 1, 2, 0, 2, 3 };
    }
```

There is a handy way to write making a mesh so that we don't have to say `mesh.vertices = ...etc`

```cs
    void Start()
    {
        Mesh mesh = new Mesh
        {
            vertices = new Vector3[] { new Vector3(-1, -1, 0), new Vector3(-1, 1, 0), new Vector3(1, 1, 0), new Vector3(1, -1, 0) },
            triangles = new int[] { 0, 1, 2, 0, 2, 3 }
        };
    }
```

Now we can run the `RecalculateNormals` function.

```cs
        Mesh mesh = new Mesh
        {
            vertices = new Vector3[] { new Vector3(-1, -1, 0), new Vector3(-1, 1, 0), new Vector3(1, 1, 0), new Vector3(1, -1, 0) },
            triangles = new int[] { 0, 1, 2, 0, 2, 3 }
        };
        mesh.RecalculateNormals();
        
        mesh.RecalculateBounds();
```

We also add a line of code that recalculates the "bounds" of the mesh. This tells the renderer what 3D space the mesh takes up as if it were enclosed in a Box (A box is easy to code with and very performant for volume calculations). 

Then finally we set the gameObjects mesh by getting the `MeshFilter` component and setting the mesh variable to the mesh we just created.

```cs
    gameObject.GetComponent<MeshFilter>().mesh = mesh;
```

Now hit play. And Voila! (If you get an error, make sure your triangles aren't trying to access vertices that don't exist, or just [join our Discord server](https://discord.gg/Gp7YEUkVHC) for help).

If you click on the gameObject we created and then click the `f` key to focus on the gameObject (f stands for "focus"), you should then be able to hold the `option` key and orbit around it with the mouse.

You will see that the color of the quad is pink! That just means it has no material for it. Make a new material we can use for all our meshes. Your project should look like this.

```
    Assets/
    |___White.mat (Material) <- new material
    |___Quad/
    |   |___Quad
    |   |___Quad
```

Add the material to the quad gameObject. Now when you hit play you should see a white-colored square.

# UVs and textures
Almost all terrains have textures for grass, dirt, and rocks.

A texture is just an image. The renderer takes care of 'warping' the textures onto our triangles, all we have to do is tell it which vertex a specific part of our texture should "map" to. 

The textures "UV size" in Unity is always from 0 to 1, regardless of the texture's actual size in pixels.

![2D uv texture](/Assets/2D_uv_texture.png)

A UV in Unity's mesh format is represented as a `Vector2`.

A Value of `Vector2(0, 0)` in a UV would get the bottom left corner of our texture.

![2D uv texture coordinate](/Assets/2D_uv_texture_coordinate.png)

How can we use this to map UV texture coordinates to a vertex? Well, each vertex in the mesh gets a UV. So putting a UV at the 0 index of the UV array would correspond to the first vertex in the mesh.

```
    mesh.vertices = { Vector3(-1, -1, 0) }
    mesh.uv       = { Vector2(0, 0) }
```

Any subsequent UVs would reference the vertex with the same index.

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

(pseudo code means "fake" explanation code)

This would yield the following

![2D textured triangle](/Assets/2D_textured_triangle.png)

To texture the quad we only have to add one more UV, since there is one UV per vertex

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0), Vector3(1, -1, 0) }
    mesh.uv = { Vector2(0, 0), Vector2(0, 1), Vector2(1, 1), Vector2(1, 0) }
    mesh.triangles = { 0, 1, 2, 0, 2, 3 }
```

![2D textured quad](/Assets/2D_textured_quad.png)

# Adding UVs to the quad in Unity
From the Resources folder of this repository get the file called "Texture.png.zip". Unzip it and place it at the root of the Project.

```
    Assets/
    |___Texture.png <- new texture file
    |___White (Material)
    |___Quad/
```

Now double click the "White" material, and drag the texture onto the little square next to the "albedo" (color) property. This sets the surface color of the mesh to be the texture. Also, set the smoothness of the material to be 0. This will make our meshes look less shiny. 

Now open up the Quad.cs script and add the code UVs for UVs of the quad. It is the same as the code we showed in the explanation.

```cs
    Mesh mesh = new Mesh
    {
        vertices = new Vector3[] { new Vector3(-1, -1, 0), new Vector3(-1, 1, 0), new Vector3(1, 1, 0), new Vector3(1, -1, 0) },
        uv = new Vector2[] { new Vector2(0, 0), new Vector2(0, 1), new Vector2(1, 1), new Vector2(1, 0) },
        triangles = new int[] { 0, 1, 2, 0, 2, 3 }
    };
```

Run this new code and you should see a textured square!

![textured quad screenshot](/Assets/textured_quad_screenshot.png)

# Making a voxel in Unity
Voxel terrains are made of the surface of thousands of small voxels. If we just placed a bunch of cubes together to make a terrain you might see something like the following.

![2D voxel terrain unoptimized](/Assets/2D_voxel_terrain_unoptimized.png)

You may notice the inefficiency here. The side of a cube or voxel that is not visible is still being generated and rendered. This will lead to a horrible performance in a large world. 

To fix this we can generate the individual sides of a voxel separately by checking if there is a neighbor voxel next to it, if there is a neighbor we don't generate that side of the voxel.

![2D voxel terrain optimized](/Assets/2D_voxel_terrain_optimized.png)

Add a new "micro project", aka a new folder, to our Unity project, and add the files you see listed.

```
    Assets/
    |___White (Material)
    |___Quad/
    |___Voxel/           <- new folder
    |   |___Voxel.unity  <- new scene
    |   |___Voxel.cs     <- new script called "Voxel"
```

Now open up the Voxel scene and add an empty GameObject. Open up Voxel.cs and force Unity to add the `MeshFilter` and `MeshRenderer` components as before (by using the RequireComponent attribute).

This time instead of hard coding our vertices and triangles we are going to be smart. We will make an array of all 8 possible vertices for a cube, that we can always reference. We'll make a new script to contain all this called DataDefs.cs

```
    Assets/
    |___DataDefs.cs      <- new script
    |___White (Material)
    |___Quad/
    |___Voxel/
```

Now add the `Vertices` array to DataDefs.cs

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

We remove the `: Monobehaviour` from the `DataDefs` class since we won't need this script to be a component of a gameObject. The vertices array (or as we call it "lookup table") has all 8 possible vertices for the corners of a cube or voxel.

To be able to get the vertices per side of a cube, we make a "lookup" table (a 2D array) that gets 4 vertices based on the side of the cube we want.

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

We've made the `BuildOrder` lookup table (for getting the vertices per side) so that the order we get the vertices works with a specific triangles pattern `0, 1, 2, 2, 1, 3`. When you see the phrase "left face" we are talking about the quad that makes up a particular side of the cube.

We mark the class as well as the arrays `static` for a reason. Anything marked as `static` gets saved to a special part of our programs for data that will be accessed a lot and doesn't change. A result of this is there is only one copy of that data and data can be accessed fast. Also, the arrays are marked as `readonly. This tells the compiler that the data can only be read and not modified. The correct word is "not mutated" or "immutable". This allows any Thread or Job (when multithreading later on) to access the arrays since there is only one copy of them and they aren't allowed to be modified. A result of nothing being allowed to change is that the C# Job system won't complain when we use the lookup tables in a Job.

We will start simple and use the lookup tables to make one side of a voxel. For now, open up the Quad.cs code and change so it uses the lookup tables for generating our vertices rather than, as we did before, "hard coding" the vertices ("hard coding" means that the code was written by hand with specific numbers, rather than using a lookup table or helpful function to do it for us).

```cs
    // ... snipping irrelevant code
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
            uv = new Vector2[] { new Vector2(0, 0), new Vector2(0, 1), new Vector2(1, 0), new Vector2(1, 1) },
            triangles = new int[] { 0, 1, 2, 2, 1, 3 }
        };
        mesh.RecalculateBounds();
        mesh.RecalculateNormals();

        gameObject.GetComponent<MeshFilter>().mesh = mesh;
    }
```

We change the triangles to follow the triangle pattern that works with the `BuildOrder` pattern we mentioned before.

The `Vertices` lookup table lets us put in a number to get a specific vertex. The `BuildOrder` table gives us the number to use to put into the `Vertices` table, based on the side of the cube we want. Currently, we are asking for side `0` which is why you see all those `0`'s in the `BuildOrder` table. The `0, 1, 2, 3` gets the vertex for a corner of the face of the cube we want (Remember, when you see the phrase "left face" or "face 0" we are talking about the quad that makes up that particular side of the cube).

Also the uv's change from `(0, 0), (0, 1), (1, 1), (1, 0)` to `(0, 0), (0, 1), (1, 0), (1, 1)` since the positions of the vertices that they refer to have changed.

Now if you run this you should see a side of a cube, aka a quad.

# making a voxel
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

(We use the naming convention `m_variable` to represent private member class variables, this helps to distinguish them from the mesh variable names. We recommend you copy us for an optimal course experience.)

The `m_vertexIndex` is used to keep track of the current newest vertex. That way new vertices being added don't overwrite each other but get added on after the already added vertices. You can visualize this as

```
vertexIndex = 4

                            \/- current vertexIndex
vertices = [v0, v1, v2, v3, null, null, null, null]
                            \__ new vertices to be added
```

The m_triangleIndex serves the same purpose except for the triangles.

Make a new function that uses the lookup tables to make a whole cube. The `side` variable looks up each side of the voxel in the lookup table. Also, a voxel (AKA cube) has 6 sides, so we make a for loop to go through each side in the lookup table.

```cs
public class Voxel : MonoBehaviour
{
    // ... snipping irrelevant code

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

We add the current vertexIndex to the triangles as an offset when setting the triangles to make sure that the triangles are referring to their set of vertices and not the previous set of vertices that are for another side of the voxel.

We do a similar procedure when putting our vertices and triangles *into* the mesh arrays.

```cs
// getting the index for *where* to put vertices in the array
m_vertices[m_vertexIndex + 0]
m_vertices[m_vertexIndex + 1]
m_vertices[m_vertexIndex + 2]
m_vertices[m_vertexIndex + 3]

// getting the index for *where* to put the triangles into the array
m_triangles[m_triangleIndex + 0]
m_triangles[m_triangleIndex + 1]
m_triangles[m_triangleIndex + 2]
m_triangles[m_triangleIndex + 3]
m_triangles[m_triangleIndex + 4]
m_triangles[m_triangleIndex + 5]
```

We can now initialize (aka set the values of) all of the member variables in `Start()`, and Draw the voxel using the `DrawVoxel()` function we just made. 

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

We then set our mesh variables data to the arrays we made, and convert the NativeArray's to a normal C# Array (since the mesh takes in a regular array) using `.ToArray()`. 

We then set the MeshFilters mesh to our `m_mesh` variable. We also make sure to Calculate the normals and Bounds.

You might notice `Allocator.Temp` this is an enum (an enum lets us make a type of "options" system. WHen you make an enum you give the options it can represent). This is telling the Allocator to use a really fast memory fetch, but that memory can only exist for 4 frames (ever heard of frames per second? fps?). 

There are other enums like `Allocator.Persistent` that let us have the memory we want for as long as we need, but they take much longer to find the memory we want, so it's better to use a `Temp` allocator. `Allocator.TempJob` is a special allocator for Job's and multithreading that is based on `Allocator.Temp` but is optimized specifically for multithreading.

Also since we are using NativeCollections we have to manually free our memory, much like in C or C++.

"native" means it uses actual pointers (AKA references) to memory and not copies of everything.

```cs
    private void Start()
    {
        //... snipping out previous code ...//

        m_vertices.Dispose();
        m_triangles.Dispose();
    }
```

C# normally makes copies of everything to make our code "safe", as well as using a "Garbage Collector" (or GC for short). A GC is code in your program automatically added by C# that goes automatically goes through memory and free's memory you are done using.

If you hit play you should have a voxel! (Make sure to set your Material) 

![final_voxel](Assets/final_voxel.png)

See you next week!
