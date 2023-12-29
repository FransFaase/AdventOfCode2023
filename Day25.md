# Day 25 of Advent of Code 2023

I first wanted to finish [day 24](Day24.md) before I would work on this. I
already have had ample time to think about how to appraoch this, and I think
I have found a good way, which involves [Floyd–Warshall algorithm](https://en.wikipedia.org/wiki/Floyd%E2%80%93Warshall_algorithm)
and counting how often a certain connection is used.

My input contains 1202 lines.

```c

#define SIZE 1202

bool graph[SIZE][SIZE];

int main(int argc, char *argv[])
{
    read_file("input/day25.txt");
    if (SIZE != nr_lines)
        printf("SIZE should be %d\n", nr_lines);
    read_graph();
    solve1();
}

void solve1()
{
}

void read_graph()
{
    for (int i = 0; i < SIZE; i++)
        for (int j = 0; j < SIZE; j++)
            graph[i][j] = FALSE;
            
    for (int i = 0; i < nr_lines; i++)
    {
        printf("%s: ", lines[i]);
        char *s = lines[i] + 4;
        while (*s == ' ')
        {
            s++;
            bool found = FALSE;
            for (int j = 0; j < nr_lines; j++)
            {
                if (strncmp(lines[j], s, 3) == 0)
                {
                    graph[i][j] = TRUE;
                    graph[j][i] = TRUE;
                    found = TRUE;
                    printf(" %d", j);
                }
            }
            if (!found)
                printf(" NOT_FOUND '%c%c%c'", s[0],s[1],s[2]);
            s += 3;
            printf("\n");
        }
    }    
}
```

Now left implements Floyd-Warshall and see what are the edges
with the highest number of paths, just to get an idea if the
approach that I have in mind is going to work.

```c
#define NR 40
void solve1()
{
    floyd();
    
    int max[NR];
    int max_i[NR];
    int max_j[NR];
    int n = 0;
    
    for (int i = 0; i < SIZE-1; i++)
        for (int j = i + 1; j < SIZE; j++)
        {
            int v = visits[i][j];
            int vi = i;
            int vj = j;
            for (int k = 0; k < n; k++)
            {
                if (v > max[k])
                {
                    int h = max[k]; max[k] = v; v = h;
                    h = max_i[k]; max_i[k] = vi; vi = h;
                    h = max_j[k]; max_j[k] = vj; vj = h;
                }
            }
            if (n < NR)
            {
                max[n] = v;
                max_i[n] = vi;
                max_j[n] = vj;
                n++;
            }
        }
    for (int i = 0; i < NR; i++)
    {
        printf("%4d %4d: %4d\n", max_i[i], max_j[i], max[i]);
    }
}

#define MAX_DIST 4000

int distance[SIZE][SIZE];
int next[SIZE][SIZE];
int visits[SIZE][SIZE];

void floyd()
{
    for (int i = 0; i < SIZE; i++)
        for (int j = 0; j < SIZE; j++)
        {
            distance[i][j] = i == j ? 0 : graph[i][j] ? 1 : MAX_DIST;
            next[i][j] = j;
            visits[i][j] = 0;
        }
    
    for (int j = 0; j < SIZE; j++)
        for (int i = 0; i < SIZE; i++)
            for (int k = 0; k < SIZE; k++)
            {
                int min_dist = distance[i][j] + distance[j][k];
                if (min_dist < distance[i][k])
                {
                    distance[i][k] = min_dist;
                    next[i][k] = next[i][j];
                }
            }
    for (int i = 0; i < SIZE; i++)
        for (int j = 0; j < SIZE; j++)
        {
            //printf("From %d to %d: ", i, j);
            int k = i;
            while(next[k][j] != j)
            {
                int n_k = next[k][j];
                visits[k][n_k]++;
                //printf("%d (%d)", k, distance[k][n_k]);
                //if (distance[k][n_k] != 1)
                //    printf("ERROR");
                k = n_k;
            }
            //printf("\n");
        }
}
```

There is indeed one edge with signicantly more visits than the other. Lets
construct a recursive method.

```c
void solve1()
{
    search(0);
}

int solution_i[3];
int solution_j[3];

bool search(int d)
{
    floyd();

    bool connected = TRUE;
    for (int i = 0; i < SIZE && connected; i++)
        for (int j = 0; j < SIZE && connected; j++)
            connected = distance[i][j] < MAX_DIST;
    if (!connected)
        printf("%*.*s- disconnected\n", d*2, d*2, "");

    if (d == 3)
    {
        if (!connected)
        {
            found_solution();
            return TRUE;
        }
        return FALSE;
    }

    int max[NR];
    int max_i[NR];
    int max_j[NR];
    int n = 0;
    
    for (int i = 0; i < SIZE-1; i++)
        for (int j = i + 1; j < SIZE; j++)
        {
            int v = visits[i][j];
            int vi = i;
            int vj = j;
            for (int k = 0; k < n; k++)
            {
                if (v > max[k])
                {
                    int h = max[k]; max[k] = v; v = h;
                    h = max_i[k]; max_i[k] = vi; vi = h;
                    h = max_j[k]; max_j[k] = vj; vj = h;
                }
            }
            if (n < NR)
            {
                max[n] = v;
                max_i[n] = vi;
                max_j[n] = vj;
                n++;
            }
        }
    for (int i = 0; i < NR; i++)
    {
        printf("%*.*sTrying %4d %4d: %4d\n", d*2, d*2, "", max_i[i], max_j[i], max[i]);
        solution_i[d] = max_i[i];
        solution_j[d] = max_j[i];
        graph[max_i[i]][max_j[i]] = FALSE;
        graph[max_j[i]][max_i[i]] = FALSE;
        bool f = search(d + 1);
        graph[max_i[i]][max_j[i]] = TRUE;
        graph[max_j[i]][max_i[i]] = TRUE;
        if (f)
            return TRUE;
    }
    return FALSE;
}

void found_solution()
{
    for (int i = 0; i < 3; i++)
        printf(" %4d,%4d", solution_i[i], solution_j[i]);
    printf("\n");
}
```

Turns out, that I wrong about the number of nodes in the graph. Some nodes
are not mentioned at the start of a line. So, have to start over a bit.

```c

typedef struct node node_t;
struct node
{
    char name[4];
    node_t *next;
};

node_t *nodes = 0;
int nr_nodes = 0;

int node_nr(char *name)
{
    node_t **ref = &nodes;
    int i = 0;
    for (; *ref != 0; ref = &(*ref)->next, i++)
        if (strncmp(name, (*ref)->name, 3) == 0)
            return i;
    nr_nodes++;
    (*ref) = (node_t*)malloc(sizeof(node_t));
    strncpy((*ref)->name, name, 3);
    (*ref)->name[3] = '\0';
    (*ref)->next = 0;
    return i;
}
    
void read_graph()
{
    for (int i = 0; i < nr_lines; i++)
    {
        node_nr(lines[i]);
        char *s = lines[i] + 4;
        while (*s == ' ')
        {
            s++;
            node_nr(s);
            s += 3;
        }
    }
    printf("%d\n", nr_nodes);
}

```

So, there are 1479 nodes in my input.

```c
#define SIZE 1479

void read_graph()
{
    for (int i = 0; i < SIZE; i++)
        for (int j = 0; j < SIZE; j++)
            graph[i][j] = FALSE;

    for (int i = 0; i < nr_lines; i++)
    {
        int nr1 = node_nr(lines[i]);
        printf("%s | %d: ", lines[i], nr1);
        char *s = lines[i] + 4;
        while (*s == ' ')
        {
            s++;
            int nr2 = node_nr(s);
            graph[nr1][nr2] = TRUE;
            graph[nr2][nr1] = TRUE;
            printf(" %d", nr2);
            s += 3;
        }
        printf("\n");
    }
    printf("%d\n", nr_nodes);
}

void floyd()
{
    for (int i = 0; i < nr_nodes; i++)
        for (int j = 0; j < nr_nodes; j++)
        {
            distance[i][j] = i == j ? 0 : graph[i][j] ? 1 : MAX_DIST;
            next[i][j] = j;
            visits[i][j] = 0;
        }
    
    for (int j = 0; j < nr_nodes; j++)
        for (int i = 0; i < nr_nodes; i++)
            for (int k = 0; k < nr_nodes; k++)
            {
                int min_dist = distance[i][j] + distance[j][k];
                if (min_dist < distance[i][k])
                {
                    distance[i][k] = min_dist;
                    next[i][k] = next[i][j];
                }
            }
    for (int i = 0; i < nr_nodes; i++)
        for (int j = 0; j < nr_nodes; j++)
        {
            //printf("From %d to %d: ", i, j);
            int k = i;
            while(next[k][j] != j)
            {
                int n_k = next[k][j];
                visits[k][n_k]++;
                //printf("%d (%d)", k, distance[k][n_k]);
                //if (distance[k][n_k] != 1)
                //    printf("ERROR");
                k = n_k;
            }
            //printf("\n");
        }
}

bool search(int d)
{
    floyd();

    bool connected = TRUE;
    for (int i = 0; i < nr_nodes && connected; i++)
        for (int j = 0; j < nr_nodes && connected; j++)
            connected = distance[i][j] < MAX_DIST;
    if (!connected)
        printf("%*.*s- disconnected\n", d*2, d*2, "");

    if (d == 3)
    {
        if (!connected)
        {
            found_solution();
            return TRUE;
        }
        return FALSE;
    }

    int max[NR];
    int max_i[NR];
    int max_j[NR];
    int n = 0;
    
    for (int i = 0; i < SIZE-1; i++)
        for (int j = i + 1; j < SIZE; j++)
        {
            int v = visits[i][j];
            int vi = i;
            int vj = j;
            for (int k = 0; k < n; k++)
            {
                if (v > max[k])
                {
                    int h = max[k]; max[k] = v; v = h;
                    h = max_i[k]; max_i[k] = vi; vi = h;
                    h = max_j[k]; max_j[k] = vj; vj = h;
                }
            }
            if (n < NR)
            {
                max[n] = v;
                max_i[n] = vi;
                max_j[n] = vj;
                n++;
            }
        }
    for (int i = 0; i < NR; i++)
    {
        printf("%*.*sTrying %4d %4d: %4d\n", d*2, d*2, "", max_i[i], max_j[i], max[i]);
        solution_i[d] = max_i[i];
        solution_j[d] = max_j[i];
        graph[max_i[i]][max_j[i]] = FALSE;
        graph[max_j[i]][max_i[i]] = FALSE;
        bool f = search(d + 1);
        graph[max_i[i]][max_j[i]] = TRUE;
        graph[max_j[i]][max_i[i]] = TRUE;
        if (f)
            return TRUE;
    }
    return FALSE;
}

```

Okay, that is running, rather slow, but for the example input, it does find a solution.
But I am bit worried that for the last, it was not near the edge with the most visits.

Another attempt

```c
#define NR 100

void bare_floyd()
{
    for (int i = 0; i < nr_nodes; i++)
        for (int j = 0; j < nr_nodes; j++)
        {
            distance[i][j] = i == j ? 0 : graph[i][j] ? 1 : MAX_DIST;
            next[i][j] = j;
            visits[i][j] = 0;
        }
    
    for (int j = 0; j < nr_nodes; j++)
        for (int i = 0; i < nr_nodes; i++)
            for (int k = 0; k < nr_nodes; k++)
            {
                int min_dist = distance[i][j] + distance[j][k];
                if (min_dist < distance[i][k])
                {
                    distance[i][k] = min_dist;
                    next[i][k] = next[i][j];
                }
            }
}

void calc_visits()
{
    for (int i = 0; i < nr_nodes; i++)
        for (int j = 0; j < nr_nodes; j++)
            //if (distance[i][j] >= 8)
            {
                //printf("From %d to %d: ", i, j);
                int k = i;
                while(next[k][j] != j)
                {
                    int n_k = next[k][j];
                    visits[k][n_k]++;
                    //printf("%d (%d)", k, distance[k][n_k]);
                    //if (distance[k][n_k] != 1)
                    //    printf("ERROR");
                    k = n_k;
                }
                //printf("\n");
            }
}

int dists[2000];

void calc_freq()
{
    for (int i = 0; i < 2000; i++)
        dists[i] = 0;
    
    for (int i = 0; i < nr_nodes-1; i++)
        for (int j = i + 1; j < nr_nodes; j++)
            dists[distance[i][j]]++;
            
    for (int i = 0; i < 2000; i++)
        if (dists[i] != 0)
            printf("%4d: %5d\n", i, dists[i]);
}

int d_dists[2000];

void calc_d_freq()
{
    for (int i = 0; i < 2000; i++)
        d_dists[i] = 0;
    
    node_t *node = nodes;
    for (int i = 0; i < nr_nodes; i++, node = node->next)
    {
        int d = 0;
        for (int j = 0; j < nr_nodes; j++)
            if (graph[i][j])
                d++;
        if (d == 1)
            printf("%d %s\n", i, node->name);
        d_dists[d]++;
    }
    for (int i = 0; i < 2000; i++)
        if (d_dists[i] != 0)
            printf("%4d: %5d\n", i, d_dists[i]);
}

void solve1_()
{
    calc_d_freq();
    
    bare_floyd();
    
    calc_freq();
    
    calc_visits();
    
    int max[NR];
    int max_i[NR];
    int max_j[NR];
    int n = 0;
    
    for (int i = 0; i < nr_nodes-1; i++)
        for (int j = i + 1; j < nr_nodes; j++)
        {
            int v = visits[i][j];
            int vi = i;
            int vj = j;
            for (int k = 0; k < n; k++)
            {
                if (v > max[k])
                {
                    int h = max[k]; max[k] = v; v = h;
                    h = max_i[k]; max_i[k] = vi; vi = h;
                    h = max_j[k]; max_j[k] = vj; vj = h;
                }
            }
            if (n < NR)
            {
                max[n] = v;
                max_i[n] = vi;
                max_j[n] = vj;
                n++;
            }
        }
        
    for (int i = 0; i < n; i++)
        printf("%4d %4d  %4d\n", max_i[i], max_j[i], max[i]);
    
    for (int i = 3; i < n; i++)
    {
        graph[max_i[i]][max_j[i]] = FALSE;
        graph[max_j[i]][max_i[i]] = FALSE;
        for (int j = 1; j < i; j++)
        {
            graph[max_i[j]][max_j[j]] = FALSE;
            graph[max_j[j]][max_i[j]] = FALSE;
            for (int k = 0; k < j; k++)
            {
                graph[max_i[k]][max_j[k]] = FALSE;
                graph[max_j[k]][max_i[k]] = FALSE;
                
                printf("Try %4d,%4d  %4d,%4d  %4d,%4d: ",
                    max_i[i], max_j[i],
                    max_i[j], max_j[j],
                    max_i[k], max_j[k]);
                
                bare_floyd();

                bool connected = TRUE;
                for (int i = 0; i < nr_nodes && connected; i++)
                    for (int j = 0; j < nr_nodes && connected; j++)
                        connected = distance[i][j] < MAX_DIST;
                if (!connected)
                {
                    printf("Found\n");
                    return;
                }
                printf("\n");
                graph[max_i[k]][max_j[k]] = TRUE;
                graph[max_j[k]][max_i[k]] = TRUE;
            }
            graph[max_i[j]][max_j[j]] = TRUE;
            graph[max_j[j]][max_i[j]] = TRUE;
        }
        graph[max_i[i]][max_j[i]] = TRUE;
        graph[max_j[i]][max_i[i]] = TRUE;
    }
}
```

Okay, I just discovered that my initial method worked perfectly,
finding the solution without any retracting, but that there was
an error in the `node_nr` function.
(For this, I renamed the last occurence of `solve1` into `solve1_`.)
Now only need to write a function that calculates the size of both
components. Calculating the size of just one is enough.

```c
void found_solution()
{
    int other = -1;
    int nr_0 = 0;
    int nr_other = 0;
    for (int i = 0; i < nr_nodes; i++)
        if (distance[0][i] < MAX_DIST)
            nr_0++;
        else if (other == -1)
        {
            other = i;
            nr_other = 1;
        }
        else if (distance[other][i] < MAX_DIST)
            nr_other++;
        else
            printf("%d in third component\n", i);
    printf("%d %d %d %d: %d\n",
        nr_0, nr_other, nr_0 + nr_other, nr_nodes, nr_0 * nr_other);
        
}
```

At 22:53 (CET), I got all the stars.


### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day25.md >day25.c; gcc -g -Wall day25.c -o day25; ./day25
```

