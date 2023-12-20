# Day 16 of Advent of Code 2023

At 9:55, I just read the introduction a bit, not much focusing on the
details.

At 14:47, I continued. I think I am first going to find out the size
of the input and assume it is fixed in my code:

```c
int main(int argc, char *argv[])
{
    read_file("input/day16.txt");
    solve1();
}

void solve1()
{
    int nr_cols = strlen(lines[0]);
    printf("%d %d\n", nr_lines, nr_cols);
}
```

Okay, it is 110. Lets just define an array with that size.

```c
#define SIZE 110

char mat[SIZE][SIZE];
#define D_UP    1
#define D_RIGHT 2
#define D_DOWN  4
#define D_LEFT  8

#define SET(D) if ((mat[i][j] & (D)) != (D)) { more = TRUE; mat[i][j] |= (D); }

void solve1()
{
    for (int i = 0; i < SIZE; i++)
        for (int j = 0; j < SIZE; j++)
            mat[i][j] = 0;
            
    for (bool more = TRUE; more; )
    {
        more = FALSE;
        
        for (int i = 0; i < SIZE; i++)
            for (int j = 0; j < SIZE; j++)
            {
                if ((j > 0 && (mat[i][j-1] & D_RIGHT) != 0) || (i == 0 && j == 0))
                {
                    //printf("x %c\n", lines[i][j]);
                    switch (lines[i][j])
                    {
                        case '.':  SET(D_RIGHT); break;
                        case '\\': SET(D_DOWN); break;
                        case '/':  SET(D_UP); break;
                        case '-':  SET(D_RIGHT); break;
                        case '|':  SET(D_UP | D_DOWN); break;
                    }
                }
                if (i > 0 && (mat[i-1][j] & D_DOWN) != 0)
                {
                    switch (lines[i][j])
                    {
                        case '.':  SET(D_DOWN); break;
                        case '\\': SET(D_RIGHT); break;
                        case '/':  SET(D_LEFT); break;
                        case '-':  SET(D_LEFT | D_RIGHT); break;
                        case '|':  SET(D_DOWN); break;
                    }
                }
                if (j + 1 < SIZE && (mat[i][j+1] & D_LEFT) != 0)
                {
                    switch (lines[i][j])
                    {
                        case '.':  SET(D_LEFT); break;
                        case '\\': SET(D_UP); break;
                        case '/':  SET(D_DOWN); break;
                        case '-':  SET(D_LEFT); break;
                        case '|':  SET(D_UP | D_DOWN); break;
                    }
                }
                if (i + 1 < SIZE && (mat[i+1][j] & D_UP) != 0)
                {
                    switch (lines[i][j])
                    {
                        case '.':  SET(D_UP); break;
                        case '\\': SET(D_LEFT); break;
                        case '/':  SET(D_RIGHT); break;
                        case '-':  SET(D_LEFT | D_RIGHT); break;
                        case '|':  SET(D_UP); break;
                    }
                }
            }

        num_t sum = 0;
        for (int i = 0; i < SIZE; i++)
            for (int j = 0; j < SIZE; j++)
                if (mat[i][j] != 0)
                    sum++;
        printf("%lld\n", sum);
    }
    
}
```
I needed a bit of debugging, because I had used `mat` instead
of `lines` in the above code. At 15:33, it returned the correct
answer to the first half of the puzzle.

### Second part of the puzzle.

Lets rewrite the above code a little.

