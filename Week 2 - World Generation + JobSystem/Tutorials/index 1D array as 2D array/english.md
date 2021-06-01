# Authors
 - Abdiel Lopez (github.com/PaperPrototype)

# Intro
This is a question i had a lot so I want to help those who have the same question

# Indexing 1D
A 1D array in memory looks like this

```
| 1 | 2 | 3 |
```

to index into it we just go through each item.

# Indexing 2D
A 2D array in memory looks like this

```
| 1 | 2 | 3 | 1 | 2 | 3 | 1 | 2 | 3 |
```

but the compiler takes care of treating it like this

```
level 2 =   |_1_|_2_|_3_| |_1_|_2_|_3_| |_1_|_2_|_3_|
level 1 =  1|___________|2|___________|3|___________|
```

when we index into the first iterator, the x would be the first iterator, we are just accessing level 1

```
array[x, y]
```

(! TODO its late and I need to go to bed sorry)