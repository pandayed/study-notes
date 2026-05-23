A rotated sorted array is a sorted array that has been shifted from some pivot point.

Example:

```text
Original:  [0,1,2,4,5,6,7]
Rotated :  [4,5,6,7,0,1,2]
```

The important observation is:

- There is exactly one point where the order breaks.
- That breaking point contains the minimum element.
- The minimum element is also called the pivot.

In the above example:

```text
[4,5,6,7,0,1,2]
           ^
         minimum
```

Notice:

```text
4,5,6,7 > 0,1,2
```

You can visualize the array as a valley:

```text
4 5 6 7 0 1 2
        ^
      valley bottom
```

Our goal is to find that valley bottom efficiently.

---

# Important Observation

A rotated sorted array always consists of two independently sorted parts.

Example:

```text
[4,5,6,7] [0,1,2]
```

Both halves are sorted individually.

The minimum element lies exactly at the boundary where sorting breaks.

Another important edge case:

- One half may contain `0` elements.

Example:

```text
[1,2,3,4,5]
```

This is technically a rotated array where the second half has size `0`.

---

# Why Binary Search Works

This problem is still based on sorted order.

The array is not fully sorted, but one side of the array is always guaranteed to be sorted.

That is enough for binary search.

Instead of asking:

```text
"Did we find the target?"
```

we ask:

```text
"Which side is properly sorted?"
```

Then we discard the perfectly sorted side and move toward the unsorted side, because the minimum lies there.

---

# Binary Search Setup

We maintain:

```cpp
low
high
mid
```

At every step:

```cpp
mid = (low + high) / 2
```

---

# Core Logic (No Duplicates)


Discard the perfectly sorted part.

For this, we compare:

```cpp
nums[mid] with nums[high]
```

There are two cases.

---

## Case 1: `nums[mid] > nums[high]`

Example:

```text
[4,5,6,7,0,1,2]
       ^
      mid
               ^
             high
```

Here:

```text
7 > 2
```

This means:

- Left side is not properly sorted.
- The rotation point lies to the right of `mid`.
- Minimum cannot be at `mid`.

So:

```cpp
low = mid + 1;
```

We safely discard the left side.

---

## Case 2: `nums[mid] < nums[high]`

Example:

```text
[4,5,1,2,3]
     ^
    mid
         ^
       high
```

Here:

```text
1 < 3
```

This means:

- Right side is properly sorted.
- The minimum may be `mid` itself.

This is extremely important.

So we do:

```cpp
high = mid;
```

NOT:

```cpp
high = mid - 1;
```

---

# Why `high = mid - 1` is Wrong

Consider:

```text
[4,5,1,2,3]
```

```text
low = 0
high = 4
mid = 2
```

```text
nums[mid] = 1
nums[high] = 3
```

Since:

```text
1 < 3
```

you know the right side is sorted.

But if you do:

```cpp
high = mid - 1;
```

then you remove the actual minimum.

You lose `1`.

That is why:

```cpp
high = mid;
```

is mandatory.

---

# Important Principle

At every step:

- Discard the side that is perfectly sorted.
- Move toward the side that contains disorder.

Because the disorder contains the rotation point.

And the rotation point contains the minimum.

---

# What If We Compare `nums[low] < nums[mid]` Instead?

This also works.

But the interpretation changes.

If:

```cpp
nums[low] <= nums[mid]
```

then:

- Left side is properly sorted.
- Minimum lies on the right side.
- Unless the entire array is already sorted.

Example:

```text
[4,5,6,7,0,1,2]
```

```text
low = 0
mid = 3
```

```text
4 < 7
```

So left side is sorted.

Minimum must be on the right.

---

However, comparing with `high` is usually cleaner for this specific problem because:

- We directly reason about where the minimum can exist.
- The condition naturally handles the case where `mid` itself may be the minimum.
- The duplicate handling becomes easier later.

---

# Complete Solution Without Duplicates

```cpp
int findMin(vector<int>& nums) {

    int low = 0;
    int high = nums.size() - 1;

    while(low < high){

        int mid = (low + high) / 2;

        if(nums[mid] > nums[high]){

            low = mid + 1;

        } else {

            high = mid;
        }
    }

    return nums[low];
}
```

---

# What Changes When Duplicates Exist?

Now consider:

```text
[2,2,2,0,1,2]
```

The earlier logic mostly works.

But duplicates create ambiguity.

---

# Ambiguous Situation

Suppose:

```cpp
nums[mid] == nums[high]
```

Example:

```text
[3,3,1,3]
```

```text
low = 0
high = 3
mid = 1
```

```text
nums[mid] = 3
nums[high] = 3
```

Now you cannot determine:

- Which side is sorted.
    
- Where the minimum lies.
    

Because duplicates destroy the ordering information.

---

# Why `high--` Works

When:

```cpp
nums[mid] == nums[high]
```

we do:

```cpp
high--;
```

Why is this safe?

Because if:

```text
nums[high] == nums[mid]
```

then removing `high` does not remove the only copy of the minimum.

Two possibilities exist.

---

## Possibility 1

The minimum lies somewhere before `high`.

Then removing `high` changes nothing.

---

## Possibility 2

`nums[high]` itself is the minimum.

But since:

```text
nums[mid] == nums[high]
```

another identical value still exists inside the range.

So the minimum is still preserved.

Thus:

```cpp
high--;
```

is always safe.

---

# Why `low++` Does NOT Work

Consider:

```text
[1,2,2,2,2,2]
```

Suppose:

```text
low = 0
high = 5
mid = 2
```

```text
nums[mid] = 2
nums[high] = 2
```

If you do:

```cpp
low++;
```

you remove:

```text
1
```

which is the actual minimum.

You permanently lose the answer.

That is why:

```cpp
high--
```

works,

but:

```cpp
low++
```

does not.

---

# Final Code (With Duplicates)

```cpp
int findMin(vector<int>& nums) {

    int low = 0;
    int high = nums.size() - 1;

    while(low < high){

        int mid = (low + high) / 2;

        if(nums[mid] > nums[high]){

            low = mid + 1;

        }
        else if(nums[mid] < nums[high]){

            high = mid;

        }
        else{

            high--;
        }
    }

    return nums[low];
}
```

---

# Time Complexity

Without duplicates:

```text
O(log n)
```

With duplicates:

```text
Worst case: O(n)
```

Example worst case:

```text
[1,1,1,1,1,1,1]
```

Because every step only reduces:

```text
high--
```

---

# Key Lessons to Remember

- Rotated sorted arrays contain two independently sorted regions.
- The minimum lies where sorting breaks.
- Binary search works because one side is always sorted.
- If `nums[mid] > nums[high]`, minimum is on the right.
- If `nums[mid] < nums[high]`, minimum may be `mid`.
- Therefore use:

```cpp
high = mid;
```

not:

```cpp
high = mid - 1;
```

- Duplicates destroy ordering information.
- When `nums[mid] == nums[high]`, safely remove `high`.
- Never remove `low` in that ambiguous case.