# Voxel types
To have different types of voxels we need a way to represent them. We will use numbers to represent the different voxel types and we will store those numbers in a 3D array. These numbers will reference a lookup table (an array) that has all our different voxel types. Each voxel type can hold any information we want. We could store textures, colors, even prefab gameObjects that should be instantiated instead of a voxel! You could make a car building game! (And you should try this!) For us though we will just be storing colors for our voxels. We will have to pass this data to our Jobs. The colors are vertex colors.