# Day 6 of Advent of Code 2023

I started reading the puzzle text around 8:45 (CET) and
I started writing some code aroun 9:00. Although, I understood
this was just a parabolic function, I decided to simply
count the wins. When I saw the input, I started to wonder
what the second half of the puzzle was going to be with
such a short input.

```c
int main(int argc, char *argv[])
{
    read_file("input/day06.txt");
    solve1();
}

void solve1()
{
    char *t_line = lines[0];
    char *d_line = lines[1];
    
    num_t answer = 1;
    
    for (int i = 0; i < 4; i++)
    {
        while (!is_digit(*t_line)) t_line++;
        num_t t = parse_number(&t_line);
        while (!is_digit(*d_line)) d_line++;
        num_t d = parse_number(&d_line);
        
        answer *= number_wins(t, d);
    }
    printf("%lld\n", answer);    
}

num_t number_wins(num_t t, num_t d)
{
    printf("%lld %lld\n", t, d);
    num_t result = 0;
    for (int i = 0; i < t; i++)
        if (i * (t - i) > d)
            result++;
    return result;
}

```

At 9:13, I found the solution for the first part of the puzzle.

### Second part of the puzzle.

Okay, now I understand. I quickly realized that it was
going to take some time to using the above function. I realize
that it is enough to find one solution to win, to calculate
the total number of ways to win. I first changed the `number_wins`
function to print the result for each race.

```c
num_t number_wins(num_t t, num_t d)
{
    num_t result = 0;
    for (int i = 0; i < t; i++)
        if (i * (t - i) > d)
            result++;
    printf("%lld %lld: %lld\n", t, d, result);
    
    return result;
}
```

Then I modified the function to calculate the number of ways
to win after finding the first solution:


```c
num_t number_wins(num_t t, num_t d)
{
    num_t result = 0;
    num_t i;
    for (i = 0; i < t; i++)
        if (i * (t - i) > d)
            break;
    result = t - 2 * i + 1;
    printf("%lld %lld: %lld\n", t, d, result);
    
    return result;
}
```

At 9:25, I got the above code working, after making one small
correction, namely adding the `+ 1` in the expression to calculate
the number of ways. Now, lets adapt the code for parsing the
digits as a single number on each line of the input:

```c
int main(int argc, char *argv[])
{
    ...
    solve2();
}

void solve2()
{
    num_t t = 0;
    for (char *s = lines[0]; *s != '\0'; s++)
        if (is_digit(*s))
            t = 10 * t + *s - '0';
    num_t d = 0;
    for (char *s = lines[1]; *s != '\0'; s++)
        if (is_digit(*s))
            d = 10 * d + *s - '0';
    printf("%lld\n", number_wins(t, d));
}
```

At 9:28, that found the solution.
    
### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day06.md >day06.c; gcc day06.c -o day06; ./day06
```

