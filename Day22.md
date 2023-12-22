# Day 22 of Advent of Code 2023

I already glimpsed at the puzzle shortly after it became available.
At 11:43, I started reading it again. At 15:16, I wrote some code
to determine the minimum and maximum values of the bricks

```c
int main(int argc, char *argv[])
{
    read_file("input/day22.txt");
    solve1();
}

void solve1()
{
    int min_x = 1000000;
    int max_x = 0;
    int min_y = 1000000;
    int max_y = 0;
    int min_z = 1000000;
    int max_z = 0;
    for (int i = 0; i < nr_lines; i++)
    {
        char *s = lines[i];
        num_t x = parse_number(&s); if (x < min_x) min_x = x; else if (x > max_x) max_x = x; s++;
        num_t y = parse_number(&s); if (y < min_y) min_y = y; else if (y > max_y) max_y = y; s++;
        num_t z = parse_number(&s); if (z < min_z) min_z = z; else if (z > max_z) max_z = z; s++;
              x = parse_number(&s); if (x < min_x) min_x = x; else if (x > max_x) max_x = x; s++;
              y = parse_number(&s); if (y < min_y) min_y = y; else if (y > max_y) max_y = y; s++;
              z = parse_number(&s); if (z < min_z) min_z = z; else if (z > max_z) max_z = z; s++;
    }
    printf("%d - %d\n", min_x, max_x);
    printf("%d - %d\n", min_y, max_y);
    printf("%d - %d\n", min_z, max_z);
    printf("%lld\n", nr_lines);
}
```

At 15:34, I found the minimum and maximum values. They are not that large, I think, I am going
to use some array to represent which positions are filled with which brick.

```c
int cube[10][10][314];

void solve1()
{
    for (int x = 0; x <= 9; x++)
        for (int y = 0; y <= 9; y++)
            for (int z = 1; z <= 313; z++)
                cube[x][y][z] = -1;
                
    for (int i = 0; i < nr_lines; i++)
    {
        char *s = lines[i];
        num_t x1 = parse_number(&s); s++;
        num_t y1 = parse_number(&s); s++;
        num_t z1 = parse_number(&s); s++;
        num_t x2 = parse_number(&s); s++;
        num_t y2 = parse_number(&s); s++;
        num_t z2 = parse_number(&s);
        
        for (;;)
        {
            if (cube[x1][y1][z1] != -1)
                printf("Error %d %lld,%lld,%lld\n", i, x1, y1, z1);
            cube[x1][y1][z1] = i;
            if (x1 == x2 && y1 == y2 && z1 == z2)
                break;
            if (x2 < x1) x1--; else if (x2 > x1) x1++;
            if (y2 < y1) y1--; else if (y2 > y1) y1++;
            if (z2 < z1) z1--; else if (z2 > z1) z1++;
        }
    }
}
```

At 15:48, I finished the above code. Now just sweep from top to bottom
to see if there are any bricks that do not support other bricks and can
be removed. I increased the height of the cube from `313` to `314` in the above code.

```c
void solve1()
{
    ...
    int answer = 0;
    
    for (bool more = TRUE; more; )
    {
        more = FALSE;
        
        for (int z = 312; z > 0; z--)
        {
            //printf("z = %d\n", z);
            //for (int x = 0; x < 10; x++)
            //{
            //    for (int y = 0; y < 10; y++)
            //        printf("%5d", cube[x][y][z]);
            //    printf("\n");
            //}
            //printf("\n");        
            for (int x = 0; x < 10; x++)
                for (int y = 0; y < 10; y++)
                    if (cube[x][y][z] != -1)
                    {
                        int brick = cube[x][y][z];
                        bool supports = FALSE;
                        for (int x1 = 0; x1 < 10 && !supports; x1++)
                            for (int y1 = 0; y1 < 10 && !supports; y1++)
                                supports = cube[x1][y1][z] == brick && cube[x1][y1][z+1] != -1;
                        if (!supports)
                        {
                            more = TRUE;
                            answer++;
                            for (int x1 = 0; x1 < 10; x1++)
                                for (int y1 = 0; y1 < 10; y1++)
                                    if (cube[x1][y1][z] == brick)
                                        cube[x1][y1][z] = -1;
                        }
                    }
        }
        printf("%d\n", answer);
    }
}
```

At 16:08, I submitted the first answer with the above code,
but it not correct. It looks like I interpreted the puzzle
incorrectly.

