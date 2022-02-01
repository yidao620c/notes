# 爬楼梯

假设你正在爬楼梯。需要 n阶你才能到达楼顶。
每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？

注意：给定 n 是一个正整数。

难度系数：★★

## 实现

这是一个典型的动态规划问题。第n个台阶只能从第n-1或者n-2个上来。
到第n-1个台阶的走法 + 第n-2个台阶的走法 = 到第n个台阶的走法，
已经知道了第1个和第2个台阶的走法，一路加上去。

可以回忆一下，是不是跟Fibonacci数列一样？

```python
class Solution:
    def climb_stairs(self, n: int) -> int:
        if n == 1:
            return 1
        if n == 2:
            return 2
        a, b = 1, 2
        for i in range(3, n + 1):
            a, b = b, a + b

        return b


if __name__ == '__main__':
    import sys
    while True:
        line = sys.stdin.readline().strip()
        if line == '':
            break
        print(Solution().climb_stairs(int(line)))
```

