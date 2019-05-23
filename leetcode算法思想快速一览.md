# leetcode算法思想快速一览

[原文链接](https://www.cnblogs.com/AkazaAkari/p/5987337.html)

整理了一下思路，想深入了解还得多去写，无奈时间紧迫的情况下抛砖引玉也不失为下策：

## 1.Two Sum [Easy] 

[原文链接](https://www.jianshu.com/p/8a7e8ee22c8c)

Given an array of integers, return indices of the two numbers such that they add up to a specific target.
You may assume that each input would have exactly one solution.

给定一个整数数组，返回两个数的下标，使这两个数的和等于指定的目标数，假定每个输入的数组都有恰好一个解

Example:

	Given nums = [2, 7, 11, 15], target = 9,
	Because nums[0] + nums[1] = 2 + 7 = 9,
	return [0, 1].

### (Java) Version 1 Time: 98ms: O(N^2)

在数组中从前往后搜索，每一个数都和其余的数进行求和判断，如果有结果则直接输出，题目中已经假设只有一个解，不需要再向后搜索

	public class Solution {
	    public int[] twoSum(int[] nums, int target) {
	        for (int i = 0; i < nums.length; i++)
	            for (int j = 0; j < nums.length; j++) {
	                if (i == j)
	                    continue;
	                if (nums[i] + nums[j] == target)
	                    return new int[] { i, j };
	            }
	        return nums;
	    }
	}

### (Java) Version 2 Time: 52ms: O(N^2/2)

对上一个算法的改进在于，第一次循环搜索的数之前的数不需要再在第二个循环重新判断一次，每一次循环都只需要和后面的数作比较，前面的数在上一次循环中已经被比较过一次了，同时，因为不需要同前面的数进行比较，那么，比较可以跳过自己，直接从i+1开始，省去了判断i==j的步骤

	public class Solution {
	    public int[] twoSum(int[] nums, int target) {
	        for (int i = 0; i < nums.length; i++)
	            for (int j = i+1; j < nums.length; j++) {
	                if (nums[i] + nums[j] == target)
	                    return new int[] { i, j };
	            }
	        return nums;
	    }
	}

### 排序: O(NlogN)

和为目标值，一般的思路是先排序，然后取两点坐标分别从首尾向中间移动。若和为目标值则返回两点坐标。若和大于目标值，右端坐标值-1，反之左端坐标值+1 

### (Java) Version 3 Time: 8ms (By plover ): O(N)

运用Java的Map，构造Map键为当前判断的数的期望目标数，值为数的下标，通过搜索键来判断是否存在对应的期望目标数，并可以快速找到其下标，不需要重新循环一次。巧妙应用Map把数组中的数和下标绑定在一起，达到快速获得数及其下标的目的

	public class Solution {
	    public int[] twoSum(int[] nums, int target) {
	        HashMap<Integer, Integer> pair=new HashMap<>();
	        int result[] = new int[2];
	        for(int i=0;i<nums.length;i++){
	            if(pair.containsKey(nums[i])){
	                result[0]=pair.get(nums[i]);
	                result[1]=i;
	                return result;
	            }
	            else{
	                pair.put(target-nums[i], i);
	            }
	        }
	    return result;
	    }
	}


## 2.Add Two Numbers [Medium]

[原文链接](https://www.jianshu.com/p/d47643a2ce3a)

You are given two non-empty linked lists representing two non-negative(非负) integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

Example:

	Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
	Output: 7 -> 0 -> 8
	Explanation: 342 + 465 = 807.


### 思路：

1. 这种需要求和，进位频繁的工作选取一个全局变量来存储临时的和。（比如两个数的个位分别是9和8，那么先不必考虑进位，这个全局变量的值就是17，运算完成后，和的个位值为 全局变量%10, 然后执行 全局变量 /= 10,得出全局变量的剩余值。在计算十位的时候要加上这个剩余值。）

1. 这道题主要还顺带考察了链表的用法。

1. 注意while循环自带逻辑判断。这两个链表的长度不一，所以循环结构可以这样写：
	
		while(链表1当前节点不为空且链表2当前节点不为空) ;//两个链表都还有值
		while(链表1当前节点不为空); //链表2已空
		while(链表2当前节点不为空); //链表1已空
		if(全局变量 > 0) //如果全局变量>0 说明有进位操作，还需要加长结果链表

### Java

	struct ListNode {
	    int val;
	    struct ListNode *next;
	};

	public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode dummyHead = new ListNode(-1), cur = dummyHead;
        ListNode p = l1, q = l2;
        int carry = 0, sum;	//存储进位信息
        while (p != null || q != null) {
            sum = (p == null ? 0 : p.val) + (q == null ? 0 : q.val) + carry;
            carry = sum / 10;
            cur.next = new ListNode(sum % 10);
            cur = cur.next;
            if (p != null) {
                p = p.next;
            }
            if (q != null) {
                q = q.next;
            }
        }
        if (carry > 0) {
            cur.next = new ListNode(carry);
        }
        return dummyHead.next;
    }

## 3.Longest Substring Without Repeating Characters [Medium]

[原文链接](https://blog.csdn.net/mapoos/article/details/87212093)

Given a string, find the length of the longest substring without repeating characters.

Example 1:

	Input: "abcabcbb"
	Output: 3 
	Explanation: The answer is "abc", with the length of 3. 

Example 2:

	Input: "bbbbb"
	Output: 1
	Explanation: The answer is "b", with the length of 1.

Example 3:

	Input: "pwwkew"
	Output: 3
	Explanation: The answer is "wke", with the length of 3. 
				 Note that the answer must be a substring, "pwke" is a subsequence and not a substring.

### 题意

给定一个字符串，找到一个子串，使得子串内没有重复的字符，求该子串的最大长度。

### 思路：

1. 普通解法直接暴力三重循环，O(N^3)

1. 较优解法是使用一个“可伸缩”的数组框在字符串上滑动，先向右移动右端点，判断右端点的字符在不在被“伸缩数组”框住的子串中，如果重复则不断向右移动左端点，直到将重复的字符排除在“伸缩数组”外。这样相当于字符串中的每一个字符都要走一遍，复杂度为O(2N) = O(N)。

1. 对上述方法的一个优化是，用一个数组记录字符串中每一个字符对应的索引位置，这样在右端点字符重复时，左端点可以直接跳到子串中字符重复的地方，不需要一步一步向右移动左端点。这样只需要右端点走一遍字符串即可，复杂度直接为O(N)。

### 代码实现

	class Solution {
	    public int lengthOfLongestSubstring(String s) {
	        int left = 0;
	        // right指向子串的右边一个字符
	        int right = 0;
	        int maxLen = 0;
	        Set<Character> set = new HashSet<>();
	        while (right < s.length()) {
	            if (set.contains(s.charAt(right))) {
	                set.remove(s.charAt(left++));
	            } else {
	                set.add(s.charAt(right++));
	                maxLen = Math.max(right - left, maxLen);
	            }
	        }
	        return maxLen;
	    }
	}

### 代码实现 - 优化

	class Solution {
	    public int lengthOfLongestSubstring(String s) {
	        int left = 0;
	        // right指向子串的最后一个字符
	        int right = 0;
	        int maxLen = 0;
	        // 下标记录字符，值记录该字符在字符串中对应的索引+1
	        // 实际上记录的是左端点应该跳转到的位置
	        int[] hash = new int[128];
	        while (right < s.length()) {
	            left = Math.max(left, hash[s.charAt(right)]);
	            maxLen = Math.max(maxLen, right - left + 1);
	            hash[s.charAt(right)] = ++right;
	        }
	        return maxLen;
	    }
	}

## 4.Median of Two Sorted Arrays [Hard]

[原文链接](https://www.jianshu.com/p/8462f6cdaa41)

There are two sorted arrays nums1 and nums2 of size m and n respectively.

Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).

Example 1:

	nums1 = [1, 3]
	nums2 = [2]
	
	The median is 2.0

Example 2:

	nums1 = [1, 2]
	nums2 = [3, 4]
	
	The median is (2 + 3)/2 = 2.5

有两个大小为 m 和 n 的排序数组 nums1 和 nums2 。请找出两个排序数组的中位数并且总的运行时间复杂度为 O(log (m+n)) 。

### 思路：

1. 设数组1的长度为m 数组2的长度为n总数就是m+n 我们就是要找两个数组中第m+n/2大的元素的值。我们设m+n/2为k

1. 如果a1[k/2-1]大于a2[k/2-1]，那么a2[0]到a2[k/2-1]的所有值当中都不可能有中值。原因如下。
	
	就算取a2[0]~a2[k/2-1]当中最大的值作为中值a2[k/2-1]，那么需要从a1中取出k/2+1个数排在a2[k/2-1]的左边才能凑够k个数，让a2[k/2]成为两数组第k大的元素。然而a1中取出k/2个最小的元素当中已经有比a2[k/2-1]大的元素了，所以并不能凑齐k-1个比a2[k/2]小的数。

1. 上述是理想情况下(k/2-1既小于a1的长度也小于a2的长度)。如果不是，那么就用短的数组的长度代替这个值进行相同的判断。

1. 然后将删除不可能的元素的两个数组重新判断，这回要找的是第 k -(k/2-1)或这 k-短的那个数组长度 大的值。删除元素的数组的长度和首指针都进行更新。

findKth方法写的非常经典。更能看出这道题的解题思路：

1. 保持A是短的那一个数组，B是长的

2. 平分k, 一半在A，一半在B （如果A的长度不足K/2,那就pa就指到最后一个）

3. 如果pa的值 < pb的值，那证明第K个数肯定不会出现在pa之前，递归，把A数组pa之前的砍掉，同理递归砍B数组。

4. 递归到 m == 0 （短的数组用完了） 就返回 B[k - 1], 或者k == 1（找第一个数）就返回min(A第一个数，B第一个数）。

		double findKth(int a[], int m , int b[], int n, int k)
		{
			if(m > n)
				return findKth(b,n,a,m,k);
			
			if(m == 0)
				return b[k-1];
			
			if(k == 1)
				return min(a[0], b[0]);
			
			int pa = min(k/2, m), pb = k-pa;
			
			if(a[pa -1] < b[pb-1])
				return findKth(a + pa, m - pa, b, n, k - pa);
			else if(a[pa - 1] > b[pb - a])
				return findKth(a, m, b + pb, n - pb, k - pb);
			else 
				return a[pa - 1]; 
			return 0;
		}
			
		double findMid(int A[], int m, int B[], int n)
		{
			int total = m + n;
			if(total & 0x1)
				return findKth(A,m,B,n, total/2+1);
			else
				return (findKth(A,m,B,n, total/2) + findKth(A,m,B,n, total/2 + 1)) / 2; 
		}