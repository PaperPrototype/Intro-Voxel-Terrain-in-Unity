# Meshes Concept
If we want to make a voxel engine, we are going to need to understand what meshes are, since we will be manipulating meshes a lot.


### Vertices
A mesh is really just some information on how to represent a shape. Most meshes are sets of three positions in space to make triangles to represent their shape. This is because triangles can be used to make any other shape. In Unity's mesh format a triangle is made using 3 positions. The "correct" word used for "positions in space" is vertices (singular is vertex). In Unity a vertex is just a Vector3. Let's make a triangle using 3 vertices.

(This isn't valid C# code but don't worry once you get the concept you'll take off really fast)

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1 0) }
```

It looks like this in 2D (ignoring z).

![three vertices](/Resources/assets/three_vertices.png)


### Triangles ("connecting" our vertices)
Now this won't actually work. In a mesh we need sets of 3 numbers that tell us which vertices to use to make a our triangle. 

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0) }
    mesh.triangles = { 0, 1, 2 }
```

(If you think this is pointless... then yes at this point it is. But when you want more than 1 triangle in a mesh it makes sense).

In our list of "triangles" (they're just a set of 3 `int`s) we are saying, get vertex at `0` (computers count from 0 not 1) and make that the first vertex in our triangle. Then get vertex at `1` for the second vertex. Then get vertex at `2` for the third vertex. These "triangle" ints are called indexes. An index tells us where in a list (or array) to find something.

![triangle mesh](/Resources/assets/triangle_mesh.png)


###  a quad / square (two triangles)
To make a square shape (a "quad" in 3D jargon) we're going to add 1 more vertex so we have 4 vertices in position to make a square.
```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0), Vector3(1, -1, 0) }
    mesh.triangles = { 0, 1, 2 }
```

![four vertices](/Resources/assets/four_vertices.png)

We need to add another set of 3 ints (referred to as a triangle) to tell Unity which vertices to make the second triangle.

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0), Vector3(1, -1, 0) }
    mesh.triangles = { 0, 1, 2, 0, 2, 3 }
```

And voila!
![quad mesh](/Resources/assets/quad_mesh.png)


### Normals (which side of the triangle to render)
We need a "direction" to tell Unity which side of the quad is up. These are called Normals. Normals tell the exact direction that is "upwards" from a surface or triangle. Here are normals telling us which direction is "upwards" on our triangle. (we are looking at it from the top = y).

![correct surface normal](/Resources/assets/correct_normal.png)

But what if the normal faces in a direction that is not 90 degrees angled from the surface? The renderer doesn't know anything, so it will just shade the surface's color as if it was at the angle the normal is saying its at.

![incorrrect surface normal](/Resources/assets/incorrect_normal.png)

This could let us make surfaces, that are made of many triangles, that are not smooth look as if they were smooth.

The format for normals in a Unity mesh is each vertex needs a normal. We will assume z+ is facing toward us.

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0), Vector3(1, -1, 0) }
    mesh.normals = { Vector3(0, 0, 1), Vector3(0, 0, 1), Vector3(0, 0, 1), Vector3(0, 0, 1) }
    mesh.triangles = { 0, 1, 2, 0, 2, 3 }
```

Normals are usually a Unit Vector. Unit Vector just means that the distance from the origin (`Vector3(0, 0, 0)`) to the "position" or "end" of our Normal needs to be 1. The distance from `Vector3(0, 0, 0)` to one of our normals `Vector3(0, 0, 1)` is obviously 1, so we're fine. 

![magnitude of normal](/Resources/assets/normal_magnitude.png)

The distance of a vector is called it's Magnitude. Also Normals are always just giving us a direction, so they don't need to be positioned near the vertices.

Unity has a built in function for generating normals for us.

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0), Vector3(1, -1, 0) }
    mesh.triangles = { 0, 1, 2, 0, 2, 3 }
    mesh.RecalculateNormals()
