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

1. 对于一个长度为n的已排序数列a，若n为奇数，中位数为a[n/2+1], 若n为偶数，则中位数(a[n/2]+a[n/2+1])/2,如果我们可以在两个数列中求出第K小的元素，便可以解决该问题。

2. 不妨设数列A元素个数为n，数列B元素个数为m，各自升序排序，求第k小元素

	1. 取A[k/2]，B[k/2]比较，如果 A[k/2]>B[k/2] 那么，所求的元素必然不在B的前k/2个元素中(证明反证法);反之，必然不在A的前k/2个元素中，于是我们可以将A或B数列的前k/2元素删去，求剩下两个数列的k-k/2小元素，于是得到了数据规模变小的同类问题，递归解决。
	
	1. 如果 k / 2 大于某数列个数，所求元素必然不在另一数列的前k / 2个元素中，同上操作就好。

### 穷举遍历法  

时间复杂度O(N),虽然也可以实现功能，但是不符合O(log (m+n))的要求

	def findMedianSortedArrays(self, nums1, nums2):
		"""
        :type nums1: List[int]
        :type nums2: List[int]
        :rtype: float
        """
        l = len(nums1) + len(nums2)
        print(l)
        if l % 2 == 0:
            return (self.findMedian(nums1, nums2, int(l/2))+self.findMedian(nums1, nums2, int(l/2)+1))/2.0
        else:
            return self.findMedian(nums1, nums2, int(l/2)+1)

    def findMedian(self, nums1, nums2, l):

        p = 0
        q = 0
        for i in range(0, l-1):
            if p >= len(nums1) and q < len(nums2):
                q = q + 1

            elif p < len(nums1) and q >= len(nums2):
                p = p + 1

            elif nums1[p] > nums2[q]:
                q = q + 1

            else:
                p = p + 1

        if p >= len(nums1):
            return nums2[q]

        elif q >= len(nums2):
            return nums1[p]

        else:
            return nums1[p] if nums1[p] < nums2[q] else nums2[q]

### 递归：

findKth方法写的非常经典。更能看出这道题的解题思路：

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
			return findKth(A,m,B,n,total/2+1);
		else
			return (findKth(A,m,B,n,total/2) + findKth(A,m,B,n,total/2 + 1)) / 2;
	}

### 二分查找

1. 首先在随机位置将A分成两部分：

	|left_A | right_A|
	|---|---|
	|A [0]，A [1]，...，A [i-1] | A [i]，A [i + 1]，...，A [m-1]|

	由于A有m个元素，所以有m + 1种切割（i = 0〜m）。其中：left_A.size() = i，right_A.size() = m-i。注意：当i = 0时，left_A为空，当i = m时，right_A为空。

2. 同样，在随机位置将B切成两部分：

	|left_B | right_B|
	|---|---|
	|B [0]，B [1]，...，B [j-1] | B [j]，B [j + 1]，...，B [n-1]|

3. 将left_A和left_B放入一个集合，并将right_A和right_B放入另一个集合。把它们命名为left_part和right_part：

	|left_part | right_part|
	|---|---|
	|A [0]，A [1]，...，A [i-1] | A [i]，A [i + 1]，...，A [m-1]|
	|B [0]，B [1]，...，B [j-1] | B [j]，B [j + 1]，...，B [n-1]|

	如果我们可以确保：
	
	- left_part.size() == right_part.size()
	- max（left_part）<= min（right_part）

	那么我们将{A，B}中的所有元素划分为两个长度相等的部分，一个部分总是大于另一个部分。然后中值=（max（left_part）+ min（right_part））/ 2。

	为了确保这两个条件，只需要确保：
	
	- i + j == m-i + n-j（或：m-i + n-j + 1）, 如果n> = m，我们只需要设置：i = 0〜m，j =（m + n + 1）/ 2-i
	- B [j-1] <= A [i]，A [i-1] <= B [j]

	为什么一定要n> = m？因为必须确保j是合法索引，因为0 <= i <= m和j =（m + n + 1）/ 2-i。如果n <m，则j可以是负数，这将导致错误的结果。

