# Day 23 of Advent of Code 2023

At 13:10 (CET), I started reading the puzzle text. I am not sure how tricky it is.
Because each tile may only be visited once, dead-end cannot be taken. Lets
see what the input looks like when we eliminate those from the input.

```c
int main(int argc, char *argv[])
{
    read_file("input/day23.txt");
    nr_cols = strlen(lines[0]);
    solve1();
}

int nr_cols;

void solve1()
{
    eliminate_dead_ends();
    for (int i = 0; i < nr_lines; i++)
        printf("%s\n", lines[i]);
    printf("%d, %d\n", nr_lines, nr_cols);
}

int nr_neighbours(int i, int j)
{
    return   (lines[i-1][j] != '#' ? 1 : 0)
           + (lines[i+1][j] != '#' ? 1 : 0)
           + (lines[i][j-1] != '#' ? 1 : 0)
           + (lines[i][j+1] != '#' ? 1 : 0);
}

void eliminate_dead_ends()
{
    for (bool more = TRUE; more;)
    {
        more = FALSE;
        for (int i = 1; i < nr_lines - 1; i++)
            for (int j = 1; j < nr_cols - 1; j++)
                if (lines[i][j] != '#' && nr_neighbours(i,j) == 1)
                {
                    lines[i][j] = '#';
                    more = TRUE;
                }
        if (more)
            for (int i = nr_lines - 2; i > 0; i--)
                for (int j = nr_cols - 2; j > 0; j--)
                    if (lines[i][j] != '#' && nr_neighbours(i,j) == 1)
                        lines[i][j] = '#';
        printf("%d\n", more);
    }
}
```

At 13:22, it looks like there are non. I wonder how I should solve this.
Build a graph of all the paths or simply use a recursive function to
travel over the input. I guess, a recursive function is needed anyways.

Every cross point can only be visited once.

It looks like many cross points have sliders. Lets check that.

```c
void solve1()
{
    for (int i = 1; i < nr_lines - 1; i++)
        for (int j = 1; j < nr_cols - 1; j++)
            if (lines[i][j] != '#' && nr_neighbours(i,j) > 2)
            {
                printf("%3d %3d: ", i, j);
                int nr_in = 0;
                int nr_out = 0;
                if (lines[i][j-1] == '>') nr_in++; else if (lines[i][j-1] == '<') nr_out++;
                if (lines[i][j+1] == '<') nr_in++; else if (lines[i][j+1] == '>') nr_out++;
                if (lines[i-1][j] == 'v') nr_in++; else if (lines[i-1][j] == '^') nr_out++;
                if (lines[i+1][j] == '^') nr_in++; else if (lines[i+1][j] == 'v') nr_out++;
                int nr = nr_neighbours(i,j);
                printf("%d %d %d", nr, nr_in, nr_out);
                if (nr_in + nr_out == nr)
                    printf(" OK");
                printf("\n");
            }
}
```

At 14:07, that seems to be the case. Which means we can solve this by keeping the distance
for every location. Lets try that.

```c
int dist[200][200];

void process(int i, int j)
{
    if (lines[i][j] == '#' || dist[i][j] > 0)
        return;
    int nr = nr_neighbours(i,j);
    if (nr == 2)
    {
        switch (lines[i][j])
        {
            case '>': dist[i][j] = dist[i][j-1]; break;
            case '<': dist[i][j] = dist[i][j+1]; break;
            case 'v': dist[i][j] = dist[i-1][j]; break;
            case '^': dist[i][j] = dist[i+1][j]; break;
            case '.': dist[i][j] = dist[i][j-1] + dist[i][j+1] + dist[i-1][j] + dist[i+1][j]; break;
        }
        if (dist[i][j] > 0)
            dist[i][j]++;
    }
    else if (nr > 2)
    {
        int max_dist = 0;
        if (lines[i][j-1] == '>') { int d = dist[i][j-1]; if (d == 0) return; else if (d > max_dist) max_dist = d; } 
        if (lines[i][j+1] == '<') { int d = dist[i][j+1]; if (d == 0) return; else if (d > max_dist) max_dist = d; } 
        if (lines[i-1][j] == 'v') { int d = dist[i-1][j]; if (d == 0) return; else if (d > max_dist) max_dist = d; } 
        if (lines[i+1][j] == '^') { int d = dist[i+1][j]; if (d == 0) return; else if (d > max_dist) max_dist = d; }
        dist[i][j] = max_dist + 1;
    }
} 

void solve1()
{
    for (int i = 0; i < nr_lines; i++)
        for (int j = 1; j < nr_cols; j++)
            dist[i][j] = 0;
    
    dist[1][1] = 2;
    
    while (dist[nr_lines-2][nr_cols-2] == 0)
    {
        for (int i = 1; i < nr_lines - 1; i++)
            for (int j = 1; j < nr_cols - 1; j++)
                process(i, j);
        for (int i = nr_lines - 2; i > 0; i--)
            for (int j = nr_cols - 2; j > 0; j--)
                process(i, j);
        printf("*\n");
    }
    printf("%d\n", dist[nr_lines-2][nr_cols-2] + 1);
}        
```

