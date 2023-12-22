# Day 21 of Advent of Code 2023


I started reading the puzzle around 11:30 (CET) and thought a bit about it.
At 11:44, I started writing this page. I guess, I am going to use the input
for all the work.

```c
int main(int argc, char *argv[])
{
	read_file("input/day21.txt");
	solve1();
}

void solve1()
{
	int nr_cols = strlen(lines[0]);
	for (int s = 0; s < 32; s++)
	{
		for (int i = 0; i < nr_lines; i++)
			for (int j = 0; j < nr_cols; j++)
				if (lines[i][j] == 'S' || lines[i][j] == 'O')
				{
					if (i > 0 && lines[i - 1][j] == '.')
						lines[i - 1][j] = 'o';
					if (i + 1 < nr_lines && lines[i + 1][j] == '.')
						lines[i + 1][j] = 'o';
					if (j > 0 && lines[i][j - 1] == '.')
						lines[i][j - 1] = 'o';
					if (j + 1 < nr_cols && lines[i][j + 1] == '.')
						lines[i][j + 1] = 'o';
				}
		for (int i = 0; i < nr_lines; i++)
			for (int j = 0; j < nr_cols; j++)
				if (lines[i][j] == 'o')
				{
					if (i > 0 && lines[i - 1][j] == '.')
						lines[i - 1][j] = 'O';
					if (i + 1 < nr_lines && lines[i + 1][j] == '.')
						lines[i + 1][j] = 'O';
					if (j > 0 && lines[i][j - 1] == '.')
						lines[i][j - 1] = 'O';
					if (j + 1 < nr_cols && lines[i][j + 1] == '.')
						lines[i][j + 1] = 'O';
				}
	}
	num_t answer = 0;
	for (int i = 0; i < nr_lines; i++)
		for (int j = 0; j < nr_cols; j++)
			if (lines[i][j] == 'S' || lines[i][j] == 'O')
				answer++;
	printf("%lld\n", answer);
}
```

### Second part of the puzzle

At 23:17, I continued with the second part. I already noticed some that the garden
has some wide lanes.

```c
int main(int argc, char *argv[])
{
	...
	solve2();
}

void solve2()
{
	for (int i = 0; i < nr_lines; i++)
		printf("%s\n", lines[i]);
	printf("\n");
	
	int nr_cols = strlen(lines[0]);
	printf("%d %d %d\n", nr_cols, 26501365L % nr_cols, 26501365L / nr_cols);
}
```

At 23:46, lets do some experiment:

```c
void solve2()
{
	...
	for (int s = 0; s < 33; s++)
	{
		for (int i = 0; i < nr_lines; i++)
			for (int j = 0; j < nr_cols; j++)
				if (lines[i][j] == 'S' || lines[i][j] == 'O')
				{
					if (i > 0 && lines[i - 1][j] == '.')
						lines[i - 1][j] = 'o';
					if (i + 1 < nr_lines && lines[i + 1][j] == '.')
						lines[i + 1][j] = 'o';
					if (j > 0 && lines[i][j - 1] == '.')
						lines[i][j - 1] = 'o';
					if (j + 1 < nr_cols && lines[i][j + 1] == '.')
						lines[i][j + 1] = 'o';
				}
		for (int i = 0; i < nr_lines; i++)
			for (int j = 0; j < nr_cols; j++)
				if (lines[i][j] == 'o')
				{
					if (i > 0 && lines[i - 1][j] == '.')
						lines[i - 1][j] = 'O';
					if (i + 1 < nr_lines && lines[i + 1][j] == '.')
						lines[i + 1][j] = 'O';
					if (j > 0 && lines[i][j - 1] == '.')
						lines[i][j - 1] = 'O';
					if (j + 1 < nr_cols && lines[i][j + 1] == '.')
						lines[i][j + 1] = 'O';
				}
	}
	for (int i = 0; i < nr_lines; i++)
		printf("%s\n", lines[i]);
	printf("\n");

	num_t full = 0;
	for (int i = 0; i < nr_lines; i++)
		for (int j = 0; j < nr_cols; j++)
			if (lines[i][j] == 'S' || lines[i][j] == 'O')
				full++;
	printf("%lld\n", full);
	int width = (32 + 33) * 2;
	printf("%d %d %d\n", width, 26501365L % width, 26501365L / width);

}
```

