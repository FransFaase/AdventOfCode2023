# Day 5 of Advent of Code 2023

I woke up at 7:20 (CET) and it was around 8:00 that I started reading the puzzle
text. A rather long one for a method definint a mapping. And it annoys me that
the text does not explain what to do if the ranges overlap. Is it the first that
goes over the latter, or the other way around, or should they be applied. But in
the last case, the categories do not make sense. I doubting a bit whether I want
to continue with this.

I will start with writing a function for parsing a number and add it to
[the standard library](Std.md).

At 8:43, lets see if we can write a solution where the input is parsed, instead
of writing a lot of code to store all the values in data structures. First write
a function that reads the seed numbers.

```c
int main(int argc, char *argv)
{
	read_file("input/day05.txt");
	solve1();
}

void solve1()
{
	num_t min_sol = -1;
	
	for (char *seeds = lines[0] + 6; *seeds == ' ';)
	{
		seeds++;
		num_t seed = parse_number(&seeds);
		
		num_t answer = process1(seed);
		
		if (min_sol == -1 || answer < min_sol)
			min_sol = answer;
	}
	printf("%lld\n", min_sol);
}

num_t process1(num_t seed)
{
	printf("%lld\n", seed);
	return 0;
}
```

Okay, the above code does print the seeds (and a zero).
Now, lets see if we can implement something for the processing:

```c
num_t process1(num_t seed)
{
	printf("Start %lld\n", seed);
	for (int line_nr = 1; line_nr < nr_lines; )
	{
		int nr_mappings = 0;
		line_nr += 2;
		while (line_nr < nr_lines && is_digit(lines[line_nr][0]))
		{
			char *line = lines[line_nr];
			num_t to = parse_number(&line);
			line++;
			num_t from = parse_number(&line);
			line++;
			num_t length = parse_number(&line);
			
			if (from <= seed && seed < from + length && nr_mappings == 0)
			{
				seed = to + seed - from;
				nr_mappings++;
				printf("To: %lld\n", seed);
			}
			line_nr++;
		}
		if (nr_mappings > 1)
			printf("More than one\n");
		printf("\n");
	}
	printf("Result %lld\n", seed);
	return seed;
}
```

At 9:17, I did find the correct answer with the above code. Lets read the second
part of the puzzle.

### Second part of the puzzle.

I read the second part of the puzzle. As usual, it is a bit harder. I still could give
it a try with a brute force method.

```c
int main(int argc, char *argv)
{
	...
	solve2();
}

void solve2()
{
	num_t min_sol = -1;
	
	for (char *seeds = lines[0] + 6; *seeds == ' ';)
	{
		seeds++;
		num_t from = parse_number(&seeds);
		seeds++;
		num_t length = parse_number(&seeds);
		printf("from %lld length %lld\n", from, length);	
		for (num_t seed = from; seed < from + length; seed++)
		{
			num_t answer = process1(seed);
			
			if (min_sol == -1 || answer < min_sol)
				min_sol = answer;
		}
	}
	printf("%lld\n", min_sol);
}
```

I did let it run for some hours, but it did not make enough progress.

At 17:54, I think I am going to develop some library for manipulating collections
of ranges. Let first write the function to find the answer and next implement.

```c
void solve2()
{
	range_t *ranges = 0;
	for (char *seeds = lines[0] + 6; *seeds == ' ';)
	{
		seeds++;
		num_t from = parse_number(&seeds);
		seeds++;
		num_t length = parse_number(&seeds);
		ranges_add_range(&ranges, from, from + length);
		ranges_print(ranges, stdout); printf("\n");
	}
	ranges_print(ranges, stdout); printf("\n");
	
	for (int line_nr = 1; line_nr < nr_lines; )
	{
		int nr_mappings = 0;
		line_nr += 2;
		
		range_t *new_ranges = 0;
		
		while (line_nr < nr_lines && is_digit(lines[line_nr][0]))
		{
			char *line = lines[line_nr];
			num_t to = parse_number(&line);
			line++;
			num_t from = parse_number(&line);
			line++;
			num_t length = parse_number(&line);
			printf("map %lld %lld %lld\n", to, from, length);
		
			range_t *selection = ranges_extract_range(&ranges, from, from + length);
			ranges_print(selection, stdout); printf("\n");
			
			num_t shift = to - from;
			
			for (range_t *range = selection; range != 0; range = range->next)
			{
				ranges_add_range(&new_ranges, range->from + shift, range->to + shift);
			}
			line_nr++;
		}
		
		ranges = new_ranges;
		ranges_print(ranges, stdout); printf("\n");
	}
	printf("%lld\n", ranges->from);
}

```

I spend almost the whole evening implementing the range functions. Also wrote some
test code, which was really needed to get it working. Then, the above code, after
removing a small bug (forgot to incremenat `line_nr`) in the for loop nested in the
while loop, which resulted in an infinite loop. In finding this, I added some
print statements. And than, at 22:42, I the program returned the correct code.
	
### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day05.md >day05.c; gcc day05.c -o day05; ./day05
```