4. 在[0，m]中搜索i，找到一个切分点i（j =（m + n + 1）/ 2-i）,使得B [j-1] <= A [i]和A [i-1] <= B[j]。
	
	我们可以按照以下步骤进行搜索：
	
	1. 设置imin = 0，imax = m，然后开始搜索[imin，imax]
	1. 设置i =（imin + imax）/ 2，j =（m + n + 1）/ 2-i
	1. 现在left_part.size() == right_part.size()。而且只有3种情况：
	
		- B[j-1] <= A [i]和A [i-1] <= B [j], 意味着找到了切分点i，停止搜索。
		- B[j-1]> A [i], 意味着A [i]太小。必须调整i得到B [j-1] <= A [i]。而i只能增加不能减小，因为当i减小时j将增加，因此，B [j-1]增加并且A [i]减小，永远不会满足。所以必须增加i。也就是说，调整搜索范围为[i + 1，imax]。因此，imin = i + 1和然后回到第<2>步。
		- A [i-1]> B [j], 意味着A [i-1]太大，必须减少i得到A [i-1] <= B [j]。因此，设置imax = i-1然后回到第<2>步。

5. 当找到切分点i时，中值为：max（A [i-1]，B [j-1]）（当m + n是奇数时）或（max（A [i-1]，B [j-1]）+ min（A [i]，B [j]））/ 2（当m + n为偶数时）

		def findMedianSortedArrays(self, nums1, nums2):
	        # 有一个数组为空
	        if len(nums1) == 0 and len(nums2) > 0:
	            return nums2[int(len(nums2)/2)] if len(nums2) % 2 == 1 else (nums2[int(len(nums2)/2)-1]+nums2[int(len(nums2)/2)])/2.0
	        if len(nums2) == 0 and len(nums1) > 0:
	            return nums1[int(len(nums1)/2)] if len(nums1) % 2 == 1 else (nums1[int(len(nums1)/2)-1]+nums1[int(len(nums1)/2)])/2.0
	
	        # 保证n > m
	        if len(nums1) > len(nums2):
	            return self.findMedianSortedArrays(nums2, nums1)
	
	        m, n = len(nums1), len(nums2)
	        imin, imax = 0, m
	
	        '''
	        用i,j分别将两个数组随机分成两部分(这里取中间)，nums1长度m,nums2为n
	        分别为nums1_left,nums1_right,nums2_left,nums2_right
	        我们再将nums1_left,nums2_left归为nums_left
	             将nums1_right,nums2_right归为nums_right
	             
	        只要我们确保：
	            1.len(nums_left) = len(nums_right)
	            2.max(nums_left) <= min(nums_left)
	            
	        那么中值就为:(max(nums_left)+min(nums_left))/2
	        
	        为了满足条件1，需使得i+j = m-i+n-j+ 则 j = (m+n+1)/2
	        为了满足条件2，需使得：
	            nums1[i-1] <= nums2[j]
	            nums2[j-1] <= nums1[i]
	            
	        所以，我们只要找到满足条件的i的位置即可
	        
	        '''
	        while imin <= imax:
	            i = int((imin+imax)/2)
	            j = int((m+n+1)/2) - i
	
	            # i太小增加i
	            if i < m and j > 0 and nums1[i] < nums2[j-1]:
	                imin = i + 1
	
	            # i太大减少i
	            elif i > 0 and j < n and nums1[i-1] > nums2[j]:
	                imax = i-1
	
	            else:
	                
	                # i或j为边界值
	                if (i == 0):
	                    max_left = nums2[j-1]
	
	                elif (j == 0):
	                    max_left = nums1[i-1]
	
	                else:
	                    max_left = nums1[i-1] if nums1[i-1] > nums2[j-1] else nums2[j-1]
	
	                break
	        
	        # 数组大小和奇数
	        if (m+n) % 2 == 1:
	            return max_left
	        
	        if i == m:
	            min_right = nums2[j]
	
	        elif j == n:
	            min_right = nums1[i]
	
	        else:
	            min_right = nums1[i] if nums1[i] < nums2[j] else nums2[j]
	
	        return (max_left+min_right)/2.0

