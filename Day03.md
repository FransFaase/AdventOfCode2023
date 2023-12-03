# Day 3 of Advent of Code 2023

I only started at 17:21 (CET). I got the idea to develop a function
to read the input and return an array with pointers to the lines.
And put in a kind of standard library of function to be used. So,
lets do that first:

```c
#include "stdio.h"
#include "malloc.h"
#include "unistd.h"


typedef char *char_p;

char **read_file(const char *name, int *nr_lines)
{
    *nr_lines = 0;
    FILE *f = fopen(name, "r");
    if (f == 0)
    {
        return 0;
    }
    int fh = fileno(f);
    size_t length = lseek(fh, 0L, SEEK_END);
    lseek(fh, 0L, SEEK_SET);
    char *data = (char*)malloc(length);
    length = read(fh, data, length);
    fclose(f);
    
    // count the number of lines in the file
    for (int i = 0; i < length; i++)
        if (data[i] == '\n')
            (*nr_lines)++;
    
    
    char **lines = (char**)malloc(*nr_lines * sizeof(char_p));
    char *s = data;
    for (int i = 0; i < *nr_lines; i++)
    {
        lines[i] = s;
        
        
        while (*s != '\n')
            s++;
        *s++ = '\0';
    }
    
    return lines;
}
```

Okay, that took me until 17:49 to implement. Lets write some
code to read the input and print it again. Just to check if
everything works.

```c
int main(int argc, char *argv)
{
    int nr_lines;
    char **input = read_file("input/day03.txt", &nr_lines);
    
    for (int i = 0; i < nr_lines; i++)
        printf("%s\n", input[i]);
}
```

At 18:15, I finished the above. Still had to make a small fix
to the `read_file` function: `*nr_lines++` should not be confused with `(*nr_lines)++`.

Now start working on the first puzzle:
```c
int main(int argc, char *argv)
{
    int nr_lines;
    char **input = read_file("input/day03.txt", &nr_lines);
    
    solve1(input, nr_lines);
}

void solve1(char **input, int nr_lines)
{
    int nr_cols = strlen(input[0]);

    num_t sum = 0;
    
    for (int i = 0; i < nr_lines; i++)
    {
        for (int j = 0; j < nr_cols; j++)
            if (input[i][j] == '-' || ('0' <= input[i][j] && input[i][j] <= '9'))
            {
                int start_j = j;
                int sign = 1;
                if (input[i][j] == '-')
                {
                    sign = -1;
                    j++;
                }
                int num = 0;
                while ('0' <= input[i][j] && input[i][j] <= '9')
                    num = 10 * num + input[i][j++] - '0';
                char symbol = ' ';
                for (int i2 = i - 1; i2 <= i + 1 && symbol == ' '; i2++)
                    for (int j2 = start_j - 1; j2 <= j && symbol == ' '; j2++)
                        if (0 <= i2 && i2 < nr_lines && 0 <= j2 && j2 < nr_cols)
                        {
                            char ch = input[i2][j2];
                            if (ch != '.' && !('0' <= ch && ch <= '9'))
                                symbol = ch;
                        }
                j--;
                if (symbol != ' ')
                    sum += num;
            }
    }
    
    printf("%lld\n", sum);
}

```

At 18:33, the above code returned a number and the number appears to be correct,
a little to my surprise.

### Second part of the puzzle.

I had to think a little bit about how I am going to solve this. I think I
am just going to build a list of for all the gears and count the number of
numbers close to it.

```c
int main(int argc, char *argv)
{
    ...
    solve2(input, nr_lines);
}

typedef struct gear gear_t;
struct gear
{
    int i;
    int j;
    int n;
    num_t v;
};

void solve2(char **input, int nr_lines)
{
    int nr_cols = strlen(input[0]);

    int nr_gears = 0;
    for (int i = 0; i < nr_lines; i++)
        for (int j = 0; j < nr_cols; j++)
            if (input[i][j] == '*')
                nr_gears++;
    
    gear_t *gears = (gear_t*)malloc(nr_gears * sizeof(gear_t));

    nr_gears = 0;
    for (int i = 0; i < nr_lines; i++)
        for (int j = 0; j < nr_cols; j++)
            if (input[i][j] == '*')
            {
                gears[nr_gears].i = i;
                gears[nr_gears].j = j;
                gears[nr_gears].n = 0;
                gears[nr_gears].v = 1;
                nr_gears++;
            }
    
    for (int i = 0; i < nr_lines; i++)
    {
        for (int j = 0; j < nr_cols; j++)
            if (input[i][j] == '-' || ('0' <= input[i][j] && input[i][j] <= '9'))
            {
                int start_j = j;
                int sign = 1;
                if (input[i][j] == '-')
                {
                    sign = -1;
                    j++;
                }
                int num = 0;
                while ('0' <= input[i][j] && input[i][j] <= '9')
                    num = 10 * num + input[i][j++] - '0';
                char symbol = ' ';
                for (int i2 = i - 1; i2 <= i + 1 && symbol == ' '; i2++)
                    for (int j2 = start_j - 1; j2 <= j && symbol == ' '; j2++)
                        if (0 <= i2 && i2 < nr_lines && 0 <= j2 && j2 < nr_cols)
                        {
                            if (input[i2][j2] == '*')
                            {
                                for (int g = 0; g < nr_gears; g++)
                                    if (gears[g].i == i2 && gears[g].j == j2)
                                    {
                                        gears[g].n++;
                                        gears[g].v *= num;
                                        break;
                                    }
                            }
                        }
                j--;
            }
    }

    num_t sum = 0;
    for (int g = 0; g < nr_gears; g++)
        if (gears[g].n == 2)
            sum += gears[g].v;
    
    printf("%lld\n", sum);
}

```

After fixing a number of small bugs in the above code, the program returned a correct answer at 19:03.
 

### Some standard definitions

```c
typedef long long num_t;
typedef int bool;
#define FALSE 0
#define TRUE 1
```

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Day03.md >day03.c; gcc day03.c -o day03; ./day03
```
