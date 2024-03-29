# Overview
- Storing voxel data + building
- Saving voxel data changes to file and loading them again

# Intro
To make the terrain editable we have to store some sort of data. Say we remove a block. If we want to see the change we have to "update" the mesh to show the change. One way is to just redraw the whole chunk's mesh and account for the change. But when we redraw the chunk's mesh we can't still use the Noise for checking where there are or aren't voxels since it will not have acounted for the change that we made to the data.

Instead we will create a 3D array of voxel data and set its values with the Noise. Then when drawing the chunk we reference the data to check if a voxel is solid or not.

Whenever we want to remove a voxel we update the 3D arrays data, and then redraw the mesh. Since the mesh drawing code is referencing the array you will see the change in the resulting new mesh.

We could use booleans (true or false) in the array, to represent solid or not solid voxel's, but it is likely we will want many different types of voxel's. So instead we can store a number that can represent all the different voxel types we might want. Then we can use the voxel type (aka number) in the array as lookup numbers in a `VoxelType` lookup table. 

We will **not** be making a lookup table of different voxel types. For now there is only 2 types of voxels 0 = air (not solid) and 1 = dirt (solid). The `IsSolid` function will check the data array to see if a voxel is solid or not.

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

We chose to use the smallest possible number type to hold our voxel data, a byte. A byte can hold numbers between 0 and 255. That will give us a total of 256 possible voxel types. You might notice that `data` is a 1D array. We will have to learn how to index a 1D array as if it were a 3D array because later on to pass the data to a Job we have to use a `NativeArray`, and to my knowledge it is not possible to create a 3D `NativeArray`.

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

If the y position is greater than the desired height we return 0, which we will say stands for air. Else we return 1 which could be dirt or grass. Currently 1 just stands for a solid voxel.

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

In the IsSolid function we *will* make (later) it will just be doing...

```
    if number = 0 
        return false // air
```

Now we can go ahead and set the chunk's data using the `GetPerlinVoxel()` function.

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

You might notice the `?` in the iterator of the data array. How are we going to fill up a 1D array with 3D data? Well, when we make a 3D array it's actually just linear (1D) in memory, but, the compiler takes care of indexing it as 3D for us. Now we don't have the compiler to help us anymore, so we have to learn how to index it as 3D ourselves. (The following is a cooy/paste of week 2's tutorial on indexing a 1D array as if it was a 3D array).

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

Now go to Chunk2.cs and replace the `?` with our new indexing function.

```cs
        data[Utils.GetIndex(x, y, z)] = GetPerlinVoxel(x, y, z);
```

Finally we can make the IsSolid function.

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

Now, if we are inside of the data array, we get the voxelType (A byte number). If it is 0 we return false (because 0 is air), otherwise we return true saying that the voxel is solid.

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

Now go into Unity and set up a Chunk GameObject (as we have done before). If you hit play you should see a chunk!

# Player
To build and edit the chunks voxel data first we need a player that can walk around, so that we can then build an interaction system with the terrain chunk data. We could also use the scene camera (the one you use without realizing when you edit a scene) as the player.

For this course we are focused on teaching you the concepts you will need, and not on how to implement them in all the different cool ways, so we'll stick with making a player.

Lets make a simple player. Make a new folder called Player and make a PlayerContoller script in it.

```
Assets/
    |___...some files...
    |___Player/
        |___PlayerController.cs
```

Open up the script and we will first work on movement for our Player.

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
            m_rb.velocity += transform.up * jumpVelocity;
        }
    }
}
```

First we force Unity to have the `Rigidbody` component attatched to the gameObject. We make a variable referencer `m_rb` so we can access the `RigidBody` component. In start we set `m_rb` to the `RigidBody` component of the gameObject.

We make the `UpdateMovement` function to hold all of the movement code. In the `UpdateMovement` function we check if the space key is pressed. If it is, we add to the current velocity by using the players current up direction and multiplying it by the jumpVelocity. (This will help make sure the player controller code works with the planet tutorial from last week).

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
Now with a working player we can get to actually editing the chunks data, then redrawing the chunk mesh to simulate building! We could also make a simple destruction system by getting all the collisiion points on the chunks mesh collidrr and then changing the chunks data with those collision points.

We will raycast to the terrain.

![terrain raycast](/Assets/terrain_raycast.png)

The green line represents the raycast. The blue dot is the hit position. The blue line is what is called a normal, this is the same as the normals we talked about in week 0. Whenever we "cast" or shoot a Ray in Unity we can get the hitPoint and hitNormal.

We can subtract half of the normal from the hitPoint to get a positon that is in the center of a voxel.

![hit point to center of voxel](/Assets/hit_point_to_center_of_voxel.png)

Then we can use the voxel position, round it to an `int`, and use it as an index into the array of data in the chunk.

The voxel are `1 x 1 x 1` so rounding the postion to an index is fine. If the cubes were `0.5 x 0.5 x 0.5` then we wuld have to multiply the voxel position we got by 2 to convert it to an index that corresponds to the correct voxel.

Make a new script called `PlayerBuilding` in the micro project.

```cs
using UnityEngine;
using Unity.Mathematics;