```c
void fill_cube()
{
    for (int x = 0; x <= 9; x++)
        for (int y = 0; y <= 9; y++)
            for (int z = 1; z <= 313; z++)
                cube[x][y][z] = -1;
                
    for (int i = 0; i < nr_lines; i++)
    {
        char *s = lines[i];
        num_t x1 = parse_number(&s); s++;
        num_t y1 = parse_number(&s); s++;
        num_t z1 = parse_number(&s); s++;
        num_t x2 = parse_number(&s); s++;
        num_t y2 = parse_number(&s); s++;
        num_t z2 = parse_number(&s);
        
        for (;;)
        {
            if (cube[x1][y1][z1] != -1)
                printf("Error %d %lld,%lld,%lld\n", i, x1, y1, z1);
            cube[x1][y1][z1] = i;
            if (x1 == x2 && y1 == y2 && z1 == z2)
                break;
            if (x2 < x1) x1--; else if (x2 > x1) x1++;
            if (y2 < y1) y1--; else if (y2 > y1) y1++;
            if (z2 < z1) z1--; else if (z2 > z1) z1++;
        }
    }
}

void solve1()
{
    fill_cube();
    
    bool processed_bricks[4000];
    for (int i = 0; i < nr_lines; i++)
        processed_bricks[i] = FALSE;
        
    int answer = 0;
    for (int z = 312; z >= 1; z--)
    {
        printf("\nz = %d\n", z);
        for (int x = 0; x < 10; x++)
            for (int y = 0; y < 10; y++)
                if (cube[x][y][z] != -1)
                {
                    int brick = cube[x][y][z];
                    if (processed_bricks[brick])
                        continue;
                    processed_bricks[brick] = TRUE;
                    
                    printf("process %d: ", brick);
                    
                    bool not_supports = TRUE;
                    for (int x1 = 0; x1 < 10 && not_supports; x1++)
                        for (int y1 = 0; y1 < 10 && not_supports; y1++)
                            if (cube[x1][y1][z] == brick && cube[x1][y1][z+1] != -1 && cube[x1][y1][z+1] != brick)
                            {
                                // brick has some brick above it
                                int above_brick = cube[x1][y1][z+1];
                                printf("(supports %d: ", above_brick);
                                
                                // check if brick above is supported by some other brick
                                bool supported_by_other = FALSE;
                                for (int x2 = 0; x2 < 10; x2++)
                                    for (int y2 = 0; y2 < 10; y2++)
                                        if (   cube[x2][y2][z] != -1
                                            && cube[x2][y2][z] != brick
                                            && cube[x2][y2][z+1] == above_brick)
                                        {
                                            printf(" also %d", cube[x2][y2][z]);
                                            supported_by_other = TRUE;
                                        }
                                printf(")");  
                                if (!supported_by_other)
                                    not_supports = FALSE;
                            }
                    if (not_supports)
                    {
                        printf("%d can be removed", brick);
                        answer++;
                    }
                    printf("\n");
                }
    }
    printf("%d\n", answer);
}
```

At 17:12, still did not find a solution. Lets try another approach and determine
the support relationship between bricks. I guess we need that for the second part as well.

```c
typedef struct brick brick_t;
struct brick
{
    int nr_supporting;
    int supports[10];
    
    int nr_support_by;
    int support_by[10];
};

brick_t bricks[2000];

void fill_bricks()
{
    for (int i = 0; i < nr_lines; i++)
    {
        bricks[i].nr_supporting = 0;
        bricks[i].nr_support_by = 0;
    }
    
    for (int z = 1; z < 312; z++)
        for (int x = 0; x < 10; x++)
            for (int y = 0; y < 10; y++)
                if (cube[x][y][z] != -1 && cube[x][y][z+1] != -1 && cube[x][y][z] != cube[x][y][z+1])
                {
                    int bottom = cube[x][y][z];
                    int top = cube[x][y][z+1];
                    bool found;

                    found = FALSE;
                    for (int i = 0; i < bricks[bottom].nr_supporting && !found; i++)
                        found = bricks[bottom].supports[i] == top;
                    if (!found)
                    {
                        printf("%d supports %d", top, bottom);
                        bricks[bottom].supports[bricks[bottom].nr_supporting++] = top;
                    }

                    found = FALSE;
                    for (int i = 0; i < bricks[top].nr_support_by && !found; i++)
                        found = bricks[top].support_by[i] == bottom;
                    if (!found)
                    {
                        printf(" and reverse\n");
                        bricks[top].support_by[bricks[top].nr_support_by++] = bottom;
                    }
                }
}

void solve1()
{
    fill_cube();
    fill_bricks();
    
    int answer = 0;
    for (int i = 0; i < nr_lines; i++)
        if (bricks[i].nr_supporting == 0)
        {
            printf("%d supports nothing\n", i);
            answer++;
        }
        else
        {
            printf("%d supports: ", i);
            bool more_than_one = FALSE;
            for (int j = 0; j < bricks[i].nr_supporting; j++)
            {
                int top = bricks[i].supports[j];
                printf(" %d (%d)", top, bricks[top].nr_support_by);
                if (bricks[top].nr_support_by > 1)
                    more_than_one = TRUE;
            }
            if (more_than_one)
            {
                printf(" can be removed");
                answer++;
            }
            printf("\n");
        }
    printf("%d\n", answer);
}
```

At 17:52, looks like I forgot about the bricks falling down.

