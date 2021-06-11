# Intro
To make the terrain editable we have to store some sort of data. Say we remove a block. If we want to see the change we have to "update" the mesh to show the change. One way is to just redraw the whole chunk's mesh and account for the change. But when we redraw the chunk's mesh we can't still use the Noise for checking where there are or aren't voxels since it will not have acounted for the change we have made.

We can store a 3D array of voxel data using the Noise. Then we when checking if a voxel should be solid we can reference the 3D array, and whenever we want to make a change like removing a voxel we can change the 3D array, and since the is solid depends on the array if we redraw the mesh you will see the mesh missing that voxel.

We could use booleans (true or false) in the array, to represent solid or not solid voxel's, but it is likely we will want many different types of voxel's. So instead we can store a number that can represent all the different voxel types we might want, and then we can use those numbers for a voxelType lookup table. For now we will not make a lookup table but will just have the IsSolid function check if a voxel is of type 0 then it is not solid (esentially the "air" voxel type) otherwise it is solid.

Make a new micro project called "EditableChunk" TODO

```
Assets/
    |___
```

TODO make first person player

TODO once I get to the saving part
We will change the `byte[,,]` array to be inside of a struct called `ChunkData`. <-- not working, since we cannot have a nullable type in a struct in a NativeArray

This is because if we tried to put a `byte[,,]` array into a NativeArray in a Job we will get an error (outside of a Job you won't get any errors) "The type `byte[*,*,*]` must be a non-nullable value type in order to be used as a parameter `T` in the generic type or method `NativeArray<T>`". This just mean's that in a Job a ntive array can't take a "nullable" type". The type may be "null" (not set to anything) because it is an array. This is because arrays are actually always pointers to memory if you have to use the "new" keyword in front, unless marked as static which would make them more like a lookup table (still an array just in a special part of our program where all the const's and statics live).

TODO When deserializing and loading
Also when we Deserialize the file we could try and cast the deserialization to a `byte[,,]` and then do `chunkData.data = (byte[,,])formatter.Deserialize(fileStream);` which would mean we could get rid of the ChunkData struct altogether... except that this won't work and will throw an error. So we will just stick with casting to the ChunkData. Also don't forget that structs can be stored in a NativeArray and be passed to a Job through the NativeArray.