public class PlayerBuilding : MonoBehaviour
{
    public Camera cam;
    public Chunk2 chunk;

    private void Update()
    {
        // if mouse if clicked
        if (Input.GetMouseButtonDown(0))
        {
            // store raycast git info
            RaycastHit hit;

            // shoot a ray
            Ray ray = cam.ScreenPointToRay(Input.mousePosition);

            // if raycast            \/ pass hit info back "out" to the hit variable
            if (Physics.Raycast(ray, out hit))
            {
                // get the positon inside of the voxel
                Vector3 desiredPoint = hit.point - (hit.normal / 2);

                // make a rounded positon we can use for indexing the chunk data
                int3 gridIndex = new int3
                    (
                        Mathf.RoundToInt(desiredPoint.x - chunk.gameObject.transform.position.x),
                        Mathf.RoundToInt(desiredPoint.y - chunk.gameObject.transform.position.y),
                        Mathf.RoundToInt(desiredPoint.z - chunk.gameObject.transform.position.z)
                    );

                // change data
                chunk.data[Utils.GetIndex(gridIndex.x, gridIndex.y, gridIndex.z)] = 0;

                // redraw
                chunk.DrawChunk();
            }
        }
    }
}
```

We take in a reference to the chunk and to our camera. In `Update` we check if the primary mouse button (left click) was pressed, if it is we edit the chunks data at the position the ray hit. We have to make sure to convert from a position to something that can be used for indexing the chunk. We then redraw the chunk's mesh.

It is very important that we subtract the chunks gameObject position from the calculating of the gridIndex. This gives us a "local" position that we can then use for indexing into the chunks data.

![chunk data editing](/Assets/) TODO

This is because the chunk may not always be at the origin

Now go into the scene and add this to the Player gameObject. Set the references to the camera and chunk. But you will not see anything being built if you hit play. This is because the raycast is hitting the inside of our players capsule collider and never hitting the terrain. Go to the Player gameObject and set it's layer to "ignore raycast". When it asks if you want to change all the children gameObject as well make sure yu click "yes, change children".

And now we should have Building!

We can have the chunk do the redrawing for us when we change the data by making a function that edits the chunk data based on a a position and then sets a variable `needsDrawn` equal to true. In `Chunk2` add the following.

```cs
// snip
using Unity.Mathematics;

// snip
public class Chunk2 : MonoBehaviour
{
    public byte[] data;
    public bool needsDrawn;

    // snip

    private void Start()
    {
        data = new byte[DataDefs.chunkSize * DataDefs.chunkSize * DataDefs.chunkSize];
        needsDrawn = false;

        // snip
    }

    public void EditChunkData(Vector3 worldPosition, byte voxelType)
    {
        int3 gridIndex = new int3
            (
                Mathf.RoundToInt(worldPosition.x - gameObject.transform.position.x),
                Mathf.RoundToInt(worldPosition.y - gameObject.transform.position.y),
                Mathf.RoundToInt(worldPosition.z - gameObject.transform.position.z)
            );

        data[Utils.GetIndex(gridIndex.x, gridIndex.y, gridIndex.z)] = voxelType;

        needsDrawn = true;
    }
}
```

We also have to add the `using Unity.Mathematics;` because we are using Unity's `int3`.

In update we check if ``needsDrawn` is true, if it is we redraw the chunk and set `needsDrawn` to false.

```cs
public class Chunk2 : MonoBehaviour
{
    // snip
    private void Start()
    {
        // snip
    }

    private void Update()
    {
        if (needsDrawn == true)
        {
            DrawChunk();
            needsDrawn = false;
        }
    }

    // snip
}
```

Now in the `PlayerBuilding` script we can change the code to be

```cs
using UnityEngine;

public class PlayerBuilding : MonoBehaviour
{
    public Camera cam;
    public Chunk2 chunk;

    private void Update()
    {
        if (Input.GetMouseButtonDown(0))
        {
            RaycastHit hit;

            Ray ray = cam.ScreenPointToRay(Input.mousePosition);

            if (Physics.Raycast(ray, out hit))
            {
                Vector3 desiredPoint = hit.point - (hit.normal / 2);

                chunk.EditChunkData(desiredPoint, 0);
            }
        }
    }
}
```

Now we can edit the chunk with ease!

# Chunk Saving
This is all nice and dandy, but once we stop the game and then play again the changes we made to the terrain are gone! Lets add some saving capabilities to our chunk!

