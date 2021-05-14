# In theory
If we want to make a voxel engine are going to need to understand what meshes are, since we will be manipulating meshes a lot. 

A mesh is really just some information on how to represent a shape. Most meshes use triangles to represent every other shape. This is because triangles are the simplest shape. In Unity's mesh format a triangle is made using 3 positions. Actually the word used for meshes is Vertex, but its just a Vector3.

To make a triangle, which is what meshes are made of we need 3 vertices.
`
mesh.vertices = [Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3()]
`