## 5.Longest Palindromic Substring [Medium]

[原文链接]（https://www.cnblogs.com/grandyang/p/4464476.html）

Given a string s, find the longest palindromic substring in s. You may assume that the maximum length of s is 1000.

Example 1:

	Input: "babad"
	Output: "bab"
	Note: "aba" is also a valid answer.

Example 2:

	Input: "cbbd"
	Output: "bb"

### 思路：

这道题让我们求最长回文子串，首先说下什么是回文串，就是正读反读都一样的字符串，比如 "bob", "level", "noon" 等等。那么最长回文子串就是在一个字符串中的那个最长的回文子串。LeetCode 中关于回文串的题共有五道，除了这道，其他的四道为 Palindrome Number，Validate Palindrome，Palindrome Partitioning，Palindrome Partitioning II，我们知道传统的验证回文串的方法就是两个两个的对称验证是否相等，那么对于找回文字串的问题，就要以每一个字符为中心，像两边扩散来寻找回文串，这个算法的时间复杂度是 O(n*n)，可以通过 OJ，就是要注意奇偶情况，由于回文串的长度可奇可偶，比如 "bob" 是奇数形式的回文，"noon" 就是偶数形式的回文，两种形式的回文都要搜索，对于奇数形式的，我们就从遍历到的位置为中心，向两边进行扩散，对于偶数情况，我们就把当前位置和下一个位置当作偶数行回文的最中间两个字符，然后向两边进行搜索，参见代码如下：

### 解法一：

	class Solution {
	public:
	    string longestPalindrome(string s) {
	        if (s.size() < 2) return s;
	        int n = s.size(), maxLen = 0, start = 0;
	        for (int i = 0; i < n - 1; ++i) {
	            searchPalindrome(s, i, i, start, maxLen);
	            searchPalindrome(s, i, i + 1, start, maxLen);
	        }
	        return s.substr(start, maxLen);
	    }
	    void searchPalindrome(string s, int left, int right, int& start, int& maxLen) {
	        while (left >= 0 && right < s.size() && s[left] == s[right]) {
	            --left; ++right;
	        }
	        if (maxLen < right - left - 1) {
	            start = left + 1;
	            maxLen = right - left - 1;
	        }
	    }
	};

我们也可以不使用子函数，直接在一个函数中搞定，我们还是要定义两个变量 start 和 maxLen，分别表示最长回文子串的起点跟长度，在遍历s中的字符的时候，我们首先判断剩余的字符数是否小于等于 maxLen 的一半，是的话表明就算从当前到末尾到子串是半个回文串，那么整个回文串长度最多也就是 maxLen，既然 maxLen 无法再变长了，计算这些就没有意义，直接在当前位置 break 掉就行了。否则就要继续判断，我们用两个变量left和right分别指向当前位置，然后我们先要做的是向右遍历跳过重复项，这个操作很必要，比如对于 noon，i在第一个o的位置，如果我们以o为最中心往两边扩散，是无法得到长度为4的回文串的，只有先跳过重复，此时left指向第一个o，right指向第二个o，然后再向两边扩散。而对于 bob，i在第一个o的位置时，无法向右跳过重复，此时 left 和 right 同时指向o，再向两边扩散也是正确的，所以可以同时处理奇数和偶数的回文串，之后的操作就是更新 maxLen 和 start 了，跟上面的操作一样，参见代码如下： 