At 14:30, the above code did not return the correct answer. Maybe a one-off error?
Lets try it on the example input. I am totally clear what is defined as the number
of steps. The above code returns 95, one more than the 94 steps mentioned in the
puzzle description.

At 14:38, I submitted the result from the above code minus one and it was accepted as
the correct answer.

### Second part of the puzzle.

Okay, the second part is what I expected to be. Yes, the requires a recursive
approach.

At 17:17, I started working on the second half. I think, I am first going to
test the code on the example input. I think, I start with building a graph
of all the crossings and their connections. From each crossing there are at
most 4 connections to other crossings.

```c
int main(int argc, char *argv[])
{
    ...
    solve2();
}

typedef struct crossing crossing_t;
struct crossing
{
    int i, j;
    crossing_t *other[4];
    int dist[4];
    bool target;
    crossing_t *next;
};

crossing_t *crossings = 0;
crossing_t **ref_next = &crossings;

crossing_t *find_crossing(int i, int j)
{
    for (crossing_t *crossing = crossings; crossing != 0; crossing = crossing->next) 
        if (crossing->i == i && crossing->j == j)
            return crossing;
    crossing_t *crossing = (crossing_t*)malloc(sizeof(crossing_t));
    crossing->i = i;
    crossing->j = j;
    for (int d = 0; d < 4; d++)
        crossing->other[d] = 0;
    crossing->target = i == nr_lines - 2 && j == nr_cols - 2;
    crossing->next = 0;
    *ref_next = crossing;
    ref_next = &crossing->next;
    return crossing;
}

void solve2()
{
    // Lets close the entry and the exit
    lines[0][1] = '#';
    lines[nr_lines - 1][nr_cols - 2] = '#';

    int di[4] = { 1,  0, -1,  0 };
    int dj[4] = { 0,  1,  0, -1 };
    
    find_crossing(1, 1);
    for (crossing_t *crossing = crossings; crossing != 0; crossing = crossing->next)
    {
        for (int sd = 0; sd < 4; sd++)
            if (crossing->other[sd] == 0)
            {
                int i = crossing->i;
                int j = crossing->j;
                int d = sd;
                int steps = 0;
                if (lines[i+di[d]][j+dj[d]] != '#')
                {
                    for (;;)
                    {
                        steps++;
                        i += di[d];
                        j += dj[d];
                        if (nr_neighbours(i, j) != 2)
                            break;
                        int rd = (d + 2) % 4;
                        for (int nd = 0; nd < 4; nd++)
                            if (nd != rd && lines[i+di[nd]][j+dj[nd]] != '#')
                            {
                                d = nd;
                                break;
                            }
                    }
                    d = (d + 2) % 4;
                    
                    crossing_t *to = find_crossing(i, j);
                    crossing->other[sd] = to;
                    crossing->dist[sd] = steps;
                    to->other[d] = crossing;
                    to->dist[d] = steps;
                    printf("%d,%d -%d- %d,%d\n", crossing->i, crossing->j,  steps, i, j);
                }
            }
    }
}
```

Lets continue implementing a recursive function to find the longest path.

```c
struct crossing
{
    ...
    bool visited;
};

int max_steps = 0;

void find_connection(crossing_t *from, crossing_t *to, int steps)
{
    if (from == to)
    {
        if (steps > max_steps)
        {
            max_steps = steps;
            printf("%d\n", steps);
        }
    }
    bool poss[8] = { 1, 1, 1, 1, 1, 1, 1, 1 };
    for (;;)
    {
        int max = 0;
        int max_i = -1;
        for (int i = 0; i < 8; i++)
            if (poss[i])
            {
                crossing_t *con = i < 4 ? from : to;
                int i2 = i % 4;
                if (con->other[i2] == 0 || con->other[i2]->visited)
                    poss[i] = FALSE;
                else if (con->dist[i2] > max)
                {
                    max = con->dist[i2];
                    max_i = i;
                }
            }
        if (max_i == -1)
            break;
        
        if (max_i < 4)
        {
            crossing_t *next = from->other[max_i];
            from->visited = TRUE;
            find_connection(next, to, steps + max);
            from->visited = FALSE;
        }
        else
        {
            crossing_t *next = to->other[max_i - 4];
            to->visited = TRUE;
            find_connection(from, next, steps + max);
            to->visited = FALSE;
        }
        poss[max_i] = FALSE;
    }
}

void solve2()
{
    ...
    crossing_t *target = 0;
    
    for (crossing_t *crossing = crossings; crossing != 0; crossing = crossing->next)
    {
        crossing->visited = FALSE;
        if (crossing->target)
            target = crossing;
    }
    find_connection(crossings, target, 0);
}
```

