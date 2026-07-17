# Extended DSA Question Bank — Detailed Explanations (Java)
### Matches sections: Arrays, Strings, Stacks & Queues, Trees, Heaps, Graphs

For each pattern: **concept**, **why it works**, then full Java solution with complexity.
At the end of each section: a **checklist** of every numbered problem you listed, so you can track progress.

---

## SECTION 1: ARRAYS — Two Pointers Pattern

**Concept:** When an array is sorted (or can be sorted), use two indices — one from the start, one from the end (or both moving forward at different speeds) — to avoid nested loops. Converts O(n²) brute force into O(n) or O(n log n).

### Q2. Three Sum
**Explanation:** Sort the array. Fix one element, then use two-pointer technique on the remaining subarray to find pairs that sum to the negative of the fixed element. Skip duplicates to avoid repeated triplets.
```java
public List<List<Integer>> threeSum(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> result = new ArrayList<>();
    for (int i = 0; i < nums.length - 2; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) continue; // skip duplicate anchors
        int left = i + 1, right = nums.length - 1;
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            if (sum == 0) {
                result.add(Arrays.asList(nums[i], nums[left], nums[right]));
                while (left < right && nums[left] == nums[left + 1]) left++;
                while (left < right && nums[right] == nums[right - 1]) right--;
                left++; right--;
            } else if (sum < 0) left++;
            else right--;
        }
    }
    return result;
}
// Time: O(n^2), Space: O(1) excluding output
```

### Q4. Container With Most Water
(Already covered — see previous cheat sheet, Q1. Same file for reference.)

### Q5. Trapping Rain Water
**Explanation:** For each index, water trapped = min(maxLeft, maxRight) - height[i]. Use two pointers tracking running max from both sides so you never need extra arrays.
```java
public int trap(int[] height) {
    int left = 0, right = height.length - 1;
    int leftMax = 0, rightMax = 0, water = 0;
    while (left < right) {
        if (height[left] < height[right]) {
            leftMax = Math.max(leftMax, height[left]);
            water += leftMax - height[left];
            left++;
        } else {
            rightMax = Math.max(rightMax, height[right]);
            water += rightMax - height[right];
            right--;
        }
    }
    return water;
}
// Time: O(n), Space: O(1)
```

### Q7. Move Zeroes to End
**Explanation:** Maintain a pointer for the "next non-zero position." Swap every non-zero element you find into that position, preserving relative order.
```java
public void moveZeroes(int[] nums) {
    int insertPos = 0;
    for (int num : nums) {
        if (num != 0) nums[insertPos++] = num;
    }
    while (insertPos < nums.length) nums[insertPos++] = 0;
}
// Time: O(n), Space: O(1)
```

### Q8. Sort Colors (Dutch National Flag)
**Explanation:** Three pointers: `low` (boundary for 0s), `mid` (current), `high` (boundary for 2s). Swap based on value at `mid`, single pass.
```java
public void sortColors(int[] nums) {
    int low = 0, mid = 0, high = nums.length - 1;
    while (mid <= high) {
        if (nums[mid] == 0) {
            swap(nums, low++, mid++);
        } else if (nums[mid] == 1) {
            mid++;
        } else {
            swap(nums, mid, high--);
        }
    }
}
private void swap(int[] nums, int i, int j) {
    int temp = nums[i]; nums[i] = nums[j]; nums[j] = temp;
}
// Time: O(n), Space: O(1)
```

### Q9. Minimum Size Subarray Sum
**Explanation:** Sliding window: expand right to grow sum, shrink left while sum ≥ target, tracking minimum window length.
```java
public int minSubArrayLen(int target, int[] nums) {
    int left = 0, sum = 0, minLen = Integer.MAX_VALUE;
    for (int right = 0; right < nums.length; right++) {
        sum += nums[right];
        while (sum >= target) {
            minLen = Math.min(minLen, right - left + 1);
            sum -= nums[left++];
        }
    }
    return minLen == Integer.MAX_VALUE ? 0 : minLen;
}
// Time: O(n), Space: O(1)
```

