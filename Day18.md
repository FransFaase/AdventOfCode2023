# Day 18 of Advent of Code 2023

At 13:54, I started to read the puzzle text.
At 13:58, I read the text. Again, this is just calculating
some area enclosed by a polygon, like we did before on an
earlier day. I do not see what has to be done with the colour.
Looks like it is needed for the second part of the puzzle.

```c
int main(int argc, char *argv[])
{
	read_file("input/day18.txt");
	solve1();
}

void solve1()
{
	num_t x = 0;
	num_t y = 0;
	num_t nr_cubics = 0;
	num_t area = 0;
	for (int i = 0; i < nr_lines; i++)
	{
		char *s = lines[i];
		char direction = *s;
		s += 2;
		num_t steps = parse_number(&s);
		
		nr_cubics += steps;
		switch (direction)
		{
			case 'R':
				x += steps;
				break;
			case 'D':
				y += steps;
				area += x;
				break;
			case 'L':
				x -= steps;
				break;
			case 'U':
				y -= steps;
				area -= x;
				break;
		}
	}
	printf("%lld %lld %lld\n", nr_cubics, area, area + nr_cubics / 2 + 1); 
}
```

At 14:13, although the above code did return an answer, it is not correct. The value is
too low. I get some idea that the path is crossing itself and that my simple
method of calculating the area does not work. Lets test it on the exmple
input to exclude a simple arithmetic mistake.

At 14:16, I discover that it does not return the correct answer.

```c
void solve1()
{
	num_t x = 0;
	num_t y = 0;
	num_t nr_cubics = 0;
	num_t area = 0;
	for (int i = 0; i < nr_lines; i++)
	{
		char *s = lines[i];
		char direction = *s;
		s += 2;
		num_t steps = parse_number(&s);
		
		nr_cubics += steps;
		switch (direction)
		{
			case 'R':
				x += steps;
				break;
			case 'D':
				y += steps;
				area += x * steps;
				break;
			case 'L':
				x -= steps;
				break;
			case 'U':
				y -= steps;
				area -= x * steps;
				break;
		}
	}
	printf("%lld %lld %lld\n", nr_cubics, area, area + nr_cubics / 2 + 1); 
}
```

There was an error in the calculation of `area`. It required to be
multiplied with `steps`. At 14:18, the above code returned the correct
value for the first part of the puzzle.


### Second part of the puzzle.

The second part is not that hard. Same algorithm, different way to parse
the input.

```c
int main(int argc, char *argv[])
{
	...
	solve2();
}

void solve2()
{
	num_t x = 0;
	num_t y = 0;
	num_t nr_cubics = 0;
	num_t area = 0;
	for (int i = 0; i < nr_lines; i++)
	{
		char *s = strchr(lines[i], '#');
		s++;
		num_t steps = 0;
		for (int i = 0; i < 5; i++, s++)
			steps = 16 * steps + *s - ('0' <= *s && *s <= '9' ? '0' : 'a' - 10);

		nr_cubics += steps;
		switch (*s)
		{
			case '0':
				x += steps;
				break;
			case '1':
				y += steps;
				area += x;
				break;
			case '2':
				x -= steps;
				break;
			case '3':
				y -= steps;
				area -= x;
				break;
		}
	}
	printf("%lld %lld %lld\n", nr_cubics, area, area + nr_cubics / 2 + 1); 
}
```

At 14:27, it did not return the correct answer.

```c
void solve2()
{
	num_t x = 0;
	num_t y = 0;
	num_t nr_cubics = 0;
	num_t area = 0;
	for (int i = 0; i < nr_lines; i++)
	{
		char *s = strchr(lines[i], '#');
		s++;
		num_t steps = 0;
		for (int i = 0; i < 5; i++, s++)
			steps = 16 * steps + *s - ('0' <= *s && *s <= '9' ? '0' : 'a' - 10);

		printf("%lld %c\n", steps, *s);
		nr_cubics += steps;
		switch (*s)
		{
			case '0':
				x += steps;
				break;
			case '1':
				y += steps;
				area += x * steps;
				break;
			case '2':
				x -= steps;
				break;
			case '3':
				y -= steps;
				area -= x * steps;
				break;
		}
	}
	printf("%lld %lld %lld\n", nr_cubics, area, area + nr_cubics / 2 + 1); 
}
```

I did some debugging with the example input.
Okay, it seems for `solve2`, I made a copy of the `solve1` with the error.
After I fixed this (again), it returned the correct answer at 14:32.

### Prolog

