# Some standard definition and generic code

First some standard includes:

```c
#include "stdio.h"
#include "stdlib.h"
#include "malloc.h"
#include "unistd.h"
```

Some standard type definitions:
```c
typedef long long num_t;
typedef int bool;
#define FALSE 0
#define TRUE 1

```

A function to read the input into a array of lines:

```c
typedef char *char_p;

char **lines;
int nr_lines;

void read_file(const char *name)
{
    nr_lines = 0;
    FILE *f = fopen(name, "r");
    if (f == 0)
    {
        return;
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
            nr_lines++;
    
    lines = (char**)malloc(nr_lines * sizeof(char_p));
    char *s = data;
    for (int i = 0; i < nr_lines; i++)
    {
        lines[i] = s;
        
        while (*s != '\n')
        {
            if (*s == '\r)
                *s = '\0';
            s++;
        }
        *s++ = '\0';
    }
}
```

A function to parse a number from a string.

```c

bool is_digit(char c) { return '0' <= c && c <= '9'; }

num_t parse_number(char **s)
{
    int sign = 1;
    if (**s == '-')
    {
        sign = -1;
        (*s)++;
    }
    num_t value = 0;
    for (;is_digit(**s); (*s)++)
        value = 10 * value + **s - '0';
    return sign * value;
}
```

### Range functions

