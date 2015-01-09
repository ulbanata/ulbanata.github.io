---
layout: post
title: nth Fibonacci Number in log(n) Time
date: 2015-01-07 20:40:45
disqus: y
---

0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 987, 1597, 2584

---

While I was a Fellow at [MakerSquare](http://www.makersquare.com) teaching the algorithms curriculum, I taught a lesson on the Fibonacci Sequence. The goal of the lesson was to show that while recursion is a very helpful tool and can elegantly save many problems, using it without thinking about the consequences can be dangerous. The exercise was for students to code both a recursive version and an iterative version to find the nth Fibonacci number. The students then had to determine which was faster and explain why. The iterative version runs in linear time, while the recursive version runs in exponential time.

One enterprising student took to Google to check solutions and found a solution that ran even faster than the iterative solution.

```ruby
def fib(n)
  a = 1
  b = 0
  p = 0
  q = 1

  while n > 0
    if n % 2 == 0
      p, q = p**2 + q**2, 2*p*q + q**2
      n /= 2
    else
      a, b = b*q + a*q + a*p, b*p + a*q
      n -= 1
    end
  end

  b
end
```

When we first saw this, nothing really made sense. It looked like *a* and *b* were the two numbers keeping track of the Fibonacci sequence, but what were *p* and *q*? Running it would produce the correct answer, but we couldn't explain why. Well, I recently found the solution [here](http://www.kendyck.com/archives/2005/05/13/solution-to-sicp-exercise-119), which is answering a question found [here](http://mitpress.mit.edu/sicp/full-text/book/book-Z-H-11.html#%_thm_1.19). The solution (and original question) is a bit dense, so I wanted to break it down.

---

For reference, here is an iterative version:

```ruby
def it_fib(n)
  a = 1
  b = 0
  (1..n).each do |_|
    holder = a
    a += b
    b = holder
  end
  b
end
```

---

Let's start with the beginning of the question:

> Recall the transformation of the state variables a and b in the fib-iter process
> of section 1.2.2: a &#8592; a + b and b &#8592; a. Call this transformation T, and observe that
> applying T over and over again n times, starting with 1 and 0, produces the pair Fib(n + 1) and Fib(n).

The "fib-iter process of section 1.2.2" can be seen above in the ruby iterative version. The variables *a* and *b* are set to the 1st and 0th Fibonacci numbers, respectively. To find the next number in the sequence, a is set to the sum of a and b, and b is set to a's old value. The question states that these two statements, a &#8592; a + b and b &#8592; a, are a transformation called T. The terminology might seem foreign, but a transformation in this context is changing the state of a and b to get the next two numbers in the Fibonacci sequence. Below is a table of each of these tranformations:

| n | a | b |
| :---: | :---: | :---: |
| 0 | 1 | 0 |
| 1 | 1 | 1 |
| 2 | 2 | 1 |
| 3 | 3 | 2 |
| 4 | 5 | 3 |
| 5 | 8 | 5 |
| 6 | 13 | 8 |
| 7 | 21 | 13 |
| 8 | 34 | 21 |
| 9 | 55 | 34 |
| 10 | 89 | 55 |

Going from row to row is a transformation, computing the new state. Each row is a state.

---

Now we need to find a pattern in the transformation. What pattern repeats each time?

Instead of starting with 1 and 0 like our last table, let's start with variables a and b, where a = fib(n+1) and b = fib(n). This means that a and b can be any two values in the Fibonacci sequence as long as they are adjacent in the sequence and b is smaller than a. So a could be 55 and b is 34. Or a could be 3 and b would be 2. Any numbers here will suffice, we are trying to find a general pattern. Let's build that table again with the general equation for the Fibonacci sequence:

| n | a | b |
| :-: | :-: | :-: |
| n | a | b |
| n+1 | a + b | a |
| n+2 | 2a + b | a + b |
| n+3 | 3a + 2b | 2a + b |
| n+4 | 5a + 3b | 3a + 2b |
| n+5 | 8a + 5b | 5a + 3b |
| n+6 | 13a + 8b | 8a + 5b |
| n+7 | 21a + 13b | 13a + 8b |
| n+8 | 34a + 21b | 21a + 13b |
| n+9 | 55a + 34b | 34a + 21b |
| n+10 | 89a + 55b | 55a + 34b |

Any two numbers in the Fibonacci Sequence that we plug into a and b in the table above will output the next 10 Fibonacci numbers. To check the table, we can plug in a = 1 and b = 0, the same starting point as the earlier table.

Instead of having to find the pattern ourselves, the question gives us the pattern we are supposed to use:

> Now consider T to be the special case of p = 0 and q = 1 in a family of
> transformations Tpq, where Tpq transforms the pair (a,b) according to a &#8592;
> bq + aq + ap and b &#8592; bp + aq.

Here, we are finally introduced to the *p* and *q* variables. They are the pattern in the transformation we've been trying to find. Instead of having a &#8592; a + b and b &#8592; a to find the next two numbers in the sequence, we will be using a &#8592; bq + aq + ap and b &#8592; bp + aq to find any number in the sequence if we know the correct p and q. What are the correct p's and q's? Let's build our table again to find out:

| n | a | b | p | q |
| :-: | :-: | :-: | :-: | :-: |
| n | a | b | n/a | n/a |
| n+1 | a + b | a | 0 | 1 |
| n+2 | 2a + b | a + b | 1 | 1|
| n+3 | 3a + 2b | 2a + b | 1 | 2 |
| n+4 | 5a + 3b | 3a + 2b | 2 | 3|
| n+5 | 8a + 5b | 5a + 3b | 3 | 5 |
| n+6 | 13a + 8b | 8a + 5b | 5 | 8 |
| n+7 | 21a + 13b | 13a + 8b | 8 | 13 |
| n+8 | 34a + 21b | 21a + 13b | 13 | 21 |
| n+9 | 55a + 34b | 34a + 21b | 21 | 34 |
| n+10 | 89a + 55b | 55a + 34b | 34 | 55 |

The correct *p* and *q* for a given Fibonacci number end up forming their own Fibonacci sequence.

---

Now that we have the transformation down, let's look at the last part of the question:

> Show that if we apply such a transformation Tpq twice, the effect is the same as using a single transformation Tp'q' of the same form, and compute p' and q' in terms of p and q. This gives us an explicit way to square these transformations, and thus we can compute Tn using successive squaring, as in the fast-expt procedure.

What happens if we apply the transformation twice? The first transformation looks like this:

a &#8592; bq + aq + ap

b &#8592; bp + aq

Applying the transformation gives us:

a &#8592; (bp + aq)q + (bq + aq + ap)q + (bq + aq + ap)p

b &#8592; (bp + aq)p + (bq + aq + ap)q

To get it into a consistent formula, we'll need to use some algebra. We can expand it to:

a &#8592; bpq + aq<sup>2</sup> + bq<sup>2</sup> + aq<sup>2</sup> + apq + bqp + aqp + ap<sup>2</sup>

b &#8592; bp<sup>2</sup> + aqp + bq<sup>2</sup> + aq<sup>2</sup> + apq

And then manipulate it to the form:

a &#8592; b(q<sup>2</sup> + 2pq) + a(q<sup>2</sup> + 2pq) + a(p<sup>2</sup> + q<sup>2</sup>)

b &#8592; b(p<sup>2</sup> + q<sup>2</sup>) + a(q<sup>2</sup> + 2pq)

The two lines above should look familiar. They are very similar to a &#8592; bq + aq + ap and b &#8592; bp + aq!

So what do we have at this point? We took the basic algorithm for finding the next numbers in the Fibonacci sequence, a &#8592; a + b and b &#8592; a. Then we looked for a pattern in how we compute the sequence. That resulted in the transformation a &#8592; bq + aq + ap and b &#8592; bp + aq, where p and q were numbers that can be used to find a value in the Fibonacci sequence. Finally, we applied our transformation twice and, through a little algebra, resulted in a transformation that looks very similar to our initial transformation.

If our two transformations are that similar, perhaps we can compute new values of p and q to find a Fibonacci number faster. Let's plug in p = 0 and q = 1 into our new transformation:

a &#8592; b(q<sup>2</sup> + 2pq) + a(q<sup>2</sup> + 2pq) + a(p<sup>2</sup> + q<sup>2</sup>)

b &#8592; b(p<sup>2</sup> + q<sup>2</sup>) + a(q<sup>2</sup> + 2pq)

Which results in:

a &#8592; b(1 + 2*0*1) + a(1 + 2*0*1) + a(0 + 1)

b &#8592; b(0 + 1) + a(1 + 2*0*1)

And finally:

a &#8592; b + 2a

b &#8592; b + a

Looking at our table above, when p = 0 and q = 1, our new transformation results in a and b values that are the same as when p = 1 and q = 1. We can continue plugging in the values of p and q that we receive into our new transformation. This results in the following table:

| p | q | a &#8592; | b &#8592; | new p | new q |
| :-: | :-: | :-: | :-: | :-: | :-: |
| 0 | 1 | b + 2a | b + a | 1 | 1|
| 1 | 1 | 3b + 5a | 2b + 3a | 2 | 3 |
| 2 | 3 | 21b + 34a | 13b + 21a | 13 | 21 |
| 13 | 21 | 987b + 1597a | 610b + 987a | 1346269 | 2178309 |

The p's and q's are growing fast using the new transformation! The results are all part of the Fibonacci sequence, but how fast are the p's and q's growing?

| n | Fib | q?|
| :-: | :-: | :-: |
| 0 | 0 | No |
| 1 | 1 | Yes |
| 2 | 1 | Yes |
| 3 | 2 | No |
| 4 | 3 | Yes |
| 5 | 5 | No |
| 6 | 8 | No |
| 7 | 13 | No |
| 8 | 21 | Yes |
| 9 | 34 | No |
| 10 | 55 | No |
| 11 | 89 | No |
| 12 | 144 | No |
| 13 | 233 | No |
| 14 | 377 | No |
| 15 | 610 | No |
| 16 | 987 | Yes |

They're growing by 2<sup>n</sup>! The new transformation will be used to quickly climb through the p and q values we need until we reach the correct ones to compute the nth Fibonacci number. To get to the correct p and q value, we can use our new transformation to compute the values of p and q, resulting in this:

p &#8592; (p<sup>2</sup> + q<sup>2</sup>)

q &#8592; (q<sup>2</sup> + 2pq)

You'll notice that this is the same thing that the Ruby program is doing:

```ruby
def fib(n)
  a = 1
  b = 0
  p = 0
  q = 1

  while n > 0
    if n % 2 == 0
      p, q = p**2 + q**2, 2*p*q + q**2
      n /= 2
    else
      a, b = b*q + a*q + a*p, b*p + a*q
      n -= 1
    end
  end

  b
end
```

---

The final question is why we care if it is odd or even. This is to compute nth Fibonacci numbers where n isn't a power of 2. If you've ever converted a number to binary, you've seen the same type of algorithm.

Let's step through the program for n = 7.

State:<br>
n = 7<br>
a = 1<br>
b = 0<br>
p = 0<br>
q = 1

We enter the while loop.

n % 2 == 1, so we run the else block.

```ruby
else
  a, b = b*q + a*q + a*p, b*p + a*q
  n -= 1
end
```

New State:<br>
n = 6<br>
a = 1<br>
b = 1<br>
p = 0<br>
q = 1

n % 2 == 0, so we run the if block.

```ruby
if n % 2 == 0
  p, q = p**2 + q**2, 2*p*q + q**2
  n /= 2
```

New State:<br>
n = 3<br>
a = 1<br>
b = 1<br>
p = 1<br>
q = 1

n % 2 == 1, so we run the else block.

```ruby
else
  a, b = b*q + a*q + a*p, b*p + a*q
  n -= 1
end
```

New State:<br>
n = 2<br>
a = 3<br>
b = 2<br>
p = 1<br>
q = 1

n % 2 == 0, so we run the if block.

```ruby
if n % 2 == 0
  p, q = p**2 + q**2, 2*p*q + q**2
  n /= 2
```

New State:<br>
n = 1<br>
a = 3<br>
b = 2<br>
p = 2<br>
q = 3

n % 2 == 1, so we run the else block.

```ruby

else
  a, b = b*q + a*q + a*p, b*p + a*q
  n -= 1
end
```

New State:<br>
n = 0<br>
a = 21<br>
b = 13<br>
p = 2<br>
q = 3

We exit the loop because n == 0 and return the value 13, which is the 7th Fibonacci number. This took a total of 5 runs through the loop, versus the 7 that would be needed for the iterative version.

---

Stepping through for n = 16:

State:<br>
n = 16<br>
a = 1<br>
b = 0<br>
p = 0<br>
q = 1

n % 2 == 0, so we run the if block.

```ruby
if n % 2 == 0
  p, q = p**2 + q**2, 2*p*q + q**2
  n /= 2
```

New State:<br>
n = 8<br>
a = 1<br>
b = 0<br>
p = 1<br>
q = 1

n % 2 == 0, so we run the if block.

```ruby
if n % 2 == 0
  p, q = p**2 + q**2, 2*p*q + q**2
  n /= 2
```

New State:<br>
n = 4<br>
a = 1<br>
b = 0<br>
p = 2<br>
q = 3

n % 2 == 0, so we run the if block.

```ruby
if n % 2 == 0
  p, q = p**2 + q**2, 2*p*q + q**2
  n /= 2
```

New State:<br>
n = 2<br>
a = 1<br>
b = 0<br>
p = 13<br>
q = 21

n % 2 == 0, so we run the if block.

```ruby
if n % 2 == 0
  p, q = p**2 + q**2, 2*p*q + q**2
  n /= 2
```

New State:<br>
n = 1<br>
a = 1<br>
b = 0<br>
p = 610<br>
q = 987

n % 2 == 1, so we run the else block.

```ruby
else
  a, b = b*q + a*q + a*p, b*p + a*q
  n -= 1
end
```

New State:<br>
n = 0<br>
a = 1597<br>
b = 987<br>
p = 610<br>
q = 987

We exit the loop because n == 0 and return the value 987, which is the 16th Fibonacci number. This took a total of 5 runs through the loop, versus the 16 that would be needed for the iterative version.

---

The O(n) of the algorithm has a worst case of log(2n), which occurs when you try to find the nth Fibonacci number where n is 1 less than a power of 2 (7, 15, 31, etc.). When one of these numbers is ran, the algorithm has to alternate between the if/else statements because n jumps between even and odd each iteration. The best case is using an n that is a power of 2 (8, 16, 32, etc.). In this case, the algorithm uses the If block until n reaches 1, where it solves for the Fibonacci number. This results in the O(n) being log(n) + 2.

---

And there you have it, a log(n) algorithm for finding the nth Fibonacci number.
