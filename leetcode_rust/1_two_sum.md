## 1 - Two Sum

### Problem statement
Given an array of integers, return **indices** of the two numbers
such that they add up to a specific target.

You may assume that each input would have **exactly** one solution,
and you may no use the same element twice.

**Example**:

```
Given nums = [2, 7, 11, 15], target = 9,
Because nums[0] = nums[1] = 2 + 7 = 9,
return [0, 1].
```

Taken from:
[leetcode](https://leetcode.com/problems/two-sum/)

### Approach and Analysis

#### 1 - Brute force Approach
The clear brute-force solution is to add all combinations of
numbers together by iterating through the array twice. You
can keep track of the indices at each stage to ensure that you're
never checking the same number twice.

The pseudocode for this looks something like:

```
for (first_element, first_index) in array:
  for (second_element, second_index) in array:
    if (first_element + second_element == total
      && first_index !== second_index):
      return [first_index, second_index]
    else:
      continue
```

Clearly this is far from the ideal solution if we want to optimize
for time:
- 0(n^2) time complexity
  - Because we iterate through the array twice
- 0(1) space complexity
  - We only need to store 2 values as we iterate

#### 2 - Hashmap Approach
Using a hashmap can help us trade some of our time for space.
We can iterate through all of the elements and insert them into
the hashmap as we find them. This allows us to look up existing
elements in *0(1)* time, which is a convenient way of finding
the complement of our current element (remember the complement
is the sum minus the current element). If we find our complement
in the hashmap, we are done and can return the indices of our
two elements; the hashmap index be stored then returned as the
value in the map while the other index we can track.

The pseudocode for this looks something like:

```
map = new HashMap()
for (element, index) in array:
  complement = target - element
  if complement in map:
    return [map.index_of(complement), index)]
  else:
    map.insert((complement, index))
```

This solution trades space for speed and will probably be the
optimal solution in most scenarios:
- 0(n) time complexity
  - We only iterate through the array once
- 0(n) space complexity
  - We may have to store up to all the elements in the HashMap
    (if the complement is at the end)

### Rust solution (approach #2)

A rust solution follows the hashmap pseudocode from above
quite closely:

Because part of the purpose of this blog is to learn Rust,
I'm going to highlight details of the language basics that
I'll omit later.

```rust
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