Okay, another experiment, just to make things sure:

```c
void solve2()
{
	...
	read_file("input/day21.txt");

	for (int s = 0; s < 131; s++)
		if (s % 2 == 0)
		{
			for (int i = 0; i < nr_lines; i++)
				for (int j = 0; j < nr_cols; j++)
					if (lines[i][j] == 'S' || lines[i][j] == 'O')
					{
						if (lines[(i + 1) % nr_cols][j] == '.')
							lines[(i + 1) % nr_cols][j] = 'o';
						if (lines[i][(j + 1) % nr_cols] == '.')
							lines[i][(j + 1) % nr_cols] = 'o';
					}
		}
		else
		{
			for (int i = 0; i < nr_lines; i++)
				for (int j = 0; j < nr_cols; j++)
					if (lines[i][j] == 'o')
					{
						if (lines[(i + 1) % nr_cols][j] == '.')
							lines[(i + 1) % nr_cols][j] = 'O';
						if (lines[i][(j + 1) % nr_cols] == '.')
							lines[i][(j + 1) % nr_cols] = 'O';
					}
		}
	for (int i = 0; i < nr_lines; i++)
		printf("%s\n", lines[i]);
	printf("\n");
}
```

Okay, so it indeed 131 steps to the next 'S'.

```c
void solve2()
{
	read_file("input/day21.txt");

	num_t total_steps = 26501365L;
	num_t nr_gardens = total_steps / nr_lines;
	num_t left = total_steps % nr_lines;

	num_t plotsLeft;
	num_t plotsTotal;

	for (int s = 0; s < nr_lines; s++)
	{
		if (s % 2 == 0)
		{
			for (int i = 0; i < nr_lines; i++)
				for (int j = 0; j < nr_lines; j++)
					if (lines[i][j] == 'S' || lines[i][j] == 'O')
					{
						if (i > 0 && lines[i - 1][j] == '.')
							lines[i - 1][j] = 'o';
						if (i + 1 < nr_lines && lines[i + 1][j] == '.')
							lines[i + 1][j] = 'o';
						if (j > 0 && lines[i][j - 1] == '.')
							lines[i][j - 1] = 'o';
						if (j + 1 < nr_lines && lines[i][j + 1] == '.')
							lines[i][j + 1] = 'o';
					}
		}
		else
		{
			for (int i = 0; i < nr_lines; i++)
				for (int j = 0; j < nr_lines; j++)
					if (lines[i][j] == 'o')
					{
						if (i > 0 && lines[i - 1][j] == '.')
							lines[i - 1][j] = 'O';
						if (i + 1 < nr_lines && lines[i + 1][j] == '.')
							lines[i + 1][j] = 'O';
						if (j > 0 && lines[i][j - 1] == '.')
							lines[i][j - 1] = 'O';
						if (j + 1 < nr_lines && lines[i][j + 1] == '.')
							lines[i][j + 1] = 'O';
					}
		}

		if (s + 1 == left || s + 1== nr_lines)
		{
			for (int i = 0; i < nr_lines; i++)
				printf("%s\n", lines[i]);
			printf("\n");
			num_t visited = 0;
			for (int i = 0; i < nr_lines; i++)
				for (int j = 0; j < nr_lines; j++)
					if (lines[i][j] == 'S' || lines[i][j] == 'O' || lines[i][j] == 'o')
						visited++;
			
			if (s + 1 == left)
				plotsLeft = visited;
			if (s + 1 == nr_lines)
				plotsTotal = visited;
		}
	}
	
	printf("plotsLeft = %lld\n", plotsLeft);
	printf("plotsTotal = %lld\n", plotsTotal);
	printf("left = %lld\n", left);
	printf("nr_gardens = %lld\n", nr_gardens);
	printf("tot = %lld\n", 2 * nr_gardens * (nr_gardens + 1));
	
	num_t answer = plotsTotal * 2 * nr_gardens * (nr_gardens + 1) + plotsLeft;
	
	
	printf("%lld\n", answer);
	printf("%lld\n", total_steps * total_steps + (total_steps + 1) * (total_steps + 1));
}
```

