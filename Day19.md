# Day 19 of Advent of Code 2023

At 10:44, started reading the puzzle description
At 10:49, started writing this page. I will try for
a solution that simply parses the input. First write
some code to parse the parse and output them.

```c
int main(int argc, char *argv[])
{
    read_file("input/day19.txt");
    solve1();
}

void solve1()
{
    num_t answer = 0;
    for (int i = 0; i < nr_lines; i++)
    {
        char *line = lines[i];
        if (*line != '{') continue;
        
        line += 3;
        num_t x = parse_number(&line);
        line += 3;
        num_t m = parse_number(&line);
        line += 3;
        num_t a = parse_number(&line);
        line += 3;
        num_t s = parse_number(&line);
        printf("{x=%lld,m=%lld,a=%lld,s=%lld}\n", x, m, a, s);
    }
}
```

At 11:02, a diff with the output and the input returns the rule.
Lets expand the above code a bit further;

I stopped quite soon after this and only continued working on this
at 1:12, December 20.


```c
void solve1()
{
    num_t answer = 0;
    for (int i = 0; i < nr_lines; i++)
    {
        char *line = lines[i];
        if (*line != '{') continue;
        
        line += 3;
        num_t x = parse_number(&line);
        line += 3;
        num_t m = parse_number(&line);
        line += 3;
        num_t a = parse_number(&line);
        line += 3;
        num_t s = parse_number(&line);
        
        char *rule = "in";
        int rule_len = 2;
        
        printf("Go\n");
        for (bool go = TRUE; go;)
        {
            printf("Rule %s %d\n", rule, rule_len);
            int r = 0;
            for (; r < nr_lines; r++)
                if (strncmp(lines[r], rule, rule_len) == 0 && lines[r][rule_len] == '{')
                    break;
            
            char *line = lines[r] + rule_len + 1;
            
            printf("Rule %s\n", lines[r]);
            for (;;)
            {
                printf(" At %s\n", line);
                bool accept = TRUE;
                if (line[1] == '<' || line[1] == '>')
                {
                    num_t var_value = 0;
                    
                    switch (*line++)
                    {
                        case 'x' : var_value = x; break;
                        case 'm' : var_value = m; break;
                        case 'a' : var_value = a; break;
                        case 's' : var_value = s; break;
                    }
                    char comp = *line++;
                    num_t const_value = parse_number(&line);
                    line++;
                    
                    accept = comp == '<' ? var_value < const_value : var_value > const_value;
                }
                if (accept)
                {
                    printf("Accept '%s'\n", line);
                    if (*line == 'A')
                    {
                        answer += x + m + a + s;
                        go = FALSE;
                        break;
                    }
                    if (*line == 'R')
                    {
                        go = FALSE;
                        break;
                    }
                    rule = line;
                    rule_len = 0;
                    while (rule[rule_len] != ',' && rule[rule_len] != '})
                        rule_len++;
                    break;
                }
                else
                {
                    while (*line != ',' && *line != '}')
                        line++;
                    line++;
                }
            }
        }
    }
    printf("%lld\n", answer);
}
```

At 1:31, after some debugging, the above code returned a correct answer.


### Second part of the puzzle.

Wow, the second part is really at a completely different level. Okay, maybe not.
Just have to make it recursive. Lets first change the above code to something
recursive.

```c

int main(int argc, char *argv[])
{
    ...
    solve2();
}

num_t answer2 = 0;


void solve2()
{
    for (int i = 0; i < nr_lines; i++)
    {
        char *line = lines[i];
        if (*line != '{') continue;
        
        line += 3;
        num_t x = parse_number(&line);
        line += 3;
        num_t m = parse_number(&line);
        line += 3;
        num_t a = parse_number(&line);
        line += 3;
        num_t s = parse_number(&line);

        find("in", 2, x, m, a, s);
    }

    printf("%lld\n", answer2);
}

void find(char *rule, int rule_len, num_t x, num_t m, num_t a, num_t s)
{    
    //printf("Go\n");
    for (bool go = TRUE; go;)
    {
        //printf("Rule %s %d\n", rule, rule_len);
        int r = 0;
        for (; r < nr_lines; r++)
            if (strncmp(lines[r], rule, rule_len) == 0 && lines[r][rule_len] == '{')
                break;
        
        char *line = lines[r] + rule_len + 1;
        
        //printf("Rule %s\n", lines[r]);
        for (;;)
        {
            //printf(" At %s\n", line);
            bool accept = TRUE;
            if (line[1] == '<' || line[1] == '>')
            {
                num_t var_value = 0;
                
                switch (*line++)
                {
                    case 'x' : var_value = x; break;
                    case 'm' : var_value = m; break;
                    case 'a' : var_value = a; break;
                    case 's' : var_value = s; break;
                }
                char comp = *line++;
                num_t const_value = parse_number(&line);
                line++;
                
                accept = comp == '<' ? var_value < const_value : var_value > const_value;
            }
            if (accept)
            {
                //printf("Accept '%s'\n", line);
                if (*line == 'A')
                {
                    answer2 += x + m + a + s;
                    go = FALSE;
                    return;
                }
                if (*line == 'R')
                {
                    go = FALSE;
                    return;
                }
                int next_rule_len = 0;
                while (line[next_rule_len] != ',' && line[next_rule_len] != '})
                    next_rule_len++;
                find(line, next_rule_len, x, m, a, s);
                return;
            }
            else
            {
                while (*line != ',' && *line != '}')
                    line++;
                line++;
            }
        }
    }
}
```