This puzzle could have been a lot harder, in case the path had crossed itself.
That would have required to implement a polygon intersection algorithm or
something else a bit more complicated that for each horizontal line calculate
the most left and right locations that have been dug.

I made an attempt to implement it:

```c
int main(int argc, char *argv[])
{
	...
	solve2b();
}

typedef struct min_max_range min_max_range_t;
struct min_max_range
{
	num_t from;
	num_t to;    // included
	num_t min;
	num_t max;
	min_max_range_t *next;
};

void min_max_range_add(min_max_range_t **ref, num_t from, num_t to, num_t min, num_t max)
{
	printf("\nadd %lld %lld %lld %lld :\n", from, to, min, max);
	for (;;)
		if (*ref == 0 || to < (*ref)->from)
		{
			printf("- new %lld:%lld %lld:%lld\n", from, to, min, max);
			min_max_range_t * new_range = (min_max_range_t*)malloc(sizeof(min_max_range_t));
			new_range->from = from;
			new_range->to = to;
			new_range->min = min;
			new_range->max = max;
			new_range->next = *ref;
			*ref = new_range;
			break;
		}
		else if ((*ref)->to < from)
			ref = &(*ref)->next;
		else
		{
			if (from < (*ref)->from)
			{
				min_max_range_t * new_range = (min_max_range_t*)malloc(sizeof(min_max_range_t));
				new_range->from = from;
				new_range->to = (*ref)->from - 1;
				from = (*ref)->from;
				new_range->min = min;
				new_range->max = max;
				new_range->next = *ref;
				printf("- append %lld:%lld %lld:%lld, continue with %lld-%lld %lld-%lld\n", new_range->from, new_range->to, min, max, from, to, min, max);
				*ref = new_range;
				ref = &new_range->next;
			}
			else if ((*ref)->from < from)
			{
				printf("- split-f %lld:%lld %lld:%lld in ", (*ref)->from, (*ref)->to, (*ref)->min, (*ref)->max);
				min_max_range_t * new_range = (min_max_range_t*)malloc(sizeof(min_max_range_t));
				new_range->from = (*ref)->from;
				new_range->to = from - 1;
				(*ref)->from = from;
				new_range->min = (*ref)->min;
				new_range->max = (*ref)->max;
				new_range->next = *ref;
				printf(" %lld:%lld %lld:%lld and %lld:%lld %lld:%lld\n", new_range->from, new_range->to, new_range->min, new_range->max, (*ref)->from, (*ref)->to, (*ref)->min, (*ref)->max);
				*ref = new_range;
				ref = &new_range->next;
			}
			
			if (to < (*ref)->to)
			{
				printf("- split-b %lld:%lld %lld:%lld in ", (*ref)->from, (*ref)->to, (*ref)->min, (*ref)->max);
				min_max_range_t * new_range = (min_max_range_t*)malloc(sizeof(min_max_range_t));
				new_range->from = (*ref)->from;
				new_range->to = to;
				(*ref)->from = to + 1;
				new_range->min = (*ref)->min;
				if (min < new_range->min)
					new_range->min = min;
				new_range->max = (*ref)->max;
				if (max > new_range->max)
					new_range->max = max;
				new_range->next = *ref;
				printf(" %lld:%lld %lld:%lld and %lld:%lld %lld:%lld\n", new_range->from, new_range->to, new_range->min, new_range->max, (*ref)->from, (*ref)->to, (*ref)->min, (*ref)->max);
				*ref = new_range;
				break;
			}
			else
			{
				if (min < (*ref)->min)
				{
					printf("- decrease %lld:%lld %lld:%lld to %lld:%lld\n", (*ref)->from, (*ref)->to, (*ref)->min, (*ref)->max, min, (*ref)->max);
					(*ref)->min = min;
				}
				if (max > (*ref)->max)
				{
					printf("- increase %lld:%lld %lld:%lld to %lld:%lld\n", (*ref)->from, (*ref)->to, (*ref)->min, (*ref)->max, (*ref)->min, max); 
					(*ref)->max = max;
				}
				if ((*ref)->to == to)
					break;
				from = (*ref)->to + 1;
				printf("- continue with %lld:%lld %lld:%lld\n", from, to, min, max);
				ref = &(*ref)->next;
			}
		}
}

void min_max_range_print(min_max_range_t *ranges)
{
	num_t area = 0;
	for (; ranges; ranges = ranges->next)
	{
		printf("[%lld:%lld, %lld:%lld] ", ranges->from, ranges->to, ranges->min, ranges->max);
		area += (1 + ranges->to - ranges->from) * (1 + ranges->max - ranges->min);
	}
	printf("%lld\n", area);
}

void test_min_max_range_impl()
{
	min_max_range_t *ranges = 0;
	min_max_range_add(&ranges, 20, 29, 10, 19);
	min_max_range_print(ranges);
	min_max_range_add(&ranges, 0, 9, 20, 29);
	min_max_range_print(ranges);
	min_max_range_add(&ranges, 40, 49, 30, 39);
	min_max_range_print(ranges);
	min_max_range_add(&ranges, 20, 29, 11, 18);
	min_max_range_print(ranges);
	min_max_range_add(&ranges, 20, 30, 9, 22);
	min_max_range_print(ranges);
	min_max_range_add(&ranges, 8, 14, 22, 28);
	min_max_range_print(ranges);
	min_max_range_add(&ranges, 19, 22, 22, 28);
	min_max_range_print(ranges);
	min_max_range_add(&ranges, 27, 32, 18, 33);
	min_max_range_print(ranges);
}


void solve2b()
{
	test_min_max_range_impl();
	
	//return;

	num_t x = 0;
	num_t y = 0;
	min_max_range_t *ranges = 0;
	for (int i = 0; i < nr_lines; i++)
	{
		char *s = strchr(lines[i], '#');
		s++;
		num_t steps = 0;
		for (int i = 0; i < 5; i++, s++)
			steps = 16 * steps + *s - ('0' <= *s && *s <= '9' ? '0' : 'a' - 10);

		//printf("%lld %c\n", steps, *s);
		switch (*s)
		{
			case '0':
				min_max_range_add(&ranges, x, x + steps, y, y);
				x += steps;
				break;
			case '1':
				min_max_range_add(&ranges, x, x, y, y + steps);
				y += steps;
				break;
			case '2':
				min_max_range_add(&ranges, x - steps, x, y, y);
				x -= steps;
				break;
			case '3':
				min_max_range_add(&ranges, x, x, y - steps, y);
				y -= steps;
				break;
		}
	}

	num_t area = 0;
	min_max_range_t *prev = 0;
	for (min_max_range_t *range = ranges; range != 0; range = range->next)
	{
		if (prev != 0 && prev->to + 1 != range->from)
			printf("Error\n");
		area += (1 + range->to - range->from) * (1 + range->max - range->min);
		prev = range;
	}
	printf("%lld\n", area); 
}
```