```

`RecalculateNormals()` requires us to already have our `vertices` and `triangles` set. But there is one other thing. If we are using Unity's function it needs to know which side of the triangle/quad to render. This is because Unity only renders 1 side of a mesh. This is called backface culling. Unity uses the order we put the triangles in to determine which side to render. The way we can remember is clockwise order will make the normals face us

(taken from https://forum.unity.com/threads/unity-has-a-clockwise-winding-order.129923/#post-3198466)
![clockwise triangle order](/Resources/assets/clockwise_triangle.png)

... and counter-clockwise will make it face away from us.

![counter clockwise triangle](/Resources/assets/counter_clockwise_triangle.png)


# Making the quad in Unity
We will be making lots of micro Unity projects to test out what we learn. Make a new Unity project. Set it to 3D. Set it up like the diagram below with a folder for this micro project called Quad.

```
    Assets/
    |___Quad/
    |   |___Quad.unity (Scene)
    |   |___Quad.cs (Script)
```

In the scene make an empty GameObject. Now open the script in VS Code or whatever you're using to code. We are going to want a MeshRenderer component and a MeshFilter component. Lets force Unity to have these on our GameObject. Above the Quad class add

```
    [RequireComponent(typeof(MeshRenderer))]
    [RequireComponent(typeof(MeshFilter))]
```

Now go back to the scene and add the Quad script to the empty GameObject. It should automatically also add the needed components. Or you can just manually add them, but if you click play and it doesn't have them Unity will warn you.

Now we make our mesh as before. In `Start()` add

```
    Mesh mesh = new Mesh
    {
        vertices = new Vector3[] { new Vector3(-1, -1, 0), new Vector3(-1, 1, 0), new Vector3(1, 1, 0), new Vector3(1, -1, 0) },
        triangles = new int[] { 0, 1, 2, 0, 2, 3 }
    };
    mesh.RecalculateBounds();
    mesh.RecalculateNormals();
```

Also add a `RecalculateBounds()` this tells the renderer what 3D space the mesh takes up as if it were enclosed in a Box. (A box is easy to work with and very performant). Now add the mesh to our MeshFilter in `Start()`

```
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


# Making a voxel in Unity
Voxel terrains are made of the surface of thousands of small voxels. Voxel means 3D pixel. If we were to make a terrain out of voxels and then slice it, it would look like this

![2D voxel terrain unoptimized]()

You may notice the inefficiency here. The side of a voxel that is not visible is still being rendered. This leads to horrible performance in a large world. To fix this we can generate the individual sides of a voxel separately and then check if the voxel has a neighbor, if it does then we just don't generate that side.

![2D voxel terrain optimized]()

Add a new micro project to our Unity project so that looks like this

```
    Assets/
    |___White (Material)
    |___Quad/
    |___Voxel/
    |   |___Voxel.unity
    |   |___Voxel.cs
```

Now open up the Voxel scene and add an empty GameObject. Open up Voxel.cs and force Unity to add our components as before. Now instead of hard coding our triangle we are going to be smart. We will make an array of all 8 possible vertices for a cube that we can always reference. We'll make a new script that we can reference called Data.cs

```
    Assets/
    |___Data.cs
    |___White (Material)
    |___Quad/
    |___Voxel/
```

Change Data.cs to the following

```
public static class Data
{
    public static readonly Vector3[] Vertices = new Vector3[8]
    {
        new Vector3(0.0f, 0.0f, 0.0f),
        new Vector3(1.0f, 0.0f, 0.0f),
        new Vector3(1.0f, 1.0f, 0.0f),
        new Vector3(0.0f, 1.0f, 0.0f),
        new Vector3(0.0f, 0.0f, 1.0f),
        new Vector3(1.0f, 0.0f, 1.0f),
        new Vector3(1.0f, 1.0f, 1.0f),
        new Vector3(0.0f, 1.0f, 1.0f)
    };
}
```

This is all 8 possible vertices for the corners of a voxel. We need a way to find all the vertices per side of the voxel. For this we'll make a lookup table (a 2D array) that gets 4 vertices based on the side of the cube we want.

```
    public static readonly int[,] BuildOrder = new int[6, 4]
    {
        // right, left, up, down, front, back

        //0 1 2 2 1 3
        
        {1, 2, 5, 6},  // right face
        {4, 7, 0, 3}, // left face
        
        {3, 7, 2, 6}, // up face
        {1, 5, 0, 4}, // down face
        
        {5, 6, 4, 7}, // front face
        {0, 3, 1, 2} // back face
    };
```

