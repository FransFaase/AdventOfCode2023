# Day 17 of Advent of Code 2023

I woke up at 5:52, stayed a bit in bed and decided to get up and give
it a try. At 6:02, I opened my laptop and started reading.
At 6:12 (CET), lets first determine the size:

```c
int main(int argc, char *argv[])
{
    read_file("input/day17.txt");
    solve1();
}

void solve1()
{
    int nr_col = strlen(lines[0]);
    printf("%d %d\n", nr_lines, nr_col);
}
```

At 6:16, the size is 141 in both directions.

```c

/*
#define SIZE
num_t left[SIZE][SIZE];
num_t right[SIZE][SIZE];
num_t down[SIZE][SIZE];
num_t up[SIZE][SIZE];

void solve1()
{
    int nr_col = strlen(lines[0]);

    num_t max = 1;
    for (int i = 0; i < SIZE; i++)
        for (int j = 0; j < SIZE: j++)
            max += lines[i][j] - '0';
    
    for (int i = 0; i < SIZE; i++)
        for (int j = 0; j < SIZE: j++)
            left[i][j] = right[i][j] = down[i][j] = up[i][j] = max;
    
    left[0][0] = 0;
    down[0][0] = 0;
    
    for (bool more = TRUE; more; )
    {
        more = FALSE;
        
        for (int i = 0; i < SIZE; i++)
            for (int j = 0; j < SIZE; j++)
            {
                int sum = lines[i][j] - '0';
                for (int s = 1; s <= 3 && j - s >= 0; s++)
                {
                    int val = sum + up[i][j - s];
                    if (val < left[i][j]) left[i][j] = val;
                    
                    val = sum + down[i][j - s];
                    if (val < left[i][j]) left[i][j] = val;

                    sum += lines[i][j - s] - '0';
        
}
*/
```

At 6:31, I realize that I only need horzontal and vertical.

```c

#define SIZE 141
num_t horz[SIZE][SIZE];
num_t vert[SIZE][SIZE];

void solve1()
{
    int nr_col = strlen(lines[0]);

    num_t max = 1;
    for (int i = 0; i < SIZE; i++)
        for (int j = 0; j < SIZE; j++)
            max += lines[i][j] - '0';
    
    for (int i = 0; i < SIZE; i++)
        for (int j = 0; j < SIZE; j++)
            horz[i][j] = vert[i][j] = max;
    
    horz[0][0] = 0;
    vert[0][0] = 0;
    
    for (bool more = TRUE; more; )
    {
        more = FALSE;
        
        for (int i = 0; i < SIZE; i++)
            for (int j = 0; j < SIZE; j++)
            {
                int sum = lines[i][j] - '0';
                for (int s = 1; s <= 3 && j - s >= 0; s++)
                {
                    int val = sum + vert[i][j - s];
                    if (val < horz[i][j]) { horz[i][j] = val; more = TRUE; }

                    int sum = lines[i][j - s] - '0';
                }
                
                sum = lines[i][j] - '0';
                for (int s = 1; s <= 3 && j + s < SIZE; s++)
                {
                    int val = sum + vert[i][j + s];
                    if (val < horz[i][j]) { horz[i][j] = val; more = TRUE; }

                    int sum = lines[i][j + s] - '0';
                }
                
                sum = lines[i][j] - '0';
                for (int s = 1; s <= 3 && i - s >= 0; s++)
                {
                    int val = sum + horz[i - s][j];
                    if (val < vert[i][j]) { vert[i][j] = val; more = TRUE; }

                    int sum = lines[i - s][j] - '0';
                }
                
                sum = lines[i][j] - '0';
                for (int s = 1; s <= 3 && i + s < SIZE; s++)
                {
                    int val = sum + horz[i + s][j];
                    if (val < vert[i][j]) { vert[i][j] = val; more = TRUE; }

                    int sum = lines[i + s][j] - '0';
                }
            }
            
        printf("%lld\n", horz[SIZE-1][SIZE-1] < vert[SIZE-1][SIZE-1] ? horz[SIZE-1][SIZE-1] : vert[SIZE-1][SIZE-1]);
    }
}
```
At 6:52, I summitted the first wrong answer produced by the above code.
Lets debug it with the example input.

