# Day 11 of Advent of Code 2023

At 7:44 (CET), I started reading the puzzle text and at 7:48,
I started writing this text. It looks like I have to store
some data in the arrays to solve this puzzle. The distance
between the galaxies is just the sum of the differences between
the coordinates minus two. No, not exactly. It should be minus
one when distances in x and y are the same. Okay, left first
write some code to cound the number of galaxies and find the
numbers of the empty rows and columns.

At 7:59, it looks like I am mistaken about the number of steps.
It is just the sum of the differences.


```c
int main(int argc, char *argv[])
{
    read_file("input/day11.txt");
    solve1();
}

typedef bool *bool_p;

bool *extra_col;
bool *extra_line;

typedef struct coord coord_t;
struct coord
{
    num_t x, y;
};

coord_t *galaxies;
int nr_galaxies = 0;

void read_gal_pos()
{
    int nr_col = strlen(lines[0]);
    
    bool *empty_col    = (bool*)malloc(nr_col * sizeof(bool_p));
    for (int j = 0; j < nr_col; j++)
    {
        empty_col[j] = TRUE;
        for (int i = 0; i < nr_lines; i++)
            if (lines[i][j] == '#')
            {
                nr_galaxies++;            
                empty_col[j] = FALSE;
            }
    }
    
    galaxies = (coord_t*)malloc(nr_galaxies * sizeof(coord_t));
    
    int gal_nr = 0;
    int y = 0;
    
    for (int i = 0; i < nr_lines; i++, y++)
    {
        bool empty_row = TRUE;
        int x = 0;
        for (int j = 0; j < nr_col; j++, x++)
        {
            if (lines[i][j] == '#')
            {
                galaxies[gal_nr].x = x;
                galaxies[gal_nr].y = y;
                gal_nr++;
                empty_row = FALSE;
            }
            if (empty_col[j])
                x++;
        }
        if (empty_row)
            y++;
    }
}

num_t abs_num(num_t v) { return v < 0 ? -v : v; }


void solve1()
{
    read_gal_pos();
    
    num_t sum = 0;
    for (int i = 1; i < nr_galaxies; i++)
        for (int j = 0; j < i; j++)
            sum += abs_num(galaxies[i].x - galaxies[j].x) + abs_num(galaxies[i].y - galaxies[j].y);
    printf("%lld\n", sum); 
}

```

At 8:22, I was ready writing the above.
At 8:23, I forgot a semicollon at line 89. At 8:25, I changed `(bool**)` into `(bool*)` on line 41.
At 8:26, the code returned the answer for the first half of the puzzle. 

### Second part of the puzzle.

It looks like the second part is not that much harder.

```c
int main(int argc, char *argv[])
{
    ...
    solve2();
}

void solve2()
{
    int nr_col = strlen(lines[0]);
    
    bool *empty_col    = (bool*)malloc(nr_col * sizeof(bool_p));
    for (int j = 0; j < nr_col; j++)
    {
        empty_col[j] = TRUE;
        for (int i = 0; i < nr_lines; i++)
            if (lines[i][j] == '#')
            {
                nr_galaxies++;            
                empty_col[j] = FALSE;
            }
    }
    
    galaxies = (coord_t*)malloc(nr_galaxies * sizeof(coord_t));
    
    int gal_nr = 0;
    int y = 0;
    
    for (int i = 0; i < nr_lines; i++, y++)
    {
        bool empty_row = TRUE;
        int x = 0;
        for (int j = 0; j < nr_col; j++, x++)
        {
            if (lines[i][j] == '#')
            {
                galaxies[gal_nr].x = x;
                galaxies[gal_nr].y = y;
                gal_nr++;
                empty_row = FALSE;
            }
            if (empty_col[j])
                x += 999999;
        }
        if (empty_row)
            y += 999999;
    }
    
    num_t sum = 0;
    for (int i = 1; i < nr_galaxies; i++)
        for (int j = 0; j < i; j++)
            sum += abs_num(galaxies[i].x - galaxies[j].x) + abs_num(galaxies[i].y - galaxies[j].y);
    printf("%lld\n", sum); 
}

```

