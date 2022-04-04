# Future courses
Do more research and finish section 4. Section 5 and onwards will probably become their own courses. Ideas for next courses are shown below (subject to change)
- Voxel Terrain LOD + Planets in Unity (C#)
- Voxel Terrain Multiplayer in Unity (C#)
    - pre-requisite: Intro to Networking for Games
- Voxel Terrain Smoothing + Marching Cubes
- Voxel Terrain Inventory
    - pre-requisite: Simple Inventory

World building + Saving
 - World DataStorage + DataCalc + DataDraw = JobWorld2
 - World DataEdit = Buildable World
 - World DataSave + Load = Persistent world

if we change the chunkSize then all the files will be broken (which we want), so include chunkSize in filename.

Figure out a simple reliable (and forward compatible) solution to storing the terrain data for multiplayer and LOD systems. Be extremely simple like a Database store for the terrain. Performance is important, but simplicity of implementation is the most important. Feel free to reach out with solutions you've made.

# Resources
Eldermarkkis Marching Cubes project https://github.com/Eldemarkki/Marching-Cubes-Terrain
