# Day 8 of Advent of Code 2023

I started reading the puzzle aroung 9:00 (CET) while preparing
my lunch. Around 9:18, I started coding.

```c
int main(int argc, char *argv[])
{
    read_file("input/day08.txt");
    solve1();
}

void solve1()
{
    char pos[4] = "AAA";
    num_t count = 0;
    char *s = lines[0];
    while (strcmp(pos, "ZZZ") != 0)
    {
        if (*s == '\0')
            s = lines[0];
        for (int i = 2; i < nr_lines; i++)
            if (strncmp(pos, lines[i], 3) == 0)
            {
                strncpy(pos, lines[i] + (*s == 'L' ? 7 : 12), 3);
                pos[3] = '\0';
                break;
            }
        printf("%s ", pos);
        count++;
        s++;
    }
    printf("%lld\n", count);
}
```

Around 9:30, I found a solution, but it turned out to be wrong.
I had forgotten to increment the left/right sequence pointer, so
I added `s++;` at the end of the loop. At 9:34, I found the correct
answer to the first half of the puzzle.

### Second part of the puzzle.

At 9:37, I finished reading the second half of the puzzle and have
come up with an efficient algorithm to calculate the answer, involving
having to find the least common multiplier. But I have to go work first.

At 18:41, I continued. During the day, I realized that it might not be
as simple as I thought when I wrote the above. Let see. First, I am
going to rewrite the above solution:

```c
void solve1()
{
    char pos[4] = "AAA";
    num_t count = 0;
    int j = 0;
    char *directions = lines[0];
    while (strcmp(pos, "ZZZ") != 0)
    {
        if (directions[j] == '\0')
            j = 0;
        for (int i = 2; i < nr_lines; i++)
            if (strncmp(pos, lines[i], 3) == 0)
            {
                strncpy(pos, lines[i] + (directions[j] == 'L' ? 7 : 12), 3);
                pos[3] = '\0';
                break;
            }
        printf("%s ", pos);
        count++;
        j++;
    }
    printf("%lld\n", count);
}
```

At 18:48, I finished the above.

```c
int main(int argc, char *argv[])
{
    ...
    solve2();
}

typedef struct state state_t, *state_p;
struct state
{
    int j;
    int nr_steps;
    state_t *to;
    state_t *next;
};


state_t **states = 0;

void solve2()
{
    states = (state_p*)malloc(nr_lines * sizeof(state_p));
    for (int i = 0; i < nr_lines; i++)
        states[i] = 0;

    char *directions = lines[0];
    
    for (int i = 2; i < nr_lines; i++)
        if (lines[i][2] == 'A')
        {
            state_t *cur = (state_p)malloc(sizeof(state_t));
            cur->j = 0;
            cur->next = states[i];
            states[i] = cur;
            
            char pos[4];
            strncpy(pos, lines[i], 3); pos[3] = '\0';
            printf("%s: ", pos);
            int steps = 0;
            int i2 = i;
            for (int j = 0; ; j++)
            {
                if (directions[j] == '\0')
                    j = 0;
                strncpy(pos, lines[i2] + (directions[j] == 'L' ? 7 : 12), 3); pos[3] = '\0';
                steps++;
                for (int i3 = 2; i3 < nr_lines; i3++)
                    if (strncmp(pos, lines[i3], 3) == 0)
                    {
                        i2 = i3;
                        break;
                    }
                if (pos[2] == 'Z')
                {
                    printf(" %d %d:%s", steps, j, pos);
                    state_t *next = 0;
                    bool found = FALSE;
                    for (state_t *state = states[i2]; state != 0; state = state->next)
                        if (state->j == j)
                        {
                            next = state;
                            found = TRUE;
                            break;
                        }
                    if (!found)
                    {
                        next = (state_p)malloc(sizeof(state_t));
                        next->j = j;
                        next->next = states[i2];
                        states[i2] = next;
                    }
                    cur->nr_steps = steps;
                    cur->to = next;
                    if (found)
                        break;
                    steps = 0;
                    next = cur;
                }
            }
            printf("\n");
        }
}
                
```

Okay, it looks rather simple. The number of steps for the 'A' node to its 'Z' node
is the same as the number of steps to the its 'Z' node. So, it is enough to count
the number of steps from the 'A' node to its 'Z' node and calculate the smallest
common multiplier of these numbers. All the code with respect to states can be left
out.

```c
void solve2()
{
    num_t answer = 1;

    char *directions = lines[0];
    
    for (int i = 2; i < nr_lines; i++)
        if (lines[i][2] == 'A')
        {
            char pos[4];
            strncpy(pos, lines[i], 3); pos[3] = '\0';
            printf("%s: ", pos);
            int steps = 0;
            int i2 = i;
            for (int j = 0; ; j++)
            {
                if (directions[j] == '\0')
                    j = 0;
                strncpy(pos, lines[i2] + (directions[j] == 'L' ? 7 : 12), 3); pos[3] = '\0';
                steps++;
                if (pos[2] == 'Z')
                {
                    printf(" %d %d:%s", steps, j, pos);
                    break;
                }
                for (int i3 = 2; i3 < nr_lines; i3++)
                    if (strncmp(pos, lines[i3], 3) == 0)
                    {
                        i2 = i3;
                        break;
                    }
            }
            answer = scm(answer, steps);
            printf("\n");
        }
    printf("%lld\n", answer);
}
                
```

At 19:54, I found the correct answer with the above code.


### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day08.md >day08.c; gcc day08.c -o day08; ./day08
```