The above code gave a much higher value for the example input.
Lets add some debugging code.

```c
crossing_t *froms[100];
int nr_froms = 0;
crossing_t *tos[100];
int nr_tos = 0;

void find_connection(crossing_t *from, crossing_t *to, int steps)
{
    if (from == to)
    {
        if (steps > max_steps)
        {
            max_steps = steps;
            printf("%d: ", steps + 2);
            
            for (int i = 0; i < nr_froms; i++)
                printf(" %d,%d", froms[i]->i, froms[i]->j);
            printf(" | %d,%d | ", from->i, from->j);
            for (int i = nr_tos - 1; i >= 0; i--)
                printf(" %d,%d", tos[i]->i, tos[i]->j);
            printf("\n");
        }
        return;
    }
    bool poss[8] = { 1, 1, 1, 1, 1, 1, 1, 1 };
    for (;;)
    {
        int max = 0;
        int max_i = -1;
        for (int i = 0; i < 8; i++)
            if (poss[i])
            {
                crossing_t *con = i < 4 ? from : to;
                int i2 = i % 4;
                if (con->other[i2] == 0 || con->other[i2]->visited)
                    poss[i] = FALSE;
                else if (con->dist[i2] > max)
                {
                    max = con->dist[i2];
                    max_i = i;
                }
            }
        if (max_i == -1)
            break;
        
        if (max_i < 4)
        {
            crossing_t *next = from->other[max_i];
            from->visited = TRUE;
            froms[nr_froms++] = from;
            find_connection(next, to, steps + max);
            nr_froms--;
            from->visited = FALSE;
        }
        else
        {
            crossing_t *next = to->other[max_i - 4];
            to->visited = TRUE;
            tos[nr_tos++] = to;
            find_connection(from, next, steps + max);
            nr_tos--;
            to->visited = FALSE;
        }
        poss[max_i] = FALSE;
    }
}
```

