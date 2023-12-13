# Day 13 of Advent of Code 2023


At 7:05 (CET), I started reading the puzzle text. The first part looks
easy. I fear that the second part will be complicated. Like stiching
all the patterns together. At 7.12, I started writing this text. Let's
dive in.

```c
int main(int argc, char *argv[])
{
	read_file("input/day13.txt");
	int i = 0;
	while (i < nr_lines)
	{
		char **rect = lines + i;
		int nr_cols = strlen(rect[0]);
		int nr_rows = 0;
		for (; i < nr_lines && lines[i][0] != '\0'; i++)
			nr_rows++;
		if (i < nr_lines)
			i++;
		process(rect, nr_rows, nr_cols);
	}
}

void process(char **rect, int nr_rows, int nr_cols)
{
	solve1(rect, nr_rows, nr_cols);
}

void solve1(char **rect, int nr_rows, int nr_cols)
{
	for (int i = 0; i < nr_rows; i++)
		printf("%s\n", rect[i]);
	printf("\n");
}
```

At 7:23, I verified that the above code does produce the input, with
the exception of an additional empty line at the bottom.

```c
num_t answer1 = 0;

int main(int argc, char *argv[])
{
	...
	printf("%lld\n", answer1);
}

void solve1(char **rect, int nr_rows, int nr_cols)
{
	for (int j = 0; j < nr_cols - 1; j++)
	{
		bool mirror = TRUE;
		for (int j2 = 0; j2 <= j && j + 1 + j2 < nr_cols && mirror; j2++)
			for (int i = 0; i < nr_rows && mirror; i++)
				mirror = rect[i][j - j2] == rect[i][j + 1 + j2];
		
		if (mirror)
		{
			printf("nr_col %d\n", j + 1);
			answer1 += j + 1;
			return;
		}
	}

	for (int i = 0; i < nr_rows - 1; i++)
	{
		bool mirror = TRUE;
		for (int i2 = 0; i2 <= i && i + 1 + i2 < nr_rows && mirror; i2++)
			for (int j = 0; j < nr_cols && mirror; j++)
				mirror = rect[i - i2][j] == rect[i + 1 + i2][j];
		
		if (mirror)
		{
			printf("nr_row %d\n", i + 1);
			answer1 += 100 * (i + 1);
			return;
		}
	}
	printf("None\n");
}
```

At 7:34, finished writing the above.
At 7:26, the above code found the correct answer for the first half of
the puzzle. I had to fix some small syntax errors. I forgot the insert
an `&&` in the condition of the first for loop. I had noticed it, but
forgot to fix it. And I was using `true` instead of `TRUE`. A common error
that I keep on making forgetting that I programming in C and not C++.

### Second part of the puzzle.

Looks not that hard.

```c
num_t answer2 = 0;

int main(int argc, char *argv[])
{
	...
	printf("%lld\n", answer2);
}

void process(char **rect, int nr_rows, int nr_cols)
{
    ...
	solve2(rect, nr_rows, nr_cols);
}

void solve2(char **rect, int nr_rows, int nr_cols)
{
	for (int j = 0; j < nr_cols - 1; j++)
	{
		int nr_smudge = 0;
		for (int j2 = 0; j2 <= j && j + 1 + j2 < nr_cols; j2++)
			for (int i = 0; i < nr_rows; i++)
				if (rect[i][j - j2] != rect[i][j + 1 + j2])
					nr_smudge++;
		
		if (nr_smudge == 1)
		{
			printf("nr_col %d\n", j + 1);
			answer2 += j + 1;
			return;
		}
	}

	for (int i = 0; i < nr_rows - 1; i++)
	{
		int nr_smudge = 0;
		for (int i2 = 0; i2 <= i && i + 1 + i2 < nr_rows; i2++)
			for (int j = 0; j < nr_cols; j++)
				if (rect[i - i2][j] != rect[i + 1 + i2][j])
					nr_smudge++;
		
		if (nr_smudge == 1)
		{
			printf("nr_row %d\n", i + 1);
			answer2 += 100 * (i + 1);
			return;
		}
	}
	printf("None\n");
}
```

At 7:46, the above code returned the correct answer after fixing
two minor errors, namely, forgot to remove some `mirror` occurances
in the conditions of the for loops and summed to `answer1` instead
of `answer2`.
	
### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day13.md >day13.c; gcc -g -Wall day13.c -o day13; ./day13
```