We mark the class as well as the arrays static for a reason. Anything marked as static gets saved to a special part of our programs for data that doesn't change and therefore there is only one copy of that data. This data can be accessed really fast. Also the arrays are marked as readonly. This tells the compiler that the data can only be read and not modified. The correct word is "mutated". This allows any Thread or Job to access the arrays since there is only one copy of them and they aren't allowed to be modified. We can be sure that nothing weird will happen, and the C# Job system won't complain when we use Jobs to make our meshes.

Open Voxel.cs again and create a new mesh except we are going to use our lookup tables. We are going to use a NativeArray to store the vertices and triangles. This is to get you used to using them since they are JobSystem friendly and you will need to know how to use them. 

```
using UnityEngine;
using Unity.Collections;

[RequireComponent(typeof(MeshRenderer))]
[RequireComponent(typeof(MeshFilter))]
public class Voxel : MonoBehaviour
{
    private Mesh m_mesh;
    private NativeArray<Vector3> m_vertices;
    private NativeArray<int> m_triangles;
    private int m_vertexIndex = 0;
    private int m_triangleIndex = 0;

}
```

(We use the naming convention `m_memberVariable` to represent private class member variables. You don't have to, but it'll make distinguishing things apart from each other easier later on. We recommend you copy us for an optimal course experience).

The m_vertexIndex is used to keep track of the current newest vertex. That way new vertices being added don't overwrite eachother, but get added on the end of the array. The m_triangleIndex serves the same purpose except for the triangles.

Make a new function that uses the lookup tables to make a voxel.

```
    private void DrawVoxel()
    {
        for (int side = 0; side < 6; side++)
        {
            m_vertices[m_vertexIndex + 0] = Data.Vertices[Data.BuildOrder[side, 0]];
            m_vertices[m_vertexIndex + 1] = Data.Vertices[Data.BuildOrder[side, 1]];
            m_vertices[m_vertexIndex + 2] = Data.Vertices[Data.BuildOrder[side, 2]];
            m_vertices[m_vertexIndex + 3] = Data.Vertices[Data.BuildOrder[side, 3]];

            // get the correct triangle index
            m_triangles[m_triangleIndex + 0] = m_vertexIndex + 0;
            m_triangles[m_triangleIndex + 1] = m_vertexIndex + 1;
            m_triangles[m_triangleIndex + 2] = m_vertexIndex + 2;
            m_triangles[m_triangleIndex + 3] = m_vertexIndex + 2;
            m_triangles[m_triangleIndex + 4] = m_vertexIndex + 1;
            m_triangles[m_triangleIndex + 5] = m_vertexIndex + 3;

            // increment by 4 because we only added 4 vertices
            m_vertexIndex += 4;

            // increment by 6 because we added 6 int's to our triangles array
            m_triangleIndex += 6;
        }
    }
```

And now initialize all of our member variables in `Start()` and Draw the voxel. Then set the meshes data. Then set the MeshFilters mesh to our mesh. Also Calculate our normals and Bounds.

```
    private void Start()
    {
        m_vertices = new NativeArray<Vector3>(24, Allocator.Temp);
        m_triangles = new NativeArray<int>(36, Allocator.Temp);

        DrawVoxel();

        m_mesh = new Mesh
        {
            vertices = m_vertices.ToArray(),
            triangles = m_triangles.ToArray()
        };

        m_mesh.RecalculateBounds();
        m_mesh.RecalculateNormals();

        gameObject.GetComponent<MeshFilter>().mesh = m_mesh;
    }
```

Also since we are using NativeCollections (Native meaning it uses actual pointers and not copies of everything, which is what C# normaly does to make everything "safe") we have to free our memory Manually much like in C or Rust.

```
    private void Start()
    {
        //... snipping out previous code ...//

        m_vertices.Dispose();
        m_triangles.Dispose();
    }
```

And now if you hit play you should have a voxel, but make sure to set your Material. See you next week! (Or in an hour if you want, but seriously take a 5 min break).