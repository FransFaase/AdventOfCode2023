# Day 9 of Advent of Code 2023


At 8:45, I started reading the puzzle. 

At 9:50, I started writing code. This is simply polygonal continuation.
I draw some thing on paper to figure out some details.

```c
int main(int argc, char *argv[])
{
	read_file("input/day09.txt");
	solve1();
}

void solve1()
{
	num_t sum = 0;
	for (int i = 0; i < nr_lines; i++)
	{
		num_t coeff[30];
		char *s = lines[i];
		int j;
		for (j = 0; is_digit(*s); j++)
		{
			num_t num = parse_number(&s);
			if (*s == ' ') s++;
			
			for (int k = 0; k <= j; k++)
			{
				int d = num - coeff[k];
				coeff[k] = num;
				num = d;
			}
		}
		for (int k = 0; k < j; k++)
			sum += coeff[k];
	}
	
	printf("%lld\n", sum);
}
```

At 9:04, I submitted the solution, but it was incorrect. I started to
debug it with adding some print statements as shown below.

```c
void solve1()
{
	num_t sum = 0;
	for (int i = 0; i < 1 /*nr_lines*/; i++)
	{
		num_t coeff[30];
		char *s = "1 3 6 10 15 21"; //lines[i];
		int j;
		for (j = 0; is_digit(*s) || *s == '-'; j++)
		{
			num_t num = parse_number(&s);
			if (*s == ' ') s++;
			
			printf("%lld: ", num);
			
			for (int k = 0; k <= j; k++)
			{
				int d = num - coeff[k];
				coeff[k] = num;
				printf(" %lld", num);
				num = d;
			}
			printf("\n");
		}
		for (int k = 0; k < j; k++)
			sum += coeff[k];
	}
	
	printf("%lld\n", sum);
}
```

At 9:09, I realize that my code did not deal with negative numbers
and that my initial algorithm was (very likely) correct. So, I went
back to it:

```c
void solve1()
{
	num_t sum = 0;
	for (int i = 0; i < nr_lines; i++)
	{
		num_t coeff[100];
		char *s = lines[i];
		int j;
		for (j = 0; *s != '\0'; j++)
		{
			num_t num = parse_number(&s);
			if (*s == ' ') s++;
			
			for (int k = 0; k <= j; k++)
			{
				int d = num - coeff[k];
				coeff[k] = num;
				num = d;
			}
		}
		for (int k = 0; k < j; k++)
			sum += coeff[k];
	}
	
	printf("%lld\n", sum);
}
```

At 9:18, I found the correct answer to the first half of the puzzle.
Still had to fix the `parse_number` function in [my standard library](Std.md)
to deal with negative numbers. (I got two segment faults in different parts
of the code.)

### Second part of the puzzle.

Okay, that is just reversing the input and then execute the same algorithm.
So, lets read the input in a array and process it in the reverse order.

```c
int main(int argc, char *argv[])
{
    ...
	solve2();
}

void solve2()
{
	num_t sum = 0;
	for (int i = 0; i < nr_lines; i++)
	{
		num_t numbers[100];
		int n = 0;
		char *s = lines[i];
		while (*s != '\0')
		{
			numbers[n++] = parse_number(&s);
			if (*s == ' ') s++;
		}
		
		num_t coeff[100];
		int j;
		for (j = 0; j < n; j++)
		{
			num_t num = numbers[n - 1 - j];
			if (*s == ' ') s++;
			
			for (int k = 0; k <= j; k++)
			{
				int d = num - coeff[k];
				coeff[k] = num;
				num = d;
			}
		}
		for (int k = 0; k < j; k++)
			sum += coeff[k];
	}
	
	printf("%lld\n", sum);
}
```

At 9:24 found the solution for the second half of the puzzle.

	
### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day09.md >day09.c; gcc day09.c -o day09; ./day09
```

