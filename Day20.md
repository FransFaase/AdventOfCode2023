# Day 20 of Advent of Code 2023

I started reading the puzzle at 12:55 (CET), but only for a little while.

I continued around 17:00 trying to understand the puzzle.
It looks like the conjunction module needs to remember some state
received from it inputs. So, it does require to know which pulses
have been received. It looks like you need some queue for the pulses.
At each position in the queue, you need to know the source, the
destination and the type. So something like:

```c
typedef struct pulse pulse_t;
struct pulse
{
    module_t *from;
    module_t *to;
    bool is_high;
    pulse_t *next;
};
```

We also need to model the modules. They have a list of outputs.
And for the conjunction modules a memory for the inputs.

```c
typedef struct module module_t;
struct module
{
    char *name;
    char type;
    bool on;
    cable_t *outputs;
    cable_t *inputs;
    module_t *next;
};

typedef struct cable cable_t;
struct cable
{
    module_t *from;
    bool received_high;
    cable_t *next_input;
    module_t *to;
    cable_t *next_output;
};

```

We need some kind of function that given a name, returns a module:

```c

module_t *modules = 0;

module_t *get_module(char *name)
{
    for (module_t *module = modules; module != 0; module = module->next)
        if (strcmp(module->name, name) == 0)
            return module;
    module_t *module = (module_t*)malloc(sizeof(module_t));
    module->name = copy_str(name);
    module->type = ' ';
    module->on = FALSE;
    module->outputs = 0;
    module->inputs = 0;    
    module->next = modules;
    modules = module;
    return module;
}
```

Lets write a function to parse the input:

```c

module_t *broadcaster = 0;

void parse_input()
{
    for (int i = 0; i < nr_lines; i++)
    {
        char *s = lines[i];
        module_t *module = 0;
        if (strncmp(s, "broadcaster", 11) == 0)
        {
            module = broadcaster = get_module("broadcaster");
            s += 15;
        }
        else
        {
            char type = *s++;
            char name[5];
            char *n = name;
            for (; *s != ' '; s++)
                *n++ = *s;
            *n = '\0';
            module = get_module(name);
            module->type = type;
            s += 4;
        }

        for (cable_t **ref_output = &module->outputs; ; ref_output = &(*ref_output)->next_output)
        {
            char name[5];
            char *n = name;
            for (; *s != '\0' && *s != ','; s++)
                *n++ = *s;
            *n = '\0';
            
            module_t *to_module = get_module(name);
            *ref_output = (cable_t*)malloc(sizeof(cable_t));
            (*ref_output)->to = to_module;
            (*ref_output)->next_output = 0;
            (*ref_output)->from = module;
            (*ref_output)->received_high = FALSE;
            (*ref_output)->next_input = to_module->inputs;
            to_module->inputs = (*ref_output);
            
            if (*s == '\0')
                break;
            s += 2;
        }
    }
    
    for (module_t *module = modules; module != 0; module = module->next)
    {
        printf("Module %c %s\n", module->type, module->name);
        for (cable_t *cable = module->inputs; cable != 0; cable = cable->next_input)
            printf(" input %s\n", cable->from->name);
        for (cable_t *cable = module->outputs; cable != 0; cable = cable->next_output)
            printf(" output %s\n", cable->to->name);
    }
}


int main(int argc, char *argv[])
{
    read_file("input/day20.txt");
    solve1();
}

void solve1()
{
    parse_input();
}
```

Now lets add the pulse processing:

