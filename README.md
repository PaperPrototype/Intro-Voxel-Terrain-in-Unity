Temporary showcase video https://www.youtube.com/watch?v=HvpDE3eM6v4

Planet terrain video https://www.youtube.com/watch?v=G5H7oRlr11s

# Note from author
I am currently not developing this course as I am teaching an Intro to Coding and Computers class (which will become a pre-requisite for this course). I will do more research and finish section 4. Section 5 and onwards will probably become their own courses. Ideas for next courses are shown below (subject to change)
- Voxel Terrain LOD + Planets in Unity (C#)
- Voxel Terrain Multiplayer in Unity (C#)
    - pre-requisite: Intro to Networking for Games
- Voxel Terrain Smoothing + Marching Cubes
- Voxel Terrain Inventory
    - pre-requisite: Simple Inventory

I have to figure out a simple reliable (and forward compatible) solution to storing the terrain data for multiplayer and LOD systems. I want to be extremely simple like a Database store for the terrain. Performance is important, but simplicity of implementation is the most important. Feel free to reach out with solutions you've made. A good resource is Eldermarkkis Marching Cubes project https://github.com/Eldemarkki/Marching-Cubes-Terrain

# Intro to Simplified Voxel Systems in Unity
An open source introductory course on making a voxel terrain system in Unity. For beginners to Unity that already know how to code (what functions, pointers, arrays, and return types are). The goal is to use the simplest code to achieve an easy to understand voxel terrain system that still achieves high performance using the JobSystem.

Join our Discord to get any updates https://discord.gg/Gp7YEUkVHC

View the source code of the courses projects https://github.com/PaperPrototype/VoxelSystem

# Pre-requisites
Coding and understanding of computers. 
Fundamental understanding of Unity.

# Stuck? having Issues?
Join our Discord server to get help https://discord.gg/Gp7YEUkVHC and report issues. I'm on there pretty regularly.

If something is broken we would love it if you created a new "issue" in the Issues tab describing the problem so that we can fix it and future students don't have to go through same thing you did!

# How to take this course
 - A file named "english.md" (or a different language) is the lecture that you need to read. Click on it to open it and start reading.
 - Folders with the word "Week" have a lecture and notes folder (with the lecture and notes in an english.md).
 - Folders named "Tutorial" contains tutorials for when you've finished a lecture and want to try out what you've learned!

# Contributing
If you want to contribute then read our [Guidelines](https://github.com/Nanite3D/Nanite-course-Guidelines) it shouldn't take you more than 1 minute to read. Seriously, it's super short.

# Currently being made
this course is still being written as you read this. Breaking changes may be introduced. Join the Discord for a changelog and one-on-one help from me https://discord.gg/Gp7YEUkVHC

# License
The content of all files ending in ".md" remain the sole property of Abdiel Lopez, unless otherwise specified. The source code snippets in the course are licensed under the MIT license. A complete Unity project containing all the source code can be found here https://github.com/PaperPrototype/Voxel-Terrain-System

# TODO
Make static funtion for savename (also include chunksize in savename) to prevent from loading chunk files with different chunksize.
