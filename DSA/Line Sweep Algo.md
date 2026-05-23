
# U

**Line Sweep Algorithm** = “sort events, then scan left-to-right”

It’s the go-to trick for 1D geometry problems: intervals, segments, points. Instead of checking every pair of things O(n²), you turn geometry into a timeline of events and sweep a vertical line across it.

### **Core Idea**
1. **Turn shapes into events**
   Every object creates 1-2 “events” on the x-axis: start, end, point, etc.
2. **Sort all events**
   Usually by x-coordinate. Tie-break rules matter.
3. **Sweep left → right**
   Maintain an active set / state as you process events. Update answer on the fly.

Runtime: usually `O(n log n)` from sorting + `O(n log n)` for a balanced set = `O(n log n)` total.

### **When to Use Line Sweep**
If your problem has these, think line sweep:
- **Intervals**: merge, union length, count overlaps
- **Rectangle problems**: area of union, perimeter, skyline
- **Closest pair of points**
- **Line segment intersection**
- **Meeting rooms / scheduling**

### **Classic Example 1: Merge Intervals**
Problem: Given `[[1][3], [2][6], [8][10], [15][18]]`, merge overlapping intervals.

**Line Sweep Steps**
1. **Events**: For interval `[s][e]`, create `(s, +1)` start and `(e, -1)` end
2. **Sort**: `[1,+1], [2,+1], [3,-1], [6,-1], [8,+1], [10,-1], [15,+1], [18,-1]`
   Tie-break: process ends before starts if `x` equal, so `[3,-1]` before `[3,+1]`
3. **Sweep**: Track `active = 0`
   | x | event | active | Action |
   | --- | --- | --- | --- |
   | 1 | +1 | 1 | active 0→1: start new merged `[1` |
   | 2 | +1 | 2 | still active |
   | 3 | -1 | 1 | still active |
   | 6 | -1 | 0 | active 1→0: close merged `6]`, result `[1][6]` |
   | 8 | +1 | 1 | start `[8` |
   | 10 | -1 | 0 | close `10]`, result `[8][10]` |
   | 15 | +1 | 1 | start `[15` |
   | 18 | -1 | 0 | close `18]`, result `[15][18]` |

Result: `[[1][6], [8][10], [15][18]]`

### **Classic Example 2: Skyline Problem**
Given buildings `[left][right][height]`, output skyline key points.

**Events**:
- Building start: `(x=left, h=-height, type=start)`
- Building end: `(x=right, h=height, type=end)`

We use `-height` for starts so taller buildings sort first at same x.

**Sweep**: Use a multiset/max-heap of active heights. When max height changes, add `(x, current_max)` to result.

### **Classic Example 3: Count Max Overlapping Intervals**
"Given meeting times, how many rooms needed?" = max overlaps

```python
def min_meeting_rooms(intervals):
    events = []
    for start, end in intervals:
        events.append((start, 1)) # meeting starts
        events.append((end, -1)) # meeting ends

    events.sort() # if tie, end -1 before start +1 so no fake overlap

    active = 0
    max_rooms = 0
    for _, delta in events:
        active += delta
        max_rooms = max(max_rooms, active)
    return max_rooms
```

**Key tie-break**: For counting overlaps, process ends before starts at same x. Otherwise you'd count `[1][2]` and `[2][3]` as overlapping at x=2.

### **Implementation Template**
```python
def line_sweep(items):
    events = []
    for item in items:
        # convert item to 1-2 events
        events.append((x1, event_type1, data))
        events.append((x2, event_type2, data))

    # CRITICAL: get sort order right
    events.sort(key=lambda e: (e[0], custom_tie_break(e)))

    active_set = set() # or heap, BST, etc
    result = 0

    for x, type, data in events:
        if type == START:
            active_set.add(data)
            # update result
        else: # END
            active_set.remove(data)
            # update result

    return result
```

### **Tie-Break Rules Cheatsheet**
| Problem | If same x, order |
| --- | --- |
| Merge intervals / Union length | End `-1` before Start `+1` |
| Max overlaps / Meeting rooms | End `-1` before Start `+1` |
| Skyline | Start before End, and taller start before shorter |
| Rectangle union area | In before Out |

### **2D Extension: Sweep Line + Segment Tree**
For rectangle union area/perimeter: sweep on x, use segment tree to track active y-intervals. That upgrades you to `O(n log n)` for 2D.