```c
typedef struct range range_t;
struct range
{
    num_t from;
    num_t to;
    range_t *next;
};

void ranges_add_range(range_t **ref_ranges, num_t from, num_t to)
{
    while (*ref_ranges != 0 && (*ref_ranges)->to < from)
        ref_ranges = &(*ref_ranges)->next;
    if (*ref_ranges == 0 || to < (*ref_ranges)->from)
    {
        range_t *new_range = (range_t*)malloc(sizeof(range_t));
        new_range->from = from;
        new_range->to = to;
        new_range->next = *ref_ranges;
        *ref_ranges = new_range;
        return;
    }
    if ((*ref_ranges)->from < from)
        from = (*ref_ranges)->from;
    while ((*ref_ranges)->next != 0 && (*ref_ranges)->next->from <= to)
        *ref_ranges = (*ref_ranges)->next;
    (*ref_ranges)->from = from;
    if ((*ref_ranges)->to < to)
        (*ref_ranges)->to = to;
}

void ranges_add_ranges(range_t **ref_ranges, range_t *ranges)
{
    for (; ranges != 0; ranges = ranges->next)
        ranges_add_range(ref_ranges, ranges->from, ranges->to);
}

void ranges_print(range_t *ranges, FILE *f)
{
    for (; ranges != 0; ranges = ranges->next)
    {
        fprintf(f, "%lld:%lld", ranges->from, ranges->to);
        if (ranges->next != 0)
            fprintf(f, ", ");
    }
}

range_t *ranges_extract_range(range_t **ref_ranges, num_t from, num_t to)
{
    while (*ref_ranges != 0 && (*ref_ranges)->to < from)
        ref_ranges = &(*ref_ranges)->next;
        
    if (*ref_ranges == 0 || to < (*ref_ranges)->from)
        return 0;

    if ((*ref_ranges)->from < from)
    {
        // split the range
        range_t *new_range = (range_t*)malloc(sizeof(range_t));
        new_range->from = (*ref_ranges)->from;
        new_range->to = from;
        new_range->next = (*ref_ranges);
        (*ref_ranges)->from = from;
        *ref_ranges = new_range;
        ref_ranges = &new_range->next;
    }    

    range_t *result = *ref_ranges;
    range_t **ref_result = &result;
    while ((*ref_ranges) != 0 && (*ref_ranges)->to <= to)
    {
        ref_result = &(*ref_result)->next;
        *ref_ranges = *ref_result;
    }
    if (*ref_ranges != 0)
    {
        if (to < (*ref_ranges)->from)
            (*ref_result) = 0;
        else
        {
            *ref_result = (range_t*)malloc(sizeof(range_t));
            (*ref_result)->from = (*ref_ranges)->from;
            (*ref_result)->to = to;
            (*ref_result)->next = 0;
            (*ref_ranges)->from = to;
        }
    }
    return result;
}

void test_ranges_impl()
{
    range_t *ranges = 0;
    ranges_add_range(&ranges, 30, 40);
    ranges_print(ranges, stdout); printf("\n");
    ranges_add_range(&ranges, 10, 20);
    ranges_print(ranges, stdout); printf("\n");
    ranges_add_range(&ranges, 70, 80);
    ranges_print(ranges, stdout); printf("\n");
    ranges_add_range(&ranges, 50, 60);
    ranges_print(ranges, stdout); printf("\n");
    ranges_add_range(&ranges, 10, 20);
    ranges_print(ranges, stdout); printf("\n");
    ranges_add_range(&ranges, 9, 20);
    ranges_print(ranges, stdout); printf("\n");
    ranges_add_range(&ranges, 8, 15);
    ranges_print(ranges, stdout); printf("\n");
    ranges_add_range(&ranges, 8, 21);
    ranges_print(ranges, stdout); printf("\n");
    ranges_add_range(&ranges, 10, 22);
    ranges_print(ranges, stdout); printf("\n");
    ranges_add_range(&ranges, 10, 30);
    ranges_print(ranges, stdout); printf("\n");
    ranges_add_range(&ranges, 10, 55);
    ranges_print(ranges, stdout); printf("\n");
    ranges_add_range(&ranges, 40, 80);
    ranges_print(ranges, stdout); printf("\n");
    ranges = 0;
    ranges_add_range(&ranges, 30, 40);
    ranges_add_range(&ranges, 10, 20);
    ranges_add_range(&ranges, 70, 80);
    ranges_add_range(&ranges, 50, 60);
    ranges_add_range(&ranges, 10, 20);
    range_t *result;
    result = ranges_extract_range(&ranges, 0, 5);
    ranges_print(ranges, stdout); printf(" | "); ranges_print(result, stdout); printf("\n");
    result = ranges_extract_range(&ranges, 21, 29);
    ranges_print(ranges, stdout); printf(" | "); ranges_print(result, stdout); printf("\n");
    result = ranges_extract_range(&ranges, 90, 0);
    ranges_print(ranges, stdout); printf(" | "); ranges_print(result, stdout); printf("\n");
    result = ranges_extract_range(&ranges, 0, 11);
    ranges_print(ranges, stdout); printf(" | "); ranges_print(result, stdout); printf("\n");
    result = ranges_extract_range(&ranges, 25, 31);
    ranges_print(ranges, stdout); printf(" | "); ranges_print(result, stdout); printf("\n");
    result = ranges_extract_range(&ranges, 19, 22);
    ranges_print(ranges, stdout); printf(" | "); ranges_print(result, stdout); printf("\n");
    result = ranges_extract_range(&ranges, 13, 14);
    ranges_print(ranges, stdout); printf(" | "); ranges_print(result, stdout); printf("\n");
    result = ranges_extract_range(&ranges, 14, 19);
    ranges_print(ranges, stdout); printf(" | "); ranges_print(result, stdout); printf("\n");
    result = ranges_extract_range(&ranges, 12, 52);
    ranges_print(ranges, stdout); printf(" | "); ranges_print(result, stdout); printf("\n");
    result = ranges_extract_range(&ranges, 58, 80);
    ranges_print(ranges, stdout); printf(" | "); ranges_print(result, stdout); printf("\n");
    result = ranges_extract_range(&ranges, 0, 90);
    ranges_print(ranges, stdout); printf(" | "); ranges_print(result, stdout); printf("\n");
}

```

### Greatest Common Divisor and Smallest Common Multiplier

```c
num_t gcd(num_t a, num_t b)
{
    while (a != 0)
    {
        num_t c = b % a;
        b = a;
        a = c;
    }
    return b;
}

num_t scm(num_t a, num_t b) { return a * (b / gcd(a, b)); }
num_t num_t_abs(num_t a) { return a < 0 ? -a : a; }
int num_t_sign(num_t a) { return a < 0 ? -1 : a > 0 ? 1 : 0; }

```

### Basic string function

```c
char *copy_str(char *s)
{
	char *r = (char*)malloc(strlen(s) + 1);
	strcpy(r, s);
	return r;
}
```
 