```c

pulse_t *new_pulse(module_t *from, bool is_high, module_t *to)
{
    pulse_t *pulse = (pulse_t*)malloc(sizeof(pulse_t));
    pulse->from = from;
    pulse->is_high = is_high;
    pulse->to = to;
    pulse->next = 0;
    printf(" - %s %s %s\n", from->name, is_high ? "high" : "low", to->name);
    return pulse;
}

void solve1()
{
    ...
    num_t nr_low = 0;
    num_t nr_high = 0;
    for (int i = 0; i < 1000; i++)
    {
        pulse_t *pulses = 0;
        pulse_t **ref_next_pulse = &pulses;
        for (cable_t *cable = broadcaster->outputs; cable != 0; cable = cable->next_output)
        {
            (*ref_next_pulse) = new_pulse(broadcaster, FALSE, cable->to);
            ref_next_pulse = &(*ref_next_pulse)->next;
        }
        nr_low++;
        
        for (pulse_t *pulse = pulses; pulse != 0; pulse = pulse->next)
        {
            printf("%s -%s-> %s\n", pulse->from->name, pulse->is_high ? "high" : "low", pulse->to->name);
            (*(pulse->is_high ? &nr_high : &nr_low))++;
            if (pulse->to->type == '%')
            {
                if (!pulse->is_high)
                {
                    pulse->to->on = !pulse->to->on;
                    for (cable_t *cable = pulse->to->outputs; cable != 0; cable = cable->next_output)
                    {
                        (*ref_next_pulse) = new_pulse(pulse->to, pulse->to->on, cable->to);
                        ref_next_pulse = &(*ref_next_pulse)->next;
                    }
                }
            }
            else
            {
                bool all_high = TRUE;
                for (cable_t *cable = pulse->to->inputs; cable != 0; cable = cable->next_input)
                {
                    if (cable->from == pulse->from)
                        cable->received_high = pulse->is_high;
                    printf("(input from %s %d)\n", cable->from->name, cable->received_high);
                    if (!cable->received_high)
                        all_high = FALSE;
                }
                printf(" -- all high = %d\n", all_high);
                for (cable_t *cable = pulse->to->outputs; cable != 0; cable = cable->next_output)
                {
                    (*ref_next_pulse) = new_pulse(pulse->to, !all_high, cable->to);
                    ref_next_pulse = &(*ref_next_pulse)->next;
                }
            }
        }
    }
    printf("%lld %lld\n", nr_high, nr_low);
    printf("%lld\n", nr_high * nr_low);
}
```

### Second part of the puzzle

The second part looks not that complicated.


```c
int main(int argc, char *argv[])
{
    ...
    solve2();
}

pulse_t *prev_pulses = 0;

pulse_t *new_pulse(module_t *from, bool is_high, module_t *to)
{
    pulse_t *pulse = 0;
    if (prev_pulses != 0)
    {
        pulse = prev_pulses;
        prev_pulses = pulse->next;
    }
    else
        pulse = (pulse_t*)malloc(sizeof(pulse_t));
    pulse->from = from;
    pulse->is_high = is_high;
    pulse->to = to;
    pulse->next = 0;
    return pulse;
}

void solve2()
{
    for (module_t *module = modules; module != 0; module = module->next)
        module->on = FALSE;

    module_t *rx = get_module("rx");

    num_t steps = 0;    
    for (bool go = TRUE; go; steps++)
    {
        pulse_t *pulses = 0;
        pulse_t **ref_next_pulse = &pulses;
        for (cable_t *cable = broadcaster->outputs; cable != 0; cable = cable->next_output)
        {
            (*ref_next_pulse) = new_pulse(broadcaster, FALSE, cable->to);
            ref_next_pulse = &(*ref_next_pulse)->next;
        }
        
        for (pulse_t *pulse = pulses; pulse != 0; )
        {
            if (pulse->to == rx && !pulse->is_high)
                go = FALSE;

            if (pulse->to->type == '%')
            {
                if (!pulse->is_high)
                {
                    pulse->to->on = !pulse->to->on;
                    for (cable_t *cable = pulse->to->outputs; cable != 0; cable = cable->next_output)
                    {
                        (*ref_next_pulse) = new_pulse(pulse->to, pulse->to->on, cable->to);
                        ref_next_pulse = &(*ref_next_pulse)->next;
                    }
                }
            }
            else
            {
                bool all_high = TRUE;
                for (cable_t *cable = pulse->to->inputs; cable != 0; cable = cable->next_input)
                {
                    if (cable->from == pulse->from)
                        cable->received_high = pulse->is_high;
                    //printf("(input from %s %d)\n", cable->from->name, cable->received_high);
                    if (!cable->received_high)
                        all_high = FALSE;
                }
                //printf(" -- all high = %d\n", all_high);
                for (cable_t *cable = pulse->to->outputs; cable != 0; cable = cable->next_output)
                {
                    (*ref_next_pulse) = new_pulse(pulse->to, !all_high, cable->to);
                    ref_next_pulse = &(*ref_next_pulse)->next;
                }
            }
            pulse_t *done = pulse;
            pulse = pulse->next;
            done->next = prev_pulses;
            prev_pulses = done;
        }
        if (steps % 1000000 == 0)
            printf("- %lld\n", steps);
    }
    
    printf("%lld\n", steps);
}
```

