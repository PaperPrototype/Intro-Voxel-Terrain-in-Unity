# Authors
 - Abdiel Lopez (github.com/PaperPrototype)

# Intro
This is a question I had a lot so I want to help those who have the same question

# Indexing 1D
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

# Indexing 2D
A 2D array in memory is just a normal array

```
[1][2][3][1][2][3][1][2][3]
```

but the compiler takes care of treating it like this

```
[[1][2][3]] [[1][2][3]] [[1][2][3]]
or
 [1, 2, 3]   [1, 2, 3]   [1, 2, 3]
```

if we think of a 2D array like this

```
level 1 (x) =  [   1   ][   2   ][   3   ]
level 2 (y) =  [1][2][3][1][2][3][1][2][3]
```

then when we index into the first iterator (`x` would be the first iterator) we are just accessing level 1

```
array[x, y]
```

and when we index with the second iterator `y` we are accessing level 2

So if we want to treat a 1D array like a 2D array we have to do math to iterate over it the way the compiler was. Every time `x` is increased by 1 the compiler jumped 3 items.

So if we just multiply `x` by 3, we will jump three items every time we increase `x`.

```
array1D = [1, 2, 3, 1, 2, 3, 1, 2, 3]

array1D[(x * 3) + y]

// is the same as

array2D = [[1, 2, 3], [1, 2, 3], [1, 2, 3]]

array2D[x, y]
```

3D indexing?

```
level 1 (x) =  [            1            ][            2            ][            3            ]
level 2 (y) =  [   1   ][   2   ][   3   ][   1   ][   2   ][   3   ][   1   ][   2   ][   3   ]
level 3 (z) =  [1][2][3][1][2][3][1][2][3][1][2][3][1][2][3][1][2][3][1][2][3][1][2][3][1][2][3]
```

the compiler jumps 3 * 3 (which is 9) times every time level 1 (x) is increased. And the compiler jumps 3 times every time level 2 (y) is increased, and 1 time every time level 3 (z) is increased.

```
array1D[(x * 3 * 3) + (y * 3) + z]

array3D[x, y, z]
```

and so on and so forth.

I hope this tutorial helps any of you out there! The difference between 2D array indexing and doing the 2D indexing ourselves is that either we are doing the math, or the compiler is doing it for us. That said, is doing the indexing ourselves faster? Probably not. And it only makes our code harder to read, which is why I've decided to leave it out of the course as much as possible.
