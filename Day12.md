# Day 12 of Advent of Code 2023


I started reading the puzzle text around 8:20 and writing
this file at 8:28. I am trying to solve this puzzle without
allocating any memory on the heap (thus not using `malloc`);
You could either solve this with a recursive algorithm or
use [dynamic programming](https://en.wikipedia.org/wiki/Dynamic_programming).
I think, I will go for the latter.

At 8:40, I decided to go for the recursive method first.
Probably is sufficient for the first half.


```c
int main(int argc, char *argv[])
{
	read_file("input/day12.txt");
	solve1();
}

void solve1()
{
	num_t sum = 0;
	for (int l = 0; l < nr_lines; l++)
		sum += matches1(lines[l], strchr(lines[l], ' ') + 1);
	printf("%lld\n", sum);
}

num_t matches1(char *codes, char *digits)
{
	if (*digits == '\0')
		while (*codes == '.' || *codes == '?')
			codes++;
	
	if (*codes == ' ' && *digits == '\0')
		return 1;
	
	if (*codes == ' ' || *digits == '\0')
		return 0;

	num_t result = 0;
		
	char *next_digits = digits;
	num_t num = parse_number(&next_digits);
	if (*next_digits == ',') next_digits++;
	
	int i = 0;
	while (i < num && (codes[i] == '#' || codes[i] == '?'))
		i++;

	if (i == num && codes[i] != '#')
	{
		if (codes[i] != ' ')
			i++;
		result += matches1(codes + i, next_digits);
	}
	if (codes[i] != '#')
		result += matches1(codes + 1, digits);
	return result;
}	
```

At 9:09, I finished writing the above code and at 9:15 I finished
fixing some syntax errors. The result was not correct. Lets add
some debugging and test it on the example input.

```c
void solve1()
{
	num_t sum = 0;
	for (int l = 0; l < nr_lines; l++)
	{
		num_t v = matches1(lines[l], strchr(lines[l], ' ') + 1);
		printf("Answer %lld\n", v);
		sum += v;
	}
	
	printf("%lld\n", sum);
}

num_t matches1(char *codes, char *digits)
{
	printf("Matches %s | %s ", codes, digits);
	if (*digits == '\0')
		while (*codes == '.' || *codes == '?')
			codes++;
	
	if (*codes == ' ' && *digits == '\0')
	{
		printf("=> 1\n");
		return 1;
	}
	
	if (*codes == ' ' || *digits == '\0')
	{	
		printf("=> 0\n");
		return 0;
	}
	printf("\n");
	num_t result = 0;
		
	char *next_digits = digits;
	num_t num = parse_number(&next_digits);
	if (*next_digits == ',') next_digits++;
	
	int i = 0;
	while (i < num && (codes[i] == '#' || codes[i] == '?'))
		i++;
	
	if (i == num && codes[i] != '#')
	{
		if (codes[i] != ' ')
			i++;
		result += matches1(codes + i, next_digits);
	}
	if (codes[0] != '#')
		result += matches1(codes + 1, digits);
	return result;
}	
```

At 9:43, I found the bug. Lets try it on the input.
At 9:44, that returned the correct answer.

### Second part of the puzzle.

As I expected a bit, the input has to be repeated a number of times,
such that a simple recursive method would take too much time.
We could try memorization. Lets rewrite the above function using
indexes instead of character pointers.

I took a break till 11:04.

```c
int main(int argc, char *argv[])
{
	...
	solve2();
}

char *codes;
int nr_codes;
char *s_digits;
int nr_digits;

void solve2()
{
	num_t sum = 0;
	int max_nr_codes = 0;
	int max_nr_digits = 0;
	for (int l = 0; l < nr_lines; l++)
	{
		codes = lines[l];
		nr_codes = strchr(codes, ' ') - codes;
		s_digits = codes + nr_codes + 1;
		nr_digits = 1;
		for (char *s = s_digits; *s != '\0'; s++)
			if (*s == ',')
				nr_digits++;

		if (nr_codes > max_nr_codes) max_nr_codes = nr_codes;
		if (nr_digits > max_nr_digits) max_nr_digits = nr_digits;				
		sum += matches2(0, s_digits, 0);
	}
	printf("%d %d ", max_nr_codes, max_nr_digits);
	printf("%lld\n", sum);
}

num_t matches2(int c, char *digits, int d)
{
	if (*digits == '\0')
		while (codes[c] == '.' || codes[c] == '?')
			c++;
	
	if (codes[c] == ' ' && *digits == '\0')
		return 1;
	
	if (codes[c] == ' ' || *digits == '\0')
		return 0;

	num_t result = 0;
		
	char *next_digits = digits;
	num_t num = parse_number(&next_digits);
	if (*next_digits == ',') next_digits++;
	
	int i = 0;
	while (i < num && (codes[c + i] == '#' || codes[c + i] == '?'))
		i++;

	if (i == num && codes[c + i] != '#')
	{
		if (codes[c + i] != ' ')
			i++;
		result += matches2(c + i, next_digits, d + 1);
	}
	if (codes[c] != '#')
		result += matches2(c + 1, digits, d);
	return result;
}	
```

At 11:27, the above code gave the same result. Now we have to modify
it such that it can parse the input multiple times. I am going to implement
some memorization using an array of 101 by 31 based on the maximum lengths
that I found in my input.

```c
num_t memo[120][31];

int tot_nr_codes;
int tot_nr_digits;

void solve2()
{
	num_t sum = 0;
	for (int l = 0; l < nr_lines; l++)
	{
		codes = lines[l];
		nr_codes = strchr(codes, ' ') - codes;
		s_digits = codes + nr_codes + 1;
		nr_digits = 1;
		for (char *s = s_digits; *s != '\0'; s++)
			if (*s == ',')
				nr_digits++;
		
		for (int c = 0; c < 120; c++)
			for (int d = 0; d < 31; d++)
				memo[c][d] = -1;
				
		tot_nr_codes = 5 * nr_codes;
		tot_nr_digits = 5 * nr_digits;
		printf("%s %d %d: ", lines[l], tot_nr_codes, tot_nr_digits);

		num_t result = matches2(0, 0, 0);
		printf("%lld\n", result);
		sum += result;
	}
	printf("%lld\n", sum);
}

num_t matches2(int c, char *digits, int d)
{
	if (memo[c][d] != -1)
		return memo[c][d];
	int s_c = c;

	if (d == tot_nr_digits)
		while (c < tot_nr_codes && (codes[c % nr_codes] == '.' || codes[c % nr_codes] == '?'))
			c++;
	
	if (c == tot_nr_codes && d == tot_nr_digits)
	{
		memo[s_c][d] = 1;
		printf("Match %d %d => 1\n", s_c, d);	
		return 1;
	}
	
	if (c == tot_nr_codes && d == tot_nr_digits)
	{
		memo[s_c][d] = 0;
		printf("Match %d %d => 0\n", s_c, 0);
		return 0;
	}

	num_t result = 0;
	
	if (d % nr_digits == 0)
		digits = s_digits;	
	char *next_digits = digits;
	num_t num = parse_number(&next_digits);
	if (*next_digits == ',') next_digits++;
	
	int i = 0;
	int ci = c;
	while (i < num && ci < tot_nr_codes && (codes[ci % nr_codes] == '#' || codes[ci % nr_codes] == '?'))
	{
		i++;
		ci++;
	}

	if (i == num && (ci == tot_nr_codes || codes[ci % nr_codes] != '#'))
	{
		if (ci < tot_nr_codes)
			ci++;
		result += matches2(ci, next_digits, d + 1);
	}
	if (codes[c % nr_codes] != '#' && c < tot_nr_codes)
		result += matches2(c + 1, digits, d);
	memo[s_c][d] = result;
	printf("Match %d %d %lld\n", s_c, d, result);	
	
	return result;
}
```

At 12:54, after a lot of debugging, solving some bugs and not finding
the correct answer, I read text about adding the '?'. Okay, that
changes it a bit. Lets try again:

```c
void solve2()
{
	num_t sum = 0;
	for (int l = 0; l < nr_lines; l++)
	{
		codes = lines[l];
		nr_codes = strchr(codes, ' ') - codes;
		s_digits = codes + nr_codes + 1;
		nr_digits = 1;
		for (char *s = s_digits; *s != '\0'; s++)
			if (*s == ',')
				nr_digits++;
		
		for (int c = 0; c < 120; c++)
			for (int d = 0; d < 31; d++)
				memo[c][d] = -1;
				
		tot_nr_codes = 6 * nr_codes - 1;
		tot_nr_digits = 5 * nr_digits;
		printf("%s %d %d: ", lines[l], tot_nr_codes, tot_nr_digits);

		num_t result = matches2(0, 0, 0);
		printf("%lld\n", result);
		sum += result;
	}
	printf("%lld\n", sum);
}

char code_for(int i)
{
	i = i % (nr_codes + 1);
	return i == nr_codes ? '?' : codes[i];
}

num_t matches2(int c, char *digits, int d)
{
	if (memo[c][d] != -1)
		return memo[c][d];
	int s_c = c;

	if (d == tot_nr_digits)
		while (c < tot_nr_codes && (code_for(c) != '#'))
			c++;
	
	if (c == tot_nr_codes && d == tot_nr_digits)
	{
		memo[s_c][d] = 1;
		printf("%*.*sMatch %d %d => 1\n", s_c + d, s_c + d, "", s_c, d);	
		return 1;
	}
	
	if (c == tot_nr_codes && d == tot_nr_digits)
	{
		memo[s_c][d] = 0;
		printf("%*.*sMatch %d %d => 0\n", s_c + d, s_c + d, "", s_c, 0);
		return 0;
	}

	num_t result = 0;
	
	if (d % nr_digits == 0)
		digits = s_digits;	
	char *next_digits = digits;
	num_t num = parse_number(&next_digits);
	if (*next_digits == ',') next_digits++;
	
	printf("%*.*sc = %d %c\n", s_c + d, s_c + d, "", c, code_for(c));
	if (code_for(c) != '#' && c < tot_nr_codes)
		result += matches2(c + 1, digits, d);

	int i = 0;
	int ci = c;
	while (i < num && ci < tot_nr_codes && (code_for(ci) != '.'))
	{
		i++;
		ci++;
	}
	printf("%*.*si = %d ci = %d %c\n", s_c + d, s_c + d, "", i, ci, code_for(ci));

	if (i == num && (ci == tot_nr_codes || code_for(ci) != '#'))
	{
		if (ci < tot_nr_codes)
			ci++;
		result += matches2(ci, next_digits, d + 1);
	}

	memo[s_c][d] = result;
	printf("%*.*sMatch %d %d %lld\n", s_c + d, s_c + d, "", s_c, d, result);	
	
	return result;
}
```

At 13:18, I took a break again. Lets solve this in the evening.

At 21:09, I continued, with the code below:

```c
void solve2()
{
	num_t sum = 0;
	for (int l = 0; l < nr_lines; l++)
	{
		codes = lines[l];
		nr_codes = strchr(codes, ' ') - codes;
		s_digits = codes + nr_codes + 1;
		nr_digits = 1;
		for (char *s = s_digits; *s != '\0'; s++)
			if (*s == ',')
				nr_digits++;
		
		for (int c = 0; c < 120; c++)
			for (int d = 0; d < 31; d++)
				memo[c][d] = -1;
				
		tot_nr_codes = 5 * nr_codes + 4;
		tot_nr_digits = 5 * nr_digits;

		num_t result = matches2(0, 0, 0);
		sum += result;
	}
	printf("%lld\n", sum);
}

num_t matches2(int c, char *digits, int d)
{
	if (memo[c][d] != -1)
		return memo[c][d];
	int s_c = c;

	if (d == tot_nr_digits)
		while (c < tot_nr_codes && (code_for(c) != '#'))
			c++;
	
	if (c == tot_nr_codes && d == tot_nr_digits)
	{
		memo[s_c][d] = 1;
		return 1;
	}
	
	if (c == tot_nr_codes && d == tot_nr_digits)
	{
		memo[s_c][d] = 0;
		return 0;
	}

	num_t result = 0;
	
	if (d % nr_digits == 0)
		digits = s_digits;	
	char *next_digits = digits;
	num_t num = parse_number(&next_digits);
	if (*next_digits == ',') next_digits++;
	
	if (code_for(c) != '#' && c < tot_nr_codes)
		result += matches2(c + 1, digits, d);

	int i = 0;
	int ci = c;
	while (i < num && ci < tot_nr_codes && (code_for(ci) != '.'))
	{
		i++;
		ci++;
	}

	if (i == num && (ci == tot_nr_codes || code_for(ci) != '#'))
	{
		if (ci < tot_nr_codes)
			ci++;
		result += matches2(ci, next_digits, d + 1);
	}

	memo[s_c][d] = result;
	
	return result;
}
```

The bug was that I had calculated the length of the 'codes' wrong.
At 21:16, I found the correct answer with the code above.



	
### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day12.md >day12.c; gcc -g -Wall day12.c -o day12; ./day12
```

