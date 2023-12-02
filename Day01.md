# Day 1 of Advent of Code 2023

I started at 7:50 (CET). I read the puzzle text and downloaded my
puzzle input. I am going to try to do this in one try:

```c
#include "stdio.h"

int main(int argc, char *argv)
{
	FILE *f = fopen("input/day01.txt", "r");
	
	int sum = 0;
	
	char ch = fgetc(f);
	while (!feof(f))
	{
		char first = ' ';
		char last = ' ';
		
		while (!feof(f) && ch != '\n')
		{
			if ('0' <= ch && ch <= '9')
			{
				if (first == ' ')
					first = ch;
				last = ch;
			}
			ch = fgetc(f);
		}
		if (first == ' ' || last == ' ')
		{
			printf("Error\n");
			break;
		}
		sum += 10 * (first - '0') + (last - '0');
	}
	if (!feof(f))
		ch = fgetc(f);

	printf("%d\n", sum);
}
```

I am getting the following output:
```
    1 | /* Day01.md (7.10) `<stdio.h>\?` expected:
      | ^
```
This is the output of the `MarkDownC` program, but I have no idea what it
means. Must be because it is some time ago that I used it.

Okay, it looks like the program does not support the `<stdio.h>` notation
first, I changed that into `"stdio.h"` in above.

Then the second error, I got was from the C compiler, complaining that it
does not know the `fgetch` function. I replaced that by `fgetc` (as suggested
by the compiler) in the above code.

Now the program runs, but it reports and `Error`. I see that I have placed
a statement in the wrong location. The statement for processing the end-of-line
character was accidently placed outside of the loop. Below the code where it
is put inside the loop.

```c
#include "stdio.h"

int main(int argc, char *argv)
{
	FILE *f = fopen("input/day01.txt", "r");
	
	int sum = 0;
	
	char ch = fgetc(f);
	while (!feof(f))
	{
		char first = ' ';
		char last = ' ';
		
		while (!feof(f) && ch != '\n')
		{
			if ('0' <= ch && ch <= '9')
			{
				if (first == ' ')
					first = ch;
				last = ch;
			}
			ch = fgetc(f);
		}
		if (first == ' ' || last == ' ')
		{
			printf("Error\n");
			break;
		}
		sum += 10 * (first - '0') + (last - '0');

		if (!feof(f))
			ch = fgetc(f);
	}

	printf("%d\n", sum);
}
```

At 8:18, it earned my first star. Lets continue with the second part
of the puzzle, which I guess has to do with the written numbers in the
input text. And it indead does.

The above program read the input character by character. That is not
really a good approach for the second part of the puzzle. It is better
to read the input line by line and put it into a buffer. I looked at
the input and I did not see any exceptional long lines, so, I assume a
buffer of 100 characters. Let try to do this second puzzle in one try
as well:

```c
#include "string.h"

#include "string.h"

int main(int argc, char *argv)
{
	FILE *f = fopen("input/day01.txt", "r");

	const char *digit_words[10] = { "one", "two", "three", "four", "five", "six", "seven", "eight", "nine" };

	int sum = 0;
	
	char line[100];
	while (fgets(line, 99, f))
	{
		int first = -1;
		int last = -1;
		
		for (char *s = line; *s != '\0'; s++)
		{
			int digit = -1;
			if ('0' <= *s && *s <= '9')  
			{
				digit = *s - '0';
			}
			else if (strncmp(s, "one", 3) == 0)
			{
				for (int i = 0; i < 10; i++)
				{
					int len = strlen(digit_words[i]);
					if (strncmp(s, digit_words[i], len))
					{
						digit = i + 1;
						s += len - 1;
						break;
					}
				}
			}
			if (digit != -1)
			{
				if (first == -1)
					first = digit;
				last = digit;
			}
		}

		if (first == -1 || last == -1)
		{
			printf("Error\n");
			break;
		}
		sum += 10 * first + last;
	}

	printf("%d\n", sum);
}
```

I fixed three errors in the above code. I had forgotten to add `== 0)` behind
the line with `strncmp`. I had forgotten to define the `sum` variable and
to print the result.

When I entered the answer, I got the reply: 'That's not the right answer'.
So there is a bug in the above. I added a statement to print the values
of first and last. I quickly noticed that it did not parse the the numbers
correctly. Oops, I noticed that the above code is really mixed up. At first,
I wanted to use a `strncmp` for each of the written words for the digits,
but then got an idea to put them in an array, being afraid that I would make
a mistake in the length argument. When I fixed this, the program returned
a 'Segmentation fault'. I quickly located the cause, I had defined an array
of length 10, whereas there are only 9 words that are used. 'zero' is missing.
(I already checked that it did not occur in the input using the find function
of the browser.) This resulted in the following program:

```c
int main(int argc, char *argv)
{
	FILE *f = fopen("input/day01.txt", "r");

	const char *digit_words[9] = { "one", "two", "three", "four", "five", "six", "seven", "eight", "nine" };

	int sum = 0;
	
	char line[100];
	while (fgets(line, 99, f))
	{
		int first = -1;
		int last = -1;
		
		for (char *s = line; *s != '\0'; s++)
		{
			int digit = -1;
			if ('0' <= *s && *s <= '9')  
			{
				digit = *s - '0';
			}
			else
			{
				for (int i = 0; i < 9; i++)
				{
					int len = strlen(digit_words[i]);
					if (strncmp(s, digit_words[i], len) == 0)
					{
						digit = i + 1;
						s += len - 1;
						break;
					}
				}
			}
			if (digit != -1)
			{
				if (first == -1)
					first = digit;
				last = digit;
			}
		}

		if (first == -1 || last == -1)
		{
			printf("Error\n");
			break;
		}
		printf("%d %d\n", first, last);
		sum += 10 * first + last;
	}

	printf("%d\n", sum);
}

```

This still did not return the correct answer. I performed some
debugging. In the end added code to reproduce the input and it
returned the exact input (except for the input):

```c
int main(int argc, char *argv)
{
	FILE *f = fopen("input/day01.txt", "r");

	const char *digit_words[9] = { "one", "two", "three", "four", "five", "six", "seven", "eight", "nine" };

	int sum = 0;
	
	char line[100];
	while (fgets(line, 99, f))
	{
		int first = -1;
		int last = -1;
		
		for (char *s = line; *s != '\0'; s++)
		{
			int digit = -1;
			if ('0' <= *s && *s <= '9')  
			{
				digit = *s - '0';
				printf("%c", *s);
			}
			else
			{
				for (int i = 0; i < 9; i++)
				{
					int len = strlen(digit_words[i]);
					if (strncmp(s, digit_words[i], len) == 0)
					{
						digit = i + 1;
						//s += len - 1;
						printf("%s", digit_words[i]);
						break;
					}
				}
			}
			
			if (digit != -1)
			{
				//printf("%d ", digit);
				if (first == -1)
					first = digit;
				last = digit;
			}
			else
				printf("%c", *s);
			
		}

		if (first == -1 || last == -1)
		{
			printf("Error\n");
			break;
		}
		//printf(": %d %d\n", first, last);
		sum += 10 * first + last;
	}

	printf("%d\n", sum);
}
```
There seems to be no error. But then I noticed that the last digit in
'twone' is not 'two' but 'one'. So lets comment out the `s += len - 1` line
in the above code. It does return a different answer. And indeed, it is
the correct answer. I found it at 9:14.

This definitely took more time than I had expected.

### Some standard definitions

```c
typedef int bool;
#define FALSE 0
#define TRUE 1
```

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Day01.md >day01.c; gcc day01.c -o day01; ./day01
```
