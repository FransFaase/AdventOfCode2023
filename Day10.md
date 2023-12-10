# Day 10 of Advent of Code 2023


I started reading the puzzle text around 11:11 (CET).
I first wanted to check if there was just one S in the
input file.

```c
int main(int argc, char *argv[])
{
    read_file("input/day10.txt");
    solve1();
}

void solve1()
{
    int nr_cols = strlen(lines[0]);
    
    for (int i = 0; i < nr_lines; i++)
        for (int j = 0; j < nr_cols; j++)
            if (lines[i][j] == 'S')
                printf("%d %d\n", i, j);
}
```

At 11:27, I concluded that there is but one 'S'.

```c
void solve1()
{
    int nr_cols = strlen(lines[0]);
    
    num_t max_steps = 0;
    for (int is = 0; is < nr_lines; is++)
        for (int js = 0; js < nr_cols; js++)
            if (lines[is][js] == 'S')
            {
                printf("%d %d\n", is, js);
                for (int ds = 0; ds < 4; ds++)
                {
                    num_t steps = 0;
                    num_t i = is;
                    num_t j = js;
                    int d = ds;
                    for (bool go = TRUE; go; )
                    {
                        steps++;
                        switch (d)
                        {
                            case 0: if (++j == nr_cols) go = FALSE; break;
                            case 1: if (++i == nr_lines) go = FALSE; break;
                            case 2: if (j-- == 0) go = FALSE; break;
                            case 3: if (i-- == 0) go = FALSE; break;
                        }
                        if (!go) break;
                        if (lines[i][j] == 'S')
                        {
                            if (steps > max_steps)
                                max_steps = steps;
                            break;
                        }
                        switch (d)
                        {
                            case 0:
                                switch (lines[i][j])
                                {
                                    case '-': break;
                                    case '7': d = 1; break;
                                    case 'J': d = 3; break;
                                    default: go = FALSE;
                                }
                            case 1:
                                switch (lines[i][j])
                                {
                                    case '|': break;
                                    case 'J': d = 2; break;
                                    case 'L': d = 0; break;
                                    default: go = FALSE;
                                }
                            case 2:
                                switch (lines[i][j])
                                {
                                    case '-': break;
                                    case 'L': d = 3; break;
                                    case 'F': d = 1; break;
                                    default: go = FALSE;
                                }
                            case 3:
                                switch (lines[i][j])
                                {
                                    case '|': break;
                                    case 'F': d = 0; break;
                                    case '7': d = 2; break;
                                    default: go = FALSE;
                                }
                            default: go = FALSE;
                        }
                    }
                }
            }
    printf("%lld\n", max_steps / 2);
}
```

At 11:56, I got 0 as an answer. I swapped 'i' and 'j', but then it
still returned 0. I will add some debug statements to find the
problem.

```c
void solve1()
{
    int nr_cols = strlen(lines[0]);
    
    num_t max_steps = 0;
    for (int is = 0; is < nr_lines; is++)
        for (int js = 0; js < nr_cols; js++)
            if (lines[is][js] == 'S')
            {
                printf("%d %d\n", is, js);
                for (int ds = 0; ds < 4; ds++)
                {
                    num_t steps = 0;
                    num_t i = is;
                    num_t j = js;
                    int d = ds;
                    for (bool go = TRUE; go; )
                    {
                        printf("%d at %lld %lld  =>", d, i, j);
                        steps++;
                        switch (d)
                        {
                            case 0: if (++j == nr_cols) go = FALSE; break;
                            case 1: if (++i == nr_lines) go = FALSE; break;
                            case 2: if (j-- == 0) go = FALSE; break;
                            case 3: if (i-- == 0) go = FALSE; break;
                        }
                        if (!go) break;
                        printf(" %lld %lld %c\n", i, j, lines[i][j]);
                        
                        if (lines[i][j] == 'S')
                        {
                            if (steps > max_steps)
                                max_steps = steps;
                            break;
                        }
                        switch (d)
                        {
                            case 0:
                                switch (lines[i][j])
                                {
                                    case '-': break;
                                    case '7': d = 1; break;
                                    case 'J': d = 3; break;
                                    default: go = FALSE;
                                }
                                break;
                            case 1:
                                switch (lines[i][j])
                                {
                                    case '|': break;
                                    case 'J': d = 2; break;
                                    case 'L': d = 0; break;
                                    default: go = FALSE;
                                }
                                break;
                            case 2:
                                switch (lines[i][j])
                                {
                                    case '-': break;
                                    case 'L': d = 3; break;
                                    case 'F': d = 1; break;
                                    default: go = FALSE;
                                }
                                break;
                            case 3:
                                switch (lines[i][j])
                                {
                                    case '|': break;
                                    case 'F': d = 0; break;
                                    case '7': d = 2; break;
                                    default: go = FALSE;
                                }
                                break;
                            default: go = FALSE;
                        }
                    }
                }
            }
    printf("%lld\n", max_steps / 2);
}
```