**Checklist — Section 1 (Q1-10):** ☐1 Two Sum ☐2 Three Sum ☐3 Four Sum ☐4 Container With Most Water ☐5 Trapping Rain Water ☐6 Remove Duplicates ☐7 Move Zeroes ☐8 Sort Colors ☐9 Min Size Subarray Sum ☐10 Squares of Sorted Array

---

## SECTION 2: STRINGS — Hashing & Sliding Window

**Concept:** Most string problems reduce to counting character frequencies (HashMap or int[26]/int[128] array) or maintaining a window of valid characters.

### Q51. Valid Anagram
```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    int[] count = new int[26];
    for (char c : s.toCharArray()) count[c - 'a']++;
    for (char c : t.toCharArray()) count[c - 'a']--;
    for (int c : count) if (c != 0) return false;
    return true;
}
// Time: O(n), Space: O(1) — fixed 26-size array
```

### Q52. Group Anagrams
**Explanation:** Anagrams share the same sorted string. Use the sorted string as a HashMap key to group originals.
```java
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> map = new HashMap<>();
    for (String s : strs) {
        char[] chars = s.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars);
        map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(map.values());
}
// Time: O(n * k log k), k = avg string length, Space: O(n*k)
```

### Q53. Longest Common Prefix
```java
public String longestCommonPrefix(String[] strs) {
    if (strs.length == 0) return "";
    String prefix = strs[0];
    for (int i = 1; i < strs.length; i++) {
        while (!strs[i].startsWith(prefix)) {
            prefix = prefix.substring(0, prefix.length() - 1);
            if (prefix.isEmpty()) return "";
        }
    }
    return prefix;
}
// Time: O(n*m), Space: O(1)
```

### Q55. String to Integer (atoi)
**Explanation:** Careful state machine: skip leading whitespace, handle optional sign, read digits, clamp to Integer range.
```java
public int myAtoi(String s) {
    int i = 0, n = s.length();
    while (i < n && s.charAt(i) == ' ') i++;
    if (i == n) return 0;

    int sign = 1;
    if (s.charAt(i) == '+' || s.charAt(i) == '-') {
        sign = s.charAt(i) == '-' ? -1 : 1;
        i++;
    }

    long result = 0;
    while (i < n && Character.isDigit(s.charAt(i))) {
        result = result * 10 + (s.charAt(i) - '0');
        if (result * sign > Integer.MAX_VALUE) return Integer.MAX_VALUE;
        if (result * sign < Integer.MIN_VALUE) return Integer.MIN_VALUE;
        i++;
    }
    return (int) (result * sign);
}
// Time: O(n), Space: O(1)
```

### Q56/57. Roman to Integer / Integer to Roman
```java
// Roman to Integer
public int romanToInt(String s) {
    Map<Character, Integer> values = Map.of('I',1,'V',5,'X',10,'L',50,'C',100,'D',500,'M',1000);
    int total = 0;
    for (int i = 0; i < s.length(); i++) {
        int curr = values.get(s.charAt(i));
        if (i + 1 < s.length() && curr < values.get(s.charAt(i + 1))) {
            total -= curr; // subtractive case, e.g. IV
        } else {
            total += curr;
        }
    }
    return total;
}
// Time: O(n), Space: O(1)
```

### Q63. Longest Repeating Character Replacement
**Explanation:** Sliding window where window is valid if `(window length - count of most frequent char) <= k`. Track max frequency char count as you expand.
```java
public int characterReplacement(String s, int k) {
    int[] count = new int[26];
    int left = 0, maxFreq = 0, maxLen = 0;
    for (int right = 0; right < s.length(); right++) {
        count[s.charAt(right) - 'A']++;
        maxFreq = Math.max(maxFreq, count[s.charAt(right) - 'A']);
        while ((right - left + 1) - maxFreq > k) {
            count[s.charAt(left) - 'A']--;
            left++;
        }
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
// Time: O(n), Space: O(1)
```

