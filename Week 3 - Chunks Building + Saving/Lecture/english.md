# NOTE
This lecture is still being written expect breaking changes

# Overview
- Data + Buildable = Chunk2
- File saving = Chunk2
- CalcDataJob + DrawDataJob = JobChunk2
- SaveDataJob + LoadDataJob = JobChunk2

# Intro
To make the terrain editable we have to store some sort of data. Say we remove a block. If we want to see the change we have to "update" the mesh to show the change. One way is to just redraw the whole chunk's mesh and account for the change. But when we redraw the chunk's mesh we can't still use the Noise for checking where there are or aren't voxels since it will not have acounted for the change that we made to the data.

Instead we will create a 3D array of voxel data and set its values with the Noise. Then when drawing the chunk we reference the data to check if a voxel is solid or not.

Whenever we want to remove a voxel we update the 3D arrays data, and then redraw the mesh. Since the mesh drawing code is referencing the array you will see the change in the resulting mesh.

We could use booleans (true or false) in the array, to represent solid or not solid voxel's, but it is likely we will want many different types of voxel's. So instead we can store a number that can represent all the different voxel types we might want. Then we can use the in the array as lookup numbers in a `VoxelType` lookup table. 

We will **not** be making a lookup table of different voxel types. For now the `IsSolid` function will check the data array to see if a voxel is solid or not.

Make a new micro project called "Chunk2"

```
Assets/
    |___...other files...
    |___Chunk2/
        |___Chunk2.cs
        |___Chunk2.unity
```

Paste in this boiler plate code to the Chunk2 class

```cs
using UnityEngine;
using Unity.Collections;

[RequireComponent(typeof(MeshFilter))]
[RequireComponent(typeof(MeshRenderer))]
public class Chunk2 : MonoBehaviour
{
    public byte[] data;

    private NativeArray<Vector3> m_vertices;
    private NativeArray<int> m_triangles;
    private NativeArray<Vector2> m_uvs;

    private Mesh m_mesh;
    private FastNoiseLite m_noise;

    private int m_vertexIndex;
    private int m_triangleIndex;

    private void Start()
    {
        data = new byte[DataDefs.chunkSize * DataDefs.chunkSize * DataDefs.chunkSize];

        m_noise = new FastNoiseLite();
        m_noise.SetNoiseType(FastNoiseLite.NoiseType.OpenSimplex2);
    }
}

```

We chose to use the smallest possible number type to hold our data, a byte. A byte can hold numbers between 0 and 255. That will give us a total of 256 possible voxel types. You might notice that `data` is a 1D array. We will have to learn how to index a 1D array as if it were a 3D array because later on to pass the data to a Job we have to use a `NativeArray`, and to my knowledge it is not possible to create a 3D `NativeArray`.

Now we need to make a function that we can call that will give us back a byte (number) for setting our voxel data.

```cs
public class Chunk2 : MonoBehaviour
{
    // ... snip ... //

    private byte GetPerlinVoxel(float x, float y, float z)
    {
        return // some number
    }
}
```

As a recap, for the terrain we will get a height value out of the noise, and add the chunks position as an offset

```cs
    private byte GetPerlinVoxel(float x, float y, float z)
    {
        float height = (m_noise.GetNoise(x + gameObject.transform.position.x, z + gameObject.transform.position.z);
    }
```

The noise gives us back values between -1 and 1, we add 1 to get values between 0 and 2. And then we divide it by 2 to get values between 0 and 1.

```cs
    private byte GetPerlinVoxel(float x, float y, float z)
    {
        float height = (m_noise.GetNoise(x + gameObject.transform.position.x, z + gameObject.transform.position.z) + 1) / 2;
    }
```

Now we multiply the noise value to get terrain that is higher than 0 or 1. We chose to multiply it by the `DataDefs.chunkSize` so that the highest terrain we get doesn't go higher than the chunk height.

```cs
    private byte GetPerlinVoxel(float x, float y, float z)
    {
        float height = (m_noise.GetNoise(x + gameObject.transform.position.x, z + gameObject.transform.position.z) + 1) / 2 * DataDefs.chunkSize;
    }
```

