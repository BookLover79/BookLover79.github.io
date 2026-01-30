---
title: Leetcode Hot100(简单中等题)
date: 2024-12-06
categories: 算法题
tags: [leetcode, hot100]

---



# Leetcode Hot 100(简单中等题)

## 哈希

### 1. 两数之和

```python
# 总结:
# 1. 输出下标，需要i循环
# 2. 判断第二个元素是否存在：从i+1开始
# 3. 找第二个元素的dix：从i+1开始
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        for i in range(len(nums)):
            f = target - nums[i]
            if f in nums[i + 1:]:  # 注意i + 1
                return ([i, nums.index(f, i + 1)])  # 这里也是i + 1
```

### 2. 字母异位词分组

```python
# 总结：
# 对字母排序，一样的就是异位词
# 用字典，key是排序后结果，value是之前的（优雅），合理的一对多
class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        d = defaultdict(list)  # 是list
        for s in strs:
            d[''.join(sorted(s))].append(s)
        return (list(d.values()))  # 是values
```

### 3. 最长连续序列

```python
# 总结：
# 1. 求连续的数据，所以可以先去重，加入set
# 2. 外层求全部长度最大值lth，内层cur求当前的长度
class Solution:
    def longestConsecutive(self, nums: List[int]) -> int:
        nums_set = set(nums)  # 最长连续所以要set，因为直接+1判断是否in，所以不需要sorted
        lth = 0

        for nums in nums_set:
            if nums - 1 in nums_set:
                continue
            cur_nums = nums
            cur_lth = 1  # 加入第一个
            while cur_nums + 1 in nums_set:
                cur_lth += 1
                cur_nums += 1
            lth = max(lth, cur_lth)
        return lth
```

## 双指针

### 4. 移动零

```python
class Solution:
    def moveZeroes(self, nums: List[int]) -> None:
        n = len(nums)
        left = 0
        for right in range(n):
            if nums[right] != 0:  # 把非0的丢前面去
                nums[left], nums[right] = nums[right], nums[left]
                left += 1
        return nums
```

### 5. 盛最多水的容器

```python
class Solution:
    def maxArea(self, height: List[int]) -> int:
        n = len(height)
        left, right = 0, n - 1
        ans = 0
        while left < right:  # <=的时候，=的面积为0
            area = (right - left) * min(height[left], height[right])
            ans = max(ans, area)
            if height[left] < height[right]:
                left += 1
            else:
                right -= 1
        return ans
```

### 6. 三数之和

```python
# 总结
# 1. 首先要对nums 进行排序，有了单调性再指针
# 2. 对于第一个指针，需要循环去重
# 3. 对于判断成功的，需要循环去重
class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        nums = sorted(nums)
        n = len(nums)
        ans = []
        for i in range(n):
            if i > 0 and nums[i] == nums[i-1]:  # 循环第一个数，不要和之前的相等
                continue
            left = i + 1
            right = n - 1
            while left < right:  # 俩个指针不能相遇
                value = nums[i] + nums[left] + nums[right]
                if value == 0:
                    ans.append([nums[i], nums[left], nums[right]])
                    while left < right and nums[left] == nums[left + 1]:  # left的右边肯定右right
                        left += 1
                    while left < right and nums[right] == nums[right - 1]:  # right的左边肯定还有left
                        right -= 1
                    left += 1  # ！！注意了，添加完，俩个指针都要动
                    right -= 1
                elif value < 0:
                    left += 1
                else:
                    right -= 1
        return (ans)
```

### 7. 接雨水

```python
# 总结：
# 2. 用双指针可以边记录边计算
# 1. 记录前后缀，只记录最大值is ok
height = [0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1]
# 输出：6

# 双指针 1125
n = len(height)
pre_fix = suf_fix = 0
left, right = 0, n - 1
ans = 0
while left <= right:
    pre_fix = max(pre_fix, height[left])
    suf_fix = max(suf_fix, height[right])
    if pre_fix < suf_fix:
        ans += pre_fix - height[left]
        left += 1
    else:
        ans += suf_fix - height[right]
        right -= 1
print(ans)
```

## 滑动窗口

### 8. 无重复字符的最长子串

```python
# 总结
# 1. 用set判断是否有重复元素
# 2. left和right都从左边开始，left固定+1可以用for循环
# 3. right滑动要注意范围不能超过n
# 4. occ随着窗口变化，最大值单独计算
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        n = len(s)
        occ = set()
        ans = 0
        right = -1
        for left in range(n):
            if left > 0:  # 只要不是第一个
                occ.remove(s[left - 1])  # 滑动窗口右移，删掉左边
            while right + 1 < n and s[right + 1] not in occ:  #！！这个是while
                occ.add(s[right + 1])  # 滑动窗口右移，加入右边
                right += 1  # ！！ 只要右while，就要右+=1
            ans = max(ans, right - left + 1)
        return (ans)
```

### 9. 找到字符串中所有字母异位词

```python
class Solution:
    def findAnagrams(self, s: str, p: str) -> List[int]:
        len_s, len_p = len(s), len(p)
        count_s, count_p = [0] * 26, [0] * 26
        ans = []
        if len_s < len_p:
            return []
        for i in range(len_p):
            count_s[ord(s[i]) - 97] += 1
            count_p[ord(p[i]) - 97] += 1
        if count_p == count_s:
            ans.append(0)
        for i in range(len_s - len_p):
            count_s[ord(s[i]) - 97] -= 1
            count_s[ord(s[i + len_p]) - 97] += 1
            if count_p == count_s:
                ans.append(i + 1)
        return ans
```

## 子串

### 10. 和为 K 的子数组

```python
class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        s = [0] * (len(nums) + 1)  # 0也要加进来，帮助运算
        for i, x in enumerate(nums):
            s[i + 1] = s[i] + x # 前缀和

        ans = 0
        cnt = defaultdict(int)
        for sj in s:
            ans += cnt[sj - k]  # ans必须在前
            cnt[sj] += 1  # 因为s多了一个0，如果k是0会多算一个
        return (ans)
```

## 普通数组

### 11. 最大子数组和

```python
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        n = len(nums)
        pre = ans = nums[0]  # pre主要看大于零i小于零
                             # 这里赋了0，所以下面从1开始
        for i in range(1, n):
            pre = max(nums[i], nums[i] + pre)  # 因为要求连续，所以pre为负数，直接弃了，当前的nums肯定要取
            ans = max(ans, pre)
        return (ans)
```

### 12. 合并区间