```c

bool more;

void from_right(int i, int j)
{
    switch (lines[i][j])
    {
        case '.':  SET(D_RIGHT); break;
        case '\\': SET(D_DOWN); break;
        case '/':  SET(D_UP); break;
        case '-':  SET(D_RIGHT); break;
        case '|':  SET(D_UP | D_DOWN); break;
    }
}

void from_down(int i, int j)
{
    switch (lines[i][j])
    {
        case '.':  SET(D_DOWN); break;
        case '\\': SET(D_RIGHT); break;
        case '/':  SET(D_LEFT); break;
        case '-':  SET(D_LEFT | D_RIGHT); break;
        case '|':  SET(D_DOWN); break;
    }
}

void from_left(int i, int j)
{
    switch (lines[i][j])
    {
        case '.':  SET(D_LEFT); break;
        case '\\': SET(D_UP); break;
        case '/':  SET(D_DOWN); break;
        case '-':  SET(D_LEFT); break;
        case '|':  SET(D_UP | D_DOWN); break;
    }
}

void from_up(int i, int j)
{
    switch (lines[i][j])
    {
        case '.':  SET(D_UP); break;
        case '\\': SET(D_LEFT); break;
        case '/':  SET(D_RIGHT); break;
        case '-':  SET(D_LEFT | D_RIGHT); break;
        case '|':  SET(D_UP); break;
    }
}

void init()
{
    for (int i = 0; i < SIZE; i++)
        for (int j = 0; j < SIZE; j++)
            mat[i][j] = 0;
}

void solve1()
{
    init();
    from_right(0, 0);
    
    printf("%lld\n", calc());
}

num_t calc()
{
    num_t sum = 0;
    for (more = TRUE; more; )
    {
        more = FALSE;
        
        for (int i = 0; i < SIZE; i++)
            for (int j = 0; j < SIZE; j++)
            {
                if (j > 0 && (mat[i][j-1] & D_RIGHT) != 0) from_right(i, j);
                if (i > 0 && (mat[i-1][j] & D_DOWN) != 0) from_down(i, j);
                if (j + 1 < SIZE && (mat[i][j+1] & D_LEFT) != 0) from_left(i, j);
                if (i + 1 < SIZE && (mat[i+1][j] & D_UP) != 0) from_up(i, j);
            }

        sum = 0;
        for (int i = 0; i < SIZE; i++)
            for (int j = 0; j < SIZE; j++)
                if (mat[i][j] != 0)
                    sum++;
        //printf("%lld\n", sum);
    }
    return sum;
}
```

At 15:58, that returns the answer for the first part.
Now, add some code to find the answer for the second part.

```c
int main(int argc, char *argv[])
{
    ...
    solve2();
}

void solve2()
{
    num_t max = 0;

    for (int i = 0; i < SIZE; i++)
    {
        init();
        from_right(i, 0);
        
        num_t value = calc();
        if (value > max) max = value;
        
        init();
        from_left(i, SIZE-1);
        
        value = calc();
        if (value > max) max = value;
    }
    
    for (int j = 0; j < SIZE; j++)
    {
        init();
        from_down(0, j);
        
        num_t value = calc();
        if (value > max) max = value;
        
        init();
        from_up(SIZE-1, j);
        
        value = calc();
        if (value > max) max = value;
    }
    
    printf("%lld\n", max);
}
```

At 16:04, I found the correct answer for the second half of todays
puzzle.

### Epilogue

A bit faster version:

```c
num_t calc()
{
    num_t sum = 0;
    for (more = TRUE; more; )
    {
        more = FALSE;
        
        for (int i = 0; i < SIZE; i++)
            for (int j = 0; j < SIZE; j++)
            {
                if (j > 0 && (mat[i][j-1] & D_RIGHT) != 0) from_right(i, j);
                if (i > 0 && (mat[i-1][j] & D_DOWN) != 0) from_down(i, j);
                if (j + 1 < SIZE && (mat[i][j+1] & D_LEFT) != 0) from_left(i, j);
                if (i + 1 < SIZE && (mat[i+1][j] & D_UP) != 0) from_up(i, j);
            }
        for (int j = SIZE-1; j >= 0; j--)
            for (int i = SIZE-1; i >= 0; i--)
            {
                if (j > 0 && (mat[i][j-1] & D_RIGHT) != 0) from_right(i, j);
                if (i > 0 && (mat[i-1][j] & D_DOWN) != 0) from_down(i, j);
                if (j + 1 < SIZE && (mat[i][j+1] & D_LEFT) != 0) from_left(i, j);
                if (i + 1 < SIZE && (mat[i+1][j] & D_UP) != 0) from_up(i, j);
            }

        sum = 0;
        for (int i = 0; i < SIZE; i++)
            for (int j = 0; j < SIZE; j++)
                if (mat[i][j] != 0)
                    sum++;
        //printf("%lld\n", sum);
    }
    return sum;
}
```

### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day16.md >day16.c; gcc -g -Wall day16.c -o day16; ./day16
```