But first, how? Well first we need to fiqgure out "where" we should save the data, as well was what the file should be called. Unity gives us a filePath for our games and apps to use that works across every platform. We name a chunk based on its gameObjects position. Add this boiler plate code.

```cs
// snip
using System.IO;
using System.Runtime.Serialization.Formatters.Binary;

public class Chunk2 : MonoBehaviour
{
    // snip

    public void SaveChunk()
    {
        // persistent file path
        string filePath = Application.persistentDataPath + "/chunks/" + gameObject.transform.position + ".chunk";

        // check if folders and directory exist
        if (!File.Exists(filePath))
        {
            // ... if not create the directory
            Directory.CreateDirectory(Path.GetDirectoryName(filePath));
        }
    }
}
```

We check if the direcotry where we want to store the file exists or not, if it doesn't we create the directory.

Now we need to convert our byte array to binary for file saving. We use C#'s BinaryFormatter class for this. Then we need a way to put the data into a file, we do that using a FileStream.

```cs
    public void SaveChunk()
    {
        string filePath = Application.persistentDataPath + "/chunks/" + gameObject.transform.position + ".chunk";

        // check if folders and directory exist
        if (!File.Exists(filePath))
        {
            // ... if not create the directory
            Directory.CreateDirectory(Path.GetDirectoryName(filePath));
        }

        // create formatter and get file access
        BinaryFormatter formatter = new BinaryFormatter();
        FileStream fileStream = File.Open(filePath, FileMode.OpenOrCreate);

        // save the data through the fileStream
        formatter.Serialize(fileStream, data);

        // make sure to close the fileStream!
        fileStream.Close();

        print("the chunk saved to: " + filePath);
    }
```

And then we use the formatter to save our data array to a file through the fileStream. Make sure to close the file streamm or you might end up with weird file leaks! We print a message when we save for debugging purposes.

Add a `needsSaved` variable to the chunk class. We will set `needsSaved` to true in the `EditChunkData` function as well as initializing it to false in start.

```cs
public class Chunk2 : MonoBehaviour
{
    // snip
    public bool needsSaved;

    private void Start()
    {
        // snip
        needsSaved = false;

        // snip
    }

    public void EditChunkData(Vector3 worldPosition, byte voxelType)
    {
        // snip

        needsSaved = true;
    }
```

And then we can use Unity's builtin `OnDisable` method to save our chunk if `needsSaved` is true.

```cs
public class Chunk2 : MonoBehaviour
{
    // snip

    private void OnDisable()
    {
        if (needsSaved == true)
        {
            SaveChunk();
        }
    }

    // snip
}
```

Now if you hit play edit the chunk and then stop the game you should see a message log that the chunk was saved!

But if we click play again the changes don't show up! Now we need to load the chunks data. Lets make a function that will check if a file for the chunk exists, if the file does exist then load it and return true. If the file doesn't exist we return false to let whoever uses the function know that no file exsisted for the chunk and we did not load a file.

```cs
public class Chunk2 : MonoBehaviour
{
    // snip
    public bool LoadChunk()
    {
        string filePath = Application.persistentDataPath + "/chunks/" + gameObject.transform.position + ".chunk";

        // if the file exists
        if (File.Exists(filePath))
        {
            // create formatter and get file access
            BinaryFormatter formatter = new BinaryFormatter();
            FileStream fileStream = File.Open(filePath, FileMode.Open);

            // deserialize the data and set the chunks data to be this data
            //     \______/ <-- cast the deserialize to a byte[]
            data = (byte[])formatter.Deserialize(fileStream);

            // makie sure to close the file stream!
            fileStream.Close();

            print("the chunk loaded from: " + filePath);

            return true;
        }

        // there wasn't a file to load so we return false
        return false;
    }
}
```

We create a `BinaryFormatter` for getting the data through the `FileStream`. When we "deserialize" the data from the file we cast it (convert it) to a byte array `byte[]` by putting `(byte[])` in front of the `formatter.Deserialize(fileStream)`. Then we make sure to close the fileStream! And right before we return we print a log to the console saying we've loaded the chunk (if we put any code after a `return` it won't get run!).

Then the `Start` function changes to try to load the chunk data. If there was a file to load `LoadChunk` returns true, otherwise it returns false (because no file existed for it to load). If `LoadChunk` didn't load any data we run `CalcChunkData` to calc the data ourselves.

```cs
    private void Start()
    {
        // ... snipping irrelevant code ... //
        m_noise.SetNoiseType(FastNoiseLite.NoiseType.OpenSimplex2);

        // if the chunk was not loaded and LoadChunk returned false
        if (LoadChunk() == false)
        {
            // if no chunk data was loaded we need to calc the data ourselves
            CalcChunkData();
        }
        
        // finally we can draw the chunk's mesh
        DrawChunk();
    }
```

And now if you click play, edit some voxels, then stop the game, then play again. The changes should still be there! WHOOP WHOOP!