Okay, that returns the correct answer. Now, left rebuild it.

```c

void solve2()
{
    find_all("in,", 1, 4000, 1, 4000, 1, 4000, 1, 4000);

    printf("%lld\n", answer2);
}

num_t min(num_t a, num_t b) { return a < b ? a : b; }
num_t max(num_t a, num_t b) { return a > b ? a : b; }

int depth = 0;

void find_all(char *rule, 
        num_t min_x, num_t max_x, 
        num_t min_m, num_t max_m, 
        num_t min_a, num_t max_a,
        num_t min_s, num_t max_s)
{
    printf("%*.*sFind %lld-%lld %lld-%lld %lld-%lld %lld-%lld %s\n", depth, depth, "",
           min_x, max_x, min_m, max_m, min_a, max_a, min_s, max_s, rule); 
    if (max_x < min_x || max_m < min_m || max_a < min_a || max_s < min_s)
        return;
        
    if (*rule == 'R')
        return;
    if (*rule == 'A')
    {
        printf("%*.*s Add %lld\n", depth, depth, "", (max_x - min_x + 1) * (max_m - min_m + 1) * (max_a - min_a + 1) * (max_s - min_s + 1));
        answer2 += (max_x - min_x + 1) * (max_m - min_m + 1) * (max_a - min_a + 1) * (max_s - min_s + 1);
        return;
    }
    depth += 2;
    
    int rule_len = 0;
    for (char *s = rule; *s != ',' && *s != '}'; s++)
        rule_len++;
        
    //printf("Go\n");
    for (bool go = TRUE; go;)
    {
        //printf("Rule %s %d\n", rule, rule_len);
        int r = 0;
        for (; r < nr_lines; r++)
            if (strncmp(lines[r], rule, rule_len) == 0 && lines[r][rule_len] == '{')
                break;
        
        char *line = lines[r] + rule_len + 1;
        
        printf("%*.*s- Rule %s\n", depth, depth, "", lines[r]);
        for (;;)
        {
            printf("%*.*s- At %s\n", depth, depth, "", line);
            if (line[1] == '<' || line[1] == '>')
            {
                char var = *line++;
                char comp = *line++;
                num_t value = parse_number(&line);
                line++;
                if (var == 'x' && comp == '<')
                {
                    find_all(line, min_x, min(value - 1, max_x), min_m, max_m, min_a, max_a, min_s, max_s);
                    min_x = max(value, min_x);
                }
                else if (var == 'x' && comp == '>')
                {
                    find_all(line, max(value + 1, min_x), max_x, min_m, max_m, min_a, max_a, min_s, max_s);
                    max_x = min(value, max_x);
                }
                else if (var == 'm' && comp == '<')
                {
                    find_all(line, min_x, max_x, min_m, min(value - 1, max_m), min_a, max_a, min_s, max_s);
                    min_m = max(value, min_m);
                }
                else if (var == 'm' && comp == '>')
                {
                    find_all(line, min_x, max_x, max(value + 1, min_m), max_m, min_a, max_a, min_s, max_s);
                    max_m = min(value, max_m);
                }
                else if (var == 'a' && comp == '<')
                {
                    find_all(line, min_x, max_x, min_m, max_m, min_a, min(value - 1, max_a), min_s, max_s);
                    min_a = max(value, min_a);
                }
                else if (var == 'a' && comp == '>')
                {
                    find_all(line, min_x, max_x, min_m, max_m, max(value + 1, min_a), max_a, min_s, max_s);
                    max_a = min(value, max_a);
                }
                else if (var == 's' && comp == '<')
                {
                    find_all(line, min_x, max_x, min_m, max_m, min_a, max_a, min_s, min(value - 1, max_s));
                    min_s = max(value, min_s);
                }
                else if (var == 's' && comp == '>')
                {
                    find_all(line, min_x, max_x, min_m, max_m, min_a, max_a, max(value + 1, min_s), max_s);
                    max_s = min(value, max_s);
                }

                while (*line != ',' && *line != '}')
                    line++;
                line++;
            }
            else
            {
                find_all(line, min_x, max_x, min_m, max_m, min_a, max_a, min_s, max_s);
                depth -= 2;
                return;
            }
        }
    }
}
```

At 2:48, after some extensive debugging, where I fixed one conceptual bug, and some
mixed up variable names, I submitted the correct answer for the second part (after
having found the correct answer for the example input) with the above code.


### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day19.md >day19.c; gcc -g -Wall day19.c -o day19; ./day19
```

