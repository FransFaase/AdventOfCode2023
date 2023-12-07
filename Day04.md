# Day 4 of Advent of Code 2023

I did read the puzzle description in the morning, before I went to my office.
Around 18:44 (CET), I worked a bit on [the standard library](Std.md). At
18:50, I started writing this file.

```c
int main(int argc, char *argv)
{
    read_file("input/day04.txt");
    solve1();
}

void solve1()
{
}

```

At 19:57, lets study the input file a bit. There are 10 winning numbers and the first
starts in column 11 of each line. There are 25 numbers on the card and the first starts
in column 43.

```c
void solve1()
{
    num_t sum = 0;
    for (int i = 0; i < nr_lines; i++)
    {
        char *line = lines[i];
        num_t value = 0;
        for (int j = 42; j < 117; j += 3)
            for (int w = 10; w < 40; w += 3)
                if (line[j] == line[w] && line[j+1] == line[w+1])
                {
                    value = value == 0 ? 1 : 2 * value;
                    break;
                }
        sum += value;
    }
    printf("%lld\n", sum);
}
```

At 19:14, after having fixed one small syntax error, the above code
returned the correct answer.

### Second part of the puzzle.

It seems for the second part you have to store the for each card into
an array with the number of matches. I am going to modify the above code,
to do this. The results is shown below:

```c
int *nr_matches;

typedef int *int_p;

void solve1()
{
    nr_matches = (int*)malloc(nr_lines * sizeof(int_p));
    
    num_t sum = 0;
    for (int i = 0; i < nr_lines; i++)
    {
        char *line = lines[i];
        num_t nr_match = 0;
        for (int j = 42; j < 117; j += 3)
            for (int w = 10; w < 40; w += 3)
                if (line[j] == line[w] && line[j+1] == line[w+1])
                {
                    nr_match++;
                    break;
                }
        nr_matches[i] = nr_match;
        sum += nr_match == 0 ? 0 : 1 << (nr_match - 1);
    }
    printf("%lld\n", sum);
}

int main(int argc, char *argv)
{
    ...
    solve2();
}

void solve2()
{
}

```

At 19:24, now left try to understand what we have to exactly.
I spend some time replaying the example and I think, I have
figured out how to calculate the numbers.

```c
typedef num_t *num_p;
void solve2()
{
    num_t *counts = (num_t*)malloc(nr_lines * sizeof(num_p));
    for (int i = 0; i < nr_lines; i++)
        counts[i] = 1;
        
    num_t count = 0;
    for (int i = 0; i < nr_lines; i++)
    {
        count += counts[i];
        int nr_match = nr_matches[i];
        for (int i2 = 0; i2 < nr_match && i + 1 + i2 < nr_lines; i2++)
            counts[i + 1 + i2] += counts[i];
    }
    
    printf("%lld\n", count);
}

```

At 19:55, after having fixed some small syntax errors, the above code
returned a number (larger than I had expected), which happend to be the correct
answer.

### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day04.md >day04.c; gcc day04.c -o day04; ./day04
```
