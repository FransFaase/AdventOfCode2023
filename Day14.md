# Day 14 of Advent of Code 2023

I started reading the puzzle aroung 7:35 and also did some
things in between. I guess that the second part of the
puzzle will consist of moving it in several directions,
like North, East, South, West, and so on.

```c
int main(int argc, char *argv[])
{
    read_file("input/day14.txt");
    nr_cols = strlen(lines[0]);
    solve1();
}

int nr_cols;

void solve1()
{
    MoveNorth();
    Calculate();
}

void MoveNorth()
{
    for (int j = 0; j < nr_cols; j++)
        for (int i = 0; i < nr_lines; i++)
            if (lines[i][j] == 'O')
            {
                lines[i][j] = '.';
                int i2 = i;
                while (i2 > 0 && lines[i2 - 1][j] == '.')
                    i2--;
                lines[i2][j] = 'O';
            }
}

void Calculate()
{
    num_t sum = 0;
    for (int i = 0; i < nr_lines; i++)
        for (int j = 0; j < nr_cols; j++)
            if (lines[i][j] == 'O')
                sum += nr_lines - i;
    printf("%lld\n", sum);
}

```

At 8:18, I found the solution for the first half on the first
time running the code.

### Second part of the puzzle.

At 15:32, lets continue with the second part.

```c
int main(int argc, char *argv[])
{
    ...
    solve2();
}

void MoveSouth()
{
    for (int j = 0; j < nr_cols; j++)
        for (int i = nr_lines - 1; i >= 0; i--)
            if (lines[i][j] == 'O')
            {
                lines[i][j] = '.';
                int i2 = i;
                while (i2 + 1 < nr_lines && lines[i2 + 1][j] == '.')
                    i2++;
                lines[i2][j] = 'O';
            }
}

void MoveWest()
{
    for (int i = 0; i < nr_lines; i++)
        for (int j = 0; j < nr_cols; j++)
            if (lines[i][j] == 'O')
            {
                lines[i][j] = '.';
                int j2 = j;
                while (j2 > 0 && lines[i][j2 - 1] == '.')
                    j2--;
                lines[i][j2] = 'O';
            }
}

void MoveEast()
{
    for (int i = 0; i < nr_lines; i++)
        for (int j = nr_cols - 1; j >= 0; j--)
            if (lines[i][j] == 'O')
            {
                lines[i][j] = '.';
                int j2 = j;
                while (j2 + 1 < nr_cols && lines[i][j2 + 1] == '.')
                    j2++;
                lines[i][j2] = 'O';
            }
}

void Calculate()
{
    num_t sum = 0;
    for (int i = 0; i < nr_lines; i++)
        for (int j = 0; j < nr_cols; j++)
            if (lines[i][j] == 'O')
                sum += nr_lines - i;
    printf("%10lld", sum);
}

void solve2()
{
    for (;;)
    {
        MoveNorth();
        Calculate();
        MoveWest();
        Calculate();
        MoveSouth();
        Calculate();
        MoveEast();
        Calculate();
        printf("\n");
    }
}
```

That did not converge. I had not expected that it would end into a cycle.
I left it at this for some other obligations.

```c
#define SIZE 200

void solve2()
{
    num_t past_values[SIZE];
    num_t cycles = 0;
    for (;;)
    {
        MoveNorth();
        MoveWest();
        MoveSouth();
        MoveEast();

        cycles++;
        
        num_t sum = 0;
        for (int i = 0; i < nr_lines; i++)
            for (int j = 0; j < nr_cols; j++)
                if (lines[i][j] == 'O')
                    sum += nr_lines - i;

        printf("%lld %lld", cycles, sum);
        if (cycles > SIZE)
        {
            // detect cycle
            for (int i = SIZE - 1; i > 0; i--)
                past_values[i] = past_values[i-1];
            past_values[0] = sum;
            
            for (int cycle_len = 1; cycle_len < SIZE / 4; cycle_len++)
            {
                bool good = TRUE;
                for (int i = 0; i + cycle_len < SIZE && good; i++)
                    good = past_values[i] == past_values[i + cycle_len];
                if (good)
                {
                    printf(" Cycle %d", cycle_len);
                    if ((1000000000L - cycles) % cycle_len == 0)
                    {
                        printf(" Done\n");
                        return;
                    }
                } 
            }
        }
        printf("\n");
    }
}
```

I build a cycle detector, which stops when the remaining number of steps
to go is a multiple of the cycle length. At that point the last printed
value is the answer. At 23:06, it returned the correct answer.

### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day14.md >day14.c; gcc -g -Wall day14.c -o day14; ./day14
```

