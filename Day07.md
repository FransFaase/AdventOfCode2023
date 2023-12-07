# Day 7 of Advent of Code 2023

I only started reading the puzzle around 9:24 (CET).

```
int main(int argc, char *argv[])
{
    read_file("input/day07.txt");
    solve1();
}

typedef
struct
{
    num_t card;
    num_t bid;
} hand_t;

hand_t *hands;

void solve1()
{
    read_hands();
}

char *card_kinds = "23456789TJQKA";

int larger_char(const void *a, const void *b)
{
    return *(char*)b - *(char*)a;
}

void read_hands()
{
    hands = (hand_t*)malloc(nr_lines * sizeof(hand_t));
    
    for (int i = 0; i < nr_lines; i++)
    {
        char cards[5];
        for (int j = 0; j < 5; j++)
            cards[j] = strchr(card_kinds, lines[i][j]) - card_kinds;
        
        for (int j = 0; j < 5; j++)
            printf("%c", card_kinds[cards[j]]);
        printf(" ");
        
        qsort(cards, 5, 1, larger_char);
        
        for (int j = 0; j < 5; j++)
            printf("%c", card_kinds[cards[j]]);
        printf(" ");

        char *s = lines[i] + 6;
        printf("%lld\n", parse_number(&s));
    }
}
```

At 10:04, I stopped with the above. I have to do some other things
now and think a bit more about how I am going to solve this.

At 18:59, I continued. Thought a little bit about it during the day.
I decided to start over and turn the C code above in text.

```c
int main(int argc, char *argv[])
{
    read_file("input/day07.txt");
    solve1();
}

typedef struct
{
    char nr;
    char card;
} nr_card_t;

typedef struct hand hand_t;
struct hand
{
    nr_card_t cards[5];
    num_t bid;
};

hand_t *hands;

void solve1()
{
    read_hands();
}

char *card_kinds = "23456789TJQKA";
                    

int larger_nr_card(const void *a, const void *b)
{
    nr_card_t *card_a = (nr_card_t*)a;
    nr_card_t *card_b = (nr_card_t*)b;
    int r = card_b->nr - card_a->nr;
    return r != 0 ? r : card_b->card - card_a->card;
}

void read_hands()
{
    hands = (hand_t*)malloc(nr_lines * sizeof(hand_t));
    
    for (int i = 0; i < nr_lines; i++)
    {
        nr_card_t cards[13];
        for (int k = 0; k < 13; k++)
        {
            cards[k].nr = 0;
            cards[k].card = k;
        }
        
        for (int j = 0; j < 5; j++)
        {
            int k = strchr(card_kinds, lines[i][j]) - card_kinds;
            printf("%c", card_kinds[k]); 
            cards[k].nr++;
        }
        
        printf(" ");
        
        qsort(cards, 13, sizeof(nr_card_t), larger_nr_card);
        
        for (int j = 0; j < 5; j++)
        {
            hands[i].cards[j] = cards[j];
            if (cards[j].nr > 0)
                printf("%d:%c ", cards[j].nr, card_kinds[cards[j].card]);
            else
                printf("    ");
        }
        printf(" ");

        char *s = lines[i] + 6;
        hands[i].bid = parse_number(&s);
        printf("%lld\n", hands[i].bid);
    }

    printf("\n");
}
```

```c

int smaller_hand(const void *a, const void *b)
{
    hand_t *hand_a = (hand_t*)a;
    hand_t *hand_b = (hand_t*)b;
    int r = 0;
    for (int i = 0; i < 5 && r == 0; i++)
        r = hand_a->cards[i].nr - hand_b->cards[i].nr;  
    for (int i = 0; i < 5 && r == 0; i++)
        r = hand_a->cards[i].card - hand_b->cards[i].card;  
    return r;
}

void solve1()
{
    ...
    
    qsort(hands, nr_lines, sizeof(hand_t), smaller_hand);
    
    num_t score = 0;

    for (int i = 0; i < nr_lines; i++)
    {
        for (int j = 0; j < 5; j++)
        {
            if (hands[i].cards[j].nr > 0)
                printf("%d:%c ", hands[i].cards[j].nr, card_kinds[hands[i].cards[j].card]);
            else
                printf("    ");
        }
        printf(" %lld\n", hands[i].bid);
        score = hands[i].bid * i;
    }
    
    printf("\n%lld\n", score);
    
}
```

This gave the wrong answer. Okay, I did not read the second ordering rule
carefully enough.