At 2:10, this still does not give the right answer.

Let fill a three by three copy of this.

```c

#define SIZE (131 * 3)
void solve2()
{
	read_file("input/day21.txt");

	num_t total_steps = 26501365L;
	num_t nr_gardens = total_steps / nr_lines;
	num_t left = total_steps % nr_lines;

	num_t plotsLeft;
	num_t plotsTotal;

	char mat[SIZE][SIZE];
	
	for (int i = 0; i < SIZE; i++)
		for (int j = 0; j < SIZE; j++)
			mat[i][j] = lines[i % 131][j % 131];
			
	for (int i = 0; i < SIZE; i++)
		for (int j = 0; j < SIZE; j++)
			if (mat[i][j] == 'S' && (i != 196 || j != 196))
			{
				printf("%d,%d\n", i, j);
				mat[i][j] = '.';
			}

	for (int s = 0; s < 131 + left; s++)
	{
		if (s % 2 == 0)
		{
			for (int i = 0; i < SIZE; i++)
				for (int j = 0; j < SIZE; j++)
					if (mat[i][j] == 'S' || mat[i][j] == 'O')
					{
						if (i > 0 && mat[i - 1][j] == '.')
							mat[i - 1][j] = 'o';
						if (i + 1 < SIZE && mat[i + 1][j] == '.')
							mat[i + 1][j] = 'o';
						if (j > 0 && mat[i][j - 1] == '.')
							mat[i][j - 1] = 'o';
						if (j + 1 < SIZE && mat[i][j + 1] == '.')
							mat[i][j + 1] = 'o';
					}
		}
		else
		{
			for (int i = 0; i < SIZE; i++)
				for (int j = 0; j < SIZE; j++)
					if (mat[i][j] == 'o')
					{
						if (i > 0 && mat[i - 1][j] == '.')
							mat[i - 1][j] = 'O';
						if (i + 1 < SIZE && mat[i + 1][j] == '.')
							mat[i + 1][j] = 'O';
						if (j > 0 && mat[i][j - 1] == '.')
							mat[i][j - 1] = 'O';
						if (j + 1 < SIZE && mat[i][j + 1] == '.')
							mat[i][j + 1] = 'O';
					}
		}
	}

	num_t visited = 0;
	for (int i = 0; i < SIZE; i++)
	{
		for (int j = 0; j < SIZE; j++)
		{
			printf("%c", mat[i][j]);
			if (mat[i][j] == 'S' || mat[i][j] == 'O' || mat[i][j] == 'o')
				visited++;
		}
		printf("\n");
	}
	printf("\n");
}
```

There is indeed one side, where it does not work correct. But maybe we should fill
a five by five:


