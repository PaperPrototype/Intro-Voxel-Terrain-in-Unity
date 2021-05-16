# Meshes Concept
If we want to make a voxel engine are going to need to understand what meshes are, since we will be manipulating meshes a lot. 

## vertices
A mesh is really just some information on how to represent a shape. Most meshes use triangles to represent their shape. This is because triangles can be used to make any other shape. In Unity's mesh format a triangle is made using 3 positions. Actually the "correct" word used for "positions" is Vertices. In unity its just a Vector3. Lets make a triangle using 3 vertices.

(This isn't valid C# code but for now we're just teaching you the concept)

`
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1 0) }
`

It looks like this in 2D (ignoring z).

![three vertices](/Resources/assets/three_vertices.png)

## triangles ("connecting" our vertices)
Now this won't actually work. In a mesh we need sets of 3 numbers that tell us which vertices to use to make a our triangle. 

`
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0) }
    mesh.triangles = { 0, 1, 2 }
`

(If you think this is pointless... then yes at this point it is. But when you want more than 1 triangle in a mesh it makes sense).

in our list of "triangles" (the're just a set of 3 `int`s) we are saying, get vertex at `0` (computers count from 0 not 1) and make that the first vertex in our triangle. Then get the vertex at `1` for our second vertex. Then get vertex at `2` for our third vertex. These "triangle" ints are called indexes. An index tells us where in a list (or array) to find something.

![triangle mesh](/Resources/assets/triangle_mesh.png)

## a quad / square (two triangles)
To make a square shape (quad in 3D jargon) we will need 1 more vertex. We already have vertices for 3 of the corners of our square.

`
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0), Vector3(1, -1, 0) }
    mesh.triangles = { 0, 1, 2 }
`

![four vertices](/Resources/assets/four_vertices.png)

We need to add another set of 3 ints (referred to as a triangle) to tell Unity which vertices we want to use to make our second triangle.

`
    mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, 1, 0), Vector3(1, -1, 0) }
    mesh.triangles = { 0, 1, 2, 1, 2, 3 }
`
And voila!
![quad mesh](/Resources/assets/quad_mesh.png)

## normals (which side of the triangle to render)
Can these triangle indexes be in any order? Of course! Except that the order we put them in will affect the direction of the normals.