### Q64. Minimum Window Substring
**Explanation:** Sliding window with two frequency maps — expand right until window contains all of `t`, then shrink left while still valid, tracking the smallest valid window.
```java
public String minWindow(String s, String t) {
    if (s.isEmpty() || t.isEmpty()) return "";
    Map<Character, Integer> need = new HashMap<>();
    for (char c : t.toCharArray()) need.merge(c, 1, Integer::sum);

    Map<Character, Integer> window = new HashMap<>();
    int left = 0, required = need.size(), formed = 0;
    int[] best = {-1, 0, 0}; // length, left, right

    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        window.merge(c, 1, Integer::sum);
        if (need.containsKey(c) && window.get(c).intValue() == need.get(c).intValue()) formed++;

        while (formed == required) {
            if (best[0] == -1 || right - left + 1 < best[0]) {
                best[0] = right - left + 1; best[1] = left; best[2] = right;
            }
            char leftChar = s.charAt(left);
            window.put(leftChar, window.get(leftChar) - 1);
            if (need.containsKey(leftChar) && window.get(leftChar) < need.get(leftChar)) formed--;
            left++;
        }
    }
    return best[0] == -1 ? "" : s.substring(best[1], best[2] + 1);
}
// Time: O(|s| + |t|), Space: O(|s| + |t|)
```

**Checklist — Section 2 (Q51-64):** ☐51 Valid Anagram ☐52 Group Anagrams ☐53 Longest Common Prefix ☐54 Reverse Words ☐55 atoi ☐56 Roman→Int ☐57 Int→Roman ☐58 Count and Say ☐59 ZigZag Conversion ☐60 Multiply Strings ☐61 Longest Palindrome ☐62 First Unique Char ☐63 Longest Repeating Char Replacement ☐64 Minimum Window Substring

---

## SECTION 4: STACKS & QUEUES — Monotonic Stack

**Concept:** Maintain a stack where elements are kept in increasing or decreasing order. When a new element breaks that order, you've found the "next greater/smaller" relationship for the popped elements — in one linear pass instead of nested loops.

### Q119. Next Greater Element I
```java
public int[] nextGreaterElement(int[] nums1, int[] nums2) {
    Map<Integer, Integer> nextGreater = new HashMap<>();
    Deque<Integer> stack = new ArrayDeque<>();
    for (int num : nums2) {
        while (!stack.isEmpty() && stack.peek() < num) {
            nextGreater.put(stack.pop(), num);
        }
        stack.push(num);
    }
    int[] result = new int[nums1.length];
    for (int i = 0; i < nums1.length; i++) {
        result[i] = nextGreater.getOrDefault(nums1[i], -1);
    }
    return result;
}
// Time: O(n+m), Space: O(n)
```

### Q120. Next Greater Element II (Circular)
**Explanation:** Simulate circular array by iterating `2*n` times using `i % n`, same monotonic stack idea.
```java
public int[] nextGreaterElements(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    Arrays.fill(result, -1);
    Deque<Integer> stack = new ArrayDeque<>();
    for (int i = 0; i < 2 * n; i++) {
        int num = nums[i % n];
        while (!stack.isEmpty() && nums[stack.peek()] < num) {
            result[stack.pop()] = num;
        }
        if (i < n) stack.push(i);
    }
    return result;
}
// Time: O(n), Space: O(n)
```

