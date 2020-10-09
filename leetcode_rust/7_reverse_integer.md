## 7 - Reverse Integer

### Problem statement
Given a 32-bit signed integer, reverse digits of an integer.

**Example 1**:

```
Input: 123
Output: 321
```

**Example 2**:

```
Input: -123
Output: -321
```

**Example 3**:

```
Input: 120
Output: 21
```

**Note:**
Assume we are dealing with an environment which could only store
integers within the 32-bit signed integer range: [−231,  231 − 1].
For the purpose of this problem, assume that your function returns
0 when the reversed integer overflows.

Taken from:
[leetcode](https://leetcode.com/problems/reverse-integer/)

### Approach and Analysis

#### Flipping via modulo and division
I found the best way to approach this problem was in terms of
how to get the first digit. We can do that by taking mod 10
of the number:

`123 % 10 = 3`

Subsequent digits can be obtained by dividing by 10 and repeating
this process:

```
123 % 10 = 3 <-
123 / 10 = 12
12 % 10 = 2 <-
12 / 10 = 1
1 % 10 = 1 <-
```

Now, we can wrap that all in a loop, here's the pseudocode:

```
ans = 0;
while(num > 0):
  ans = num % 10 + (ans * 10)
  num = num / 10
```

**Note**: We multiply our answer by 10 each time to effectively
"shift" it up to the next 10's position.

This pseudocode is on the way to a good solution but there are
a couple pieces missing still.

For one, this won't work if num is less than 0 (because the while
loop won't go through any iterations). This can be easily be solved
by changing to condition in the loop to `while(num != 0)`

Secondly, the problem states that we can only store integers in the
32-big signed integer range and that we should return 0 when the
reverse integer overflows. This is a huge clue that we will have to
handle this scenario.

### Aside - Integer Overflow
Integer overflow in this problem can occur when we try and reverse
a number that is too big for the data type it's being stored in.

For example, imagine for the sake of simplicity that this problem
involved 4 bits instead of 32 (for simplicity) and that the number
we wanted to reverse was 14. This could be represented in binary as
`0b1110`. We would expect that reversing 14 would return 41 but this
is in fact impossible because we cannot represent the number 41 with
only 4 bits (this highest is 15!).

If we don't check our code for overflow, we are likely to get
unexpected results when provided with extremely large values.

### Rust solution (clumsy)

```Rust
impl Solution {
  pub fn reverse(x: i32) -> i32 {
    let isPositive = x > 0;
    let mut accumulator: i32 = 0;
    let mut mutx = (x.abs());
    while (mutx >= 1) {
      let digit: i32 = mutx % 10;
      // use checked_mul to avoid overflow, evaluates the
      // `None` arm if overflow is found
      match accumulator.checked_mul(10) {
        None => return 0,
        Some(fine) => {
          accumulator = accumulator * 10;
        }
      }

      match accumulator.checked_add(digit) {
        None => return 0,
        Some(fine) => {
          accumulator = accumulator + digit;
        }
      }
      mutx = mutx / 10;
    }
    if (isPositive) {
      return accumulator;
    } else {
      return -accumulator;
    }
  }
}
```

This solution passes all the test cases that leetcode provides but
it has some quirks that we can easily avoid.

Firstly, we can remove additional logic to deal with the negative
case, namely lines 3 and 23 through 27 simply by making the fix to
our `while` loop as suggested in the previous section.

Secondly, the match statements can be chained together such that the
entire `checked_add` function is contained in the "success" arm of
the `checked_mul` function:

```Rust
impl Solution {
  pub fn reverse(x: i32) -> i32 {
    let mut accumulator: i32 = 0;
    let mut mutx = x;

    while (mutx != 0) {
      let digit: i32 = mutx % 10;
      match accumulator.checked_mul(10) {
        None => return 0,
        // By this point, tmp will contain the result of
        // evaluating `checked_mul(10)` on accumulator
        Some(tmp) => {
          match tmp.checked_add(digit) {
            None => return 0,
            Some(fine) => {
              accumulator = fine;
            }
          }
        }
      }
      mutx = mutx / 10;
    }
    return accumulator;
  }
}
```

Note that in either solution, the first thing we have to do is
create mutable variables both of the input and output. By default,
the rust compiler won't mutate x and that's a very good thing.

### What I learned by solving this problem

#### Thinking about the negative case

My first approach to tackling the negative case was not very elegant;
instead of looking at how to modify the solution to fit the case, I
thought about how to modify the case to fit the solution (taking the
absolute value and then "re-reversing" it). This pattern should serve
as a warning to me in the future that there is probably a better way
to accomplish this.

#### Checked functions

Clearly although Rust is a lot more stringent than other languages,
problems like these can still sideline you if you're not careful.
Rust's default add and multiply operations don't handle overflow
(and justifiably so - one might want to handle it their own way)
and that simply means we need to reach deeper into our toolbox for
the right tool to deal with that job. Checked functions are a useful
tool in that regard, and they seem to work great!

#### Getting used to "match"

Match is a little foreign to a Rust newbie like myself but his problem
provided some decent experience working with it. An important lesson
I learned was that the `Some` arm stores the *evaluation* of the
statement in question so there's no need for intermediate "mumbo jumbo".

### References
1. [Submission that led me to my answer](https://leetcode.com/problems/reverse-integer/discuss/293960/Rust%3A-use-checked-ops-to-check-overflow-0ms-2.3mb)
2. [Checked add for i32](https://doc.rust-lang.org/std/primitive.i32.html#method.checked_add)
