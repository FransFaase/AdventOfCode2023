# Day 24 of Advent of Code 2023

At 12:08, I started writing this page. I already read the exercise.
It is not that hard, except if you language does not have big numbers.
And also that you have to use integer arithmetic when the cutting points
can have fractional values. Lets first see, if the numbers are not too
big for 64 bit integers. I used [`wc`](https://en.wikipedia.org/wiki/Wc_%28Unix%29)
to find out that there are 300 lines in the input.

```c
int main(int argc, char *argv[])
{
    read_file("input/day24.txt");
    //min_xy = 70000;
    //max_xy = 270000;

    solve1();
}

typedef struct hailstone hailstone_t;
struct hailstone
{
    num_t x, y, z, dx, dy, dz;
};

hailstone_t hailstones[300];

void solve1()
{
    read_hailstones();
    for (int i = 0; i < nr_lines; i++)
        printf("%lld, %lld, %lld @ %lld, %lld, %lld\n",
            hailstones[i].x,
            hailstones[i].y,
            hailstones[i].z,
            hailstones[i].dx,
            hailstones[i].dy,
            hailstones[i].dz);
}

void read_hailstones()
{
    for (int i = 0; i < nr_lines; i++)
    {
        char *s = lines[i];
        hailstones[i].x = parse_number(&s); s += 2;
        hailstones[i].y = parse_number(&s); s += 2;
        hailstones[i].z = parse_number(&s); s += 3;
        hailstones[i].dx = parse_number(&s); s += 2;
        hailstones[i].dy = parse_number(&s); s += 2;
        hailstones[i].dz = parse_number(&s);
    }
}
```

Okay, the above code reproduces the input.

At 12:38, lets see if there are any hailstones that do not pass
through the target area. Because in that case it is already
impossible that it will intersect with another hailstone within
the target area.

```c
num_t min_xy = 200000000000000L;
num_t max_xy = 400000000000000L;

struct hailstone
{
    ...
    bool hit;
};

void check_target_area()
{
    for (int i = 0; i < nr_lines; i++)
    {
        printf("%3d:", i);
        num_t x = hailstones[i].x;
        num_t y = hailstones[i].y;
        num_t dx = hailstones[i].dx;
        num_t dy = hailstones[i].dy;
        hailstones[i].hit = FALSE;
        if (x > max_xy && dx > 0)
            printf(" x > max_x");
        else if (x < min_xy && dx < 0)
            printf(" x < min_x");
        else if (y > max_xy && dy > 0)
            printf(" y > max_y");
        else if (y < min_xy && dy < 0)
            printf(" y < min_y");
        else if (min_xy <= x && x <= max_xy && min_xy <= y && y <= max_xy)
        {
            printf(" start within");
            hailstones[i].hit = TRUE;
        }
        else
        {
            num_t ny = y * dx + min_xy - x;
            if (min_xy * dx <= ny && ny <= max_xy * dx)
            {
                printf(" left");
                hailstones[i].hit = TRUE;
            }
            ny = y * dx + max_xy - x;
            if (min_xy * dx <= ny && ny <= max_xy * dx)
            {
                printf(" right");
                hailstones[i].hit = TRUE;
            }
            num_t nx = x * dy + min_xy - y;
            if (min_xy * dy <= nx && nx <= max_xy * dy)
            {
                printf(" bottom");
                hailstones[i].hit = TRUE;
            }
            nx = x * dy + max_xy - y;
            if (min_xy * dy <= nx && nx <= max_xy * dy)
            {
                printf(" top");
                hailstones[i].hit = TRUE;
            }
            if (!hailstones[i].hit)
                printf(" no hit");
        }
        printf("\n");
    }
}

void solve1()
{
    read_hailstones();

    check_target_area();
}
```



```c
void solve1()
{
    ...
    int answer = 0;
    for (int i = 0; i < nr_lines - 1; i++)
        for (int j = i + 1; j < nr_lines; j++)
        {
            num_t x1 = hailstones[i].x;
            num_t y1 = hailstones[i].y;
            num_t dx1 = hailstones[i].dx;
            num_t dy1 = hailstones[i].dy;
            num_t x2 = hailstones[j].x;
            num_t y2 = hailstones[j].y;
            num_t dx2 = hailstones[j].dx;
            num_t dy2 = hailstones[j].dy;
            printf("Hailstone A: %lld, %lld @ %lld, %lld\n", x1, y1, dx1, dy1);
            printf("Hailstone B: %lld, %lld @ %lld, %lld\n", x2, y2, dx2, dy2);
            num_t dot = dx1 * dy2 - dx2 * dy1;
            //printf("Dot = %lld ", dot);
            if (dot == 0)
                printf("Hailstones' paths are parallel; they never intersect.\n");
            else
            {
                num_t sign_dot = num_t_sign(dot);
                dot = num_t_abs(dot);
                num_t s_num = sign_dot * (dx2 * (y1 - y2) + dy2 * (x2 - x1));
                //printf("\nNumA = %lld ", s_num);
                num_t s_gcd = gcd(num_t_abs(s_num), dot);
                s_num /= s_gcd;
                num_t s_denom = dot / s_gcd;
                //printf("(%lld) %lld / %lld %lf ", s_gcd, s_num, s_denom, s_num / (double)s_denom);
                bool s_before = s_num < 0;
                //if (s_before)
                //    printf("before! "); 
                //printf("at %lf, %lf\n", 
                //    x1 + dx1 * s_num / (double)s_denom, 
                //    y1 + dy1 * s_num / (double)s_denom); 
            
                num_t t_num = sign_dot * (dx1 * (y1 - y2) + dy1 * (x2 - x1));
                //printf("\nNumB = %lld ", t_num);
                num_t t_gcd = gcd(num_t_abs(t_num), dot);
                t_num /= t_gcd;
                num_t t_denom = dot / t_gcd;
                //printf("(%lld) %lld / %lld %lf ", t_gcd, t_num, t_denom, t_num / (double)t_denom);
                bool t_before = t_num < 0;
                //if (t_before)
                //    printf("before! "); 
                //printf("at %lf, %lf\n", 
                //    x2 + dx2 * t_num / (double)t_denom, 
                //    y2 + dy2 * t_num / (double)t_denom); 
        
                //printf("\n");            
                num_t x = s_denom * x1 + s_num * dx1;
                bool x_inside = s_denom * min_xy <= x && x <= s_denom * max_xy;
                //if (x_inside)
                //    printf("inside "); 
            
                num_t y = s_denom * y1 + s_num * dy1;
                bool y_inside = s_denom * min_xy <= y && y <= s_denom * max_xy;
                //if (y_inside)
                //    printf("inside "); 
                
                //if (!t_before && !s_before && x_inside && y_inside)
                //    printf("INSIDE");
                    
                if (t_before && s_before)
                    printf("Hailstones' paths crossed in the past for both hailstones.\n");
                else if (s_before)
                    printf("Hailstones' paths crossed in the past for hailstone A.\n");    
                else if (t_before)
                    printf("Hailstones' paths crossed in the past for hailstone B.\n");
                else if (x_inside && y_inside)
                {
                    printf("Hailstones' paths will cross INSIDE the test area.\n");
                    answer++;
                }
                else
                    printf("Hailstones' paths will cross outside the test area.\n");    
            }
            printf("\n");
        }
    printf("%d\n", answer);
}

```

The equations to be solved:

```
x1 + dx1 * s = x2 + dx2 * t
y1 + dy1 * s = y2 + dy2 * t

dy2 * x1 + dx1 * dy2 * s = dy2 * x2 + dx2 * dy2 * t
dx2 * y1 + dx2 * dy1 * s = dx2 * y2 + dx2 * dy2 * t
dy2 * x1 - dx2 * y1 + s * (dx1 * dy2 - dx2 * dy1) = dy2 * x2 - dx2 * y2  
s * (dx1 * dy2 - dx2 * dy1) = dy2 * x2 - dx2 * y2 - dy2 * x1 + dx2 * y1  
s * (dx1 * dy2 - dx2 * dy1) = dx2 * y1 - dx2 * y2 + dy2 * x2 - dy2 * x1   
s = (dx2 * (y1 - y2) + dy2 * (x2 - x1)) / (dx1 * dy2 - dx2 * dy1)   

dy1 * x1 + dy1 * dx1 * s = dy1 * x2 + dy1 * dx2 * t
dx1 * y1 + dy1 * dx1 * s = dx1 * y2 + dx1 * dy2 * t
dx1 * y1 - dy1 * x1 = dx1 * y2 - dy1 * x2 + (dx1 * dy2 - dy1 * dx2) * t
dx1 * y1 - dy1 * x1 - dx1 * y2 + dy1 * x2 = (dx1 * dy2 - dy1 * dx2) * t
dx1 * y1 - dx1 * y2 + dy1 * x2 - dy1 * x1 = (dx1 * dy2 - dy1 * dx2) * t
dx1 * (y1 - y2) + dy1 * (x2 - x1) = (dx1 * dy2 - dy1 * dx2) * t
t = (dx1 * (y1 - y2) + dy1 * (x2 - x1)) / (dx1 * dy2 - dy1 * dx2)

```

Around 18:00, I got the program return the correct answers for the example input,
but it did not return the correct answer for the test input. Their might be an
edge case in my puzzle input that is not in the example input or their might be
an overflow somewhere.

```c

num_t mul(num_t a, num_t b)
{
    num_t sign = num_t_sign(a) * num_t_sign(b);
    num_t abs_a = num_t_abs(a);
    num_t abs_b = num_t_abs(b);
    num_t r = abs_a * abs_b;
    if (r < a || r < b)
    {
        printf("%lld * %lld = %lld = %lld\n", a, b, sign, r);
        printf("OVERFLOW\n");
        exit(1);
    }
    return sign * r;
}

void solve1()
{
    read_hailstones();

    check_target_area();

    int answer = 0;
    for (int i = 0; i < nr_lines - 1; i++)
        for (int j = i + 1; j < nr_lines; j++)
        {
            num_t x1 = hailstones[i].x;
            num_t y1 = hailstones[i].y;
            num_t dx1 = hailstones[i].dx;
            num_t dy1 = hailstones[i].dy;
            num_t x2 = hailstones[j].x;
            num_t y2 = hailstones[j].y;
            num_t dx2 = hailstones[j].dx;
            num_t dy2 = hailstones[j].dy;
            printf("Hailstone A: %lld, %lld @ %lld, %lld\n", x1, y1, dx1, dy1);
            printf("Hailstone B: %lld, %lld @ %lld, %lld\n", x2, y2, dx2, dy2);
            num_t dot = dx1 * dy2 - dx2 * dy1;
            //printf("Dot = %lld ", dot);
            if (dot == 0)
                printf("Hailstones' paths are parallel; they never intersect.\n");
            else
            {
                num_t sign_dot = num_t_sign(dot);
                dot = num_t_abs(dot);
                num_t s_num = sign_dot * (mul(dx2, (y1 - y2)) + mul(dy2, (x2 - x1)));
                //printf("\nNumA = %lld ", s_num);
                num_t s_gcd = gcd(num_t_abs(s_num), dot);
                s_num /= s_gcd;
                num_t s_denom = dot / s_gcd;
                //printf("(%lld) %lld / %lld %lf ", s_gcd, s_num, s_denom, s_num / (double)s_denom);
                bool s_before = s_num < 0;
                //if (s_before)
                //    printf("before! "); 
                //printf("at %lf, %lf\n", 
                //    x1 + dx1 * s_num / (double)s_denom, 
                //    y1 + dy1 * s_num / (double)s_denom); 
            
                num_t t_num = sign_dot * (mul(dx1, (y1 - y2)) + mul(dy1, (x2 - x1)));
                //printf("\nNumB = %lld ", t_num);
                num_t t_gcd = gcd(num_t_abs(t_num), dot);
                t_num /= t_gcd;
                num_t t_denom = dot / t_gcd;
                //printf("(%lld) %lld / %lld %lf ", t_gcd, t_num, t_denom, t_num / (double)t_denom);
                bool t_before = t_num < 0;
                //if (t_before)
                //    printf("before! "); 
                //printf("at %lf, %lf\n", 
                //    x2 + dx2 * t_num / (double)t_denom, 
                //    y2 + dy2 * t_num / (double)t_denom); 
        
                //printf("\n");            
                num_t x = mul(s_denom, x1) + mul(s_num, dx1);
                bool x_inside = mul(s_denom, min_xy) <= x && x <= mul(s_denom, max_xy);
                //if (x_inside)
                //    printf("inside "); 
            
                num_t y = mul(s_denom, y1) + mul(s_num, dy1);
                bool y_inside = mul(s_denom, min_xy) <= y && y <= mul(s_denom, max_xy);
                //if (y_inside)
                //    printf("inside "); 
                
                //if (!t_before && !s_before && x_inside && y_inside)
                //    printf("INSIDE");
                    
                if (t_before && s_before)
                    printf("Hailstones' paths crossed in the past for both hailstones.\n");
                else if (s_before)
                    printf("Hailstones' paths crossed in the past for hailstone A.\n");    
                else if (t_before)
                    printf("Hailstones' paths crossed in the past for hailstone B.\n");
                else if (x_inside && y_inside)
                {
                    printf("Hailstones' paths will cross INSIDE the test area.\n");
                    answer++;
                }
                else
                    printf("Hailstones' paths will cross outside the test area.\n");    
            }
            printf("\n");
        }
    printf("%d\n", answer);
}

```

And yes, there is an overflow in one of the multiplications.

```
#define BIG_NUM_SIZE 20
#define BIG_NUM_MUL 1000

typedef struct bignum bignum_t;
struct bignum
{
    int sign;
    long val[BIG_NUM_SIZE];
};

void big_num_set(bignum_t *r, num_t value)
{
    //printf("set %lld ", value);
    r->sign = value >= 0 ? 1 : -1;
    value = num_t_abs(value);
    for (int i = 0; i < BIG_NUM_SIZE; i++)
    {
        r->val[i] = value % BIG_NUM_MUL;
        value = value / BIG_NUM_MUL;
    }
    //big_num_print(r); printf("\n");
}

void big_num_minus(bignum_t *r, bignum_t *a)
{
    r->sign = -1 * a->sign;
    for (int i = 0; i < BIG_NUM_SIZE; i++)
        r->val[i] = a->val[i];
    //printf("- "); big_num_print(a); printf(" = "); big_num_print(r); printf("\n");
}

void big_num_add(bignum_t *r, bignum_t *a, bignum_t *b)
{
    if (a->sign == b->sign)
    {
        r->sign = a->sign;
        long carry = 0;
        for (int i = 0; i < BIG_NUM_SIZE; i++)
        {
            long sum = a->val[i] + b->val[i] + carry;
            carry = sum / BIG_NUM_MUL;
            r->val[i] = sum % BIG_NUM_MUL;
        }
    }
    else
    {
        if (a->sign == -1) { bignum_t *h = a; a = b; b = h; }
        
        bool b_larger = FALSE;
        for (int i = BIG_NUM_SIZE-1; i >= 0; i--)
            if (b->val[i] > a->val[i])
                b_larger = TRUE;
            else if (b->val[i] < a->val[i])
                break;
        r->sign = 1;
        if (b_larger)
        {
            r->sign = -1;
            bignum_t *h = a; a = b; b = h;
        }
        
        long carry = 0;
        for (int i = 0; i < BIG_NUM_SIZE; i++)
        {
            long sub = a->val[i] - b->val[i] + carry;
            carry = 0;
            if (sub < 0)
            {
                sub += BIG_NUM_MUL;
                carry = -1;
            }
            r->val[i] = sub;
        }
    }
    if (big_num_val(a) + big_num_val(b) != big_num_val(r)) printf("DIFF: ");
    big_num_print(a); printf(" + "); big_num_print(b); printf(" = "); big_num_print(r); printf("\n");
}

int big_num_sign(bignum_t *v)
{
    for (int i = 0; i < BIG_NUM_SIZE; i++)
        if (v->val[i] != 0)
            return v->sign;
    return 0;
}

void big_num_mul(bignum_t *r, bignum_t *a, bignum_t *b)
{
    r->sign = a->sign * b->sign;
    
    for (int i = 0; i < BIG_NUM_SIZE; i++)
        r->val[i] = 0;
    
    for (int i = 0; i < BIG_NUM_SIZE; i++)
    {
        long carry = 0;
        for (int j = 0; i + j < BIG_NUM_SIZE; j++)
        {
            long sum = a->val[i] * b->val[j] + carry;
            carry = sum / BIG_NUM_MUL;
            r->val[i+j] += sum % BIG_NUM_MUL;
        }
    }
    if (big_num_val(a) * big_num_val(b) != big_num_val(r)) printf("DIFF: ");
    big_num_print(a); printf(" * "); big_num_print(b); printf(" = "); big_num_print(r); printf("\n");
}

int big_num_comp(bignum_t *a, bignum_t *b)
{
    return 
    int sign_a = big_num_sign(a);
    int sign_b = big_num_sign(b);
    if (sign_a < sign_b) return -1;
    if (sign_a > sign_b) return 1;
    if (sign_a == 0)
        return 0;
    
    int comp = 0;
    for (int i = BIG_NUM_SIZE-1; i >= 0 && comp == 0; i--)
        comp = a->val[i] - b->val[i];
    //printf("comp "); big_num_print(a); printf(" "); big_num_print(b); printf(" %d\n", sign_a * comp);
    comp = comp < 0 ? -1 : comp > 0 ? 1 : 0;
    int c = sign_a * comp;
    num_t diff = big_num_val(a) - big_num_val(b);
    int lc = diff < 0 ? -1 : diff > 0 ? 1 : 0;
    if (c != lc)
    {
        printf("DIFF: ");
        printf("comp "); big_num_print(a); printf(" "); big_num_print(b); printf(" %d*%d %d %lld %lld\n", sign_a, comp, lc, big_num_val(a), big_num_val(b));
    }
    
    return sign_a * comp;
}

num_t big_num_val(bignum_t *a)
{
    num_t v = 0;
    for (int i = BIG_NUM_SIZE-1; i >= 0; i--)
    {
        v = BIG_NUM_MUL * v + a->val[i];
    }
    //printf("Val %lld ", a->sign * v);
    //big_num_print(a); printf("\n");
    return (num_t)a->sign * v;
}    

void big_num_print(bignum_t *a)
{
    if (a->sign == -1)
        printf("-");
    bool zero = TRUE;
    for (int i = BIG_NUM_SIZE-1; i >= 0; i--)
        if (!zero)
            printf("%03ld%s", a->val[i], i > 0 ? "," : "");
        else if (a->val[i] != 0 || i == 0)
        {
            printf("%ld%s", a->val[i], i > 0 ? "," : "");
            zero = FALSE;
        }
}

void test_big_num()
{
    num_t number[10] = { 0, 1, -1, 4000, -4896344, 4896344, 666, 666, 7777, 88888};
    
    for (int i = 0; i < 10; i++)
    {
        bignum_t v; big_num_set(&v, number[i]);
        bignum_t min_v; big_num_minus(&min_v, &v);
        if (big_num_val(&min_v) != -number[i])
            printf("ERROR min %d\n", i);
            
        for (int j = 0; j < 10; j++)
        {
            bignum_t a; big_num_set(&a, number[i]); 
            bignum_t b; big_num_set(&b, number[j]);
            bignum_t r; big_num_add(&r, &a, &b);
            if (big_num_val(&r) != number[i] + number[j])
                printf("ERROR add %d %d\n", i, j);
        }
        
        for (int j = 0; j < 10; j++)
        {
            bignum_t a; big_num_set(&a, number[i]); 
            bignum_t b; big_num_set(&b, number[j]);
            bignum_t r; big_num_mul(&r, &a, &b);
            if (big_num_val(&r) != number[i] * number[j])
                printf("ERROR mul %d %d\n", i, j);
        }

        for (int j = 0; j < 10; j++)
        {
            bignum_t a; big_num_set(&a, number[i]); 
            bignum_t b; big_num_set(&b, number[j]);
            int c = big_num_comp(&a, &b);
            int cm = number[i] < number[j] ? -1 : number[i] > number[j] ? 1 : 0;
            if (c != cm)
                printf("ERROR comp %d %d  %d %d\n", i, j, c, cm);
        }
    }
}
    
```

At 19:10, lets test it the example input. First have to adapt `solve1`.

```c
void solve1()
{
    read_hailstones();

    check_target_area();
    
    bignum_t b_min_xy; big_num_set(&b_min_xy, min_xy);
    bignum_t b_max_xy; big_num_set(&b_max_xy, max_xy);

    int answer = 0;
    for (int i = 0; i < nr_lines - 1; i++)
        for (int j = i + 1; j < nr_lines; j++)
        {
            num_t x1 = hailstones[i].x;
            num_t y1 = hailstones[i].y;
            num_t dx1 = hailstones[i].dx;
            num_t dy1 = hailstones[i].dy;
            num_t x2 = hailstones[j].x;
            num_t y2 = hailstones[j].y;
            num_t dx2 = hailstones[j].dx;
            num_t dy2 = hailstones[j].dy;
            printf("Hailstone A: %lld, %lld @ %lld, %lld\n", x1, y1, dx1, dy1);
            printf("Hailstone B: %lld, %lld @ %lld, %lld\n", x2, y2, dx2, dy2);
            num_t dot = dx1 * dy2 - dx2 * dy1;
            //printf("Dot = %lld ", dot);
            if (dot == 0)
                printf("Hailstones' paths are parallel; they never intersect.\n");
            else
            {
                num_t sign_dot = num_t_sign(dot);
                dot = num_t_abs(dot);
                bignum_t b_dot; big_num_set(&b_dot, dot);
                bignum_t b_x1; big_num_set(&b_x1, x1);
                bignum_t b_x2; big_num_set(&b_x2, x2);
                bignum_t b_y1; big_num_set(&b_y1, y1);
                bignum_t b_y2; big_num_set(&b_y2, y2);
                bignum_t b_dx1; big_num_set(&b_dx1, dx1);
                bignum_t b_dy1; big_num_set(&b_dy1, dy1);
                bignum_t b_dx2; big_num_set(&b_dx2, dx2);
                bignum_t b_dy2; big_num_set(&b_dy2, dy2);
                
                bignum_t b_m_x1; big_num_minus(&b_m_x1, &b_x1);
                bignum_t b_diff_x; big_num_add(&b_diff_x, &b_m_x1, &b_x2);
                
                bignum_t b_m_y2; big_num_minus(&b_m_y2, &b_y2);
                bignum_t b_diff_y; big_num_add(&b_diff_y, &b_y1, &b_m_y2);

                bignum_t t1; big_num_mul(&t1, &b_dx2, &b_diff_y);
                bignum_t t2; big_num_mul(&t2, &b_dy2, &b_diff_x);
                bignum_t s_num; big_num_add(&s_num, &t1, &t2);
                big_num_mul_sign(&s_num, sign_dot);
                
                bool s_before = big_num_sign(&s_num) < 0;

/*                    
                bignum_t b_m_y2; big_num_minus(
                num_t s_num = sign_dot * (mul(dx2, (y1 - y2)) + mul(dy2, (x2 - x1)));
                //printf("\nNumA = %lld ", s_num);
                num_t s_gcd = gcd(num_t_abs(s_num), dot);
                s_num /= s_gcd;
                num_t s_denom = dot / s_gcd;
                //printf("(%lld) %lld / %lld %lf ", s_gcd, s_num, s_denom, s_num / (double)s_denom);
                bool s_before = s_num.sign < 0;
                //if (s_before)
                //    printf("before! "); 
                //printf("at %lf, %lf\n", 
                //    x1 + dx1 * s_num / (double)s_denom, 
                //    y1 + dy1 * s_num / (double)s_denom); 
*/

                bignum_t t3; big_num_mul(&t3, &b_dx1, &b_diff_y);
                bignum_t t4; big_num_mul(&t4, &b_dy1, &b_diff_x);
                bignum_t t_num; big_num_add(&t_num, &t3, &t4);
                big_num_mul_sign(&s_num, sign_dot);

                bool t_before = big_num_sign(&t_num) < 0;

/*                            
                num_t t_num = sign_dot * (mul(dx1, (y1 - y2)) + mul(dy1, (x2 - x1)));
                //printf("\nNumB = %lld ", t_num);
                num_t t_gcd = gcd(num_t_abs(&t_num), dot);
                t_num /= t_gcd;
                num_t t_denom = dot / t_gcd;
                //printf("(%lld) %lld / %lld %lf ", t_gcd, t_num, t_denom, t_num / (double)t_denom);
                bool t_before = t_num < 0;
                //if (t_before)
                //    printf("before! "); 
                //printf("at %lf, %lf\n", 
                //    x2 + dx2 * t_num / (double)t_denom, 
                //    y2 + dy2 * t_num / (double)t_denom); 
*/
        
                bignum_t b_min; big_num_mul(&b_min, &b_dot, &b_min_xy);
                bignum_t b_max; big_num_mul(&b_max, &b_dot, &b_max_xy);
                
                //printf("\n");
                
                bignum_t t5; big_num_mul(&t5, &b_dot, &b_x1);
                bignum_t t6; big_num_mul(&t6, &s_num, &b_dx1);
                bignum_t x; big_num_add(&x, &t5, &t6);
                
                bool x_inside = big_num_comp(&b_min, &x) <= 0 && big_num_comp(&x, &b_max) <= 0;
                
                bignum_t t7; big_num_mul(&t7, &b_dot, &b_y1);
                bignum_t t8; big_num_mul(&t8, &s_num, &b_dy1);
                bignum_t y; big_num_add(&y, &t7, &t8);
                
                bool y_inside = big_num_comp(&b_min, &y) <= 0 && big_num_comp(&y, &b_max) <= 0;

/*
                num_t x = mul(s_denom, x1) + mul(s_num, dx1);
                bool x_inside = mul(s_denom, min_xy) <= x && x <= mul(s_denom, max_xy);
                //if (x_inside)
                //    printf("inside "); 
            
                num_t y = mul(s_denom, y1) + mul(s_num, dy1);
                bool y_inside = mul(s_denom, min_xy) <= y && y <= mul(s_denom, max_xy);
                //if (y_inside)
                //    printf("inside "); 
                
                //if (!t_before && !s_before && x_inside && y_inside)
                //    printf("INSIDE");
*/
                    
                if (t_before && s_before)
                    printf("Hailstones' paths crossed in the past for both hailstones.\n");
                else if (s_before)
                    printf("Hailstones' paths crossed in the past for hailstone A.\n");    
                else if (t_before)
                    printf("Hailstones' paths crossed in the past for hailstone B.\n");
                else if (x_inside && y_inside)
                {
                    printf("Hailstones' paths will cross INSIDE the test area.\n");
                    answer++;
                }
                else
                    printf("Hailstones' paths will cross outside the test area.\n");    
            }
            printf("\n");
        }
    printf("%d\n", answer);
    
}

```

At 21:11, the above code now returns the correct answer with the example input.
I added some testing code in the code above.

At 21:57, fixed some bugs in the `big_num_t` methodes. All unit test pass now.
It still returns the same answer.

At 22:11, fixed another bug.

At 22:32, I am about to quit. There must be some edge cases that my code is not dealing
correctly. I am going online for some tips.

At 22:46, it seems that GCC has a 182 bit integer. Try that.

```c
typedef __int128 bignum_t;

void big_num_set(bignum_t *r, num_t value)
{
    *r = value;
}

void big_num_minus(bignum_t *r, bignum_t *a)
{
    *r = - *a;
}

void big_num_add(bignum_t *r, bignum_t *a, bignum_t *b)
{
    *r = *a + *b;
}

int big_num_sign(bignum_t *v)
{
    return *v < 0 ? -1 : *v > 0 ? 1 : 0;
}

void big_num_mul(bignum_t *r, bignum_t *a, bignum_t *b)
{
    *r = *a * *b;
}

int big_num_comp(bignum_t *a, bignum_t *b)
{
    return *a < *b ? -1 : *a > *b ? 1 : 0;
}

num_t big_num_val(bignum_t *a)
{
    return (num_t)*a;
}    

void big_num_print(bignum_t *a)
{
}

void big_num_mul_sign(bignum_t *a, int sign)
{
    *a = *a * sign;
}
    
```

At 23:08, using 128 bit integers does return a different answer, but it is still incorrect.
So, there is a bug in my `big_num_t` implementation and there is still a bug in my code.

```c
void solve1()
{
    read_hailstones();

    check_target_area();
    
    int answer = 0;
    for (int i = 0; i < nr_lines - 1; i++)
        for (int j = i + 1; j < nr_lines; j++)
        {
            __int128 x1 = hailstones[i].x;
            __int128 y1 = hailstones[i].y;
            __int128 dx1 = hailstones[i].dx;
            __int128 dy1 = hailstones[i].dy;
            __int128 x2 = hailstones[j].x;
            __int128 y2 = hailstones[j].y;
            __int128 dx2 = hailstones[j].dx;
            __int128 dy2 = hailstones[j].dy;
            //printf("Hailstone A: %llld, %llld @ %llld, %llld\n", x1, y1, dx1, dy1);
            //printf("Hailstone B: %llld, %llld @ %llld, %llld\n", x2, y2, dx2, dy2);
            __int128 dot = dx1 * dy2 - dx2 * dy1;
            //printf("Dot = %lld ", dot);
            if (dot == 0)
                printf("Hailstones' paths are parallel; they never intersect.\n");
            else
            {
                num_t sign_dot = num_t_sign(dot);
                dot = num_t_abs(dot);
                __int128 s_num = sign_dot * (dx2 * (y1 - y2) + dy2 * (x2 - x1));
                //printf("(%lld) %lld / %lld %lf ", s_gcd, s_num, s_denom, s_num / (double)s_denom);
                bool s_before = s_num < 0;
                //if (s_before)
                //    printf("before! "); 
                //printf("at %lf, %lf\n", 
                //    x1 + dx1 * s_num / (double)s_denom, 
                //    y1 + dy1 * s_num / (double)s_denom); 
            
                __int128 t_num = sign_dot * (dx1 * (y1 - y2) + dy1 * (x2 - x1));
                //printf("\nNumB = %lld ", t_num);
                //num_t t_gcd = gcd(num_t_abs(t_num), dot);
                //t_num /= t_gcd;
                //num_t t_denom = dot / t_gcd;
                //printf("(%lld) %lld / %lld %lf ", t_gcd, t_num, t_denom, t_num / (double)t_denom);
                bool t_before = t_num < 0;
                //if (t_before)
                //    printf("before! "); 
                //printf("at %lf, %lf\n", 
                //    x2 + dx2 * t_num / (double)t_denom, 
                //    y2 + dy2 * t_num / (double)t_denom); 
        
                //printf("\n");            
                __int128 x = dot * x1 + s_num * dx1;
                bool x_inside = dot * min_xy <= x && x <= dot * max_xy;
                //if (x_inside)
                //    printf("inside "); 
            
                __int128 y = dot * y1 + s_num * dy1;
                bool y_inside = dot * min_xy <= y && y <= dot * max_xy;
                //if (y_inside)
                //    printf("inside "); 
                
                //if (!t_before && !s_before && x_inside && y_inside)
                //    printf("INSIDE");
                    
                if (t_before && s_before)
                    printf("Hailstones' paths crossed in the past for both hailstones.\n");
                else if (s_before)
                    printf("Hailstones' paths crossed in the past for hailstone A.\n");    
                else if (t_before)
                    printf("Hailstones' paths crossed in the past for hailstone B.\n");
                else if (x_inside && y_inside)
                {
                    printf("Hailstones' paths will cross INSIDE the test area.\n");
                    answer++;
                }
                else
                    printf("Hailstones' paths will cross outside the test area.\n");    
            }
            printf("\n");
        }
    printf("%d\n", answer);
}

```

At 23:27, I found the correct answer with the above code. Conclusion must be that
there is a bug in the above implementation using 128 bit integers. I must have called one
of the methods with the wrong arguments. Maybe I will look into it later.

### Second part of the puzzle.

Lots of equations. So, you have to find a ray that hits all hailstones. Maybe we
just try some directions and see if we can find solutions for all hailstones.

Lots of notes.

```
x1 + a * dx1 - b * rdx   =
y1 + a * dy1   =   f + b * rdy
z1 + a * dz1   =   g + b * rdz

x2 + c * dx2 - d * rdx  = e 
y2 + c * dy2   =   f + d * rdy 
z2 + c * dz2   =   g + d * rdz

x1 + a * dx1 - b * rdx = x2 + c * dx2 - d * rdx

h = d - b

a * dx1 = x2-x1 + c * dx2 + h * rdx
a * dy1 = y2-y1 + c * dy2 + h * rdy

(x2-x1 + c * dx2 + h * rdx)*dy1 = (c * dy2 + h * rdy)*dx1 
(x2-x1)*dy1 + c * dx2*dy1 + h * rdx*dy1 = c * dy2*dx1 + h * rdy*dx1 
c * (dx2*dy1-dy2*dx1) = h * (rdy*dx1-rdx*dy1) - (x2-x1)*dy1 
c * (dx2*dz1-dz2*dx1) = h * (rdz*dx1-rdx*dz1) - (x2-x1)*dz1
 
(h * (rdy*dx1-rdx*dy1) - (x2-x1)*dy1)*(dx2*dz1-dz2*dx1) = (h * (rdz*dx1-rdx*dz1) - (x2-x1)*dz1)*(dx2*dy1-dy2*dx1) 

h * (rdy*dx1-rdx*dy1)*(dx2*dz1-dz2*dx1) - (x2-x1)*dy1*(dx2*dz1-dz2*dx1) = h * (rdz*dx1-rdx*dz1)*(dx2*dy1-dy2*dx1) - (x2-x1)*dz1*(dx2*dy1-dy2*dx1) 
h * (rdy*dx1-rdx*dy1)*(dx2*dz1-dz2*dx1) = h * (rdz*dx1-rdx*dz1)*(dx2*dy1-dy2*dx1) + (x2-x1)*dy1*(dx2*dz1-dz2*dx1) - (x2-x1)*dz1*(dx2*dy1-dy2*dx1) 
h * ((rdy*dx1-rdx*dy1)*(dx2*dz1-dz2*dx1)-(rdz*dx1-rdx*dz1)*(dx2*dy1-dy2*dx1)) = (x2-x1)*dy1*(dx2*dz1-dz2*dx1) - (x2-x1)*dz1*(dx2*dy1-dy2*dx1) 


x1 + a * dx1   =   e
y1 + a * dy1   =   f
z1 + a * dz1   =   g + b * rdz

x2 + c * dx2 - d * rdx  = e 
y2 + c * dy2   =   f + d * rdy 
z2 + c * dz2   =   g + d * rdz

h * (  -  ) = 
h * ((rdy*dx1-rdx*dy1)*(dx2*dz1-dz2*dx1) - (rdz*dx1-rdx*dz1)*(dx2*dy1-dy2*dx1)) (x2-x1)*dy1*(dx2*dz1-dz2*dx1) =  - (x2-x1)*dz1*(dx2*dy1-dy2*dx1) 


x1 + a * dx1 - a * rdx - b * rdx = x2 + c * dx2 - d * rdx

a * (dx1-rdx) = c * dx2 - d * rdx + b * rdx + x2 - x1
a * (dy1-rdy) = c * dy2 - d * rdy + b * rdy + y2 - y1

(c * dy2 - d * rdy + b * rdy + y2 - y1) * (dx1-rdx) = (c * dy2 - d * rdy + b * rdy + y2 - y1) * (dx1-rdx)

c * dy2 * (dx1-rdx) + (- d * rdy + b * rdy + y2 - y1) * (dx1-rdx) = (c * dy2) * (dx1-rdx) + ( - d * rdy + b * rdy + y2 - y1) * (dx1-rdx)

c * dy2*(dx1-rdx) + (- d * rdy + b * rdy + y2 - y1) * (dx1-rdx) = c * dy2*(dx1-rdx) + (- d * rdy + b * rdy + y2 - y1) * (dx1-rdx)

c * (dy2*(dx1-rdx) - dy2*(dx1-rdx)) = (- d * rdy + b * rdy + y2 - y1) * (dx1-rdx) - (- d * rdy + b * rdy + y2 - y1) * (dx1-rdx)

c * (dy2*(dx1-rdx)-dy2*(dx1-rdx)) = d * (rdy*(dx1-rdx)-rdy*(dx1-rdx) + b * (rdy*(dx1-rdx)-rdy*(dx1-rdx) + (y2-y1)*(dx1-rdx)-(y2-y1)*(dx1-rdx)
c * (dx2*(dx1-rdx)-dz2*(dx1-rdx)) = d * (rdz*(dx1-rdx)-rdz*(dx1-rdx) + b * (rdz*(dx1-rdx)-rdz*(dx1-rdx) + (z2-z1)*(dx1-rdx)-(z2-z1)*(dx1-rdx)


e = 
x1 + a * dx1   =   e + a * rdx + b * rdx
a * (dx1 - rdx) = e + b * rdx - x1


a * dx1 - b * rdx = x2 - x1 + c * dx2 - d * rdx

a = (x2 - x1 + c * dx2 + (b-d) + tdx)/dx1
a = (y2 - y1 + c * dy2 + (b-d) + tdy)/dy1
a = (z2 - z1 + c * dz2 + (b-d) + tdz)/dz1

c * dy1*dx2 + dy1*x2 - dy1*x1 + dy1*tdx =  c * dx1*dy2 + dx1*tdy + dx1*y2 - dx1*y1
c = (dx1*tdy + dx1*y2 - dx1*y1 - dy1*dx2 + dy1*x2 - dy1*x1 + dy1*tdx) / (dy1*dx2 - dx1*dy2)

y1 + a * dy1   =   f + b * rdy
z1 + a * dz1   =   g + b * rdz

x2 + c * dx2   =   e + d * rdx
y2 + c * dy2   =   f + d * rdy 
z2 + c * dz2   =   g + d * rdz

x1 + a * dx1  
x1 + a * dx1
y1 + a * dy1 
z1 + a * dz1

x2 + b * dx2
y2 + b * dy2 
z2 + b * dz2


x(a,b) = x1 + a * dx1 + t * (x2 + b * dx2 - x1 + a * dx1)
x(a,b) = x1 + a * dx1 
y(a,b) = y1 + a * dy1 + t * (y2 + b * dy2 - y1 + a * yx1)

x1 + a * dx1   =   rx
y1 + a * dy1   =   ry 
z1 + a * dz1   =   rz

x2 + c * dx2   =   rx + d * rdx
y2 + c * dy2   =   ry + d * rdy 
z2 + c * dz2   =   rz + d * rdz

x2 + c * dx2   =   x1 + a * dx1 + d * rdx
y2 + c * dy2   =   y1 + a * dy1 + d * rdy 
z2 + c * dz2   =   z1 + a * dz1 + d * rdz

x2 + e * dx2   =   x1 + a * (dx1 - dx2) + d * rdx
y2 + e * dy2   =   x1 + a * (dy1 - dy2) + d * rdy
z2 + e * dz2   =   x1 + a * (dy1 - dz2) + d * rdz

x2 - x1 + e * dx2   =   a * (dx1 - dx2) + d * rdx
y2 - y1 + e * dy2   =   a * (dy1 - dy2) + d * rdy
z2 - z1 + e * dz2   =   a * (dy1 - dz2) + d * rdz
```

#### December 26

I have been thinking about the aproach of constructing all paths of a
potential ray through all the point that two hailstones travel and
intersect that one with two other hailstone tracks.

The equations:
```
x1 + a * dx1
x2 + b * dx2
x1 + a * dx1 + c * (x2 + b * dx2 - x1 - a * dx1)
x1 + a * dx1 + c * (x2-x1) + c * b * dx2 - c * a * dx1
x3 + d * dx3
// c can be fractional
x3 + d * dx3 = x1 + a * dx1 + c * (x2-x1) + c * b * dx2 - c * a * dx1
y3 + d * dy3 = y1 + a * dy1 + c * (y2-y1) + c * b * dy2 - c * a * dy1

d * dx3 = x1-x3 + a * dx1 + c * (x2-x1) + c * b * dx2 - c * a * dx1 - x3
d * dy3 = y1 + a * dy1 + c * (y2-y1) + c * b * dy2 - c * a * dy1 - y3
(x1-x3) + a * dx1 + c * (x2-x1) + c * b * dx2 - c * a * dx1 - x3) * dy3 = (y1 + a * dy1 + c * (y2-y1) + c * b * dy2 - c * a * dy1 - y3) * dx3

  (x1-x3)*dy3 + a * dx1*dy3 + c * (x2-x1)*dy3 + c * b * dx2*dy3 - c * a * dx1*dy3 
= (y1-y3)*dx3 + a * dy1*dx3 + c * (y2-y1)*dx3 + c * b * dy2*dx3 - c * a * dy1*dx3 

  (x1-x3)*dy3-(y1-y3)*dx3 + a * (dx1*dy3-dy1*dx3) + c * ((x2-x1)*dy3-(y2-y1)*dx3) + c * b * (dx2*dy-3dy2*dx3) - c * a * (dx1*dy3-dy1*dx3) = 0
  (x1-x3)*dz3-(z1-z3)*dx3 + a * (dx1*dz3-dz1*dx3) + c * ((x2-x1)*dz3-(z2-z1)*dx3) + c * b * (dx2*dz-3dz2*dx3) - c * a * (dx1*dz3-dz1*dx3) = 0

  (y1-y3)*dz3-(z1-z3)*dy3 + a * (dy1*dz3-dz1*dy3) + c * ((y2-y1)*dz3-(z2-z1)*dy3) + c * b * (dy2*dz-3dz2*dy3) - c * a * (dy1*dz3-dz1*dy3) = 0

f1xy = (x1-x3)*dy3-(y1-y3)*dx3
f2xy = (dx1*dy3-dy1*dx3)
f3xy = ((x2-x1)*dy3-(y2-y1)*dx3)
f4xy = (dx2*dy-3dy2*dx3)
f5xy = (dx1*dy3-dy1*dx3)

  f1xy + a * f2xy + c * f3xy + c * b * f4xy + c * a * f5xy = 0
  f1xz + a * f2xz + c * f3xz + c * b * f4xz + c * a * f5xz = 0

  f1xy + a * f2xy + c * (f3xy + b * f4xy + a * f5xy) = 0
  f1xz + a * f2xz + c * (f3xz + b * f4xz + a * f5xz) = 0

  (f1xy + a * f2xy) * (f3xz + b * f4xz + a * f5xz) + c * (f3xy + b * f4xy + a * f5xy) * (f3xz + b * f4xz + a * f5xz) = 0
  (f1xz + a * f2xz) * (f3xy + b * f4xy + a * f5xy) + c * (f3xy + b * f4xy + a * f5xy) * (f3xz + b * f4xz + a * f5xz) = 0

    f1xy * (f3xz + b * f4xz + a * f5xz) + a * f2xy * (f3xz + b * f4xz + a * f5xz)
  = f1xz * (f3xy + b * f4xy + a * f5xy) + a * f2xz * (f3xy + b * f4xy + a * f5xy)

    f1xy*f3xz + b * f1xy*f4xz + a * f1xy*f5xz + a * f2xy * f3xz + a * f2xy * b * f4xz + a * f2xy * a * f5xz
  = f1xz*f3xy + b * f1xz*f4xy + a * f1xz*f5xy + a * f2xz * f3xy + a * f2xz * b * f4xy + a * f2xz * a * f5xy

    f1xy*f3xz + b * f1xy*f4xz + a * f1xy*f5xz + a * f2xy*f3xz + a * b * f2xy*f4xz + a^2 * f2xy*f5xz
  = f1xz*f3xy + b * f1xz*f4xy + a * f1xz*f5xy + a * f2xz*f3xy + a * b * f2xz*f4xy + a^2 * f2xz*f5xy
  
  a^2 * (f2xy*f5xz-f2xz*f5xy) + a * (f1xy*f5xz + f2xy*f3xz - f1xz*f5xy - f2xz*f3xy + b * (f2xy*f4xz - f2xz*f4xy)) + f1xy*f3xz - f1xz*f3xy + b * (f1xy*f4xz - f1xz*f4xy)
```

I do not get the idea that this approach is bringing us somewhere. Lets investigate the other approach:

```

x1 + a * dx1   =   e
x2 + b * dx2   =   e + d * rdx

x2 + b * dx2   =   x1 + a * dx1 + d * rdx
x2-x1 + b * dx2 - a * dx1 - d * rdx = 0

x2-x1 + b * dx2 - a * dx1 - d * rdx = 0
y2-y1 + b * dy2 - a * dy1 - d * rdy = 0

(x2-x1 + b * dx2 - a * dx1 - d * rdx)* dy2 = 0
(y2-y1 + b * dy2 - a * dy1 - d * rdy)* dx2 = 0

(x2-x1)*dy2 + b * dx2*dy2 - a * dx1*dy2 - d * rdx*dy2 = 0
(y2-y1)*dx2 + b * dx2*dy2 - a * dy1*dx2 - d * rdy*dx2 = 0
(x2-x1)*dy2-(y2-y1)*dx2 + a * (dy1*dx2-dx1*dy2) + d * (rdy*dx2-rdx*dy2) = 0

f1xy = (x2-x1)*dy2-(y2-y1)*dx2
f2xy = dy1*dx2-dx1*dy2
f3xy = rdy*dx2-rdx*dy2

f1xy + a * f2xy + d * f3xy = 0
f1xz + a * f2xz + d * f3xz = 0

f1xy * f3xz + a * f2xy * f3xz + d * f3xz * f3xy = 0
f1xz * f3xy + a * f2xz * f3xy + d * f3xz * f3xy = 0

f1xy * f3xz + a * f2xy * f3xz = f1xz * f3xy + a * f2xz * f3xy 
a * (f2xy * f3xz - f2xz * f3xy) = f1xz * f3xy - f1xy * f3xz

```
For 'a' needing to be a whole number, but restriction on the expressions.

```
f1xy = (x2-x1)*dy2-(y2-y1)*dx2
f2xy = dy1*dx2-dx1*dy2

f3xy = rdy*dx2-rdx*dy2

f1xz = (x2-x1)*dz2-(z2-z1)*dx2
f2xz = dz1*dx2-dx1*dz2

f3xz = rdz*dx2-rdx*dz2
```

```c
int main(int argc, char *argv[])
{
    read_file("input/day24.txt");
    read_hailstones();

    solve2();
}

void print128(char *name, __int128 i)
{
	char buffer[50];
	char *s = buffer + 49;
	*s = '\0';
	bool negative = FALSE;
	if (i < 0)
	{
		negative = TRUE;
		i = -i;
	}
	while (i != 0)
	{
		s--;
		*s = i % 10 + '0';
		i /= 10;
	}
	if (negative)
	{
		s--;
		*s = '-';
	}
	if (*s == '\0')
	{
		s--;
		*s = '0';
	}
	printf("%s = %s\n", name, s);
}	
	

void solve2()
{
	int range = 100;

	int i = 0;
	int j = 1;
		
    __int128 x1 = hailstones[i].x;
    __int128 y1 = hailstones[i].y;
    __int128 z1 = hailstones[i].z;
    __int128 dx1 = hailstones[i].dx;
    __int128 dy1 = hailstones[i].dy;
    __int128 dz1 = hailstones[i].dz;
    __int128 x2 = hailstones[j].x;
    __int128 y2 = hailstones[j].y;
    __int128 z2 = hailstones[j].z;
    __int128 dx2 = hailstones[j].dx;
    __int128 dy2 = hailstones[j].dy;
    __int128 dz2 = hailstones[j].dz;

	__int128 f1xy = (x2-x1)*dy2-(y2-y1)*dx2;
	__int128 f2xy = dy1*dx2-dx1*dy2;
	__int128 f1xz = (x2-x1)*dz2-(z2-z1)*dx2;
	__int128 f2xz = dz1*dx2-dx1*dz2;

	print128("f1xy", f1xy);	
	print128("f1xz", f1xz);	
	print128("f2xy", f2xy);
	print128("f2xz", f2xz);	
	
	for (int rdxi = -range; rdxi <= range; rdxi++)
	{
		__int128 rdx = rdxi;
		for (int rdyi = -range; rdyi <= range; rdyi++)
		{
			__int128 rdy = rdyi;
			for (int rdzi = -range; rdzi <= range; rdzi++)
			{
				__int128 rdz = rdzi;
				
				__int128 f3xy = rdy*dx2-rdx*dy2;
				__int128 f3xz = rdz*dx2-rdx*dz2;
				//print128(" f2xy", f3xy);
				//print128(" f3xz", f3xz);
				__int128 denom = f2xy * f3xz - f2xz * f3xy;
				__int128 num = f1xz * f3xy - f1xy * f3xz;
				//print128(" denom  ", denom);
				//print128("   num  ", num);
				if (denom != 0 && num % denom == 0)
				{
					__int128 div = num / denom;
					printf("found %d,%d,%d ", rdxi, rdyi, rdzi);
					print128(" f2xy", f3xy);
					print128(" f3xz", f3xz);
					print128(" denom  ", denom);
					print128("   num  ", num);
					print128("   div  ", div);
					
					// Now check intersection with all other hailstones
					
				}
			}
		}
	}
}
```

The above code does find some solutions. I continued working a bit on a function
checking intersections.

```
bool intersect(__int128 x1, __int128 y1, __int128 z1, __int128 dx1, __int128 dy1, __int128 dz1, int i)
{
	__int128 x2 = hailstones[i].x;
    __int128 y2 = hailstones[i].y;
    __int128 z2 = hailstones[i].z;
    __int128 dz2 = hailstones[i].dz;
    __int128 dy2 = hailstones[i].dy;
    __int128 dz2 = hailstones[i].dz;
    __int128 dot = dx1 * dy2 - dx2 * dy1;
    int _dotsign = dot < 0 ? -1 : dot > 0 ? 1 : 0;
    if (dot == 0)
    {
	    __int128 z2 = hailstones[i].z;
	    __int128 dz2 = hailstones[i].dz;
    	dot = dx1 * dz2 - dx2 * dz1;
    }
    else
    {
        __int128 s_num = dx2 * (y1 - y2) + dy2 * (x2 - x1);
        bool s_before = s_num < 0;
    
        __int128 t_num = dx1 * (y1 - y2) + dy1 * (x2 - x1);
        bool t_before = t_num < 0;

        __int128 x = dot * x1 + s_num * dx1;
        __int128 y = dot * y1 + s_num * dy1;
        bool y_inside = dot * min_xy <= y && y <= dot * max_xy;
        if (t_before && s_before)
            printf("Hailstones' paths crossed in the past for both hailstones.\n");
        else if (s_before)
            printf("Hailstones' paths crossed in the past for hailstone A.\n");    
        else if (t_before)
            printf("Hailstones' paths crossed in the past for hailstone B.\n");
        else if (x_inside && y_inside)
        {
            printf("Hailstones' paths will cross INSIDE the test area.\n");
            answer++;
        }
        else
            printf("Hailstones' paths will cross outside the test area.\n");    
    }
}
```

#### December 27

```c

bool multiple(__int128 a, __int128 b)
{
	if (a < 0) a = -a;
	if (b < 0) b = -b;
	return a % b == 0;
}
	
bool intersect(__int128 x1, __int128 y1, __int128 z1, __int128 dx1, __int128 dy1, __int128 dz1, int i, __int128 *min)
{
	__int128 x2 = hailstones[i].x;
    __int128 y2 = hailstones[i].y;
    __int128 z2 = hailstones[i].z;
    __int128 dx2 = hailstones[i].dx;
    __int128 dy2 = hailstones[i].dy;
    __int128 dz2 = hailstones[i].dz;
    __int128 s_num;
    __int128 t_num;
    __int128 dot = dx1 * dy2 - dx2 * dy1;
    const char *dot_name = "dot x-y";
    if (dot == 0)
    {
    	dot = dx1 * dz2 - dx2 * dz1;
    	dot_name = "dot x-z";
    	if (dot == 0)
    	{
    		printf("Dot x-z is 0\n");
    		return FALSE;
    	}
    	s_num = dx2 * (z1 - z2) + dz2 * (x2 - x1);
        t_num = dx1 * (z1 - z2) + dz1 * (x2 - x1);
    }
    else
    {
        s_num = dx2 * (y1 - y2) + dy2 * (x2 - x1);
        t_num = dx1 * (y1 - y2) + dy1 * (x2 - x1);
    }
    /*
    if (!multiple(s_num, dot))
    {
    	print128("s_num", s_num);
    	print128("dot", dot);
    	printf("s not multiple of %s\n", dot_name);
    	return FALSE;
    }
    s_num /= dot;
    if (!multiple(t_num, dot))
    {
    	printf("t not multiple of %s\n", dot_name);
    	return FALSE;
    }
    t_num /= dot;
	if (t_num < 0)
	{
		printf("t not positive\n");
		return FALSE;
	}
	*/
    
    if (dot * x1  + s_num * dx1 != dot * x2 + t_num * dx2)
    {
    	printf("x does not intersect\n");
    	return FALSE;
    }
    if (dot * y1  + s_num * dy1 != dot * y2 + t_num * dy2)
    {
    	printf("y does not intersect\n");
    	return FALSE;
    }
    if (dot * z1  + s_num * dz1 != dot * z2 + t_num * dz2)
    {
    	printf("z does not intersect\n");
    	return FALSE;
    }
    
    if (s_num < *min)
    	*min = s_num;
    	
    printf("Correct\n");
    return TRUE;
}

void solve2()
{
	int range = 5;

	int i = 0;
	int j = 1;
		
    __int128 x1 = hailstones[i].x;
    __int128 y1 = hailstones[i].y;
    __int128 z1 = hailstones[i].z;
    __int128 dx1 = hailstones[i].dx;
    __int128 dy1 = hailstones[i].dy;
    __int128 dz1 = hailstones[i].dz;
    __int128 x2 = hailstones[j].x;
    __int128 y2 = hailstones[j].y;
    __int128 z2 = hailstones[j].z;
    __int128 dx2 = hailstones[j].dx;
    __int128 dy2 = hailstones[j].dy;
    __int128 dz2 = hailstones[j].dz;

	__int128 f1xy = (x2-x1)*dy2-(y2-y1)*dx2;
	__int128 f2xy = dy1*dx2-dx1*dy2;
	__int128 f1xz = (x2-x1)*dz2-(z2-z1)*dx2;
	__int128 f2xz = dz1*dx2-dx1*dz2;

	print128("f1xy", f1xy);	
	print128("f1xz", f1xz);	
	print128("f2xy", f2xy);
	print128("f2xz", f2xz);	
	
	for (int rdxi = -range; rdxi <= range; rdxi++)
	{
		__int128 rdx = rdxi;
		for (int rdyi = -range; rdyi <= range; rdyi++)
		{
			int gcdxy = absgcd(rdxi, rdyi);
			__int128 rdy = rdyi;
			for (int rdzi = -range; rdzi <= range; rdzi++)
			{
				//if (absgcd(gcdxy, rdzi) != 1) continue;
				
				printf("check %d,%d,%d\n", rdxi, rdyi, rdzi);
				__int128 rdz = rdzi;
				
				__int128 f3xy = rdy*dx2-rdx*dy2;
				__int128 f3xz = rdz*dx2-rdx*dz2;
				print128(" f2xy", f3xy);
				print128(" f3xz", f3xz);
				__int128 denom = f2xy * f3xz - f2xz * f3xy;
				__int128 num = f1xz * f3xy - f1xy * f3xz;
				print128(" denom  ", denom);
				print128("   num  ", num);
				if (denom != 0 && num % denom == 0)
				{
					__int128 div = num / denom;
					printf("found %d,%d,%d ", rdxi, rdyi, rdzi);
					print128(" f2xy", f3xy);
					print128(" f3xz", f3xz);
					print128(" denom  ", denom);
					print128("   num  ", num);
					print128("   div  ", div);
					
					__int128 ax1 = x1 + div * rdx;
					__int128 ay1 = y1 + div * rdy;
					__int128 az1 = z1 + div * rdz;
					
					
					// Now check intersection with all other hailstones
					bool all_match = TRUE;
					__int128 min = 0;
					for (int i = 0; i < nr_lines && all_match; i++)
					{
						printf("%3d: ", i);
						all_match = intersect(ax1, ay1, az1, rdx, rdy, rdz, i, &min);
					}
					if (all_match)
						printf("FOUND\n");
					
				}
			}
		}
	}
}

int absgcd(int a, int b)
{
	if (a < 0) a = -a;
	if (b < 0) b = -b;
    while (a != 0)
    {
        num_t c = b % a;
        b = a;
        a = c;
    }
    return b;
}


```

#### December 28

```c
bool intersect(__int128 x1, __int128 y1, __int128 z1, __int128 dx1, __int128 dy1, __int128 dz1, int i, __int128 *min, __int128 *g)
{
	__int128 x2 = hailstones[i].x;
    __int128 y2 = hailstones[i].y;
    __int128 z2 = hailstones[i].z;
    __int128 dx2 = hailstones[i].dx;
    __int128 dy2 = hailstones[i].dy;
    __int128 dz2 = hailstones[i].dz;
    __int128 s_num;
    __int128 t_num;
    __int128 dot = dx1 * dy2 - dx2 * dy1;
    const char *dot_name = "dot x-y";
    if (dot == 0)
    {
    	dot = dx1 * dz2 - dx2 * dz1;
    	dot_name = "dot x-z";
    	if (dot == 0)
    	{
    		printf("Dot x-z is 0\n");
    		return FALSE;
    	}
    	s_num = dx2 * (z1 - z2) + dz2 * (x2 - x1);
        t_num = dx1 * (z1 - z2) + dz1 * (x2 - x1);
    }
    else
    {
        s_num = dx2 * (y1 - y2) + dy2 * (x2 - x1);
        t_num = dx1 * (y1 - y2) + dy1 * (x2 - x1);
    }
    if (!multiple(s_num, dot))
    {
    	print128("s_num", s_num);
    	print128("dot", dot);
    	printf("s not multiple of %s\n", dot_name);
    	return FALSE;
    }
    s_num /= dot;
    if (!multiple(t_num, dot))
    {
    	printf("t not multiple of %s\n", dot_name);
    	return FALSE;
    }
    t_num /= dot;
	if (t_num < 0)
	{
		printf("t not positive\n");
		return FALSE;
	}
    
    if (x1  + s_num * dx1 != x2 + t_num * dx2)
    {
    	printf("x does not intersect\n");
    	return FALSE;
    }
    if (y1  + s_num * dy1 != y2 + t_num * dy2)
    {
    	printf("y does not intersect\n");
    	return FALSE;
    }
    if (z1  + s_num * dz1 != z2 + t_num * dz2)
    {
    	printf("z does not intersect\n");
    	return FALSE;
    }
    
    if (s_num < *min)
    	*min = s_num;

	if (*g == 0)
		*g = s_num;
	else
	{
		__int128 a = s_num;
		__int128 b = *g;
		if (a < 0) a = -a;
	    while (a != 0)
	    {
	        __int128 c = b % a;
	        b = a;
	        a = c;
	    }
		*g = b;
	} 
    	
    //printf("Correct\n");
	print128("Correct at ", s_num);
	print128("           ", *g);
	//print128("   x ", x1 + s_num * dx1);
	//print128("   y ", y1 + s_num * dy1);
	//print128("   z ", z1 + s_num * dz1);
    return TRUE;
}

void solve2()
{
	int range = 400;

	int i = 0;
	int j = 1;
		
    __int128 x1 = hailstones[i].x;
    __int128 y1 = hailstones[i].y;
    __int128 z1 = hailstones[i].z;
    __int128 dx1 = hailstones[i].dx;
    __int128 dy1 = hailstones[i].dy;
    __int128 dz1 = hailstones[i].dz;
    __int128 x2 = hailstones[j].x;
    __int128 y2 = hailstones[j].y;
    __int128 z2 = hailstones[j].z;
    __int128 dx2 = hailstones[j].dx;
    __int128 dy2 = hailstones[j].dy;
    __int128 dz2 = hailstones[j].dz;

	__int128 f1xy = (x2-x1)*dy2-(y2-y1)*dx2;
	__int128 f2xy = dy1*dx2-dx1*dy2;
	__int128 f1xz = (x2-x1)*dz2-(z2-z1)*dx2;
	__int128 f2xz = dz1*dx2-dx1*dz2;
	__int128 f1yz = (y2-y1)*dz2-(z2-z1)*dy2;
	__int128 f2yz = dz1*dy2-dy1*dz2;

	print128("f1xy", f1xy);
	print128("f1xz", f1xz);
	print128("f1yz", f1yz);
	print128("f2xy", f2xy);
	print128("f2xz", f2xz);	
	print128("f2yz", f2yz);	
	
	for (int rdxi = -range; rdxi <= range; rdxi++)
	{
		__int128 rdx = rdxi;
		for (int rdyi = -range; rdyi <= range; rdyi++)
		{
			int gcdxy = absgcd(rdxi, rdyi);
			__int128 rdy = rdyi;
			for (int rdzi = 1; rdzi <= range; rdzi++)
			{
				if (absgcd(gcdxy, rdzi) != 1) continue;
				
				//printf("check %d,%d,%d\n", rdxi, rdyi, rdzi);
				__int128 rdz = rdzi;
				
				__int128 f3xy = rdy*dx2-rdx*dy2;
				__int128 f3xz = rdz*dx2-rdx*dz2;
				__int128 f3yz = rdz*dy2-rdy*dz2;
				//print128(" f3xy", f3xy);
				//print128(" f3xz", f3xz);
				//print128(" f3yz", f3yz);
				__int128 denom = f2xy * f3xz - f2xz * f3xy;
				__int128 num = f1xz * f3xy - f1xy * f3xz;
				//print128(" denom  ", denom);
				//print128("   num  ", num);
				if (denom != 0 && num % denom == 0)
				{
					__int128 div = num / denom;
					printf("found %d,%d,%d ", rdxi, rdyi, rdzi);
					//print128(" f3xy", f3xy);
					//print128(" f3xz", f3xz);
					//print128(" denom  ", denom);
					//print128("   num  ", num);
					print128("   div  ", div);
					denom = f2xy * f3yz - f2yz * f3xy;
					num = f1yz * f3xy - f1xy * f3yz;
					div = num / denom;
					print128(" denom  ", denom);
					print128("   num  ", num);
					print128("   div  ", div);
					denom = f2xz * f3yz - f2yz * f3xz;
					num = f1yz * f3xz - f1xz * f3yz;
					div = num / denom;
					print128(" denom  ", denom);
					print128("   num  ", num);
					print128("   div  ", div);
					
					__int128 ax1 = x1 + div * dx1;
					__int128 ay1 = y1 + div * dy1;
					__int128 az1 = z1 + div * dz1;
					print128("    ax1 ", ax1);
					print128("    ay1 ", ay1);
					print128("    az1 ", az1);
					
					
					// Now check intersection with all other hailstones
					bool all_match = TRUE;
					__int128 min = 0;
					__int128 g = 0;
					for (int i = 0; i < nr_lines && all_match; i++)
					{
						printf("%3d: ", i);
						all_match = intersect(ax1, ay1, az1, rdx, rdy, rdz, i, &min, &g);
					}
					if (all_match)
					{
						printf("FOUND\n");
						print128(" gcd ", g);
						print128(" min ", min);
						min -= g;
						print128("   x ", ax1 + min * rdx);
						print128("   y ", ay1 + min * rdy);
						print128("   z ", az1 + min * rdz);
						print128("ANSWER ", ax1 + min * rdx + ay1 + min * rdy + az1 + min * rdz);
					}					
				}
			}
		}
	}
}

```
The above code does find a path, but for some reason the answer is not correct.

I watched a bit of [Advent of Code 2023 Day 24: Never Tell Me The Odds](https://www.youtube.com/watch?v=guOyA7Ijqgk)
and I understood what I have done wrong.

```c
bool intersect(__int128 x1, __int128 y1, __int128 z1, __int128 dx1, __int128 dy1, __int128 dz1, int i, __int128 *min, __int128 *g)
{
	__int128 x2 = hailstones[i].x;
    __int128 y2 = hailstones[i].y;
    __int128 z2 = hailstones[i].z;
    __int128 dx2 = hailstones[i].dx;
    __int128 dy2 = hailstones[i].dy;
    __int128 dz2 = hailstones[i].dz;
    __int128 s_num;
    __int128 t_num;
    __int128 dot = dx1 * dy2 - dx2 * dy1;
    const char *dot_name = "dot x-y";
    if (dot == 0)
    {
    	dot = dx1 * dz2 - dx2 * dz1;
    	dot_name = "dot x-z";
    	if (dot == 0)
    	{
    		printf("Dot x-z is 0\n");
    		return FALSE;
    	}
    	s_num = dx2 * (z1 - z2) + dz2 * (x2 - x1);
        t_num = dx1 * (z1 - z2) + dz1 * (x2 - x1);
    }
    else
    {
        s_num = dx2 * (y1 - y2) + dy2 * (x2 - x1);
        t_num = dx1 * (y1 - y2) + dy1 * (x2 - x1);
    }
    if (!multiple(s_num, dot))
    {
    	print128("s_num", s_num);
    	print128("dot", dot);
    	printf("s not multiple of %s\n", dot_name);
    	return FALSE;
    }
    s_num /= dot;
    if (!multiple(t_num, dot))
    {
    	printf("t not multiple of %s\n", dot_name);
    	return FALSE;
    }
    t_num /= dot;
	if (t_num < 0)
	{
		printf("t not positive\n");
		return FALSE;
	}
    
    if (x1  + s_num * dx1 != x2 + t_num * dx2)
    {
    	printf("x does not intersect\n");
    	return FALSE;
    }
    if (y1  + s_num * dy1 != y2 + t_num * dy2)
    {
    	printf("y does not intersect\n");
    	return FALSE;
    }
    if (z1  + s_num * dz1 != z2 + t_num * dz2)
    {
    	printf("z does not intersect\n");
    	return FALSE;
    }
    
    if (s_num < *min)
    	*min = s_num;

	if (*g == 0)
		*g = s_num;
	else
	{
		__int128 a = s_num;
		__int128 b = *g;
		if (a < 0) a = -a;
	    while (a != 0)
	    {
	        __int128 c = b % a;
	        b = a;
	        a = c;
	    }
		*g = b;
	} 
    	
    //printf("Correct\n");
	print128("Correct at ", s_num);
	print128("           ", *g);
	print128("   x ", x1 + (t_num - s_num) * dx1);
	print128("   y ", y1 + (t_num - s_num) * dy1);
	print128("   z ", z1 + (t_num - s_num) * dz1);
	print128("ANSWER= ", x1 + (t_num - s_num) * dx1 + y1 + (t_num - s_num) * dy1 + z1 + (t_num - s_num) * dz1);
    return TRUE;
}
```

Still not the correct answer.

#### December 29

```c
bool intersect(__int128 x1, __int128 y1, __int128 z1, __int128 dx1, __int128 dy1, __int128 dz1, int i)
{
	__int128 x2 = hailstones[i].x;
    __int128 y2 = hailstones[i].y;
    __int128 z2 = hailstones[i].z;
    __int128 dx2 = hailstones[i].dx;
    __int128 dy2 = hailstones[i].dy;
    __int128 dz2 = hailstones[i].dz;
    __int128 s_num;
    __int128 t_num;
    __int128 dot = dx1 * dy2 - dx2 * dy1;
    const char *dot_name = "dot x-y";
    if (dot == 0)
    {
    	dot = dx1 * dz2 - dx2 * dz1;
    	dot_name = "dot x-z";
    	if (dot == 0)
    	{
    		printf("Dot x-z is 0\n");
    		return FALSE;
    	}
    	s_num = dx2 * (z1 - z2) + dz2 * (x2 - x1);
        t_num = dx1 * (z1 - z2) + dz1 * (x2 - x1);
    }
    else
    {
        s_num = dx2 * (y1 - y2) + dy2 * (x2 - x1);
        t_num = dx1 * (y1 - y2) + dy1 * (x2 - x1);
    }
    if (!multiple(s_num, dot))
    {
    	print128("s_num", s_num);
    	print128("dot", dot);
    	printf("s not multiple of %s\n", dot_name);
    	return FALSE;
    }
    s_num /= dot;
    if (!multiple(t_num, dot))
    {
    	printf("t not multiple of %s\n", dot_name);
    	return FALSE;
    }
    t_num /= dot;
	if (t_num < 0)
	{
		printf("t not positive\n");
		return FALSE;
	}
    
    if (x1  + s_num * dx1 != x2 + t_num * dx2)
    {
    	printf("x does not intersect\n");
    	return FALSE;
    }
    if (y1  + s_num * dy1 != y2 + t_num * dy2)
    {
    	printf("y does not intersect\n");
    	return FALSE;
    }
    if (z1  + s_num * dz1 != z2 + t_num * dz2)
    {
    	printf("z does not intersect\n");
    	return FALSE;
    }
    
    return s_num == t_num;
}

void solve2()
{
	int range = 400;

	int i = 0;
	int j = 1;
		
    __int128 x1 = hailstones[i].x;
    __int128 y1 = hailstones[i].y;
    __int128 z1 = hailstones[i].z;
    __int128 dx1 = hailstones[i].dx;
    __int128 dy1 = hailstones[i].dy;
    __int128 dz1 = hailstones[i].dz;
    __int128 x2 = hailstones[j].x;
    __int128 y2 = hailstones[j].y;
    __int128 z2 = hailstones[j].z;
    __int128 dx2 = hailstones[j].dx;
    __int128 dy2 = hailstones[j].dy;
    __int128 dz2 = hailstones[j].dz;

	__int128 f1xy = (x2-x1)*dy2-(y2-y1)*dx2;
	__int128 f2xy = dy1*dx2-dx1*dy2;
	__int128 f1xz = (x2-x1)*dz2-(z2-z1)*dx2;
	__int128 f2xz = dz1*dx2-dx1*dz2;
	__int128 f1yz = (y2-y1)*dz2-(z2-z1)*dy2;
	__int128 f2yz = dz1*dy2-dy1*dz2;

	print128("f1xy", f1xy);
	print128("f1xz", f1xz);
	print128("f1yz", f1yz);
	print128("f2xy", f2xy);
	print128("f2xz", f2xz);	
	print128("f2yz", f2yz);	
	
	for (int rdxi = -range; rdxi <= range; rdxi++)
	{
		__int128 rdx = rdxi;
		for (int rdyi = -range; rdyi <= range; rdyi++)
		{
			int gcdxy = absgcd(rdxi, rdyi);
			__int128 rdy = rdyi;
			for (int rdzi = 1; rdzi <= range; rdzi++)
			{
				if (absgcd(gcdxy, rdzi) != 1) continue;
				
				//printf("check %d,%d,%d\n", rdxi, rdyi, rdzi);
				__int128 rdz = rdzi;
				
				__int128 f3xy = rdy*dx2-rdx*dy2;
				__int128 f3xz = rdz*dx2-rdx*dz2;
				__int128 f3yz = rdz*dy2-rdy*dz2;
				//print128(" f3xy", f3xy);
				//print128(" f3xz", f3xz);
				//print128(" f3yz", f3yz);
				__int128 denom = f2xy * f3xz - f2xz * f3xy;
				__int128 num = f1xz * f3xy - f1xy * f3xz;
				//print128(" denom  ", denom);
				//print128("   num  ", num);
				if (denom != 0 && num % denom == 0)
				{
					__int128 div = num / denom;
					printf("found %d,%d,%d ", rdxi, rdyi, rdzi);
					//print128(" f3xy", f3xy);
					//print128(" f3xz", f3xz);
					//print128(" denom  ", denom);
					//print128("   num  ", num);
					print128("   div  ", div);
					denom = f2xy * f3yz - f2yz * f3xy;
					num = f1yz * f3xy - f1xy * f3yz;
					div = num / denom;
					print128(" denom  ", denom);
					print128("   num  ", num);
					print128("   div  ", div);
					denom = f2xz * f3yz - f2yz * f3xz;
					num = f1yz * f3xz - f1xz * f3yz;
					div = num / denom;
					print128(" denom  ", denom);
					print128("   num  ", num);
					print128("   div  ", div);
					
					__int128 ax1 = x1 + div * (dx1 - rdx);
					__int128 ay1 = y1 + div * (dy1 - rdy);
					__int128 az1 = z1 + div * (dz1 - rdz);
					print128("    ax1 ", ax1);
					print128("    ay1 ", ay1);
					print128("    az1 ", az1);
					
					
					// Now check intersection with all other hailstones
					bool all_match = TRUE;
					for (int i = 0; i < nr_lines && all_match; i++)
					{
						printf("%3d: ", i);
						all_match = intersect(ax1, ay1, az1, rdx, rdy, rdz, i);
					}
					if (all_match)
					{
						printf("FOUND\n");
						print128("   x ", ax1);
						print128("   y ", ay1);
						print128("   z ", az1);
						print128("ANSWER ", ax1 + ay1 + az1);
					}					
				}
			}
		}
	}
}

```

Okay, this does find the correct answer. I also realized that there might be a much simpler
and direct method to find the solution. I had not realized that the rock and the hailstone
should hit at the same time.


### Executing this page

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Std.md Day24.md >day24.c; gcc -g -Wall day24.c -o day24; ./day24
```