```python
class Solution:
    def merge(self, intervals: List[List[int]]) -> List[List[int]]:
        intervals.sort(key=lambda p: p[0]) # 对左端点排序
        ans = []
        for p in intervals:
            if ans and p[0] <= ans[-1][1]: # 判断左端点可否合并
                                           # 1. ans表是存在上一个端点
                                           # 2. <=表示可以合并
                ans[-1][1] = max(ans[-1][1], p[1])   # 合并更新右端点
            else:
                ans.append(p)
        return (ans)
```

### 13. 轮转数组

```python
class Solution:
    def rotate(self, nums: List[int], k: int) -> None:
        def reverse(i, j):
            while i < j:
                nums[i], nums[j] = nums[j], nums[i]
                i += 1
                j -= 1
        n = len(nums)
        k %= n  # 这个相当重要，不要做无谓的浪费
        reverse(0, n - 1)
        reverse(0, k - 1)
        reverse(k, n - 1)
        return (nums)
```

### 14. 除自身以外数组的乘积

```python
class Solution:
    def productExceptSelf(self, nums: List[int]) -> List[int]:
        n = len(nums)
        l, r, ans = [0] * n, [0] * n, [0] * n  # 必须分来初始化！
        l[0] = r[-1] = 1  # 初始化乘法从1开始

        for i in range(1, n): # 从1开始
            l[i] = l[i - 1] * nums[i - 1]  # 后面俩个是一致的

        for i in range(n - 1, 0, -1):  # 到1结束
            r[i - 1] = r[i] * nums[i]

        for i in range(n):  # 所有都循环
            ans[i] = l[i] * r[i]

        return (ans)
```

## 矩阵

### 15. 矩阵置零

```python
class Solution:
    def setZeroes(self, matrix: List[List[int]]) -> None:
        m, n = len(matrix), len(matrix[0])

        row, col = [False] * m, [False] * n  # 俩个一维的来指引

        for i in range(m):
            for j in range(n):
                if matrix[i][j] == 0:
                    row[i] = True
                    col[j] = True

        for i in range(m):
            for j in range(n):
                if row[i] or col[j]:
                    matrix[i][j] = 0
```

### 16. 螺旋矩阵

```python
class Solution:
    def spiralOrder(self, matrix: List[List[int]]) -> List[int]:
        l, r, t, b = 0, len(matrix[0]) - 1, 0, len(matrix) - 1 # !!前面是matrix[0]
        res = []
        while True:
            for i in range(l, r + 1): res.append(matrix[t][i]) # 后面的固定值改变
            t += 1
            if t > b: break

            for i in range(t, b + 1): res.append(matrix[i][r])
            r -= 1
            if l > r: break

            for i in range(r, l - 1, -1): res.append(matrix[b][i])
            b -= 1
            if t > b: break

            for i in range(b, t - 1, -1): res.append(matrix[i][l])
            l += 1
            if l > r: break
        return res
```

### 17. 旋转图像

```python
class Solution:
    def rotate(self, matrix: List[List[int]]) -> None:
        n = len(matrix)
        for i in range(n // 2):
            for j in range((n + 1) // 2):  # 因为是奇偶数，所以是n+1再除以2
                tmp = matrix[i][j]
                matrix[i][j] = matrix[n - 1 - j][i]
                matrix[n - 1 - j][i] = matrix[n - 1 - i][n - 1 - j]
                matrix[n - 1- i][n - 1 - j] = matrix[j][n - 1 - i]
                matrix[j][n - 1 - i] = tmp
```

### 18. 搜索二维矩阵 II

```python
class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        for row in matrix:
            idx = bisect.bisect_left(row, target)
            if idx < len(row) and row[idx] == target:
                return True
        return False
```

## 链表

### 19. 相交链表

```python
class Solution:
    def getIntersectionNode(self, headA: ListNode, headB: ListNode) -> Optional[ListNode]:
        A, B = headA, headB
        while A != B:
            A = A.next if A else headB
            B = B.next if B else headA
        return A  # 如果不相交，走A+B长度就变成null了
```

### 20. 反转链表

```python
class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        cur = head
        pre = None
        while cur:
            nxt = cur.next
            cur.next = pre
            pre = cur
            cur = nxt
        return pre
```

### 21. 回文链表

```python
class Solution:
    def middleNode(self, head:Optional[ListNode]) -> Optional[ListNode]:
        show = fast = head
        while fast and fast.next:
            show = show.next
            fast = fast.next.next
        return show

    def reverseList(self, head:Optional[ListNode]) -> Optional[ListNode]:
        pre = None
        cur = head
        while cur:
            nxt = cur.next
            cur.next = pre
            pre = cur
            cur = nxt
        return pre

    def isPalindrome(self, head: Optional[ListNode]) -> bool:
        mid = self.middleNode(head)
        head2 = self.reverseList(mid)

        while head2:  # head2更短，直接跟着他走
            if head.val != head2.val:  # 先判断，再走
                return False
            head = head.next
            head2 = head2.next
        return True
```

### 22. 环形链表

```python
class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        show = fast = head
        while fast and fast.next:
            show = show.next
            fast = fast.next.next
            if show == fast:
                return True
        return False
```

### 23. 环形链表 II

```python
class Solution:
    def detectCycle(self, head: Optional[ListNode]) -> Optional[ListNode]:
        fast, show = head, head
        while True:
            if not (fast and fast.next): return #存在空的情况，返回空
            fast = fast.next.next
            show = show.next
            if fast == show:
                break
        fast = head
        while fast != show:
            fast = fast.next
            show = show.next
        return fast
```

### 24. 合并两个有序链表

```python
class Solution:
    def mergeTwoLists(self, list1: Optional[ListNode], list2: Optional[ListNode]) -> Optional[ListNode]:
        cur = dum = ListNode(0)
        while list1 and list2:
            if list1.val <= list2.val:
                cur.next, list1 = list1, list1.next
            else:
                cur.next, list2 = list2, list2.next
            cur = cur.next #重点
        cur.next = list1 if list1 else list2
        return dum.next
```

### 25. 两数相加

````python
class Solution:
    def addTwoNumbers(self, l1: Optional[ListNode], l2: Optional[ListNode]) -> Optional[ListNode]:
        cur = dum = ListNode(0)
        carry = 0
        while l1 or l2 or carry:
            if l1:
                carry += l1.val
                l1 = l1.next
            if l2:
                carry += l2.val
                l2 = l2. next
            cur.next = ListNode(carry % 10)
            carry //= 10
            cur = cur.next
        return dum.next
````

### 26. 删除链表的倒数第 N 个结点

```python
class Solution:
    def removeNthFromEnd(self, head: Optional[ListNode], n: int) -> Optional[ListNode]:
        left = right = dum = ListNode(next = head)  # ！！多一个哨兵
        for _ in range(n):
            right = right.next
        while right.next:  # 少掉哨兵这个
            left = left.next
            right = right.next
        left.next = left.next.next  # 也是要跳过哨兵：修改的next指针，left是node
        return dum.next
```

