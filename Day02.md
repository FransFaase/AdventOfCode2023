# Day 2 of Advent of Code 2023

I started at 8:37 (CET). I read the puzzle text and downloaded my
puzzle input. At 8:42, I started writing this page. Although for
the first puzzle you do not have to descern between the games, I
am going to write the code for parsing the lines, such that I can
later adapt it for the second puzzle. (You never know if that is
really needed, though.). I am first going to write some function
to sum the numbers of the correct lines:

```c
num_t sum_good1 = 0;

void process_line(int n, char *line)
{
	if (good1(line))
		sum_good1 += n;
}
```

Next a function to check if a line is correct for the first puzzle:

```c
bool good1(char *line)
{
	char *s = line;
	for (;;)
	{
		for (;;)
		{
			int c = 0;
			for (; '0' <= *s && *s <= '9'; s++)
				c = 10 * c + *s - '0';
			if (*s == ' ')
				s++;
			if (strncmp(s, "red", 3) == 0)
			{
				s += 3;
				if (c > 12)
					return FALSE;
			}
			else if (strncmp(s, "green", 5) == 0)
			{
				s += 5;
				if (c > 13)
					return FALSE;
			}
			else if (strncmp(s, "blue", 4) == 0)
			{
				s += 4;
				if (c > 14)
					return FALSE;
			}
			if (*s != ',')
				break;
			s++;
		}
		if (*s != ';')
			break;
		s++;
	}
	return TRUE;
}
```

Now, I still need to write the function that reads a line, parses the game number
and calls the `process_line` function.

```c
#include "stdio.h"


int main(int argc, char *argv)
{
	FILE *f = fopen("input/day02.txt", "r");
	
	char line[500];
	while (fgets(line, 499, f))
	{
		char *s = line + 5;
		
		int n = 0;	
			for (; '0' <= *s && *s <= '9'; s++)
				n = 10 * n + *s - '0';
		process_line(n, s + 2);
	}
	printf("%lld\n", sum_good1);
}
```

At 9:00, I was ready writing the above. I suspect it will still contain some
errors. Lets see.

It did not show anything. the `day02.c` file looks rather empty. There is syntax
error on line 48 of this file. Okay, the `if` was missing. I have added it.

At 9:05, second try. Now `MarkDownC` processes the file correctly, but the
`gcc` compiler reports a number of errors. On line 39 there is an undeclared `n`.
Changed (three times) the `n` into `c` in the above code.

At 9:09, third try. It reports that `false` is not defined. I replaced `false`
and `true` with the all capital spelling in the above code. Also added an `l`
before the `d` in the print statement.

At 9:12, fourth try. Should have been two `l`. But it did return an answer.
But the answer is too high.

At 9:15, I noticed that at line 50, it said `s += 3;`. I changed it into `s += 4;`.
Which still returns the same incorrect answer. Lets copy the above code and add
a print function:

```c
bool good1(char *line)
{
	char *s = line;
	for (;;)
	{
		for (;;)
		{
			if (*s == ' ')
				s++;
			int c = 0;
			for (; '0' <= *s && *s <= '9'; s++)
				c = 10 * c + *s - '0';
			if (*s == ' ')
				s++;
			if (strncmp(s, "red", 3) == 0)
			{
				s += 3;
				if (c > 12)
					return FALSE;
			}
			else if (strncmp(s, "green", 5) == 0)
			{
				s += 5;
				if (c > 13)
					return FALSE;
			}
			else if (strncmp(s, "blue", 4) == 0)
			{
				s += 4;
				if (c > 14)
					return FALSE;
			}
			if (*s != ',')
				break;
			s++;
		}
		if (*s != ';')
			break;
		s++;
	}
	if (*s != '\n')
		printf("Rest: %s\n", s);
	return TRUE;
}
```

At 9:21, I added two lines to skip some space characters after
the `;` and `,` characters. Now it gives a different answer.

At 9:22, it turns out to be correct. Lets read the second puzzle
and think about it while I go on my usual business for this
saturdays.

At 9:27, I finished reading it, it does not look that hard. And
actually does not require to make a difference between the 'games'
on a line. Just find the maximums for each colour on each line.

At 9:29, I stopped.

### Second part

At 18:27, I contined. I think, I will just copy the parsing code
from above:
```c
num_t sum_sol2 = 0;

void process_line(int n, char *line)
{
	...
	sum_sol2 += calc2(line);
}

int main(int argc, char *argv)
{
	...
	printf("%lld\n", sum_sol2);
}


int calc2(char *line)
{
	char *s = line;
	int min_red = 0;
	int min_green = 0;
	int min_blue = 0;
	for (;;)
	{
		for (;;)
		{
			if (*s == ' ')
				s++;
			int c = 0;
			for (; '0' <= *s && *s <= '9'; s++)
				c = 10 * c + *s - '0';
			if (*s == ' ')
				s++;
			if (strncmp(s, "red", 3) == 0)
			{
				s += 3;
				if (c > min_red)
					min_red = c;
			}
			else if (strncmp(s, "green", 5) == 0)
			{
				s += 5;
				if (c > min_green)
					min_green = c;
			}
			else if (strncmp(s, "blue", 4) == 0)
			{
				s += 4;
				if (c > min_blue)
					min_blue = c;
			}
			if (*s != ',')
				break;
			s++;
		}
		if (*s != ';')
			break;
		s++;
	}
	if (*s != '\n')
		printf("Rest: %s\n", s);
	return min_red * min_green * min_blue;
}

```

It worked in one time and gave the right answer at 18:35.
 

### Some standard definitions

```c
typedef long long num_t;
typedef int bool;
#define FALSE 0
#define TRUE 1
```

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Day02.md >day02.c; gcc day02.c -o day02; ./day02
```