At 20:20, the above code now returns the correct answer for the
example input. I had forgotten a return statement in `from == to`
case. (Then I also placed it first inside the nested if-statement.)
But, as I had feared, the code takes a long time to solve the puzzle
when running on my input. I think this problem is rather close to
the [Travelling Salesman Problem](https://en.wikipedia.org/wiki/Travelling_salesman_problem).
Maybe it is possible to add some method for detecting when a
partial solution cannot result in a solution with more steps than
the currently found number of steps.

At 20:28, I just submitted the highest value it found so far and
it turned out to be correct. So, I have solved the puzzle.


### Epilogue

At 22:42, the program is still running. Could still be a very long
time, lets see if we can make it stop a bit sooner. Presume that
the maximum solution would visit all crossings, than you would enter
and leave every crossing through different paths, because you are
only allowed to visit each tile once. That would mean that the
upper limit of the maximum distance is the sum of all these together
divided by too.
If a crossing has but one path left, you should not add it.
So lets, after we have selected a crossing, calculate this for
all the crossing we have not visited yet and if this sum together
with the steps we have included is smaller than the maximum found
so far, we can stop.

```c
void find_connection(crossing_t *from, crossing_t *to, int steps)
{
    if (from == to)
    {
        if (steps >= max_steps)
        {
            max_steps = steps;
            printf("%d: ", steps + 2);
            
            for (int i = 0; i < nr_froms; i++)
                printf(" %d,%d", froms[i]->i, froms[i]->j);
            printf(" | %d,%d | ", from->i, from->j);
            for (int i = nr_tos - 1; i >= 0; i--)
                printf(" %d,%d", tos[i]->i, tos[i]->j);
            printf("\n");
        }
        return;
    }
    
    if (steps < max_steps)
    {
        int upper_limit = 0;
        for (crossing_t *crossing = crossings; crossing != 0; crossing = crossing->next)
            if (!crossing->visited)
            {
                int h1 = 0;
                int h2 = 0;
                for (int d = 0; d < 4; d++)
                    if (crossing->other[d] != 0 && !crossing->other[d]->visited)
                    {
                        if (crossing->dist[d] > h1)
                        {
                            h2 = h1;
                            h1 = crossing->dist[d];
                        }
                        else if (crossing->dist[d] > h2)
                            h2 = crossing->dist[d];
                    }
                if (crossing == from || crossing == to)
                    upper_limit += h1;
                else if (h2 > 0)
                    upper_limit += h1 + h2;
            }
        if (2 * steps + upper_limit < 2 * max_steps)
        {
            printf("2 * %d + %d < 2 * %d\n", steps, upper_limit, max_steps);
            return;
        }
    }
                
    bool poss[8] = { 1, 1, 1, 1, 1, 1, 1, 1 };
    for (;;)
    {
        int max = 0;
        int max_i = -1;
        for (int i = 0; i < 8; i++)
            if (poss[i])
            {
                crossing_t *con = i < 4 ? from : to;
                int i2 = i % 4;
                if (con->other[i2] == 0 || con->other[i2]->visited)
                    poss[i] = FALSE;
                else if (con->dist[i2] > max)
                {
                    max = con->dist[i2];
                    max_i = i;
                }
            }
        if (max_i == -1)
            break;
        
        if (max_i < 4)
        {
            crossing_t *next = from->other[max_i];
            from->visited = TRUE;
            froms[nr_froms++] = from;
            find_connection(next, to, steps + max);
            nr_froms--;
            from->visited = FALSE;
        }
        else
        {
            crossing_t *next = to->other[max_i - 4];
            to->visited = TRUE;
            tos[nr_tos++] = to;
            find_connection(from, next, steps + max);
            nr_tos--;
            to->visited = FALSE;
        }
        poss[max_i] = FALSE;
    }
}
```

It looks like it even faster find the solution, but then still
takes a long time to finish.

I added some print statements and than realized the bug that causes it
to take so long. Once it has selected the largest from one of the sides,
it should only try others from that side. So, lets fix it:

```c
void find_connection(crossing_t *from, crossing_t *to, int steps)
{
    if (from == to)
    {
        if (steps >= max_steps)
        {
            max_steps = steps;
            printf("%d: ", steps + 2);
            
            for (int i = 0; i < nr_froms; i++)
                printf(" %d,%d", froms[i]->i, froms[i]->j);
            printf(" | %d,%d | ", from->i, from->j);
            for (int i = nr_tos - 1; i >= 0; i--)
                printf(" %d,%d", tos[i]->i, tos[i]->j);
            printf("\n");
        }
        return;
    }
    
    if (steps < max_steps)
    {
        int upper_limit = 0;
        for (crossing_t *crossing = crossings; crossing != 0; crossing = crossing->next)
            if (!crossing->visited)
            {
                int h1 = 0;
                int h2 = 0;
                for (int d = 0; d < 4; d++)
                    if (crossing->other[d] != 0 && !crossing->other[d]->visited)
                    {
                        if (crossing->dist[d] > h1)
                        {
                            h2 = h1;
                            h1 = crossing->dist[d];
                        }
                        else if (crossing->dist[d] > h2)
                            h2 = crossing->dist[d];
                    }
                if (crossing == from || crossing == to)
                    upper_limit += h1;
                else if (h2 > 0)
                    upper_limit += h1 + h2;
            }
        if (2 * steps + upper_limit < 2 * max_steps)
        {
            return;
        }
    }
                
    bool poss[8] = { 1, 1, 1, 1, 1, 1, 1, 1 };
    for (;;)
    {
        int max = 0;
        int max_i = -1;
        for (int i = 0; i < 8; i++)
            if (poss[i])
            {
                crossing_t *con = i < 4 ? from : to;
                int i2 = i % 4;
                if (con->other[i2] == 0 || con->other[i2]->visited)
                    poss[i] = FALSE;
                else if (con->dist[i2] > max)
                {
                    max = con->dist[i2];
                    max_i = i;
                }
            }
        if (max_i == -1)
            break;
        
        if (max_i < 4)
        {
            crossing_t *next = from->other[max_i];
            from->visited = TRUE;
            froms[nr_froms++] = from;
            find_connection(next, to, steps + max);
            nr_froms--;
            from->visited = FALSE;
            poss[4] = poss[5] = poss[6] = poss[7] = FALSE;
            
        }
        else
        {
            crossing_t *next = to->other[max_i - 4];
            to->visited = TRUE;
            tos[nr_tos++] = to;
            find_connection(from, next, steps + max);
            nr_tos--;
            to->visited = FALSE;
            poss[0] = poss[1] = poss[2] = poss[3] = FALSE;
        }
        poss[max_i] = FALSE;
    }
}
```

Okay, that finishes it within a second. Actually, less than 60 miliseconds.
Without the extra cutting of condition it takes a little over 3 seconds.


### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day23.md >day23.c; gcc -g -Wall day23.c -o day23; ./day23
```

