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
            s++;
        *s++ = '\0';
    }
}
```