```c
void solve1()
{
    num_t max = 1;
    for (int i = 0; i < SIZE; i++)
        for (int j = 0; j < SIZE; j++)
            max += lines[i][j] - '0';
    
    for (int i = 0; i < SIZE; i++)
        for (int j = 0; j < SIZE; j++)
            horz[i][j] = vert[i][j] = max;
    
    horz[0][0] = 0;
    vert[0][0] = 0;
    
    for (bool more = TRUE; more; )
    {
        more = FALSE;
        
        for (int i = 0; i < SIZE; i++)
            for (int j = 0; j < SIZE; j++)
            {
                int sum = lines[i][j] - '0';
                for (int s = 1; s <= 3 && j - s >= 0; s++)
                {
                    int val = sum + vert[i][j - s];
                    if (val < horz[i][j]) { horz[i][j] = val; more = TRUE; }

                    sum += lines[i][j - s] - '0';
                }
                
                sum = lines[i][j] - '0';
                for (int s = 1; s <= 3 && j + s < SIZE; s++)
                {
                    int val = sum + vert[i][j + s];
                    if (val < horz[i][j]) { horz[i][j] = val; more = TRUE; }

                    sum += lines[i][j + s] - '0';
                }
                
                sum = lines[i][j] - '0';
                for (int s = 1; s <= 3 && i - s >= 0; s++)
                {
                    int val = sum + horz[i - s][j];
                    if (val < vert[i][j]) { vert[i][j] = val; more = TRUE; }

                    sum += lines[i - s][j] - '0';
                }
                
                sum = lines[i][j] - '0';
                for (int s = 1; s <= 3 && i + s < SIZE; s++)
                {
                    int val = sum + horz[i + s][j];
                    if (val < vert[i][j]) { vert[i][j] = val; more = TRUE; }

                    sum += lines[i + s][j] - '0';
                }
            }
        //for (int i = 0; i < SIZE; i++)
        //{
        //    for (int j = 0; j < SIZE; j++)
        //        printf(" %4d,%3d", vert[i][j], horz[i][j]);
        //    printf("\n");
        //}

        printf("%lld\n", horz[SIZE-1][SIZE-1] < vert[SIZE-1][SIZE-1] ? horz[SIZE-1][SIZE-1] : vert[SIZE-1][SIZE-1]);
    }
}
```

It looks like I made a stupid coding mistake, which I could have found
if I had heeded the warnings that the compiler was producing. After I
fixed the coding mistakes, I found the correct answer to the first half
of the puzzle at 7:03.


### Second part of the puzzle.

This is relatively easy. Just make some simple modifications to the code.

```c

int main(int argc, char *argv[])
{
    ...
    solve2();
}

void solve2()
{
    num_t max = 1;
    for (int i = 0; i < SIZE; i++)
        for (int j = 0; j < SIZE; j++)
            max += lines[i][j] - '0';
    
    for (int i = 0; i < SIZE; i++)
        for (int j = 0; j < SIZE; j++)
            horz[i][j] = vert[i][j] = max;
    
    horz[0][0] = 0;
    vert[0][0] = 0;
    
    for (bool more = TRUE; more; )
    {
        more = FALSE;
        
        for (int i = 0; i < SIZE; i++)
            for (int j = 0; j < SIZE; j++)
            {
                int sum = lines[i][j] - '0';
                for (int s = 1; s <= 10 && j - s >= 0; s++)
                {
                    int val = sum + vert[i][j - s];
                    if (s > 3 && val < horz[i][j]) { horz[i][j] = val; more = TRUE; }

                    sum += lines[i][j - s] - '0';
                }
                
                sum = lines[i][j] - '0';
                for (int s = 1; s <= 10 && j + s < SIZE; s++)
                {
                    int val = sum + vert[i][j + s];
                    if (s > 3 && val < horz[i][j]) { horz[i][j] = val; more = TRUE; }

                    sum += lines[i][j + s] - '0';
                }
                
                sum = lines[i][j] - '0';
                for (int s = 1; s <= 10 && i - s >= 0; s++)
                {
                    int val = sum + horz[i - s][j];
                    if (s > 3 && val < vert[i][j]) { vert[i][j] = val; more = TRUE; }

                    sum += lines[i - s][j] - '0';
                }
                
                sum = lines[i][j] - '0';
                for (int s = 1; s <= 10 && i + s < SIZE; s++)
                {
                    int val = sum + horz[i + s][j];
                    if (s > 3 && val < vert[i][j]) { vert[i][j] = val; more = TRUE; }

                    sum += lines[i + s][j] - '0';
                }
            }

        printf("%lld\n", horz[SIZE-1][SIZE-1] < vert[SIZE-1][SIZE-1] ? horz[SIZE-1][SIZE-1] : vert[SIZE-1][SIZE-1]);
    }
    
}
```

At 7:08, I found the correct answer with the above code.

### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day17.md >day17.c; gcc -g -Wall day17.c -o day17; ./day17
```