### 解法二：

	class Solution {
	public:
	    string longestPalindrome(string s) {
	        if (s.size() < 2) return s;
	        int n = s.size(), maxLen = 0, start = 0;
	        for (int i = 0; i < n;) {
	            if (n - i <= maxLen / 2) break;
	            int left = i, right = i;
	            while (right < n - 1 && s[right + 1] == s[right]) ++right;
	            i = right + 1;
	            while (right < n - 1 && left > 0 && s[right + 1] == s[left - 1]) {
	                ++right; --left;
	            }
	            if (maxLen < right - left + 1) {
	                maxLen = right - left + 1;
	                start = left;
	            }
	        }
	        return s.substr(start, maxLen);
	    }
	};

此题还可以用动态规划 Dynamic Programming 来解，根 Palindrome Partitioning II 的解法很类似，我们维护一个二维数组 dp，其中 dp[i][j] 表示字符串区间 [i, j] 是否为回文串，当 i = j 时，只有一个字符，肯定是回文串，如果 i = j + 1，说明是相邻字符，此时需要判断 s[i] 是否等于 s[j]，如果i和j不相邻，即 i - j >= 2 时，除了判断 s[i] 和 s[j] 相等之外，dp[i + 1][j - 1] 若为真，就是回文串，通过以上分析，可以写出递推式如下：

	dp[i, j] = 1  （i == j）
	         = s[i] == s[j] （j = i + 1）
	         = s[i] == s[j] && dp[i + 1][j - 1] （j > i + 1）

示意图：

|||0|1|2|3|4|
|---|---|---|---|---|---|---|
|||b|a|b|a|d|
|0|b|1|0|
|1|a||1|0|
|2|b|||1|0|
|3|a||||1|0|
|4|d|||||1|

这里有个有趣的现象就是如果我把下面的代码中的二维数组由 int 改为 vector<vector<int>> 后，就会超时，这说明 int 型的二维数组访问执行速度完爆 std 的 vector 啊，所以以后尽可能的还是用最原始的数据类型吧。

### 解法三：

	class Solution {
	public:
	    string longestPalindrome(string s) {
	        if (s.empty()) return "";
	        int dp[s.size()][s.size()] = {0}, left = 0, right = 0, len = 0;
	        for (int i = 0; i < s.size(); ++i) {
	            dp[i][i] = 1;
	            for (int j = 0; j < i; ++j) {
	                dp[j][i] = (s[i] == s[j] && (i - j < 2 || dp[j + 1][i - 1]));
	                if (dp[j][i] && len < i - j + 1) {
	                    len = i - j + 1;
	                    left = j;
	                    right = i;
	                }
	            }
	        }
	        return s.substr(left, right - left + 1);
	    }
	};

最后要来的就是大名鼎鼎的马拉车算法 Manacher's Algorithm，这个算法的神奇之处在于将时间复杂度提升到了 O(n) 这种逆天的地步，而算法本身也设计的很巧妙，很值得我们掌握，参见我另一篇专门介绍马拉车算法的博客 [Manacher's Algorithm 马拉车算法](http://www.cnblogs.com/grandyang/p/4475985.html)，代码实现如下：

![](https://images2018.cnblogs.com/blog/1414940/201808/1414940-20180801000600386-448004483.jpg)

### 解法四：

	class Solution {
	public:
	    string longestPalindrome(string s) {
	        string t ="$#";
	        for (int i = 0; i < s.size(); ++i) {
	            t += s[i];
	            t += '#';
	        }
	        int p[t.size()] = {0}, id = 0, mx = 0, resId = 0, resMx = 0;
	        for (int i = 1; i < t.size(); ++i) {
	            p[i] = mx > i ? min(p[2 * id - i], mx - i) : 1;
	            while (t[i + p[i]] == t[i - p[i]]) ++p[i];
	            if (mx < i + p[i]) {
	                mx = i + p[i];
	                id = i;
	            }
	            if (resMx < p[i]) {
	                resMx = p[i];
	                resId = i;
	            }
	        }
	        return s.substr((resId - resMx) / 2, resMx - 1);
	    }
	};