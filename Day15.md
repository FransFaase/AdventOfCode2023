# Day 15 of Advent of Code 2023

I started reading the text at 6:01 (CET).
It is just calculatin hashes. Seems rather straightforward.

```c
int main(int argc, char *argv[])
{
    read_file("input/day15.txt");
    solve1();
}

void solve1()
{
    char *s = lines[0];
    num_t sum = 0;
    for (;; s++)
    {
        int hash = 0;
        for (;*s != '\0' && *s != ','; s++)
            hash = (hash + *s) * 17 % 256;
        sum += hash;
        if (*s == '\0')
            break;
    }
    printf("%lld\n", sum);
}
```

At 6:11, the above code returned the correct answer. I had
to fix some small syntax error and I had to insert the `s++` code
in the for-statement after I realized that the program was running
in an infinite loop.

### Second part of the puzzle.

```c
int main(int argc, char *argv[])
{
    ...
    solve2();
}

typedef struct lens lens_t;
struct lens
{
    char name[40];
    num_t focus;
    lens_t *next;
};

void solve2()
{
    lens_t *lenses[256];
    for (int i = 0; i < 256; i++)
        lenses[i] = 0;
        
    char *s = lines[0];
    for (;; s++)
    {
        int hash = 0;
        char *r = s;
        for (;*s != '=' && *s != '-'; s++)
            hash = (hash + *s) * 17 % 256;

        if (*s == '-')
        {
            *s++ = '\0';
            for (lens_t **ref = &lenses[hash]; *ref != 0; ref = &(*ref)->next)
                if (strcmp((*ref)->name, r) == 0)
                {
                    *ref = (*ref)->next;
                    break;
                }
        }
        else
        {
            *s++ = '\0';
            num_t focus = parse_number(&s); 
            bool found = FALSE;
            lens_t **ref = &lenses[hash];
            for (; *ref != 0; ref = &(*ref)->next)
                if (strcmp((*ref)->name, r) == 0)
                {
                    found = TRUE;
                    (*ref)->focus = focus;
                }
            if (!found)
            {
                (*ref) = (lens_t*)malloc(sizeof(lens_t));
                strcpy((*ref)->name, r);
                (*ref)->focus = focus;
                (*ref)->next = 0;
            }
        }
        if (*s == '\0')
            break;
    }
}
```

At 6:32, I finished the above and it executed without causing
any 'segment fault`. I continued with the summation.

```c
void solve2()
{
    ...
    
    num_t sum = 0;
    for (int i = 0; i < 256; i++)
    {
        num_t l = 1;
        for (lens_t *lens = lenses[i]; lens != 0; lens = lens->next, l++)
            sum += (i + 1) * l * lens->focus;
    }
    printf("%lld\n", sum);
}
```

At 6:37, the above code returned the correct answer for the
second half of the puzzle.

### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day15.md >day15.c; gcc -g -Wall day15.c -o day15; ./day15
```