```c
struct hand
{
    ...
    int values[5];
};

void read_hands()
{
    hands = (hand_t*)malloc(nr_lines * sizeof(hand_t));
    
    for (int i = 0; i < nr_lines; i++)
    {
        nr_card_t cards[13];
        for (int k = 0; k < 13; k++)
        {
            cards[k].nr = 0;
            cards[k].card = k;
        }
        
        for (int j = 0; j < 5; j++)
        {
            int k = strchr(card_kinds, lines[i][j]) - card_kinds;
            if (k < 0 || k >= 13) { printf("ERROR\n"); exit(1); }
            printf("%c", card_kinds[k]); 
            cards[k].nr++;
            hands[i].values[j] = k;
        }
        
        printf(" ");
        
        qsort(cards, 13, sizeof(nr_card_t), larger_nr_card);
        
        for (int j = 0; j < 5; j++)
        {
            hands[i].cards[j] = cards[j];
            if (cards[j].nr > 0)
                printf("%d:%c ", cards[j].nr, card_kinds[cards[j].card]);
            else
                printf("    ");
        }
        printf(" ");

        char *s = lines[i] + 6;
        hands[i].bid = parse_number(&s);
        printf("%lld\n", hands[i].bid);
    }

    printf("\n");
}

int smaller_hand(const void *a, const void *b)
{
    hand_t *hand_a = (hand_t*)a;
    hand_t *hand_b = (hand_t*)b;
    int r = 0;
    for (int i = 0; i < 5 && r == 0; i++)
        r = hand_a->cards[i].nr - hand_b->cards[i].nr;  
    for (int i = 0; i < 5 && r == 0; i++)
        r = hand_a->values[i] - hand_b->values[i];  
    return r;
}

void solve1()
{
    read_hands();
    
    qsort(hands, nr_lines, sizeof(hand_t), smaller_hand);
    
    num_t score = 0;

    for (int i = 0; i < nr_lines; i++)
    {
        for (int j = 0; j < 5; j++)
        {
            if (hands[i].cards[j].nr > 0)
                printf("%d ", hands[i].cards[j].nr);
            else
                printf("  ");
        }
        for (int j = 0; j < 5; j++)
            printf("%c", card_kinds[hands[i].values[j]]);
        printf(" %lld %d %lld %lld\n", hands[i].bid, i + 1, hands[i].bid * (i + 1), score);
        score += hands[i].bid * (i + 1);
    }
    
    printf("\n%lld\n", score);
}
```

Did still did not give the right answer. I tried the program on the
example input and it gave the right answer.

At 21:27, okay, I have been staring at this for a long time only because my
correct answer was rejected because I submitted my answers too quickly
after each other and I did not realize that this was the reason why it
was rejected.

### Second part of the puzzle.

```c
int main(int argc, char *argv[])
{
    ...
    solve2();
}

char *card_kinds2 = "J23456789TQKA";

void solve2()
{
    hands = (hand_t*)malloc(nr_lines * sizeof(hand_t));
    
    for (int i = 0; i < nr_lines; i++)
    {
        nr_card_t cards[13];
        for (int k = 0; k < 13; k++)
        {
            cards[k].nr = 0;
            cards[k].card = k;
        }
        
        for (int j = 0; j < 5; j++)
        {
            int k = strchr(card_kinds2, lines[i][j]) - card_kinds2;
            if (k < 0 || k >= 13) { printf("ERROR\n"); exit(1); }
            printf("%c", card_kinds[k]); 
            cards[k].nr++;
            hands[i].values[j] = k;
        }
        
        printf(" ");
        
        int jv = cards[0].nr;
        cards[0].nr = 0;
        qsort(cards, 13, sizeof(nr_card_t), larger_nr_card);
        cards[0].nr += jv;
        
        for (int j = 0; j < 5; j++)
        {
            hands[i].cards[j] = cards[j];
            if (cards[j].nr > 0)
                printf("%d:%c ", cards[j].nr, card_kinds[cards[j].card]);
            else
                printf("    ");
        }
        printf(" ");

        char *s = lines[i] + 6;
        hands[i].bid = parse_number(&s);
        printf("%lld\n", hands[i].bid);
    }

    printf("\n");
    
    qsort(hands, nr_lines, sizeof(hand_t), smaller_hand);
    
    num_t score = 0;

    for (int i = 0; i < nr_lines; i++)
    {
        for (int j = 0; j < 5; j++)
        {
            if (hands[i].cards[j].nr > 0)
                printf("%d ", hands[i].cards[j].nr);
            else
                printf("  ");
        }
        for (int j = 0; j < 5; j++)
            printf("%c", card_kinds2[hands[i].values[j]]);
        printf(" %lld %d %lld %lld\n", hands[i].bid, i + 1, hands[i].bid * (i + 1), score);
        score += hands[i].bid * (i + 1);
    }
    
    printf("\n%lld\n", score);
}

```

At 21:46, I found the answer to the second part of the puzzle.

    
### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day07.md >day07.c; gcc day07.c -o day07; ./day07
```

