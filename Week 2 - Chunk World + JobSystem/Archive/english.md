Before we were rounding the 1's place for checking if the player had passed a cube. Since the cubes were only 1 x 1 x 1. But now we have to round to the 16's place. 

To do this we need to scale down from 16 (assuming `Data.chunkSize` is 16) to the one's place, so that we can then round the 1's place numbers, and multiply those by 16 (assuming that `Data.chunkSize` is 16) to get back only multiples of 16 (eg. 32, 64, 128, 512).

We can convert the player's positon to the one's place by dividing the players position by 16. As an example say the player is at 33.5 in the `x`, if we divide 33.5 by 16 we get 2.09375, if you then round this you get 2. Now we can multiply 2 by 16 and get back 32, a multiple of 16!

We can now use the rounded numbers from `GetRoundedPos()` to check if the player has passed a chunk.


```cs
    private Vector3 GetRoundedPos()
    {
        return new Vector3(Mathf.Round(center.position.x / Data.chunkSize), 0, Mathf.Round(center.position.z / Data.chunkSize)) * Data.chunkSize;
    }
```