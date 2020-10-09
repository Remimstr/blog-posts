<F20><F20>## 13 - Roman to Integer

### Problem statement
Roman numerals are represented by seven different symbols: `I`, `V`, `X`, `L`, `C`, `D` and `M`.

```
Symbol  Value
I       1
V       5
X       10
L       50
C       100
D       500
M       1000
```

For example, two is written as `II` in Roman numeral, just two one's added together.
Twelve is written as, `XII`, which is simply `X` + `II`. The number twenty seven is
written as `XXVII`, which is `XX` + `V` + `II`.

Roman numerals are usually written largest to smallest from left to right. However,
the numeral for four is not `IIII`. Instead, the number four is written as `IV`.
Because the one is before the five we subtract it making four. The same principle
applies to the number nine, which is written as `IX`. There are six instances where
subtraction is used:

- `I` can be placed before `V` (5) and `X` (10) to make 4 and 9.
- `X` can be placed before `L` (50) and `C` (100) to make 40 and 90.
- `C` can be placed before `D` (500) and `M` (1000) to make 400 and 900.

Given a roman numeral, convert it to an integer. Input is guaranteed to be within
the range from 1 to 3999.

**Example 1**:

```
Input: "III"
Output: 3
```

**Example 2**:

```
Input: "IV"
Output: 4
```

**Example 3**:

```
Input: "IX"
Output: 9
```

**Example 4**:

```
Input: "LVIII"
Output: 58
Explanation: L = 50, V = 5, III = 3.
```

**Exmpale 5**:

```
Input: "MCMXCIV"
Output: 1994
Explanation: M = 1000, CM = 900, XC = 90, and IV = 4.
```

Taken from:
[leetcode](https://leetcode.com/problems/roman-to-integer/)

### Approach and Analysis

#### Approach #1
Setting aside the one big exception for the time being, the basic
mechanism for turning our roman numeral into an integer is pretty
straightforward. We can iterate through each character of the string,
mapping each one to its equivalent numeric value as we go. Finally,
we can add these to a variable that tracks our total sum. After having
passed through all of the characters, we can return this sum as the
output of our program.

To address the case where the next character is bigger than the current
one (and we should subtract it instead of adding it to `sum`), we can look
ahead to the next value and perform the correct piece of logic accordingly.

#### Complexity

The complexity for this approach is as follows:
- O(n) time complexity
- O(1) space complexity
		- No additional space is needed

### Rust solution (Approach #1)

```Rust
fn parse(c: char) -> i32 {
  match c {
    'I' => 1,
    'V' => 5,
    'X' => 10,
    'L' => 50,
    'C' => 100,
    'D' => 500,
    'M' => 1000,
    _ => 0
  }
}

impl Solution {
  pub fn roman_to_int(s: String) -> i32 {
    // The vector of characters expects to be indexed
    // by a value of type "usize"
    let mut i: usize = 0;
    let mut sum: i32 = 0;
    let char_vec: Vec<char> = s.chars().collect();
    let vec_len = char_vec.len();

    while (i < vec_len) {
      let curr: i32 = parse(char_vec[i]);
      // Look ahead to the next element if possible
      if (i + 1) < vec_len {
        let next: i32 = parse(char_vec[i+1]);
        if (next > curr) {
          sum -= 2 * curr;
        }
      }
      sum += curr;
      i += 1;
    }
    sum
  }
}
```

One quirk of this solution's implementation is that I've elected
to subtract double the current value in the instance where subtraction
is used.

Our logic in this scenario is the following:

1. If next exists and next > curr, we subtract 2 * curr from sum
2. For each element, add curr

For scenarios where `1` and `2` apply, the net result consists of
subtracting curr from sum.

This saves us from needing implement logic like so:

1. If next exists and next > curr, subtract curr from sum
2. If next exists and next < curr, add curr to sum
3. If next does not exist, add curr to sum

Notice the duplicate logic in `2` and `3`

### Bonus solution (Using iterators)

```Rust
fn parse(c: char) -> i32 {
  match c {
    'I' => 1,
    'V' => 5,
    'X' => 10,
    'L' => 50,
    'C' => 100,
    'D' => 500,
    'M' => 1000,
    _ => 0
  }
}

impl Solution {
  pub fn roman_to_int(s: String) -> i32 {
    let mut iter = s.chars().map(|c| parse(c)).peekable();
    let mut acc = 0;
      loop {
        match iter.next() {
          Some(x) => {
            match iter.peek() {
              // Peek returns a reference to
              // the next element
              Some(&p) => {
                if p > x {
                  acc -= 2*x
                }
              },
              None => {}
            };
            acc += x;
          },
          None => break
        }
      }
    acc
  }
}
```

This solution was an exercise in using only iterators to answer
the question. Accomplishing this requires that we transform the
iterator into one which has a "peek" function that we can use
to look at the next value without iterating over it.

While interesting as a proof of concept, in practice, this
solution has one glaring issue. Because we have to handle
"Options" that are returned from the iterator methods `.next()`
and `.peek()`, the code has many levels of nesting that make
it hard to read.

Additionally, according to leetcode, this solution takes about
4 times as long to run (0 ms vs. 4 ms).

### What I learned by solving this problem

My first solution to this problem, being straightforward, didn't
teach me anything new about rust. The part of the problem that
proved most challenging was the logic surrounding the "lookahead"
and ensuring that I didn't attempt to index the array at a position
that doesn't exist.

#### Working with Iterators

By finagling my way to a solution involving iterators, I learned
that there are some scenarios in which it simply doesn't make
sense to use this paradigm.

While rust's `peekable` struct did make an iterator solution
possible, simply using loops was much simpler.

As a point of interest, somebody did post an incredibly elegant
solution using iterators: [iterator solution (https://leetcode.com/problems/roman-to-integer/discuss/374718/Rust-pattern-matching-without-extra-allocation).
This solution took advantage of a point in the problem description
that I ignored: "Roman numerals are usually writte largest to smallest
from left to right". I learned from this that reading deeper into
the question may uncover assumptions that allow us to achieve the
most elegant solution.

### References
1. [peekable (official rust docs)](https://doc.rust-lang.org/beta/std/iter/struct.Peekable.html)