### 27. 两两交换链表中的节点

```python
class Solution:
    def swapPairs(self, head: Optional[ListNode]) -> Optional[ListNode]:
        node0 = dum = ListNode(next=head)
        node1 = head
        while node1 and node1.next: #确保总结俩个有，前后可以为null
            node2 = node1.next
            node3 = node2.next

            node0.next = node2 #注意：前面是next，修改的是指向的指针
            node2.next = node1
            node1.next = node3

            node0 = node1
            node1 = node3

        return dum.next
```

### 28. 随机链表的复制

```python
class Solution:
    def copyRandomList(self, head: 'Optional[Node]') -> 'Optional[Node]':
        if not head: return  # 先定义边界情况
        dic = {}  # defaultdict很难给参数
        cur = head
        while cur:
            dic[cur] = Node(cur.val)
            cur = cur.next
        cur = head
        while cur:
            dic[cur].next = dic.get(cur.next)
            dic[cur].random = dic.get(cur.random)
            cur = cur.next  # 一定要记住，这咋总忘
        return dic[head]
```

### 29. 排序链表

```python
# 归并排序
class Solution:
    def sortList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        if not head or not head.next: return head # 只有一个
        show, fast = head, head.next # ！切开了
        while fast and fast.next:
            show = show.next
            fast = fast.next.next
        mid = show.next
        show.next = None #分割成俩部分

        left, right = self.sortList(head), self.sortList(mid) # 归并排序

        # 变成排序序列部分
        # 重新放到h上去，res是tmp
        h = res = ListNode(0)
        while left and right:
            if left.val < right.val:
                h.next, left = left, left.next
            else:
                h.next, right = right, right.next
            h = h.next
        h.next = left if left else right
        return res.next
```

### 30. LRU 缓存

```python
class Node:
    __slots__ = 'prev', 'next', 'key', 'value'

    def __init__(self, key=0, value=0):
        self.key = key
        self.value = value

class LRUCache:

    def __init__(self, capacity: int):
        self.capacity = capacity
        self.dummy = Node()
        self.dummy.prev = self.dummy
        self.dummy.next = self.dummy
        self.key_to_node = dict()

    def remove(self, x: Node) -> Node:
        x.prev.next = x.next  # 删掉一个
        x.next.prev = x.prev

    def push_front(self, x: Node) -> Node:
        x.prev = self.dummy
        x.next = self.dummy.next  # next!!
        x.prev.next = x
        x.next.prev = x

    def get_node(self, key: int) -> Optional[Node]:
        if key not in self.key_to_node:
            return None
        node = self.key_to_node[key]
        self.remove(node)
        self.push_front(node)
        return node

    def get(self, key: int) -> int:
        node = self.get_node(key)
        return node.value if node else -1

    def put(self, key: int, value: int) -> None:
        node = self.get_node(key)
        if node:
            node.value = value
            return
        self.key_to_node[key] = node = Node(key, value)
        self.push_front(node)
        if len(self.key_to_node) > self.capacity:
            back_node = self.dummy.prev
            del self.key_to_node[back_node.key]
            self.remove(back_node)
```

## 二叉树

### 31. 二叉树的中序遍历

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def inorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        WHITE, GRAY = 0, 1
        stack = [(WHITE, root)]
        res = []
        while stack:  # 不需要dfs，因为while了
            color, node = stack.pop()
            if node is None: continue  # 这里判断了None！之前无需处理
            if color == WHITE:
                stack.append((WHITE, node.right))
                stack.append((GRAY, node))
                stack.append((WHITE, node.left))
            else:
                res.append(node.val)
        return res
```

### 32. 二叉树的最大深度

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        if root is None:
            return 0
        l_depth = self.maxDepth(root.left)
        r_depth = self.maxDepth(root.right)
        return max(l_depth, r_depth) +1
```

### 33. 翻转二叉树

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def invertTree(self, root: Optional[TreeNode]) -> Optional[TreeNode]:
        if root is None:
            return None #主要是为了终止，有啥返回啥
        left = self.invertTree(root.left)
        right = self.invertTree(root.right)
        root.left = right #交换在呢
        root.right = left
        return root #交换完返回
```

### 34. 对称二叉树

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def isSymmetric(self, root: Optional[TreeNode]) -> bool:
        return self.isSameTrue(root.left, root.right)

    def isSameTrue(self, p:Optional[TreeNode], q:Optional[TreeNode]) -> bool:
        if p is None or q is None:
            return p is q  # 用==也可以；==判断的是值；is判断的是指针
        return p.val == q.val and self.isSameTrue(p.left, q.right) and self.isSameTrue(p.right, q.left)
```

### 35. 二叉树的直径

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def diameterOfBinaryTree(self, root: Optional[TreeNode]) -> int:
        ans = 0
        def dfs(node:Optional[TreeNode]) ->int:
            if node is None:
                return -1  # 减去最后一个node

            l_len = dfs(node.left) + 1
            r_len = dfs(node.right) + 1
            nonlocal ans
            ans = max(ans, l_len + r_len)
            return max(l_len, r_len)
        dfs(root)
        return ans
```

### 36. 二叉树的层序遍历

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if root is None:
            return []  # 没有就返回空

        q = deque([root]) # 先把root加进来
        ans = []

        while q:
            vals=[]
            for _ in range(len(q)):
                node = q.popleft()
                vals.append(node.val)  # 不用判断node是否有值？因为root判断过了，后面加入的时候判断过了
                if node.left : q.append(node.left)
                if node.right :q.append(node.right)
            ans.append(vals)
        return ans
```

### 37. 将有序数组转换为二叉搜索树

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def sortedArrayToBST(self, nums: List[int]) -> Optional[TreeNode]:
        if not nums:
            return None  # 空树就是None
        m = len(nums) // 2
        return TreeNode(nums[m], self.sortedArrayToBST(nums[:m]), self.sortedArrayToBST(nums[m + 1:])) #cool
```

### 38. 验证二叉搜索树

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def isValidBST(self, root: Optional[TreeNode], left = -inf, right = inf) -> bool:  # ！！还修改了参数
        if root is None:
            return True
        x = root.val
        return left < x < right and self.isValidBST(root.left, left, x) and self.isValidBST(root.right, x, right)
```

