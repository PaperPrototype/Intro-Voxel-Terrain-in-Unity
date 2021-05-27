# Contributing
If anything is defined incorrectly or you have some knowledge of compiler keywords that was not covered then please make a pull request, and add / fix it.

# Lecture Vocab
Vertex (singular) - Vertices (plural)

    A 3D position used to tell where a mesh's triangle corners will be.

Index (singular) - Indexes? Indices? (plural)

    An index tells us where in a list, or array, we can find something.

Triangle(s)

    A set of 3 numbers telling us which vertices to use to make one corner of a triangle.
    The order that you put these ints affects the direction of the normals.
    [Triangle Mesh Wikipedia](https://en.wikipedia.org/wiki/Triangle_mesh)
    [Polygon Mesh Wikpedia](https://en.wikipedia.org/wiki/Polygon_mesh)

Normal(s)

    The direction that is 90 degrees "upwards" from a surface, for lighting purposes. Also the side of a triangle to render. Each vertex has a normal.
    [Normals Wikpiedia](https://en.wikipedia.org/wiki/Normal_(geometry))
    [Vertex Normal Wikipedia](https://en.wikipedia.org/wiki/Vertex_normal)

UV(s)

    The position on a texture to map to a specific vertex of a triangle. A UV of `Vector2(0, 0)` gets the bottom left of a texture. 
    ![2D uv texture coordinate](/Assets/2D_uv_texture_coordinate.png)

Unit Vector

    A vector who's magnitude is 1. The megnitude of a vector is just the distance from the start, or origin, to the "end" position that the vector is giving.
    ![magnitude of a vector](/Resources/assets/normal_magnitude.png)

Immutable

    A value that can **only** be read and not changed, AKA immutable.

Mutable

    A value that can be changed and read, AKA mutated.

Voxel

    Analogous to the word "pixel", vo standing for "volume" (instead of pixel's "picture") and el representing "element". Similar formations with el for "element" include the words "pixel" and "texel".
    [Voxel Wikipedia](https://en.wikipedia.org/wiki/Voxel)

Pseudo code

    commented code 
    
    ```
        // if key pressed
        // move player
        // else
        // stop player
    ```

    or example code that is for demonstration but wouldn't actually work

    ```
        mesh.uv = { Vector2(0, 0), Vector2(0, 1), Vector2(1, 1) }
    ```

Thread

    Most modern processors have multiple cores. Having multiple cores is that same as having several processors packed in a box and saying the "box" is a processor with many "cores". Each core on a processor has 2 Threads, which just means it can process 2 things at once.

Processor Cores

    A normal processor has 

Processor / Central Processing Unit (AKA CPU)

    A processor takes 0's and 1's and processes them. (I know that doesn't tell you much, but that is essentally what they are). They also have super small super fast memory "caches" for storing information that needs processed, these are SRAM (Static RAM) and are not to be confused with DRAM (Dynamic RAM) which is much slower.

Job 

    A `struct` with an `Execute()` function. The `Execute()` function is called by the JobSystem on a separate thread.

JobSystem

    A system by Unity that does multithreading by scheduling and running Jobs to run on differrent threads.


# Keywords

`static`

    Marks variable as a "global" and stroes it ina special part of our program.

`readonly`

    Marks variables as "immutable". External classes can only read and not write to it.