The above code, did not return the correct answer. Lets try it on the example input.

```c
void solve2()
{
    int nr_col = strlen(lines[0]);
    nr_galaxies = 0;
    
    bool *empty_col    = (bool*)malloc(nr_col * sizeof(bool_p));
    for (int j = 0; j < nr_col; j++)
    {
        printf("scan col %d: ", j);
        empty_col[j] = TRUE;
        for (int i = 0; i < nr_lines; i++)
            if (lines[i][j] == '#')
            {
                nr_galaxies++;
                printf("gal %lld,%lld ", i, j);    
                empty_col[j] = FALSE;
            }
        printf("col %d empty %d\n", j, empty_col[j]);
    }
    printf("---\n");
    
    galaxies = (coord_t*)malloc(nr_galaxies * sizeof(coord_t));
    
    int gal_nr = 0;
    int y = 0;
    
    for (int i = 0; i < nr_lines; i++, y++)
    {
        printf("row %d: %d\n", i, y);
        bool empty_row = TRUE;
        int x = 0;
        for (int j = 0; j < nr_col; j++, x++)
        {
            if (i == 0)
                printf("col %d: %d\n", j, x);
            if (lines[i][j] == '#')
            {
                galaxies[gal_nr].x = x;
                galaxies[gal_nr].y = y;
                gal_nr++;
                empty_row = FALSE;
            }
            if (empty_col[j])
            {
                if (i == 0)
                    printf("expand col %d\n", j);
                x += 9;
            }
        }
        if (empty_row)
        {
            printf("expand row %d\n", i);
            y += 9;
        }
    }
    
    num_t sum = 0;
    for (int i = 1; i < nr_galaxies; i++)
        for (int j = 0; j < i; j++)
        {
            printf("dist %d-%d: %lld + %lld = %lld\n",
                i, j, abs_num(galaxies[i].x - galaxies[j].x), abs_num(galaxies[i].y - galaxies[j].y),
                abs_num(galaxies[i].x - galaxies[j].x) + abs_num(galaxies[i].y - galaxies[j].y));
             sum += abs_num(galaxies[i].x - galaxies[j].x) + abs_num(galaxies[i].y - galaxies[j].y);
         }
    printf("%lld\n", sum); 
}
```

After a lot of debugging with the above code, it looks like there were only one bug: I forgot
to initialize `nr_galaxies` to zero again at the start of the `solve2` function. Which gives:

```c
void solve2()
{
    nr_galaxies = 0;
    int nr_col = strlen(lines[0]);
    
    bool *empty_col    = (bool*)malloc(nr_col * sizeof(bool_p));
    for (int j = 0; j < nr_col; j++)
    {
        empty_col[j] = TRUE;
        for (int i = 0; i < nr_lines; i++)
            if (lines[i][j] == '#')
            {
                nr_galaxies++;            
                empty_col[j] = FALSE;
            }
    }
    
    galaxies = (coord_t*)malloc(nr_galaxies * sizeof(coord_t));
    
    int gal_nr = 0;
    int y = 0;
    
    for (int i = 0; i < nr_lines; i++, y++)
    {
        bool empty_row = TRUE;
        int x = 0;
        for (int j = 0; j < nr_col; j++, x++)
        {
            if (lines[i][j] == '#')
            {
                galaxies[gal_nr].x = x;
                galaxies[gal_nr].y = y;
                gal_nr++;
                empty_row = FALSE;
            }
            if (empty_col[j])
                x += 999999;
        }
        if (empty_row)
            y += 999999;
    }
    
    num_t sum = 0;
    for (int i = 1; i < nr_galaxies; i++)
        for (int j = 0; j < i; j++)
            sum += abs_num(galaxies[i].x - galaxies[j].x) + abs_num(galaxies[i].y - galaxies[j].y);
    printf("%lld\n", sum); 
}

```

At 9:04, the above code returned the correct answer.

    
### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day11.md >day11.c; gcc -g -Wall day11.c -o day11; ./day11
```