### 39. 二叉搜索树中第 K 小的元素

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def kthSmallest(self, root: Optional[TreeNode], k: int) -> int:
        def dfs(node: Optional[TreeNode]):
            if node is None:
                return -1  # 找不到设定为-1
            left_res = dfs(node.left)
            if left_res != -1:  # 精髓，这个值都可以，随便拿来判断的
                return left_res
            nonlocal k
            k -= 1
            if k == 0:  # 很nice的想法
                return node.val
            return dfs(node.right)
        return dfs(root)  # 这里没要ans，直接return就行
```

### 40. 二叉树的右视图

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def rightSideView(self, root: Optional[TreeNode]) -> List[int]:
        ans = []
        def dfs(node: Optional[TreeNode], depth:int) -> None:
            if node is None:  # 对于空的不用操作
                return
            if depth == len(ans):     # 因为depth + 1，所以无需减一
                ans.append(node.val)  # 绝妙，一层就一个
                                      # 注意不是return是apend，一层加一个
            dfs(node.right, depth + 1)
            dfs(node.left, depth + 1)
        dfs(root, 0)
        return ans
```

### 41. 二叉树展开为链表

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    head = None

    def flatten(self, root: Optional[TreeNode]) -> None:
        if root is None:
            return  # 链表也不需要什么返回值
        self.flatten(root.right)  # 先展开右边，
        self.flatten(root.left)
        root.left = None                # 都是在root上做改变，依次加入
        root.right = self.head
        self.head = root                # 迭代
```

### 42. 从前序与中序遍历序列构造二叉树

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        if not preorder: # preorder 是列表，所以是not表示为空
            return None
        left_size = inorder.index(preorder[0])
        left = self.buildTree(preorder[1 : 1 + left_size], inorder[: left_size])
        right = self.buildTree(preorder[1 + left_size:], inorder[1 + left_size: ])

        return TreeNode(preorder[0], left, right)
```

### 43. 路径总和 III

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def pathSum(self, root: Optional[TreeNode], targetSum: int) -> int:
        ans = 0
        cnt = defaultdict(int)
        cnt[0] = 1  # ！！0 一定要加进来
        def dfs(node:Optional[TreeNode], s:int)->None:
            if node is None:
                return
            nonlocal ans
            s += node.val  # s是前缀和，加上所有val
            ans += cnt[s - targetSum]  # 前缀和 - k

            cnt[s] += 1  # 前缀和加一
            dfs(node.left, s)
            dfs(node.right, s)
            cnt[s] -= 1  # 回复现场，走完left，走right要恢复
                         # s保存在参数里面了，ans就加所以都不需要保护

        dfs(root, 0)
        return ans
```

### 44. 二叉树的最近公共祖先

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
    def lowestCommonAncestor(self, root: 'TreeNode', p: 'TreeNode', q: 'TreeNode') -> 'TreeNode':
        if root in (None, p, q):
            return root
        left = self.lowestCommonAncestor(root.left, p, q)
        right = self.lowestCommonAncestor(root.right, p, q)
        if left and right: #都有，返回公共祖先
            return root
        return left or right #在一个分支，返回该分支
```

## 图论

### 45. 岛屿数量

```python
class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        m, n = len(grid), len(grid[0])

        def dfs(i: int, j: int) -> None:
            if i < 0 or i >= m or j < 0 or j >= n or grid[i][j] != '1':  # 一定是字符串
                return
            grid[i][j] = '2'  # 已访问过给换个数，就不会再访问了
            dfs(i - 1, j)
            dfs(i + 1, j)
            dfs(i, j - 1)
            dfs(i, j + 1)

        ans = 0
        for i, row in enumerate(grid):  # 有意思的双重enumerate循环
            for j, c in enumerate(row):
                if c == '1':
                    dfs(i, j)
                    ans += 1
        return ans
```

### 46. 腐烂的橘子

```python
class Solution:
    def orangesRotting(self, grid: List[List[int]]) -> int:
        row, col = len(grid), len(grid[0])
        rotten = {(i, j) for i in range(row) for j in range(col) if grid[i][j] == 2}  # 优雅！
        fresh = {(i, j) for i in range(row) for j in range(col) if grid[i][j] == 1}
        time = 0
        while fresh:
            if not rotten: return -1  # 边界情况，不可能腐烂所有橘子
            rotten = {(i + di, j + dj) for i, j in rotten for di, dj in [(0, 1), (0, -1), (1, 0), (-1, 0)] if (i + di, j + dj) in fresh}
            # 一定要加[]
            # roten前面不用加enumerate，它会多加一个idx维度，并不需要，直接取is ok
            fresh -= rotten
            time += 1
        return time
```

### 47. 课程表

```python
class Solution:
    def canFinish(self, numCourses: int, prerequisites: List[List[int]]) -> bool:
        indegrees = [0 for _ in range(numCourses)] # 入度
        adjacency = [[] for _ in range(numCourses)] # 邻接表
        # [0, 0, 0, 0, 0, 0, 0, 0, 0, 0][[], [], [], [], [], [], [], [], [], []]

        # indegrees = [0] * numCourses   # 入度
        # adjacency = [[] * numCourses]  # 邻接表
        # [0, 0, 0, 0, 0, 0, 0, 0, 0, 0][[]]  # 一定要for循环，直接*就没了

        queue = deque()  # defaultdict是字典，deque()是双端队列

        for cur, pre in prerequisites:  # 是in，不是enumerate
            indegrees[cur] += 1 #构建入度list
            adjacency[pre].append(cur)  # 因为是列表，所以是append

        for i in range(len(indegrees)):
            if not indegrees[i]: queue.append(i) #入度为0开始删除

        while queue:
            pre = queue.popleft() #取出来
            numCourses -= 1 #！！课程删掉（接下来就回删完掉）
            for cur in adjacency[pre]: #取出该课程对应的所有后面的课程
                indegrees[cur] -= 1 #删除邻接
                if not indegrees[cur]: queue.append(cur) #删掉后，把新得到的入度为0的加入
        return not numCourses
```

### 48. 实现 Trie (前缀树)

```python
class Node():
    def __init__(self):
        self.son = {}
        self.end = False

class Trie:
    def __init__(self):
        self.root = Node()

    def insert(self, word: str) -> None:
        cur = self.root
        for c in word:
            if c not in cur.son:
                cur.son[c] = Node()
            cur = cur.son[c]
        cur.end = True

    def find(self, word: str) -> int:
        cur = self.root
        for c in word:
            if c not in cur.son:
                return 0
            cur = cur.son[c]
        return 2 if cur.end else 1

    def search(self, word: str) -> bool:
        return self.find(word) == 2

    def startsWith(self, prefix: str) -> bool:
        return self.find(prefix) != 0
```

## 回溯

### 49. 全排列