```c

#define FSIZE (2000)
#define SUM(X, Y) printf("%20lld %13lld %20lld\n", X * Y, X, Y); answer += X * Y;

void solve2()
{
	read_file("input/day21.txt");

	num_t total_steps = 26501365L;
	num_t nr_gardens = total_steps / nr_lines;
	num_t left = total_steps % nr_lines;

	num_t plotsLeft;
	num_t plotsTotal;

	char mat[FSIZE][FSIZE];
	
	int times = nr_gardens % 2 == 0 ? 9 : 7;
	int size = times * nr_lines;
	
	for (int i = 0; i < size; i++)
		for (int j = 0; j < size; j++)
			mat[i][j] = lines[i % nr_lines][j % nr_lines];
	
	int center = (times / 2) * nr_lines + nr_lines / 2;
	printf("center = %d\n", center);
	
	for (int i = 0; i < size; i++)
		for (int j = 0; j < size; j++)
			if (mat[i][j] == 'S' && (i != center || j != center))
			{
				printf("%d,%d\n", i, j);
				mat[i][j] = '.';
			}

	for (int s = 0; s < (times / 2) * nr_lines + left; s++)
	{
		if (s % 2 == 0)
		{
			for (int i = 0; i < size; i++)
				for (int j = 0; j < size; j++)
					if (mat[i][j] == 'S' || mat[i][j] == 'O')
					{
						if (i > 0 && mat[i - 1][j] == '.')
							mat[i - 1][j] = 'o';
						if (i + 1 < size && mat[i + 1][j] == '.')
							mat[i + 1][j] = 'o';
						if (j > 0 && mat[i][j - 1] == '.')
							mat[i][j - 1] = 'o';
						if (j + 1 < size && mat[i][j + 1] == '.')
							mat[i][j + 1] = 'o';
					}
		}
		else
		{
			for (int i = 0; i < size; i++)
				for (int j = 0; j < size; j++)
					if (mat[i][j] == 'o')
					{
						if (i > 0 && mat[i - 1][j] == '.')
							mat[i - 1][j] = 'O';
						if (i + 1 < size && mat[i + 1][j] == '.')
							mat[i + 1][j] = 'O';
						if (j > 0 && mat[i][j - 1] == '.')
							mat[i][j - 1] = 'O';
						if (j + 1 < size && mat[i][j + 1] == '.')
							mat[i][j + 1] = 'O';
					}
		}
	}

	num_t counts[10][10];
	for (int i = 0; i < times; i++)
		for (int j = 0; j < times; j++)
			counts[i][j] = 0;
	for (int i = 0; i < size; i++)
	{
		for (int j = 0; j < size; j++)
		{
			printf("%c", mat[i][j]);
			if (mat[i][j] == 'S' || mat[i][j] == 'O')
				counts[i/nr_lines][j/nr_lines]++;
		}
		printf("\n");
	}
	
	num_t sum = 0;
	for (int i = 0; i < times; i++)
	{
		for (int j = 0; j < times; j++)
		{
			sum += counts[i][j];
			printf("%8lld", counts[i][j]);
		}
		printf("\n");
	}

	int m = times / 2;
	int b = times - 1;	

	num_t inside1;
	num_t inside2;
	if (nr_gardens % 2 == 0)
	{
		inside2 = nr_gardens * nr_gardens - 4;
		inside1 = (nr_gardens - 1) * (nr_gardens - 1);
	}
	else
	{
		inside1 = nr_gardens * nr_gardens - 4;
		inside2 = (nr_gardens - 1) * (nr_gardens - 1);
	}

	printf("left = %lld\n", left);
	printf("gardens = %lld\n", nr_gardens);
	num_t answer = 0;
	
	num_t one = 1;
	SUM(one, counts[0][m]);
	SUM(one, counts[m][0]);
	SUM(one, counts[b][m]);
	SUM(one, counts[m][b]);
	SUM(one, counts[1][m]);
	SUM(one, counts[m][1]);
	SUM(one, counts[b-1][m]);
	SUM(one, counts[m][b-1]);
	SUM(inside1, counts[m][m]);
	SUM(inside2, counts[m][m+1]);
	SUM(nr_gardens, counts[0][m-1]);
	SUM(nr_gardens, counts[0][m+1]);
	SUM(nr_gardens, counts[b][m-1]);
	SUM(nr_gardens, counts[b][m+1]);
	SUM((nr_gardens - 1), counts[  1][m-1]);
	SUM((nr_gardens - 1), counts[  1][m+1]);
	SUM((nr_gardens - 1), counts[b-1][m-1]);
	SUM((nr_gardens - 1), counts[b-1][m+1]);
	printf("%20lld\n", answer);
	printf("%lld\n", nr_gardens * nr_lines + left);
}
```

At 4:58, December 22, I am getting a bit mad.