Now we can check if the y position is greater than the desired height we return 0, which we will say stands for air. Else we return 1 which could be dirt or grass. Currently 1 just stands for a solid voxel.

```cs
    private byte GetPerlinVoxel(float x, float y, float z)
    {
        float height = (m_noise.GetNoise(x + gameObject.transform.position.x, z + gameObject.transform.position.z) + 1) / 2 * DataDefs.chunkSize;

        if (y >= height)
        {
            return 0; // air
        }
        else
        {
            return 1; // solid (the only "voxelType")
        }
    }
```

In the IsSolid function we *will* make it will just be doing...

```
    if number = 0 
        return false // air
```

Now we can go ahead and set the chunk data using the `GetPerlinVoxel()` function.

```cs
public class Chunk2 : MonoBehaviour
{
    // ... snip ... //

    private void CalcChunkData()
    {
        for (int x = 0; x < DataDefs.chunkSize; x++)
        {
            for (int y = 0; y < DataDefs.chunkSize; y++)
            {
                for (int z = 0; z < DataDefs.chunkSize; z++)
                {
                    data[?] = GetPerlinVoxel(x, y, z);
                }
            }
        }
    }
}
```

You might notice the `?` in the iterator of the data array. How are we going to fill up a 1D array with 3D data? Well, when we make a 3D array it's actually just linear (1D) in memory, but, the compiler takes care of indexing it as 3D for us. Now we don't have the compiler to help us anymore, so we have to learn how to index it as 3D ourselves. (The following is a cpoy/paste of week 2's tutorial).

## Indexing 1D
A 1D array in memory looks like this

```
[1][2][3][4][5][6][7][8][9]
```

to index into it we just go through each item.

```cs
for (x = 0; x < 9; x++)
{
    array1D[x]
}
```

## Indexing 2D
A 2D array **in memory** is just a like normal array

```
[1][2][3][1][2][3][1][2][3]
```

but the compiler has taken care of treating it like this

```
[[1][2][3]] [[1][2][3]] [[1][2][3]]
or
[[1, 2, 3],  [1, 2, 3],  [1, 2, 3]]
```

so we can think of a 2D array like this

```
level 1 (x) =  [   1   ][   2   ][   3   ]
level 2 (y) =  [1][2][3][1][2][3][1][2][3]
```

when we index into the first iterator (`x` would be the first iterator) we are just accessing level 1

```
array[x, y]
```

and when we index with the second iterator `y` we are accessing level 2

So if we want to treat a 1D array like a 2D array we have to do math to iterate over it the way the compiler was. 

Whenever `x` was increased by 1 the compiler jumped 3 items.

So if we just multiply `x` by 3, we will jump three items every time we increase `x`.

```
array1D = [1, 2, 3, 1, 2, 3, 1, 2, 3]

array1D[(x * 3) + y]

// is the same as

array2D = [[1, 2, 3], [1, 2, 3], [1, 2, 3]]

array2D[x, y]
```

## Indexing 3D
3D indexing?

```
level 1 (x) =  [            1            ][            2            ][            3            ]
level 2 (y) =  [   1   ][   2   ][   3   ][   1   ][   2   ][   3   ][   1   ][   2   ][   3   ]
level 3 (z) =  [1][2][3][1][2][3][1][2][3][1][2][3][1][2][3][1][2][3][1][2][3][1][2][3][1][2][3]
```

the compiler jumps 3 * 3 (which is 9) times every time level 1 (x) is increased. And the compiler jumps 3 times every time level 2 (y) is increased, and it jumps 1 time every time level 3 (z) is increased.

```
array1D[(x * 3 * 3) + (y * 3) + z]

array3D[x, y, z]
```

So if we want to index into our chunks data array we can make a function that takes in `x` `y` and `z`, does the math, and then gives us back the number for 1D indexing.

Make a new script in the root of our project called Utils.cs

```
Assets/
    |___...other files...
    |___Utils.cs
```

Change it to be a static class and add the following function.

```cs
using UnityEngine;

public static class Utils
{
    public static int GetIndex(int x, int y, int z)
    {
        return (x * DataDefs.chunkSize * DataDefs.chunkSize) + (y * DataDefs.chunkSize) + z;
    }
}
```

The `GetIndex` function takes in 3 indexes that we would normally use for a 3D array. It multiplies these numbers appropriately and gives us a number that we can use to index into our data array.

Now go to Chunk2.cs and change the data arrays indexing to.

```cs
        data[Utils.GetIndex(x, y, z)] = GetPerlinVoxel(x, y, z);
```

Now  we can make the IsSolid function.

```cs
public class Chunk2 : MonoBehaviour
{
    // ... snip ... //

    private bool IsSolid(int x, int y, int z)
    {
        // if inside bounds of data
        if (x >= 0 && x < DataDefs.chunkSize &&
            y >= 0 && y < DataDefs.chunkSize &&
            z >= 0 && z < DataDefs.chunkSize)
        {

        }
        else
        {
            // this is where we would check for neighbor chunks
            return false;
        }

    }
}
```

We first make sure that we aren't trying to get an index that is outside of the array. If the index is outside of the array we would check neighbor chunks for their voxel data.

Then if we are inside of the data array we get the voxelType (A byte). If it is 0 we return false (because 0 is air), otherwise we return true saying that the voxel is solid.

```cs
    private bool IsSolid(int x, int y, int z)
    {
        // if inside bounds of data
        if (x >= 0 && x < DataDefs.chunkSize &&
            y >= 0 && y < DataDefs.chunkSize &&
            z >= 0 && z < DataDefs.chunkSize)
        {
            byte voxelType = data[Utils.GetIndex(x, y, z)];

            if (voxelType == 0) return false;
            else return true;
        }
        else
        {
            // this is where we would check for neighbor chunks
            return false;
        }

    }
```

Now we can make our DrawChunk and DrawVoxel functions which don't change at all from week 1's chunk code

```cs
public class Chunk2 : MonoBehaviour
{
    // ... snip ... //

    public void DrawChunk()
    {
        m_vertexIndex = 0;
        m_triangleIndex = 0;

        m_vertices = new NativeArray<Vector3>(24 * DataDefs.chunkSize * DataDefs.chunkSize * DataDefs.chunkSize / 2, Allocator.Temp);
        m_triangles = new NativeArray<int>(36 * DataDefs.chunkSize * DataDefs.chunkSize * DataDefs.chunkSize / 2, Allocator.Temp);
        m_uvs = new NativeArray<Vector2>(24 * DataDefs.chunkSize * DataDefs.chunkSize * DataDefs.chunkSize / 2, Allocator.Temp);

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

        m_mesh = new Mesh
        {
            vertices = m_vertices.Slice<Vector3>(0, m_vertexIndex).ToArray(),
            triangles = m_triangles.Slice<int>(0, m_triangleIndex).ToArray(),
            uv = m_uvs.Slice<Vector2>(0, m_vertexIndex).ToArray()
        };

        m_mesh.RecalculateBounds();
        m_mesh.RecalculateNormals();

        gameObject.GetComponent<MeshFilter>().mesh = m_mesh;

        m_vertices.Dispose();
        m_triangles.Dispose();
        m_uvs.Dispose();
    }


    private void DrawVoxel(int x, int y, int z)
    {
        for (int side = 0; side < 6; side++)
        {
            if (!IsSolid(DataDefs.NeighborOffset[side].x + x, DataDefs.NeighborOffset[side].y + y, DataDefs.NeighborOffset[side].z + z))
            {
                Vector3 pos = new Vector3(x, y, z);

                m_vertices[m_vertexIndex + 0] = DataDefs.Vertices[DataDefs.BuildOrder[side, 0]] + pos;
                m_vertices[m_vertexIndex + 1] = DataDefs.Vertices[DataDefs.BuildOrder[side, 1]] + pos;
                m_vertices[m_vertexIndex + 2] = DataDefs.Vertices[DataDefs.BuildOrder[side, 2]] + pos;
                m_vertices[m_vertexIndex + 3] = DataDefs.Vertices[DataDefs.BuildOrder[side, 3]] + pos;

                m_triangles[m_triangleIndex + 0] = m_vertexIndex + 0;
                m_triangles[m_triangleIndex + 1] = m_vertexIndex + 1;
                m_triangles[m_triangleIndex + 2] = m_vertexIndex + 2;
                m_triangles[m_triangleIndex + 3] = m_vertexIndex + 2;
                m_triangles[m_triangleIndex + 4] = m_vertexIndex + 1;
                m_triangles[m_triangleIndex + 5] = m_vertexIndex + 3;

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

    // ... snip ... //
}
```

Now we can change `Start()` to calculate the chunk data and draw the chunk

```cs
    private void Start()
    {
        // ... snip ... //

        CalcChunkData();
        DrawChunk();
    }
```

Now go into Unity and set up a Chunk GameObject as we have done before. If you hit play you should see a chunk!

# Player
To build and edit the chunks voxel data first we need a player that can walk around, so that we can then build an interaction system with the terrain. We could also do this through the scene editor rather than using an in game player. 

For this course we are focused on teaching you the concepts you will need, and not on how to implement them in all the different cool ways.

Lets make a simple player. Make a new folder called Player and make a PlayerContoller script in it.

```
Assets/
    |___...some files...
    |___Player/
        |___PlayerController.cs
```

Open up the script and we will first work on movement.

```cs
using UnityEngine;

[RequireComponent(typeof(Rigidbody))]
[RequireComponent(typeof(CapsuleCollider))]
public class PlayerController : MonoBehaviour
{

    // movement
    public float jumpVelocity = 6;

    private Rigidbody m_rb;

    private void Start()
    {
        m_rb = gameObject.GetComponent<Rigidbody>();
    }

    private void Update()
    {
        UpdateMovement();
    }

    private void UpdateMovement()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            m_rb.velocity += new Vector3(0, jumpVelocity, 0);
        }
    }
}
```

First we force Unity to have the `Rigidbody` component attatched to the gameObject. We make a variable reference `m_rb` so we can access the `RigidBody` component. In start we set `m_rb` to the `RigidBody` component of the gameObject.

We make the `UpdateMovement` function to hold all of the movement code. In the `UpdateMovement` function we check if the space key is pressed. If it is, we increase the y velocity by the `jumpVelocity` variable, this gives us nice yet simple and clean jumping.

Open the `Chunk2` scene and create a capsule gameObject primitve.

![add capsule primitive](/Assets/add_capsule_primitive.png)

Change its name to "Player" and add the `PlayerController` component to the Player.

Make sure to move the player up in the y so that it isn't inside of the chunk. But if you hit play the player will fall through the chunk! Change Chunk2.cs to require a MeshCollider component so that the player doesn't fall through the chunk. A MeshCollider takes in a mesh and makes a collider that works for it. Set the `MeshColliders` sharedMesh to our mesh.

```cs
// ... snip ... //
[RequireComponent(typeof(MeshCollider))]
public class Chunk2 : MonoBehaviour
{
    // ... snip ... //

    public void DrawChunk()
    {
        // ... snip ... //
        
        gameObject.GetComponent<MeshFilter>().mesh = m_mesh;
        gameObject.GetComponent<MeshCollider>().sharedMesh = m_mesh; // set

        // ... snip ... //
    }
}
```

You may need to add the MeshCollider component yourself since Unity might not register that you added `[RequireComponent(typeof(MeshCollider))]`

Now if you hit play the player should be able to jump on the chunk! 

The direction that the Heads forward vector (z positive) is "looking" will be used for the movment code, so we need to get the players look rotation working before we continue.

Rather than having to figure out weird rotation stuff, lets nest some gameObjects so that the transform system does the difficult rotation math for us. Make a new gameObject under the Player gameObject and call it "Head". Position the Head where you want the viwer to see from. Drag the camera onto the Head gameObject so that the camera is a child of the head. Reset the camera's position so that it is the same as the Heads position.

!(player controller hierarchy)[/Assets/player_controller_hierarchy.png]

Change the script to take in the Heads transform and the Cameras transform.

```cs
public class PlayerController : MonoBehaviour
{
    // ... snip ... //

    // rotation
    public float lookSpeed = 100;
    public Transform head;
    public Transform cam;

    // ... snip ... //

    private void Update() 
    {
        // ... snip ... //

        if (Input.GetMouseButton(1))
        {
            UpdateLook();
        }
    }

    private void UpdateLook()
    {
        float rotateSpeedy = Input.GetAxis("Mouse X") * lookSpeed * Time.deltaTime;
        float rotateSpeedx = -Input.GetAxis("Mouse Y") * lookSpeed * Time.deltaTime;
    }
}
```

We calculate the speed of rotation by getting a input from the mouse. We multiply the input by deltaTime to make sure the rotation stays the same speed no matter what the fps (frames per second) is.

We check if the second mouse button is being pressed (right click), and if it is, it activates player look. This will let the player click anywhere on the screen to remove / place blocks.

We apply the y rotation (right/left) to the Head, and then we apply the x rotation (up/down) to the cam, which is a child of the Head. This is important because by parenting the camera under the Head when we rotate the head in the y the camera also gets rotated. Then when we rotate the camera to look up and down 

```cs
    private void UpdateLook()
    {
        float rotateSpeedy = Input.GetAxis("Mouse X") * lookSpeed * Time.deltaTime;
        float rotateSpeedx = -Input.GetAxis("Mouse Y") * lookSpeed * Time.deltaTime;
    
        head.Rotate(Vector3.up, rotateSpeedy);
        cam.Rotate(Vector3.right, rotateSpeedx);
    }
```

Now we can go back to our movement code.

```cs
public class PlayerController : MonoBehaviour
{
    // movement
    public float jumpVelocity = 6;
    public float moveSpeed = 2;

    // ... snip ... //

    private void UpdateMovement()
    {
        // ... snip ... //

        float forwardSpeed = Input.GetAxis("Vertical") * moveSpeed * Time.deltaTime;
        float sideSpeed = Input.GetAxis("Horizontal") * moveSpeed * Time.deltaTime;
    }
}
```

We add a movement speed variable. Then calculate a forward speed by getting input from the Vertical axis (W-S and UP-DOWN keys). We do the same for the side speed. Now lets apply those to the player.

```cs
    private void UpdateMovement()
    {
        // ... snip ... //

        transform.position += head.forward * forwardSpeed; // forward
        transform.position += head.right * sideSpeed; // sideways
    }
```

Now if we go back into Unity we should see movement and look working. First you need to assign the Head and Cam transforms in the Inspector. Also in the Player gameObject we need to freeze the rotation in the x, y and z, since we will be doing all the look rotation throught the Head and Cam. Set the Players RigidBody constraints to the following.

![player rigidbody constraints](/Assets/player_rigidbody_constraints.png)

If you want you can prevent the `PlayerController` from always controlling the position of the player and only taking control when we give it input.

```cs
    private void Update()
    {
        if (Input.anyKey)
        {
            UpdateMovement();

            if (Input.GetMouseButton(1))
            {
                UpdateLook();
            }
        }
    }
```

# Building
Now with a working player we can get to actually editing the chunks data, then redrawing the chunk mesh, to simulate building! We could also make a simple destruction system by getting all the collisiion points and then changing the chunks data with those collision points.

Make a new script called PlayerBuilding 

NOTE: If your reading this I am still writing this lecture

TODO once I get to the saving part
We will change the `byte[,,]` array to be inside of a struct called `ChunkData`. <-- not working, since we cannot wrap a nullable type in a struct in a NativeArray

This is because if we tried to put a `byte[,,]` array into a NativeArray in a Job we will get an error (outside of a Job you won't get any errors) "The type `byte[*,*,*]` must be a non-nullable value type in order to be used as a parameter `T` in the generic type or method `NativeArray<T>`". This just mean's that in a Job a ntive array can't take a "nullable" type". The type may be "null" (not set to anything) because it is an array. This is because arrays are actually always pointers to memory unless marked as static which would make them more like a lookup table and not a pointer to some memory (it would end up in a special part of our program where all the const's, statics, and globals live).

TODO When deserializing and loading
Also when we Deserialize the file we could try and cast the deserialization to a `byte[,,]` and then do `chunkData.data = (byte[,,])formatter.Deserialize(fileStream);` which would mean we could get rid of the ChunkData struct altogether... except that this won't work and will throw an error. So we will just stick with casting to the ChunkData. Also don't forget that structs can be stored in a NativeArray and be passed to a Job through the NativeArray.

