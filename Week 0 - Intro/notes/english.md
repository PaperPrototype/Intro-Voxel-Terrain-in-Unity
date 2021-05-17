# Contributing
If anything is defined incorrectly or you have some knowledge of compiler keywords that was not covered then please make a pull request, and add / fix it.

# Vocab
Vertex (singular) - Vertices (plural)
    A 3D position used to tell where a mesh's triangle corners will be.
    
Index (singular) - Indexes? Indices? (plural)
    An index tells us where in a list, or array, we can find something.

Triangle(s)
    A set of 3 indexes telling us which vertices to use to make our triangle.
    The order that you put these ints affects the direction of the normals.
    [Triangle Mesh Wikipedia](https://en.wikipedia.org/wiki/Triangle_mesh)
    [Polygon Mesh Wikpedia](https://en.wikipedia.org/wiki/Polygon_mesh)

Normal(s)
    The direction that is 90 degrees "upwards" from a surface, for lighting purposes. Also the side of a triangle to render. Each vertex has a normal.
    [Normals Wikpiedia](https://en.wikipedia.org/wiki/Normal_(geometry))
    [Vertex Normal Wikipedia](https://en.wikipedia.org/wiki/Vertex_normal)

Unit Vector
    A vector who's magnitude is 1. The megnitude of a vector is just the distance from the start, or origin, to the "end" position that the vector is giving.
    ![magnitude of a vector](/Resources/assets/normal_magnitude.png)

Immutable
    A value that can **only** be read and not changed, AKA immutable.

Mutable
    A value that can be changed and read, AKA mutated.

Keyword - `static`
    Marks variable as a "global" and stroes it ina special part of our program.

Keyword - `readonly`
    Marks variables as "immutable". External classes can only read and not write to it.

Voxel
    Analogous to the word "pixel", vo standing for "volume" (instead of pixel's "picture") and el representing "element". Similar formations with el for "element" include the words "pixel" and "texel".
    [Voxel Wikipedia](https://en.wikipedia.org/wiki/Voxel)