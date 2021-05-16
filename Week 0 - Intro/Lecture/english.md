# Meshes Concept
If we want to make a voxel engine are going to need to understand what meshes are, since we will be manipulating meshes a lot. 


### vertices
A mesh is really just some information on how to represent a shape. Most meshes sets of three positions to make triangles to represent their shape. This is because triangles can be used to make any other shape. In Unity's mesh format a triangle is made using 3 positions. Actually the "correct" word used for "positions" is Vertices. In unity its just a Vector3. Lets make a triangle using 3 vertices.

(This isn't valid C# code but don't worry once you get the concept you'll take off really fast)

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1 0) }
```

It looks like this in 2D (ignoring z).

![three vertices](/Resources/assets/three_vertices.png)


### triangles ("connecting" our vertices)
Now this won't actually work. In a mesh we need sets of 3 numbers that tell us which vertices to use to make a our triangle. 

```
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0) }
    mesh.triangles = { 0, 1, 2 }
```

(If you think this is pointless... then yes at this point it is. But when you want more than 1 triangle in a mesh it makes sense).

in our list of "triangles" (they're just a set of 3 `int`s) we are saying, get vertex at `0` (computers count from 0 not 1) and make that the first vertex in our triangle. Then get vertex at `1` for the second vertex. Then get vertex at `2` for the third vertex. These "triangle" ints are called indexes. An index tells us where in a list (or array) to find something.

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


### normals (which side of the triangle to render)
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

... and counter clockwise will make it face away from us.

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
Voxel terrains are made of the surface of thousands of small voxels (voxel means 3D pixel). If we were to make a terrain out of straight up voxels and then slice it, it would look like this

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

Now open up the Voxel scene and add an empty GameObject. Open up Voxel.cs and force Unity to add our components as before. Now instead of hard coding our triangle we are going to be smart. We will make an array of all 8 possible vertices for a cube that we can always reference. We'll make a new static class that we can reference called Data.cs

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

Lets now open 