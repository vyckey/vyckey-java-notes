---
title: 单调栈
tags: [algorithm, problem]
sidebar_label: 单调栈
sidebar_position: 10
---

# 单调栈

## 一维接雨水

1. 暴力求解方案：

```java
public class Solution {
    public static void main(String[] args) {
        List<Integer> heights = Lists.newArrayList(2, 1, 4, 2, 1, 1, 3, 2, 5, 1, 3, 1, 2);
        int volume = calVolume(heights);
        System.out.println(volume);
    }
    
    /**
     * 设f(i)为第i列的水柱高度，则有
     * f(i) = min(max(h[0..i-1]), max(h[i,...]))-h[i], f(i)=max(f(i), 0)
     */
    private int calVolume(List<Integer> heights) {
        if (heights.size() <= 2) {
            return 0;
        }
        // 使用maxHeights缓存左右的最大高度，可降低复杂度到O(n)
        // int[][] maxHeights = initMaxHeights(heights);

        int[] water = new int[heights.size()];
        water[0] = 0;
        water[heights.size() - 1] = 0;
        for (int i = 1; i < heights.size() - 1; i++) {
            // int lh = maxHeights[0][i - 1];
            // int rh = maxHeights[1][i + 1];
            int lh = maxHeight(heights, 0, i);
            int rh = maxHeight(heights, i, heights.size());
            water[i] = (Math.max(Math.min(lh, rh) - heights.get(i), 0));
        }
        return Arrays.stream(water).sum();
    }

    private int maxHeight(List<Integer> heights, int l, int r) {
        int max = 0;
        for (int i = l; i < r; i++) {
            max = Math.max(max, heights.get(i));
        }
        return max;
    }

    /**
     * 创建max_heights[2][size]，max_heights[0][j]表示第j个位置左边最大高度，max_heights[1][j]表示第j个位置右边最大高度
     * max_heights[0][j+1] = max(max_heights[0][j], heights[j+1])
     * max_heights[1][j-1] = max(max_heights[1][j], heights[j-1])
     */
    private int[][] initMaxHeights(List<Integer> heights) {
        int[][] maxHeights = new int[2][heights.size()];
        maxHeights[0][0] = heights.get(0);
        maxHeights[1][heights.size() - 1] = heights.get(heights.size() - 1);
        for (int j = 0; j < heights.size() - 1; j++) {
            int k = heights.size() - j - 1;
            maxHeights[0][j + 1] = Math.max(maxHeights[0][j], heights.get(j + 1));
            maxHeights[1][k - 1] = Math.max(maxHeights[1][k], heights.get(k - 1));
        }
        return maxHeights;
    }
}
```

2. 使用双指针：

```java
public class Solution {
    public static void main(String[] args) {
        List<Integer> heights = Lists.newArrayList(2, 1, 4, 2, 1, 1, 3, 2, 5, 1, 3, 1, 2);
        int volume = calVolume(heights);
        System.out.println(volume);
    }

    private int calVolume(List<Integer> heights) {
        if (heights.size() <= 2) {
            return 0;
        }

        int left = 0, right = heights.size() - 1;
        int leftMaxHeight = heights.get(0), rightMaxHeight = heights.get(heights.size() - 1);
        int volume = 0;
        while (left <= right) {
            // 一边遍历一遍计算最大高度
            leftMaxHeight = Math.max(leftMaxHeight, heights.get(left));
            rightMaxHeight = Math.max(rightMaxHeight, heights.get(right));

            if (leftMaxHeight < rightMaxHeight) {
                volume += (leftMaxHeight - heights.get(left));
                left++;
            } else {
                volume += (rightMaxHeight - heights.get(right));
                right--;
            }
        }
        return volume;
    }
}
```

3. 使用单调栈：

```java
public class Solution {
    public static void main(String[] args) {
        List<Integer> heights = Lists.newArrayList(2, 1, 4, 2, 1, 1, 3, 2, 5, 1, 3, 1, 2);
        int volume = calVolume(heights);
        System.out.println(volume);
    }

    /**
     * Stack[..., prev, current] <==push== i ?(heights[current] > heights[i])
     * or S = w * h = (i-prev-1) * (max(heights[prev],heights[i])-heights[current])
     */ 
    private int calVolume(List<Integer> heights) {
        if (heights.size() <= 2) {
            return 0;
        }

        int volume = 0;
        // 单调递减
        Deque<Integer> deque = new ArrayDeque<>();
        for (int i = 0; i < heights.size(); i++) {
            Integer height = heights.get(i);
            // 保持单调递减
            while (!deque.isEmpty() && heights.get(deque.peek()) <= height) {
                int cur = deque.pop();
                if (deque.isEmpty()) {
                    break;
                }
                int prev = deque.peek();
                int w = i - prev - 1;
                int h = Math.min(heights.get(prev), height) - heights.get(cur);
                volume += w * h;
            }
            deque.push(i);
        }
        return volume;
    }
}
```

# 参考资料