```python
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        n = len(nums)
        ans = []
        def dfs(i):  # i是前面多少个固定了
            if i == n - 1: # 也可以是n，但是自己和自己无需改变，下面的循环不会执行
                ans.append(nums.copy())  # 不同的nums加入res
                return
            for j in range(i, n):
                nums[i], nums[j] = nums[j], nums[i]  # 只交换后面的，前面的固定了
                dfs(i + 1)  # ！！变i
                nums[i], nums[j] = nums[j], nums[i]
        dfs(0)
        return ans
```

### 50. 子集

```python
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        ans = []
        path = []
        n = len(nums)
        def dfs(i):
            ans.append(path.copy())
            for j in range(i, n):
                path.append(nums[j])
                dfs(j + 1)
                path.pop()
        dfs(0)
        return ans
```

### 51. 电话号码的字母组合

```python
MAPPING = "", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"
class Solution:
    def letterCombinations(self, digits: str) -> List[str]:
        n = len(digits)
        if n == 0:  # 需要处理一下，不然加入""
            return []
        ans = []
        path = [''] * n  # 一定是n个
        def dfs(i):
            if i == n:  # 0到n-1次（n个）加入字符完成！在n次只是append
                ans.append(''.join(path))  # 变成字符串了，不用copy了
                return # 记得return
            for c in MAPPING[int(digits[i])]:   # i是第i个数字：digits[i]
                path[i] = c
                dfs(i + 1)
        dfs(0)
        return ans
```

### 52. 组合总和

```python
class Solution:
    def combinationSum(self, candidates: List[int], target: int) -> List[List[int]]:
        ans = []
        path = []
        def dfs(i, left):
            if left == 0:  # 合理的情况
                ans.append(path.copy())  # 不copy结果会为最后清空的path：null
                return
            if i == len(candidates) or left < 0:  # 不合理的情况
                return
            dfs(i + 1, left)  # 不选当前的candidate

            path.append(candidates[i])
            dfs(i, left - candidates[i])  # 选当前的candidate
            path.pop()
        dfs(0, target)
        return ans
```

### 53. 括号生成

```python
class Solution:
    def generateParenthesis(self, n: int) -> List[str]:
        m = n * 2  # 不一样的地方，俩个double一下
        ans = []
        path = [''] * m
        def dfs(i, open):
            if i == m:
                ans.append(''.join(path))
            if open < n:  # ！！限制左括号最多填写n个
                path[i] = '('
                dfs(i + 1, open + 1)
            if i - open < open:  # 限制右括号最多填n个
                path[i] = ")"
                dfs(i + 1, open)  # open不用+1了
        dfs(0, 0)
        return ans
```

### 54. 单词搜索

```python
class Solution:
    def exist(self, board: List[List[str]], word: str) -> bool:
        m, n = len(board), len(board[0])  # 不需要给其他空间
        def dfs(i, j, k):
            if board[i][j] != word[k]:  # 失败的情况
                return False
            if k == len(word) - 1:  # 最后一个也成功，就成功，这个必须在上面的下面
                return True
            # 上面这俩个一定是先判断最后一个，加入，再判断成功；为了防止只有一个的情况，下面就需要走动了。
            board[i][j] = ""  # 访问过就置空，和恢复一起写
            for x, y in (i + 1, j), (i - 1, j), (i, j + 1), (i, j - 1): #这里取不需要加[]
                if 0 <= x < m and 0 <= y < n and dfs(x, y, k + 1): # 搜到多条路径，k+1是ok，k也是ok？
                    return True
            board[i][j] = word[k]  # 还原
        return any(dfs(i, j, 0) for i in range(m) for j in range(n))  # k是0开始走，循环所有起点
                                                                      # 不用[]，一个就好，用any()包着
```

### 55. 分割回文串

```python
class Solution:
    def partition(self, s: str) -> List[List[str]]:
        n = len(s)
        path = []
        ans = []
        def dfs(i):  # 分割到了长度i
            if i == n:  # 实际为n - 1，但是调入循环会加1，所以判断n
                ans.append(path.copy())
                return
            for j in range(i, n):  # 继续往后分割
                t = s[i: j + 1]  # i已经是了，判断i-n的，以j为分割点，判断i:j是不是
                if t == t[::-1]:
                    path.append(t)
                    dfs(j + 1)  # 这里的+1和上面的+1都是取不到的
                    path.pop()
        dfs(0)
        return ans
```

## 二分查找

### 56. 搜索插入位置

```python
# 总结
# 1. 初始化 +1/-1
# 2. while <= , < , + 1 <
# 3. mid +1/-1
# 4. 返回值：left、left/right、right

# 闭区间写法 [left, right]
def lower_bound(nums: List[int], target: int) -> int:
    left, right = 0, len(nums) - 1
    while left <= right:
        mid = (left + right) // 2
        if nums[mid] < target:  # 三个都是小于
            left = mid + 1
        else:
            right = mid - 1
    return left

# 左闭右开区间写法 [left, right)
def lower_bound2(nums: List[int], target: int) -> int:
    left = 0
    right = len(nums)
    while left < right:  # 区间不为空
        mid = (left + right) // 2
        if nums[mid] < target:
            left = mid + 1  # 范围缩小到 [mid+1, right)
        else:
            right = mid  # 范围缩小到 [left, mid)
    return left  # 或者 right

# 开区间写法 (left, right)
def lower_bound3(nums: List[int], target: int) -> int:
    left, right = -1, len(nums)
    while left + 1 < right:  # 区间不为空
        mid = (left + right) // 2
        if nums[mid] < target:
            left = mid  # 开区间不需要+1/-1
        else:
            right = mid
    return right

class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        return lower_bound(nums, target)
```

### 57. 搜索二维矩阵

```python
class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        m, n = len(matrix), len(matrix[0])
        left, right = -1, m * n  # 还是一样的-1
        while left + 1 < right:
            mid = (left + right) // 2
            x = matrix[mid // n][mid % n]  # 除以列，得到行
            if x == target:
                return True
            if x < target:
                left = mid
            else:
                right = mid
        return False
```

### 58. 在排序数组中查找元素的第一个和最后一个位置

```python
class Solution:
    def lower_bound(self, nums: List[int], target: int)->int:
        left, right = 0, len(nums) - 1
        while left <= right:
            mid = (left + right) // 2
            if nums[mid] < target:
                left = mid + 1
            else:
                right = mid - 1
        return left

    def searchRange(self, nums: List[int], target: int) -> List[int]:
        start = self.lower_bound(nums, target)
        if start == len(nums) or nums[start] != target:
            return [-1, -1]
        end = self.lower_bound(nums, target + 1) - 1
        return [start, end]
```

### 59. 搜索旋转排序数组