At 5:59. New idea, which I got when watching
[A Diamond in the Rough](https://www.reddit.com/r/adventofcode/comments/18njrqf/2023_day_21_a_diamond_in_the_rough/)

```c
void solve2()
{
	read_file("input/day21.txt");

	num_t total_steps = 26501365L;
	num_t nr_gardens = total_steps / nr_lines;
	num_t left = total_steps % nr_lines;

	num_t plotsLeft;
	num_t plotsTotal;

	char mat[FSIZE][FSIZE];
	
	int times = nr_gardens % 2 == 0 ? 13 : 11;
	int size = times * nr_lines;
	
	for (int i = 0; i < size; i++)
		for (int j = 0; j < size; j++)
			mat[i][j] = lines[i % nr_lines][j % nr_lines];
	
	int center = (times / 2) * nr_lines + nr_lines / 2;
	//printf("center = %d\n", center);
	
	for (int i = 0; i < size; i++)
		for (int j = 0; j < size; j++)
			if (mat[i][j] == 'S' && (i != center || j != center))
			{
				//printf("%d,%d\n", i, j);
				mat[i][j] = '.';
			}

	num_t results[4];
	int r = 0;
	for (int s = 1; s <= (times / 2) * nr_lines + left; s++)
	{
		if (s % 2 == 1)
		{
			for (int i = 0; i < size; i++)
				for (int j = 0; j < size; j++)
					if (mat[i][j] == 'S' || mat[i][j] == 'O')
					{
						if (i > 0 && mat[i - 1][j] == '.')
							mat[i - 1][j] = 'o';
						if (i + 1 < size && mat[i + 1][j] == '.')
							mat[i + 1][j] = 'o';
						if (j > 0 && mat[i][j - 1] == '.')
							mat[i][j - 1] = 'o';
						if (j + 1 < size && mat[i][j + 1] == '.')
							mat[i][j + 1] = 'o';
					}
		}
		else
		{
			for (int i = 0; i < size; i++)
				for (int j = 0; j < size; j++)
					if (mat[i][j] == 'o')
					{
						if (i > 0 && mat[i - 1][j] == '.')
							mat[i - 1][j] = 'O';
						if (i + 1 < size && mat[i + 1][j] == '.')
							mat[i + 1][j] = 'O';
						if (j > 0 && mat[i][j - 1] == '.')
							mat[i][j - 1] = 'O';
						if (j + 1 < size && mat[i][j + 1] == '.')
							mat[i][j + 1] = 'O';
					}
		}
		if ((s % (2 * nr_lines)) == left)
		{
			num_t sum = 0;
			for (int i = 0; i < size; i++)
				for (int j = 0; j < size; j++)
					if (mat[i][j] == 'o')
						sum++;
			results[r] = sum;
			printf("%4d: %10lld\n", s, results[r]);
			r++;
		}					
	}
	
	num_t diffs[3];
	for (int i = 0; i < 3; i++)
	{
		diffs[i] = results[i + 1] - results[i];
		printf("%d %lld %lld %lld\n", i, results[i + 1], results[i], diffs[i]);
	}
	for (int i = 0; i < 2; i++)
		printf("%d %lld\n", i, diffs[i + 1] - diffs[i]);
}
```

At 13:30, I continued. How to derive a parabolic equation from the first three values?

```
f(x) = ax^2 + bx + c

f(0) = k  =>           c = k
f(1) = l  =>  a +  b + c = l  =>  a +  b = l - k
f(2) = m  => 4a + 2b + c = m  => 4a + 2b = m - k

solve:
   -2a - 2b = -2l + 2k
    4a + 2b = m - k
    2a      = -2l + 2k + m - k
     a      = -l + k/2 + m
     
solve:
    4a + 4b = 4l - 4k
   -4a - 2b = -m + k
         2b = 4l - 4k - m + k
          b = 2l - 3k/2 - m/2
```

It does look wierd. Maybe, just try to figure it out in code:

```c
void solve2()
{
	...
	num_t a = diffs[2] - diffs[1];
	for (int i = 0; i < 10; i++)
		printf("%10d %14lld\n", left + i * 2 * nr_lines, results[0] + i * (results[1] - results[0]) + i * (i - 1) * a / 2);
	num_t x = nr_gardens / 2;
	printf("\n%10lld %14lld\n", left + x * 2 * nr_lines, results[0] + x * (results[1] - results[0]) + x * (x - 1) * a / 2);
}
```

At 14:24, I submitted the result of the above. And it is still incorrect. And, I think why.
I changed `s % 2 == 0` into `s % 2 == 1` in the above code. But I have to wait 15 minutes
before I can submit it.

At 14:51, I submitted the answer, but it is still incorrect. I did some debugging,
not reflected in the above code, to discover that it seems I submitted (by accident)
a previous (incorrect) answer.
At 15:24, I submitted the last and correct answer.

    
### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day21.md >day21.c; gcc -g -Wall day21.c -o day21; ./day21
```