Okay, it is not that easy.

I let the above program run for a bit more than 3961000000 steps without it finding
an answer. In the meantime, I have been analyzing the input, especially focussing on
the conjunction modules. I got some idea that there are four generators that all have
to come together.

```c
void solve2()
{
    for (module_t *module = modules; module != 0; module = module->next)
        module->on = FALSE;

    num_t prev[4];
    num_t freq[4];
    for (int i = 0; i < 4; i++)
        prev[i] = freq[i] = 0;
    module_t *rx = get_module("rx");

    num_t steps = 0;    
    for (bool go = TRUE; go; steps++)
    {
        pulse_t *pulses = 0;
        pulse_t **ref_next_pulse = &pulses;
        for (cable_t *cable = broadcaster->outputs; cable != 0; cable = cable->next_output)
        {
            (*ref_next_pulse) = new_pulse(broadcaster, FALSE, cable->to);
            ref_next_pulse = &(*ref_next_pulse)->next;
        }
        
        for (pulse_t *pulse = pulses; pulse != 0; )
        {
            if (!pulse->is_high)
            {
                int i = 0;
                for (cable_t *rxinc = rx->inputs->from->inputs; rxinc != 0; rxinc = rxinc->next_input, i++)
                    if (pulse->to == rxinc->from)
                    {
                        freq[i] = steps - prev[i];
                        prev[i] = steps;
                        printf("%s %lld\n", rxinc->from->name, freq[i]);
                    }
            }
            if (pulse->to->type == '%')
            {
                if (!pulse->is_high)
                {
                    pulse->to->on = !pulse->to->on;
                    for (cable_t *cable = pulse->to->outputs; cable != 0; cable = cable->next_output)
                    {
                        (*ref_next_pulse) = new_pulse(pulse->to, pulse->to->on, cable->to);
                        ref_next_pulse = &(*ref_next_pulse)->next;
                    }
                }
            }
            else
            {
                bool all_high = TRUE;
                for (cable_t *cable = pulse->to->inputs; cable != 0; cable = cable->next_input)
                {
                    if (cable->from == pulse->from)
                        cable->received_high = pulse->is_high;
                    //printf("(input from %s %d)\n", cable->from->name, cable->received_high);
                    if (!cable->received_high)
                        all_high = FALSE;
                }
                //printf(" -- all high = %d\n", all_high);
                for (cable_t *cable = pulse->to->outputs; cable != 0; cable = cable->next_output)
                {
                    (*ref_next_pulse) = new_pulse(pulse->to, !all_high, cable->to);
                    ref_next_pulse = &(*ref_next_pulse)->next;
                }
            }
            pulse_t *done = pulse;
            pulse = pulse->next;
            done->next = prev_pulses;
            prev_pulses = done;
        }
        
        num_t answer = 1;
        for (int i = 0; i < 4; i++)
            answer *= freq[i];
        if (answer > 0)
        {
            for (int i = 0; i < 4; i++)
                printf("%lld ", freq[i]);
            
            printf(": %lld\n", answer);
            break;
        }
    }
}
```

At 2:51, December 21, I found the correct answer with the above code. (A little to my
surprise.) I modified the above code a bit, to break the loop as soon as the answer is found.


### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day20.md >day20.c; gcc -g -Wall day20.c -o day20; ./day20
```