```python
class Solution:
    def findMin(self, nums: List[int]):
        left, right = -1, len(nums) - 1  # 最后一个作为比较的target
        while left + 1 < right:
            mid = (left + right) // 2
            if nums[mid] > nums[-1]:  # 二分是 <
                left = mid
            else:
                right = mid
        return right

    def lower_bound(self, nums: List[int], left: int, right: int, target: int) -> int:
        while left + 1 < right:
            mid = (left + right) // 2
            if nums[mid] < target:
                left = mid
            else:
                right = mid
        return right if nums[right] == target else -1  # 非常重要的判断

    def search(self, nums: List[int], target: int) -> int:
        i = self.findMin(nums)
        if target > nums[-1]:
            return self.lower_bound(nums, -1, i, target)
        return self.lower_bound(nums, i - 1, len(nums), target)
```

### 60. 寻找旋转排序数组中的最小值

```python
class Solution:
    def findMin(self, nums: List[int]) -> int:
        left, right = -1, len(nums) - 1
        while left + 1 < right:
            mid = (left + right) // 2
            if nums[mid] > nums[-1]:
                left = mid
            else:
                right = mid
        return nums[right]
```

## 栈

### 61. 有效的括号

```python
class Solution:
    def isValid(self, s: str) -> bool:
        dic = {'(':')', '{':'}', '[':']', '?':'?'}
        stack = ['?']
        for c in s:
            if c in dic:
                stack.append(c)
            elif dic[stack.pop()] != c:  # 直接放里面pop，不对的直接失败了
                return False
        return len(stack) == 1
```

### 62. 最小栈

```python
class MinStack:
    def __init__(self):
        self.stack = []
    def push(self, val: int) -> None:
        if not self.stack:
            self.stack.append((val, val))
        else:
            self.stack.append((val, min(val, self.stack[-1][1])))
    def pop(self) -> None:
        self.stack.pop()
    def top(self) -> int:
        return self.stack[-1][0]
    def getMin(self) -> int:
        return self.stack[-1][1]
```

### 63. 字符串解码

```python
class Solution:
    def decodeString(self, s: str) -> str:
        stack, res, multi = [], "", 0  # res 是保存前面的结果
        for c in s:
            if c == '[':  # 左括号加入当前的，字符一个一个加，一个一个压栈，并且清空当前
                stack.append([multi, res])
                multi, res = 0, ""
            elif c == ']':  # 右括号要进行结算了
                cur_multi, cur_res = stack.pop()  # 必须重命名，不然会把之前的覆盖掉
                res = cur_res + cur_multi * res
            elif '0' <= c <= '9':
                multi = multi * 10 + int(c)
            else:
                res += c
        return res
```

### 64. 每日温度

```python
class Solution:
    def dailyTemperatures(self, temperatures: List[int]) -> List[int]:
        n = len(temperatures)
        ans = [0] * n  # 因为后面后ans[i]，所以这里要提前指明大小
        st = []  # 单调栈
        for i in range(n - 1, -1, -1):  # 从后往前统计
            t = temperatures[i]
            while st and t >= temperatures[st[-1]]:   # 单调是不可以等于的>，所以相反的是<=
                                                      # 注意是while，所有不合法的都抛弃
                st.pop()  # 先判断是否右st，所以就是要pop st
            if st:  # 上面pop完了，可能没有了，还需判断是否
                ans[i] = st[-1] - i
            st.append(i)
        return ans
```

## 堆

### 65. 数组中的第K个最大元素

```python
class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        def maxHepify(arr, i, end):  # 大顶堆[下标0表示堆顶]
                                     # 当前需要调整的节点索引i， end值保证边界
            j = 2 * i + 1  # j为i的左子节点
            while j <= end:  # 自上而下进行调整
                if j + 1 <= end and arr[j + 1] > arr[j]:  # i的左右子节点分别为j和j+1
                    j += 1  # 取两者之间的较大者
                if arr[i] < arr[j]:
                    arr[i], arr[j] = arr[j], arr[i]
                    i = j  # 往下走：i调整为其子节点
                    j = 2 * i + 1  # j调整左子节点
                else:  # 一定要记住break！！
                    break
        # 建堆【大顶堆】
        n = len(nums)
        for i in range(n // 2 - 1, -1, -1):  # 第一个非叶子节点：n//2-1 (-1是因为从坐标0开始)
            maxHepify(nums, i, n - 1)
        for j in range(n - 1, n - k - 1, -1):
            nums[0], nums[j] = nums[j], nums[0]  # 堆顶元素（当前最大值）放置到尾部j
            maxHepify(nums, 0, j - 1)  # j不变，j-1变成尾部，并从堆顶0开始调
        return nums[-k] # 最大的依次放到最后面，最大放到n-1，然后n-2，所以k就是n-k
```

### 66. 前 K 个高频元素

```python
class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        count = collections.Counter(nums) # 注意Counter大写
        heap = [(val, key) for key, val in count.items()]
        return [item[1] for item in heapq.nlargest(k, heap)]  # item[1]是[]，而不是()
                                                              # heapq: python小根堆
                                                              # nlargest,k是返回k个
```

## 贪心算法

### 67. 买卖股票的最佳时机

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        ans = 0
        min_price = prices[0]  # 必须分开来初始化，ans要0
        for p in prices:
            min_price = min(min_price, p)
            ans = max(ans, p - min_price)
        return ans
```

### 68. 跳跃游戏

```python
class Solution:
    def canJump(self, nums: List[int]) -> bool:
        cover = 0
        for i, x in enumerate(nums): # 虽然最后一个nums的值用不到，但是需要用来判断是否出界，所以需要循环到
            if i > cover:  # ！！首先进行判断，然后才更新cover： 出范围了， 跳不到
                return False
            cover = max(cover, i + x)  # 当前格点 + 可跳范围
            if cover >= len(nums) - 1:   # 注意是>=，且len - 1，因为下标从0开始。
                                         # 到最后了
                return True
```

### 69. 跳跃游戏 II

```python
class Solution:
    def jump(self, nums: List[int]) -> int:
        ans = 0
        cur_right = 0 # 已建造的桥的右端点
        next_right = 0  # 下一座桥的右端点的最大值
        for i in range(len(nums) - 1):  # ！！考虑的是是否能到n - 1的位置，所以n - 1的值没啥用，不取最后一个，所以不用enumerate
            next_right = max(next_right, i + nums[i])
            if i == cur_right: # 到达已建造的桥的右端点
                cur_right = next_right # 造一座桥
                ans += 1
        return ans
