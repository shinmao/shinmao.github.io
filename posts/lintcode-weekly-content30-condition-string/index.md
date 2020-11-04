# LintCode Weekly Contest 30 Condition String


I ran into the "Condition String" in weekly contest of LintCode and failed to solve it. After spending some time on it after the contest ended, I found that the first idea I came up with was wrong. Here comes my writeup.

## Problem Description
Here I would give the screenshot of challenge:  
![](/10-31-20/description.png)

## First solution [Fail]
First, I tried to handle this problem with **longest increase subarray**. For example, in `acec`  
with start point from `a`, it would move forward until `e` and get the max length of 3.  
with start point from `c`, it would move forward until `e` and get the max length of 2.
However, after the contest ended, I found this idea was totally wrong. With this idea, we cannot find the case of `acc` because we also can remove the character of `e` to get the longest string. Therefore, we cannot handle this problem with longest increase subarray.

## Solution with DP - Longest Increase SubSequence
We cannot handle with longest increase subarray, but we can handle with longest increase sub-sequence. We need a DP array to memorize the current max length ending with three kinds of characters. Why this method can get the all possible cases of longest subsequence? Let me show the sample in following:  
```cpp
// dp[i]: the max length end with i
// here i would only be 0, 1, 2 which represent a,c,e or b,d,f
// take the same example 'acec' and iterate through
dp[i]    a     c     e
x        0     0     0
str[0]   1     0     0     // updated with 'a'
str[1]   1     2     0     // updated with 'ac'
str[2]   1     2     3     // e updated with 'ace', c stay still with 'ac'
str[3]   1     3     3     // c updated with 'acc', e stay still with 'ace'
```
From this example, now I can give the answer to the question. This method works because he can memorize the previous status which ends with different tails. So let's come back to the example, if `str[3]` was 'a', we can still attach it to the `dp[0]` and check whether `aa` can be our condition string of max length or not. Great, now let us move to the solution:  
```cpp
class Solution {
public:
    /**
     * @param s: a string only contains `a-f`
     * @return: The longest length that satisfies the condition
     */
    int check(vector<int> &s) {
        vector<int> dp(3, 0);
        for(int i = 0; i < s.size(); i++) {
            int c = s[i];
            
            int last = INT_MIN;
            for(int j = 0; j <= c; j++) {
                last = max(last, dp[j] + 1);
            } 
            dp[c] = last;
        }
        
        return *max_element(dp.begin(), dp.end());
    } 
    
    int conditionString(string &s) {
        // write your code here
        if(s.size() == 1)
            return 1;
            
        vector<int> ace;
        vector<int> bdf;
        for(auto c : s) {
            if(c == 'a') {
                ace.push_back(0);
            }else if(c == 'b') {
                bdf.push_back(0);
            }else if(c == 'c') {
                ace.push_back(1);
            }else if(c == 'd') {
                bdf.push_back(1);
            }else if(c == 'e') {
                ace.push_back(2);
            }else {
                bdf.push_back(2);
            }
        }
        
        return check(ace) + check(bdf);
    }
};
```
I have explained for the part of `check()`, in main function part, we just need to separate the string to two groups and handle with function individually. What needs to be care about is the definition of DP array: **length ended with index**. In this solution, time complexity is `O(n^2)`. What is interesting, the solution can be optimized!

## Optimized method for finding Longest increase subsequence
```cpp
int LIS(vector<int> & nums) {
    int n = nums.size();
    vector<int> q(n, INT_MAX);
    for(int i = 0; i < nums.size(); i++) {
        auto iter = upper_bound(q.begin(), q.end(), nums[i]);
        *iter = nums[i];
    }
    for(int i = n - 1; i >= 0; i--) {
        if(q[i] != INT_MAX)
            return i + 1;
    }
    return 1;
}
```
The main idea of this optimized method is to replace next element with the **first element in array which bigger than current element**. To help understand much easier, I would give example,
```
original array: [1, 2, 5, 7, 3, 4, 5]
q = [Max, Max, Max, Max, Max, Max, Max]
for-loop:
q = [1, Max, Max, Max, Max, Max, Max]
q = [1, 2, Max, Max, Max, Max, Max]
q = [1, 2, 5, Max, Max, Max, Max]
q = [1, 2, 5, 7, Max, Max, Max]  // 1.
q = [1, 2, 3, 7, Max, Max, Max]  // 2.
q = [1, 2, 3, 4, Max, Max, Max]
q = [1, 2, 3, 4, 5, Max, Max]
```
In second loop, remove the elements which are still `INT_MAX` then we can get the longest increase subsequence in this array, which is `[1, 2, 3, 4, 5]`. Why am I sure this is the longest one in this array? From the change I showed in above, we can find that the method would **lower the height of the whole subsequence**. Same example in above, the comment 1 ends up with 5 at the index 2 and the comment 2 ends up with 3 at the same index, and there are much more possible numbers we can decide to put after 3 than 5. Therefore, this optimized method can help to extend the length. Time complexity is only `O(nlogn)`!

## Conclusion
In fact, there was still a faster solution with `O(3N)`. If we change definition of dynamic programming, $$ dp[i][k]: LIS length of first i elements, which end up with k. $$  
This is feasible because we only have three types of letter, so we can come with this solution:  
```cpp
for(int i = 1; i < n; i++) {
    if(nums[i] == 1) {
        dp[i][1] = dp[i - 1][1] + 1;
        dp[i][2] = dp[i - 1][2];
        dp[i][3] = dp[i - 1][3];
    }else if(nums[i] == 2) {
        dp[i][1] = dp[i - 1][1];
        dp[i][2] = max(dp[i - 1][1], dp[i - 1][2]) + 1;
        dp[i][3] = dp[i - 1][3];
    }else if(nums[i] == 3) {
        dp[i][1] = dp[i - 1][1];
        dp[i][2] = dp[i - 1][2];
        dp[i][3] = max(dp[i - 1][1], max(dp[i - 1][2], dp[i - 1][3])) + 1;
    }
}
```
Well, this method is only limited to such input size that we can list out all the cases.