At 12:05, I submitted the correct answer. The bug was that I had
forgotten to add some `break;` statements. The fact that I still
sometimes forget them, after knowing C for almost 34 years, shows
that is a bad design choice to include them. (I should have the `-Wall`
command line option for the `ggc` compiler.) I did read the second
half of the puzzle and went on to do some other things.

### Second part of the puzzle.

At 13:39, I continue with the second half. I had some time to think
about it. I think it is possible to solve this with a rather simple
algorithm. I think you can calculate the number of enclosed locations,
if you know the length of the path and its area. Say the length is _l_
and the enclosed are is _a_. If we look to the very first path in the
puzzle description:
```
.....
.S-7.
.|.|.
.L-J.
.....
```
In this the path has a length of eight (because it contains eight
locations) and the area is four because it contains four squares.
Each of these squares as four points. For an area of four, that
means a total of sixteen. Of this sixteen, twelve are on the path,
which means there are four left and these four all are at the one
location. So, it looks like the formula should be:

(4 * _a_ - 2 * _l_ + 4) / 4

Lets see, how this works out, if we make the path a bit longer, like:
```
.....
.S--7.
.|.FJ
.L-J.
.....
```
Now the path length has increased with two to ten and the area has
increased with one to five. (4 * 5 - 2 * 10 + 4 ) / 4 = 1. Now,
add one more:
```
.....
.S--7.
.|..|
.L--J
.....
```
The length of the path remained the same, but the area was increased
with one. Giving: (4 * 6 - 2 * 10 + 4) / 4 = 2.

The area of a polygon can be calculated with the [Shoelace formula](https://en.wikipedia.org/wiki/Shoelace_formula)
which is summing the inner product of all successive points. This can
be combined in the following solution:

```c
int main(int argc, char *argv[])
{
    ...
    solve2();
}

void solve2()
{
    int nr_cols = strlen(lines[0]);
    
    for (int is = 0; is < nr_lines; is++)
        for (int js = 0; js < nr_cols; js++)
            if (lines[is][js] == 'S')
            {
                for (int ds = 0; ds < 4; ds++)
                {
                    num_t steps = 0;
                    num_t area = 0;
                    num_t i = is;
                    num_t j = js;
                    int d = ds;
                    for (bool go = TRUE; go; )
                    {
                        steps++;
                        int pi = i;
                        int pj = j;
                        switch (d)
                        {
                            case 0: if (++j == nr_cols) go = FALSE; break;
                            case 1: if (++i == nr_lines) go = FALSE; break;
                            case 2: if (j-- == 0) go = FALSE; break;
                            case 3: if (i-- == 0) go = FALSE; break;
                        }
                        area += pi * j - pj * i;
                        if (!go) break;
                        
                        if (lines[i][j] == 'S')
                        {
                            if (area < 0) area = -area;
                            printf("%lld\n", 1 + (area - steps) / 2);
                            break;
                        }
                        switch (d)
                        {
                            case 0:
                                switch (lines[i][j])
                                {
                                    case '-': break;
                                    case '7': d = 1; break;
                                    case 'J': d = 3; break;
                                    default: go = FALSE;
                                }
                                break;
                            case 1:
                                switch (lines[i][j])
                                {
                                    case '|': break;
                                    case 'J': d = 2; break;
                                    case 'L': d = 0; break;
                                    default: go = FALSE;
                                }
                                break;
                            case 2:
                                switch (lines[i][j])
                                {
                                    case '-': break;
                                    case 'L': d = 3; break;
                                    case 'F': d = 1; break;
                                    default: go = FALSE;
                                }
                                break;
                            case 3:
                                switch (lines[i][j])
                                {
                                    case '|': break;
                                    case 'F': d = 0; break;
                                    case '7': d = 2; break;
                                    default: go = FALSE;
                                }
                                break;
                            default: go = FALSE;
                        }
                    }
                }
            }
}
```

At 14:32, the above program returned the correct answer (twice actually).
So, I guess the formula is correct.

    
### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day10.md >day10.c; gcc -g -Wall day10.c -o day10; ./day10
```