The above code returns a higher answer for my puzzle input. I am not sure if the
above code is actually correct.

I want to check if any of the lines are actually overlapping:

```c

typedef struct rect rect_t;
struct rect
{
	num_t min_x, max_x, min_y, max_y;
};

rect_t rects[680];

int main(int argc, char *argv[])
{
	...
	check_overlap();
}

void check_overlap()
{
	num_t x = 0;
	num_t y = 0;
	for (int i = 0; i < nr_lines; i++)
	{
		char *s = strchr(lines[i], '#');
		s++;
		num_t steps = 0;
		for (int i = 0; i < 5; i++, s++)
			steps = 16 * steps + *s - ('0' <= *s && *s <= '9' ? '0' : 'a' - 10);

		//printf("%lld %c\n", steps, *s);
		switch (*s)
		{
			case '0':
				rects[i].min_x = x + 1;
				rects[i].max_x = x + steps;
				rects[i].min_y = y;
				rects[i].max_y = y;
				x += steps;
				break;
			case '1':
				rects[i].min_x = x;
				rects[i].max_x = x;
				rects[i].min_y = y + 1;
				rects[i].max_y = y + steps;
				y += steps;
				break;
			case '2':
				rects[i].min_x = x - steps;
				rects[i].max_x = x - 1;
				rects[i].min_y = y;
				rects[i].max_y = y;
				x -= steps;
				break;
			case '3':
				rects[i].min_x = x;
				rects[i].max_x = x;
				rects[i].min_y = y - steps;
				rects[i].max_y = y - 1;
				y -= steps;
				break;
		}
	}
	for (int i = 1; i < nr_lines; i++)
		for (int j = 0; j < i; j++)
			if (   rects[i].min_x <= rects[j].max_x && rects[j].min_x <= rects[i].max_x
			    && rects[i].min_y <= rects[j].max_y && rects[j].min_y <= rects[i].max_y)
			    printf("steps %d and %d overlap\n", i, j);
	printf("good\n");
}
```

It does not report any overlaps, which means there is probably a bug in my code.

Okay, I understand the problem. The method of calculating for each vertical line the
lowest and highest values, is the incorrect method. We should actually count lines.
So that explains the value that is too high.

### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day18.md >day18.c; gcc -g -Wall day18.c -o day18; ./day18
```

