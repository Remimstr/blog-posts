## 9 - Palindrome Number

### Problem statement
Determine whether an integer is a palindrome. An integer is a palindrome when it reads the same backward as forward.

**Example 1**:

```
Input: 121
Output: true
```

**Example 2**:

```
Input: -121
Output: false
Explanation: From left to right, it reads -121. From right to left, it becomes 121-. Therefore it is not a palindrome.
```

**Example 3**:

```
Input: 10
Output: false
Explanation: Reads 01 from right to left. Therefore it is not a palindrome.
```

Follow up:

Could you solve it without converting the integer to a string?

Taken from:
[leetcode](https://leetcode.com/problems/palindrome-number/)

### Approach and Analysis

#### Approach #1
I found the most intuitive approach to this problem was to create arrays
consisting of each digit in the integer, both in ascending and descending
order. Then they could be compared, and if found to be the same, the
integer is a palindrome.

In order to create the inverted array, we can use the same procedure as with
reverse integer of progressively taking the last digit using modulus 10
(% 10) and dividing by 10 until we reach 0.

#### Approach #2
This solution is like #1 except that instead of creating arrays,
we can simply accumulate on a number. In this case, we need only
generate the inverted number using our progressive modulus procedure,
which we can then compare to the input number.

One issue with this approach is that our inverted integer may overflow
(see reverse integer post for a discussion about this). To solve this
problem, we can invert the latter half of the integer and compare it to
the first half of the original (recall, they should match if they are
truly palindromes).

#### Complexity

The complexities for either approach is as follows:
- O(log<sub>10</sub>(n)) time complexity
  - We have to iterate through the base 10 order of magnitude
    of the input number.
- O(1) space complexity
  - This should be intuitive: this algorithm takes no more space than
    it does to store an integer or array that is the max length of an
    integer, which doesn't vary based on input size.

### Rust solution (Approach #1)

```Rust
impl Solution {
  pub fn is_palindrome(x: i32) -> bool {
    if (x < 0) {
      return false;
    }
    let mut stack = vec![];
    let mut mutx: i32 = x;

    while (mutx != 0) {
      stack.push(mutx % 10);
      mutx /= 10;
    }

    let mut reversed_stack = stack.clone();
    reversed_stack.reverse();
    for (index, element) in stack.iter().enumerate() {
      if (*element != reversed_stack[index]) {
        return false;
      }
    }
    return true;
  }
}
```

I see the biggest issue with this solution being:
- Use of high-level vector methods. This means:
  - The solution is less portable to other languages
  - Implementing it requires knowledge of vector methods

### Rust solution (Approach #2)

```Rust
impl Solution {
  pub fn is_palindrome(x: i32) -> bool {
    let mut mutx = x;

    # We need to check the case where x ends with a 0 but
    # doesn't start with 0 (ie. isn't 0). If we don't,
    # in the case where x is 10, both revertedNumber and mutx
    # will be 1, which will return true
    if (mutx < 0 || (mutx % 10 == 0) && (mutx != 0)) {
      return false;
    }

    let mut revertedNumber: i32 = 0;
    while (mutx > revertedNumber) {
      revertedNumber = revertedNumber * 10 + mutx % 10;
      mutx /= 10;
    }

    # In the case where a digit is odd, mutx will be longer than
    # the reverted number ex. when x = 12321, mutx = 12 and
    # revertedNumber = 123. We can divide the mutated number
    # by 10 to solve this problem
    return (mutx == revertedNumber) | (mutx == revertedNumber / 10);
  }
}
```

The problem with this solution is that we have to deal with edge cases
such as when end of the number is 0 and when the number is odd.

### Bonus solution (Approach #1)

I posted my solution to this problem using approach #1 in a discussion
post on leetcode and a very kind individual named
[ObliqueMotion](https://leetcode.com/obliquemotion/) provided
excellent feedback that I'll outline below:

```Rust
impl Solution {
  # Change the function sinature to avoid redeclaring x as a mut
  pub fn is_palindrome(mut x: i32) -> bool {
    # There should be no parentheses around the condition in the
    # `if` statement
    if x < 0 {
      return false;
    }
    # We may as well allocate only the space we need
    let mut stack = Vec::with_capacity(10);

    while x != 0 {
      stack.push(x % 10);
      x /= 10;
    }

    let half = stack.len() / 2;
    # This approach is a more idiomatic rust style using iterators
    stack
    # Make an interator
    .iter()
    # Yield the first `n` elements of the iterator
    .take(half)
    # Generate a new iterator with the reversed half and then
    # `zip` it to the first one
    .zip(stack.iter().rev().take(half))
    # Takes a closure that returns true or false and applies
    # it to each element and ensures that they all return `true`
    .all(|(left_hand_side, right_hand_side)| left_hand_side == right_hand_side)
  }
}
```

To help explain the iterator section, let's go through some examples:

Let's take the following arrays as examples:

`[1, 2, 3, 4]` - non-palindrome
`[1, 2, 2, 1]` - palindrome

```
stack
  .iter()
  .take(half)
```

This will result in an iterator that returns: `[1, 2]` for both examples

The next line:

```
  .zip(stack.iter().rev().take(half))
```

will result in an iterator that returns tuples where the 0th element in the tuple is
the first half of the list and the 1st element in the tuple is the reversed second
half of the list (in lockstep), conceptually represented by:

`[(1, 1), (2, 2)]` - for the palindrome example
`[(1, 3), (2, 4)]` - for the non-palindrome example

The last line hopefully makes sense considering the explanation given above.

```
  .all(|(left_hand_side, right_hand_side)| left_hand_side == right_hand_side)
```

### Bonus Solution (Approach #2)

The user who provided feedback on my solution posted his own answer using approach #2 here:
[ObliqueMotion's Solution](https://leetcode.com/problems/palindrome-number/discuss/333683/Rust-0ms-4ms)

### What I learned by solving this problem

Due to the excellent feedback that I received on my solution, I learned a lot about how to
write code in good Rust style.

Of the many things I learned throughout this exercise, getting comfortable with Iterators was probably
the most important one. I feel like this is a great start to "thinking" in a Rust way and to improve
my coding skills in general. You can look forward to seeing many more iterators in future posts!

#### `.zip`

This method took me the longest to understand but that I feel was well worth the effort. Previously I've
always approached this problem by making a for-loop in one of the vectors and then indexing the other one;
an approach that feels antiquated and inelegant compared to using iterators and the `.zip` method.

### References
1. [zip (official rust docs)](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.zip)
