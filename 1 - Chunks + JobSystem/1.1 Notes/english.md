# Definitions
Noise
    Perlin noise, Complex Noise, Fractal Brownian motion, // TODO

NativeArray
    An array type provided by Unity, under the namespace `Unity.Collections`. It allows us to access the same space in memory much like pointers in other languages. Its most common use is for the C# JobSystem where accessing the same memery that a Job has done work on would be difficult without using a NativeArray.

Interface
    A set of functions some class or struct must implement

# Keywords
`const`
    Sets a number to not be change-able. Arrays require to be initialized with a const.