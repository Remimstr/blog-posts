## 1 - Two Sum

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

#### EDIT HERE

### Rust solution (approach #2)

A rust solution follows the hashmap pseudocode from above
quite closely:

Because part of the purpose of this blog is to learn Rust,
I'm going to highlight details of the language basics that
I'll omit later.

```Rust
# import the `HashMap` function right into the namespace
use std::collections::HashMap;

impl Solution {
	# We provide our function within the implementation on type "Solution"
	pub fn two_sum(nums: Vec<i32>, target: i32) -> Vec<i32> {
		# We call the HashMap constructor using the `new` function
		let mut elements = HashMap::new();

		# Enumerate is an iterator function (std::iter::enumerate) that
		# yields (index, value) pairs
		for (index, element) in nums.iter().enumerate() {
			let complement = target - element;
			# The match function evaluates the result of a function call
			# (in this case, elements.get()) and runs code down the first
			# "arm" that it encounters.
			match elements.get(&complement) {
				# Use the !vec macro to generate a new vector
				Some(k) => return vec![*k, index as i32],
				None => ()
			}
			elements.insert(element, index as 32);
		}
		return vec![0, 0];
	}
}
```

### What I learned by solving this problem

#### Iterators

Iterator indices are stored as *usize* types. This means we need to cast
them to type *i32* before returning the final vector. Using `as` makes this
a safe cast that the compiler is ok with.

Note though that a usize integer could be bigger than i32 and the values could
end up truncated in the process.

#### HashMaps

The `.get()` function of HashMap takes a reference to an element. Thus
we need to pass the complement by taking its reference (&).

Similarly, in the `Some` arm of the match statement, where we return
the answer, we have to dereference the result (*).

It might be intuitive to add our new element to the map in the `None =>` arm
of the match statement but we cannot do this because the `map.get()` function
**borrows** map for the entirety of the block, so we can't insert into it.

### References
1. [HashMap get explanation](https://stackoverflow.com/questions/41423809/create-a-hashmapi32-i32-in-rust)
2. [enumerate (official rust docs)](https://doc.rust-lang.org/core/iter/trait.Iterator.html#method.enumerate)