```c
void drop_bricks()
{
    bool processed[2000];
    for (int i = 0; i < nr_lines; i++)
        processed[i] = FALSE;
        
    for (int z = 2; z <= 312; z++)
    {
        printf("\nz = %d\n", z);
        for (int x = 0; x < 10; x++)
            for (int y = 0; y < 10; y++)
                if (cube[x][y][z] != -1 && !processed[cube[x][y][z]])
                {
                    int brick = cube[x][y][z];
                    
                    processed[brick] = TRUE;
                    
                    int drop_z = 1;
                    for (int x1 = 0; x1 < 10 && drop_z < z; x1++)
                        for (int y1 = 0; y1 < 10 && drop_z < z; y1++)
                            if (cube[x1][y1][z] == brick)
                            {
                                int z1 = z;
                                while (z1 > 1 && cube[x1][y1][z1 - 1] == -1)
                                    z1--;
                                if (z1 > drop_z)
                                    drop_z = z1;
                            }
                    printf("Drop %d by %d\n", brick, z - drop_z);
                    if (drop_z < z)
                    {
                        int h = 1;
                        while (z + h <= 312 && cube[x][y][z + h] == brick)
                            h++;
                        printf("  h = %d\n", h);
                        for (int x1 = 0; x1 < 10 && drop_z < z; x1++)
                            for (int y1 = 0; y1 < 10 && drop_z < z; y1++)
                                if (cube[x1][y1][z] == brick)
                                    for (int i = 0; i < h; i++)
                                    {
                                        printf("  %d,%d: %d -> %d\n", x1, y1, z + i, drop_z + i), 
                                        cube[x1][y1][z + i] = -1;
                                        cube[x1][y1][drop_z + i] = brick;
                                    }
                    }
                }
    }
}                        

void solve1()
{
    fill_cube();
    drop_bricks();
    fill_bricks();
    
    int answer = 0;
    for (int i = 0; i < nr_lines; i++)
        if (bricks[i].nr_supporting == 0)
        {
            printf("%d supports nothing\n", i);
            answer++;
        }
        else
        {
            printf("%d supports: ", i);
            bool all_more_than_one = TRUE;
            for (int j = 0; j < bricks[i].nr_supporting; j++)
            {
                int top = bricks[i].supports[j];
                printf(" %d (%d)", top, bricks[top].nr_support_by);
                if (bricks[top].nr_support_by == 1)
                    all_more_than_one = FALSE;
            }
            if (all_more_than_one)
            {
                printf(" can be removed");
                answer++;
            }
            printf("\n");
        }
    printf("%d\n", answer);
}
```

At 19:06, not sure if drop works correct.

```c
void drop_bricks()
{
    for (bool more = TRUE; more;)
    {
        more = FALSE;
        for (int z = 2; z <= 312; z++)
            for (int x = 0; x < 10; x++)
                for (int y = 0; y < 10; y++)
                    if (cube[x][y][z] != -1)
                    {
                        int brick = cube[x][y][z];
                        bool can_drop = TRUE;
                        for (int x1 = 0; x1 < 10 && can_drop; x1++)
                            for (int y1 = 0; y1 < 10 && can_drop; y1++)
                                if (cube[x1][y1][z] == brick)
                                    can_drop = cube[x1][y1][z-1] == -1;
                        if (can_drop)
                        {
                            more = TRUE;
                            for (int x1 = 0; x1 < 10; x1++)
                                for (int y1 = 0; y1 < 10; y1++)
                                    if (cube[x1][y1][z] == brick)
                                    {
                                        cube[x1][y1][z] = -1;
                                        cube[x1][y1][z-1] = brick;
                                    }
                        }
                    }
    }
}
```

### Second part of the puzzle.

```c
int main(int argc, char *argv[])
{
    ...
    solve2();
}

struct brick
{
    ...
    bool dropped;
};

void solve2()
{
    num_t answer = 0;
    for (int i = 0; i < nr_lines; i++)
        if (bricks[i].nr_supporting == 0)
        {
            printf("%d supports nothing\n", i);
        }
        else
        {
            printf("%d supports: ", i);
            bool all_more_than_one = TRUE;
            for (int j = 0; j < bricks[i].nr_supporting; j++)
            {
                int top = bricks[i].supports[j];
                printf(" %d (%d)", top, bricks[top].nr_support_by);
                if (bricks[top].nr_support_by == 1)
                    all_more_than_one = FALSE;
            }
            if (all_more_than_one)
            {
                printf(" can be removed");
            }
            else
            {
                for (int j = 0; j < nr_lines; j++)
                    bricks[j].dropped = j == i;
                    
                for (bool more = TRUE; more;)
                {
                    more = FALSE;
                    
                    for (int j = 0; j < nr_lines; j++)
                        if (!bricks[j].dropped && bricks[j].nr_support_by > 0)
                        {
                            bool no_support = TRUE;
                            for (int k = 0; k < bricks[j].nr_support_by && no_support; k++)
                                no_support = bricks[bricks[j].support_by[k]].dropped;
                            if (no_support)
                            {
                                printf(" %d", j);
                                answer++;
                                bricks[j].dropped = TRUE;
                                more = TRUE;
                            }
                        }
                }
            }
            printf("\n");
        }
    printf("%lld\n", answer);
}
```

### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day22.md >day22.c; gcc -g -Wall day22.c -o day22; ./day22
```