```

### 70. 划分字母区间

```python
class Solution:
    def partitionLabels(self, s: str) -> List[int]:
        last = {c : i for i, c in enumerate(s)}  # 太酷啦！每个字母最后出现的下标
        print(last)
        start = end = 0
        ans = []
        for i, c in enumerate(s):  # 和last一致
            end = max(end, last[c])  # 更新当前区间右端点的最大值
            if i == end:  # 直到i走到end，在考虑用下一个！！
                ans.append(end - start + 1)  # 区间长度加入答案
                start = i + 1  # 下一区间起点
        return ans
```

## 动态规划

### 71. 爬楼梯

```python
class Solution:
    def climbStairs(self, n: int) -> int:
        f = [0] * (n + 1)              # 一定是n + 1，因为爬n楼，第一层是0和1，虽然0没有，但是放一下
        f[0] = f[1] = 1                # 初始化一定要有
        for i in range(2, n + 1):      # 因为上面有俩，所以从2开始，因为f是n+1，所以这里也到n+1
            f[i] = f[i - 1] + f[i - 2] # 最大范围n+1，这里也可以是n+1，所以直接f[i]
        return f[n]

        # @cache
        # def dfs(i:int) -> int:
        #     if i <= 1:
        #         return 1
        #     return dfs(i - 1) + dfs(i - 2)
        # return dfs(n)
```

### 72. 杨辉三角

```python
class Solution:
    def generate(self, numRows: int) -> List[List[int]]:
        c = [[1] * (i + 1) for i in range(numRows)]  # 总共numRows行，直接放；因为i可能为0，所以i+1
        for i in range(1, numRows):  # 因为i - 1，所以i从1开始
            for j in range(1, i):    # 因为j - 1，所以也从1开始
                c[i][j] = c[i - 1][j - 1] + c[i - 1][j]
        return c
```

### 73. 打家劫舍

```python
class Solution:
    def rob(self, nums: List[int]) -> int:
        f = [0] * (len(nums) + 2) # 所以这里+2
        for i,x in enumerate(nums): # 因为有nums就用enumerate，但是因为有相邻俩屋
            f[i + 2] = max(f[i + 1], f[i] + x)
        return f[-1] # 因为上面去max了，所以max和-1都是一样的

        # @cache
        # def dfs(i:int) -> int:
        #     if i < 0:
        #         return 0
        #     return max( dfs(i - 1), dfs(i - 2) + nums[i])
        # return dfs(len(nums) - 1)
```

### 74. 完全平方数

```python
class Solution:
    def numSquares(self, n: int) -> int:
        f = [[0] * (n + 1) for _ in range(isqrt(n) + 1)]  #需要这个数本身，所以需要+1。
                                                          # i是到了第几个平方数，j是当前的和
        f[0] = [0] + [inf] * n  # 取最小值，所以需要inf
        for i in range(1, len(f)):
            for j in range(n + 1):
                if j < i * i:
                    f[i][j] = f[i - 1][j]  #超出范围了，取上一个
                else:
                    f[i][j] = min(f[i - 1][j], f[i][j - i * i] + 1)  #取上一个或者取当前的值
        return f[isqrt(n)][n]
```

### 75. 零钱兑换

```python
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        n = len(coins)
        f = [[inf] * (amount + 1) for _ in range(n + 1)] # 所以是n + 1
        # 1. 后面这个是行，左边是列
        # 2. 因为要去最小值，就要inf
        # 3. 横轴n的coin是enumerate，所以0，n，
        f[0][0] = 0 # 00这个一定要0

        for i, x in enumerate(coins): # 固定：0,n -> 所以是i + 1
            for c in range(amount + 1):
                if c < x:
                    f[i + 1][c] = f[i][c] # i + 1还是i主要看第一层循环的起始
                else:
                    f[i + 1][c] = min(f[i][c], f[i + 1][c - x] + 1)
        ans = f[n][amount]
        return ans if ans < inf else -1  # ！！！ 对于没有找到的情况要做处理
```

### 76. 单词拆分

```python
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        max_len = max(map(len, wordDict))  # 用于限制下面 j 的循环次数
        words = set(wordDict)  # set减少重复

        n = len(s)
        f = [True] + [False] * n # 这个只有一个维度，因为总共有n个单词，第一个是为了凑数的，表示空
        for i in range(1, n + 1): # 因为n个单词，加上一个空，所以从1到n+1
            for j in range(i - 1, max(i - max_len - 1, -1), -1):  # ！！往前退j个单词，看能否分割
                if f[j] and s[j:i] in words: # f[j]是小单词，加上s[j:i]
                    f[i] = True
                    break # ！！一定要break，跳出了j的循环，不然可能被修改！！
        return f[n] # 不是最大值，给定的固定单词，所以就是最后一个
```

### 77. 最长递增子序列

```python
class Solution: # 子序列不需要连续，子串需要
    def lengthOfLIS(self, nums: List[int]) -> int:
        f = [0] * len(nums) # 注意，是一个一维的，i表示坐标0-(n-1)，所以不需要+1，值表示长度
        for i, x in enumerate(nums):
            for j, y in enumerate(nums[:i]):  #i取最后一个，j往前搜，都是一个东西，用一样的表示
                if x > y: # 最后这个数大一定是大于前面的
                    f[i] = max(f[i], f[j])  # 因为是i和前面的比，所以不是i - 1
            f[i] += 1  # 加入最后这个。继承了之前的最大，在之后再次+1，所以不断找最大
        return max(f)  # ！！不一定是最后一个最大
```

### 78. 乘积最大子数组

```python
class Solution:
    def maxProduct(self, nums: List[int]) -> int:
        n = len(nums)
        f_min = [0] * n  # 一共n个数，n个就可以
        f_max = [0] * n  # 由于负号的存在，同时统计最大最小值
        f_min[0] = f_max[0] = nums[0]  # 记得初始化

        for i in range(1, n):  # 0已经给出来了，从1开始就可以
            x = nums[i]  # 提前取一个
            f_min[i] = min(f_max[i - 1] * x, f_min[i - 1] * x, x) # 三个比大小哦，最大、最小*x和x
            f_max[i] = max(f_max[i - 1] * x, f_min[i - 1] * x, x)
        return max(f_max)
```

### 79. 分割等和子集

```python
class Solution:
    def canPartition(self, nums: List[int]) -> bool:
        s = sum(nums)
        if s % 2: # s是奇数，那不能等分
            return False
        s //= 2  # 注意这里把 s 减半了
        n = len(nums)
        f = [[False] * (s + 1) for _ in range(n + 1)]  # 判断能不能成，是基于上一个，所以+1
                                                    # 能否从 nums[0] 到 nums[i] 中选出一个和恰好等于 j 的子序列。
        f[0][0] = True  # 必须，基于此
        for i, x in enumerate(nums):  # i从0开始，所以i + 1
            for j in range(s + 1):  # s+1是因为//可能掉了1
                f[i + 1][j] = j >= x and f[i][j - x] or f[i][j] # 因为这里只可以选一次，所以都是i
                #第一个是判断还有x的容量，然后就减去x or 不要x
        return f[n][s]