### Q121. Daily Temperatures
(Already covered in the main cheat sheet — Problem #3.)

### Q122. Largest Rectangle in Histogram
**Explanation:** For each bar, find the first shorter bar to its left and right — that defines the max rectangle width using this bar's height. Use a monotonic increasing stack of indices.
```java
public int largestRectangleArea(int[] heights) {
    Deque<Integer> stack = new ArrayDeque<>();
    int maxArea = 0, n = heights.length;
    for (int i = 0; i <= n; i++) {
        int h = (i == n) ? 0 : heights[i];
        while (!stack.isEmpty() && heights[stack.peek()] >= h) {
            int height = heights[stack.pop()];
            int width = stack.isEmpty() ? i : i - stack.peek() - 1;
            maxArea = Math.max(maxArea, height * width);
        }
        stack.push(i);
    }
    return maxArea;
}
// Time: O(n), Space: O(n)
```

### Q126. Online Stock Span
**Explanation:** For each new price, pop all stack entries with price ≤ current (they're "covered" by today's span), accumulate their spans, then push current.
```java
class StockSpanner {
    private Deque<int[]> stack = new ArrayDeque<>(); // [price, span]

    public int next(int price) {
        int span = 1;
        while (!stack.isEmpty() && stack.peek()[0] <= price) {
            span += stack.pop()[1];
        }
        stack.push(new int[]{price, span});
        return span;
    }
}
// Amortized O(1) per call, Space: O(n)
```

### Q128. Asteroid Collision
**Explanation:** Use a stack to simulate collisions. A right-moving asteroid (positive) can collide with the stack's top only if it's negative — compare magnitudes to decide survivor.
```java
public int[] asteroidCollide(int[] asteroids) {
    Deque<Integer> stack = new ArrayDeque<>();
    for (int a : asteroids) {
        boolean alive = true;
        while (alive && a < 0 && !stack.isEmpty() && stack.peek() > 0) {
            int top = stack.peek();
            if (top < -a) { stack.pop(); }       // top explodes
            else if (top == -a) { stack.pop(); alive = false; } // both explode
            else { alive = false; }               // a explodes
        }
        if (alive) stack.push(a);
    }
    int[] result = new int[stack.size()];
    for (int i = result.length - 1; i >= 0; i--) result[i] = stack.pop();
    return result;
}
// Time: O(n), Space: O(n)
```

**Checklist — Section 4 (Q119-128):** ☐119 Next Greater I ☐120 Next Greater II ☐121 Daily Temperatures ☐122 Largest Rectangle ☐123 Maximal Rectangle ☐124 Sum of Subarray Minimums ☐125 Max Width Ramp ☐126 Online Stock Span ☐127 132 Pattern ☐128 Asteroid Collision

---

## SECTION 6: TREES — Traversals & Recursion

**Concept:** DFS (pre/in/post-order) explores depth-first; BFS (level-order) explores breadth-first with a queue. Most "properties" problems (depth, diameter, balance) use post-order recursion: solve children first, combine at parent.

### Q175. Binary Tree Inorder Traversal (Iterative)
**Explanation:** Simulate the recursion with an explicit stack — go as far left as possible, process node, then go right.
```java
public List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode curr = root;
    while (curr != null || !stack.isEmpty()) {
        while (curr != null) {
            stack.push(curr);
            curr = curr.left;
        }
        curr = stack.pop();
        result.add(curr.val);
        curr = curr.right;
    }
    return result;
}
// Time: O(n), Space: O(h)
```

### Q179. Binary Tree Zigzag Level Order
**Explanation:** Standard BFS level order, but reverse alternate levels (or use a deque and add left/right depending on direction).
```java
public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    boolean leftToRight = true;
    while (!queue.isEmpty()) {
        int size = queue.size();
        LinkedList<Integer> level = new LinkedList<>();
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            if (leftToRight) level.addLast(node.val);
            else level.addFirst(node.val);
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.add(level);
        leftToRight = !leftToRight;
    }
    return result;
}
// Time: O(n), Space: O(n)
```

### Q180. Binary Tree Right Side View
**Explanation:** BFS level order, but only keep the last node processed at each level (or first, if traversing right-to-left).
```java
public List<Integer> rightSideView(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    if (root == null) return result;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int size = queue.size();
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            if (i == size - 1) result.add(node.val); // last node of the level
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
    }
    return result;
}
// Time: O(n), Space: O(n)
```

### Q187. Diameter of Binary Tree
**Explanation:** Post-order recursion — at each node, diameter through it = leftHeight + rightHeight. Track global max while computing height.
```java
private int diameter = 0;

public int diameterOfBinaryTree(TreeNode root) {
    height(root);
    return diameter;
}
private int height(TreeNode node) {
    if (node == null) return 0;
    int left = height(node.left);
    int right = height(node.right);
    diameter = Math.max(diameter, left + right);
    return 1 + Math.max(left, right);
}
// Time: O(n), Space: O(h)
```

### Q188. Balanced Binary Tree
**Explanation:** Post-order recursion returning -1 as a sentinel to short-circuit as soon as imbalance found (avoids recomputation).
```java
public boolean isBalanced(TreeNode root) {
    return checkHeight(root) != -1;
}
private int checkHeight(TreeNode node) {
    if (node == null) return 0;
    int left = checkHeight(node.left);
    if (left == -1) return -1;
    int right = checkHeight(node.right);
    if (right == -1) return -1;
    if (Math.abs(left - right) > 1) return -1;
    return 1 + Math.max(left, right);
}
// Time: O(n), Space: O(h)
```

**Checklist — Section 6 (Q175-188):** ☐175 Inorder(iter) ☐176 Preorder(iter) ☐177 Postorder(iter) ☐178 Level Order ☐179 Zigzag ☐180 Right Side View ☐181 Average of Levels ☐182 N-ary Level Order ☐183 Vertical Order ☐184 Boundary Traversal ☐185 Max Depth ☐186 Min Depth ☐187 Diameter ☐188 Balanced Tree

---

## SECTION 7: HEAPS — Top-K & Two-Heap Patterns

**Concept:** Min-heap of size K keeps the K "best" (largest, closest, most frequent) elements seen so far without sorting everything. Two-heap pattern (max-heap for lower half, min-heap for upper half) finds a running median.

### Q231. Kth Largest Element in a Stream
```java
class KthLargest {
    private final PriorityQueue<Integer> heap;
    private final int k;

    public KthLargest(int k, int[] nums) {
        this.k = k;
        heap = new PriorityQueue<>();
        for (int n : nums) add(n);
    }

    public int add(int val) {
        heap.offer(val);
        if (heap.size() > k) heap.poll();
        return heap.peek();
    }
}
// add: O(log k), Space: O(k)
```

### Q234. K Closest Points to Origin
**Explanation:** Max-heap of size K on distance — if a new point is closer than the heap's farthest, swap it in.
```java
public int[][] kClosest(int[][] points, int k) {
    PriorityQueue<int[]> maxHeap = new PriorityQueue<>(
        (a, b) -> (b[0]*b[0] + b[1]*b[1]) - (a[0]*a[0] + a[1]*a[1])
    );
    for (int[] point : points) {
        maxHeap.offer(point);
        if (maxHeap.size() > k) maxHeap.poll();
    }
    return maxHeap.toArray(new int[k][]);
}
// Time: O(n log k), Space: O(k)
```

### Q236. Kth Smallest Element in a Sorted Matrix
**Explanation:** Min-heap seeded with the first element of each row; pop smallest, push its row-neighbor, repeat K times. Exploits that each row/column is sorted.
```java
public int kthSmallest(int[][] matrix, int k) {
    int n = matrix.length;
    PriorityQueue<int[]> minHeap = new PriorityQueue<>((a, b) -> a[0] - b[0]);
    for (int r = 0; r < Math.min(n, k); r++) {
        minHeap.offer(new int[]{matrix[r][0], r, 0});
    }
    int result = -1;
    for (int i = 0; i < k; i++) {
        int[] curr = minHeap.poll();
        result = curr[0];
        int row = curr[1], col = curr[2];
        if (col + 1 < n) minHeap.offer(new int[]{matrix[row][col+1], row, col+1});
    }
    return result;
}
// Time: O(k log n), Space: O(n)
```

### Q239. Task Scheduler
**Explanation:** Greedy with max-heap on frequency — always schedule the most frequent remaining task, cooling down others.
```java
public int leastInterval(char[] tasks, int n) {
    int[] freq = new int[26];
    for (char t : tasks) freq[t - 'A']++;
    PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
    for (int f : freq) if (f > 0) maxHeap.offer(f);

    int time = 0;
    while (!maxHeap.isEmpty()) {
        List<Integer> cycle = new ArrayList<>();
        for (int i = 0; i <= n; i++) {
            if (!maxHeap.isEmpty()) cycle.add(maxHeap.poll() - 1);
        }
        for (int c : cycle) if (c > 0) maxHeap.offer(c);
        time += maxHeap.isEmpty() ? cycle.size() : n + 1;
    }
    return time;
}
// Time: O(n), Space: O(1) — bounded by 26 letters
```

### Q241. Find Median from Data Stream
**Explanation:** Max-heap holds the smaller half, min-heap holds the larger half, kept balanced in size. Median is either the top of the larger heap or the average of both tops.
```java
class MedianFinder {
    private PriorityQueue<Integer> small = new PriorityQueue<>(Collections.reverseOrder()); // max-heap
    private PriorityQueue<Integer> large = new PriorityQueue<>(); // min-heap

    public void addNum(int num) {
        small.offer(num);
        large.offer(small.poll());
        if (large.size() > small.size()) {
            small.offer(large.poll());
        }
    }

    public double findMedian() {
        if (small.size() > large.size()) return small.peek();
        return (small.peek() + large.peek()) / 2.0;
    }
}
// addNum: O(log n), findMedian: O(1)
```

**Checklist — Section 7 (Q231-242):** ☐231 Kth Largest in Stream ☐232 Top K Frequent ☐233 Top K Frequent Words ☐234 K Closest Points ☐235 K Pairs Smallest Sums ☐236 Kth Smallest in Matrix ☐237 Sort Chars by Frequency ☐238 Reorganize String ☐239 Task Scheduler ☐240 Max Frequency Stack ☐241 Median from Data Stream ☐242 Sliding Window Median

---

## SECTION 8: GRAPHS — BFS & DFS Patterns

**Concept:** BFS guarantees shortest path in unweighted graphs (explore level by level). DFS explores all paths / detects cycles / finds connected components using a visited set.

### Q253. Number of Islands
(Already covered in main cheat sheet — Problem #8, via DFS. BFS version below for comparison.)
```java
public int numIslandsBFS(char[][] grid) {
    int rows = grid.length, cols = grid[0].length, count = 0;
    boolean[][] visited = new boolean[rows][cols];
    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};

    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (grid[r][c] == '1' && !visited[r][c]) {
                count++;
                Queue<int[]> queue = new LinkedList<>();
                queue.offer(new int[]{r, c});
                visited[r][c] = true;
                while (!queue.isEmpty()) {
                    int[] cell = queue.poll();
                    for (int[] d : dirs) {
                        int nr = cell[0] + d[0], nc = cell[1] + d[1];
                        if (nr >= 0 && nc >= 0 && nr < rows && nc < cols
                            && grid[nr][nc] == '1' && !visited[nr][nc]) {
                            visited[nr][nc] = true;
                            queue.offer(new int[]{nr, nc});
                        }
                    }
                }
            }
        }
    }
    return count;
}
// Time: O(rows*cols), Space: O(rows*cols)
```

### Q258. Rotting Oranges (Multi-source BFS)
**Explanation:** Start BFS from ALL rotten oranges simultaneously (multi-source). Each BFS "layer" = one minute passing.
```java
public int orangesRotting(int[][] grid) {
    int rows = grid.length, cols = grid[0].length, fresh = 0, minutes = 0;
    Queue<int[]> queue = new LinkedList<>();
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (grid[r][c] == 2) queue.offer(new int[]{r, c});
            else if (grid[r][c] == 1) fresh++;
        }
    }
    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
    while (!queue.isEmpty() && fresh > 0) {
        int size = queue.size();
        for (int i = 0; i < size; i++) {
            int[] cell = queue.poll();
            for (int[] d : dirs) {
                int nr = cell[0] + d[0], nc = cell[1] + d[1];
                if (nr >= 0 && nc >= 0 && nr < rows && nc < cols && grid[nr][nc] == 1) {
                    grid[nr][nc] = 2;
                    fresh--;
                    queue.offer(new int[]{nr, nc});
                }
            }
        }
        minutes++;
    }
    return fresh == 0 ? minutes : -1;
}
// Time: O(rows*cols), Space: O(rows*cols)
```

### Q263. Word Ladder
**Explanation:** BFS where each "neighbor" is formed by changing one letter of the current word (and checking if it's in the word set). BFS guarantees shortest transformation sequence.
```java
public int ladderLength(String beginWord, String endWord, List<String> wordList) {
    Set<String> wordSet = new HashSet<>(wordList);
    if (!wordSet.contains(endWord)) return 0;

    Queue<String> queue = new LinkedList<>();
    queue.offer(beginWord);
    int steps = 1;

    while (!queue.isEmpty()) {
        int size = queue.size();
        for (int i = 0; i < size; i++) {
            String word = queue.poll();
            if (word.equals(endWord)) return steps;
            char[] chars = word.toCharArray();
            for (int j = 0; j < chars.length; j++) {
                char original = chars[j];
                for (char c = 'a'; c <= 'z'; c++) {
                    chars[j] = c;
                    String next = new String(chars);
                    if (wordSet.contains(next)) {
                        wordSet.remove(next); // mark visited
                        queue.offer(next);
                    }
                }
                chars[j] = original;
            }
        }
        steps++;
    }
    return 0;
}
// Time: O(M^2 * N), M = word length, N = word list size, Space: O(M*N)
```

### Q268. Clone Graph
**Explanation:** DFS/BFS with a HashMap from original node → cloned node, to avoid infinite loops on cycles and to reuse already-cloned nodes.
```java
public Node cloneGraph(Node node) {
    if (node == null) return null;
    Map<Node, Node> visited = new HashMap<>();
    return dfs(node, visited);
}
private Node dfs(Node node, Map<Node, Node> visited) {
    if (visited.containsKey(node)) return visited.get(node);
    Node clone = new Node(node.val);
    visited.put(node, clone);
    for (Node neighbor : node.neighbors) {
        clone.neighbors.add(dfs(neighbor, visited));
    }
    return clone;
}
// Time: O(V+E), Space: O(V)
```

### Q269. Course Schedule (Cycle Detection)
(Already covered in main cheat sheet — Problem #9, via Kahn's/topological sort. DFS-based cycle detection below.)
```java
public boolean canFinishDFS(int numCourses, int[][] prerequisites) {
    List<List<Integer>> graph = new ArrayList<>();
    for (int i = 0; i < numCourses; i++) graph.add(new ArrayList<>());
    for (int[] p : prerequisites) graph.get(p[1]).add(p[0]);

    int[] state = new int[numCourses]; // 0=unvisited, 1=visiting, 2=done
    for (int i = 0; i < numCourses; i++) {
        if (state[i] == 0 && hasCycle(graph, state, i)) return false;
    }
    return true;
}
private boolean hasCycle(List<List<Integer>> graph, int[] state, int node) {
    state[node] = 1;
    for (int next : graph.get(node)) {
        if (state[next] == 1) return true; // back edge = cycle
        if (state[next] == 0 && hasCycle(graph, state, next)) return true;
    }
    state[node] = 2;
    return false;
}
// Time: O(V+E), Space: O(V+E)
```

**Checklist — Section 8 (Q253-269+):** ☐253 Number of Islands ☐254 Max Area of Island ☐255 Flood Fill ☐256 Pacific Atlantic ☐257 01 Matrix ☐258 Rotting Oranges ☐259 Walls and Gates ☐260 Shortest Path Binary Matrix ☐261 Snakes and Ladders ☐262 Open the Lock ☐263 Word Ladder ☐264 Word Ladder II ☐265 Min Knight Moves ☐266 Bus Routes ☐267 Cut Off Trees ☐268 Clone Graph ☐269 Course Schedule

---

## How to use this document
1. Work section by section, matching your roadmap weeks.
2. For each solved problem above: type it out yourself first, then compare.
3. For unchecked problems in the checklists: they reuse the **same pattern** already explained — solve them without new explanations needed, just apply the pattern.
4. If you get stuck on a specific unlisted problem, ask and I'll add a detailed breakdown for that one.