```

## 多维动态规划

### 80. 不同路径

```python
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:
        f = [[1] * n] + [[1] + [0] * (n - 1) for _ in range(m - 1)] # 1. 首行首列为1
                                                                    # 2. 注意第一块儿和第二块都有俩层[[]]
        for i in range(1, m):  # 下面-1, 这里从1开始
            for j in range(1, n):
                f[i][j] = f[i - 1][j] + f[i][j - 1]
        return f[m - 1][n - 1]
```

### 81. 最小路径和

```python
class Solution:
    def minPathSum(self, grid: List[List[int]]) -> int:
        for i in range(len(grid)):
            for j in range(len(grid[0])):
                if i == 0 and j == 0: continue  # 为了美观，循环自动j + 1
                elif i == 0: grid[i][j] += grid[i][j - 1] # i固定不变，所以j - 1
                elif j == 0: grid[i][j] += grid[i - 1][j]
                else: grid[i][j] += min(grid[i][j - 1], grid[i - 1][j]) # 要的是小的那个
        return grid[-1][-1]
```

### 82. 最长回文子串

```python
class Solution:
    def longestPalindrome(self, s: str) -> str:
        n = len(s)
        dp = [[False] * n for _ in range(n)]  # dp是i到j是否是回文
        ans = 1  # ！！最小肯定为1
        idx1 = idx2 = 0  # 结果的左右下标记

        for i in range(n - 1, -1, -1):  # ！！后往前，确保j走的都是i走过的
            for j in range(i, n):  # 把i走过的判断是否是回文
                dp[i][j] = dp[i + 1][j - 1] and s[i] == s[j] if j - i >= 2 else s[i] == s[j]  # 俩个以内判断是否相等；俩个以外，判断中间是以及外面相等
                if dp[i][j] and j - i + 1 > ans:  # 是回文，保存最大的ans
                    ans = j - i + 1
                    idx1, idx2 = i, j
        return s[idx1 : idx2 + 1]
```

### 83. 最长公共子序列

```python
class Solution:
    def longestCommonSubsequence(self, s: str, t: str) -> int:
        n, m = len(s), len(t)
        f = [[0] * (m + 1) for _ in range(n + 1)]  # 给定有值的使用enumerate，所以+1
        for i, x in enumerate(s):
            for j, y in enumerate(t):
                f[i + 1][j + 1] = f[i][j] + 1 if x == y else max(f[i][j + 1], f[i + 1][j])
                # 相等一起减一，否则i-1或者j-1
        return f[n][m]
```

### 84. 编辑距离

```python
class Solution:
    def minDistance(self, s: str, t: str) -> int:
        m, n = len(s), len(t)
        f = [[0] * (n + 1) for _ in range(m + 1)]
        f[0] = list(range(n + 1))  #这个和第一个n是一样的！！
                                   # 因为取min，前面肯定有一个初始化，最大的情况
                                   # 1到m，假设每一步都需要改变
        for i, x in enumerate(s):
            f[i + 1][0] = i + 1  # 注意是i从0-n，所以是i+1：第一列，假设每一步都需要改变
            for j, y in enumerate(t):
                f[i + 1][j + 1] = f[i][j] if x == y else min(f[i][j + 1], f[i + 1][j], f[i][j]) + 1
                # 相等不用变，单独处理
                # 删除、插入、替换都要做一次操作+1
        return f[n][m]
```

## 技巧

### 85. 只出现一次的数字

```python
class Solution:
    def singleNumber(self, nums: List[int]) -> int:
        x = 0
        for num in nums:
            x ^= num
        return x
```

### 86. 多数元素

```python
import collections
# from collections import List
class Solution:
    def majorityElement(self, nums: List[int]) -> int:
        counters = collections.Counter(nums)
        return max(counters.keys(), key=counters.get)
                        # 前面是keys()， 后面是key
                        # get是在counters中的值(即出现的次数)
```

### 87. 颜色分类

```python
class Solution:
    def sortColors(self, nums: List[int]) -> None:
        def swap(nums, idx1, idx2):
            nums[idx1], nums[idx2] = nums[idx2], nums[idx1]

        n = len(nums)
        if n < 2: return  # 小于2就不需要比较了

        left, i, right = 0, 0, n  # n是这样
        while i < right:  # 注意< + 注意是i，i和right临近啊
            if nums[i] == 0:
                swap(nums, left, i)
                left += 1
                i += 1
            elif nums[i] == 1:
                i += 1
            else:
                right -= 1  # right在swap前面
                swap(nums, right, i)
```

### 88. 下一个排列

```python
# 记住就要，一个数学方法的模拟
# 1. 倒序遍历数组中的元素，找到第一个相邻的升序序列 (i,j)；
# 2. 再倒序遍历，找到第一个大于 nums[i] 的元素，并将其与 nums[i] 交换位置；
# 3. 再将nums[j]及其后面的元素重新按照升序排序。
class Solution:
    def nextPermutation(self, nums: List[int]) -> None:
        n = len(nums)
        i = n - 2
        while i >= 0 and nums[i] >= nums[i + 1]:  # 找到第一个升序的i：>=
            i -= 1  # 注意：是-=
        if i >= 0:  # 存在i
            j = n - 1  # 重新从最后搜寻第一个大于i的j
            while j >= 0 and nums[i] >= nums[j]:  # 这个和上面的一样，左边都是nums[i]。
                                                  # 非<的就-1，所以>=
                j -= 1
            nums[i], nums[j] = nums[j], nums[i]

        left, right = i + 1, n - 1  # 把i之后的换序
        while left < right:
            nums[left], nums[right] = nums[right], nums[left]
            left += 1
            right -= 1
```

### 89. 寻找重复数

```python
# 完美的解法
#         0, 1, 2, 3, 4  # 环的值
# nums = [1, 3, 4, 2, 2] # 环的next
class Solution:
    def findDuplicate(self, nums: List[int]) -> int:
        slow, fast = 0, 0  # 初始化，停在0
        while True:  # True，break跑掉
            slow = nums[slow]  # next：slow在[]中
            fast = nums[nums[fast]]
            if slow == fast:  # 第一次相遇在环里
                break
        fast = 0
        while True:  # True，return跑掉
            slow = nums[slow]
            fast = nums[fast]
            if slow == fast:  # 第二次相遇在交点
                return slow  # 因为一定存在
```