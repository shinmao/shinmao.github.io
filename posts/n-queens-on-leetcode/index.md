# N Queens on Leetcode


# Introduction
I spend so much time on solving this problem. My challenge is not just solving this problem, but how to enhance the efficiency of my solution. The only one solution I come up with is **DFS + check each of previous rows**; however, this solution is too slow. Therefore, in following of my youtube video, I will explain four to five solutions, from idea building, coding by hands, and even debugging. Welcome to leave any comments on my video!

## Writeup
{{< youtube XgOZ-0Qr4rw >}}

The programming language in video is C++

For the source code in video: [Github](https://github.com/shinmao/algorithm/blob/main/leetcode/leetcode51.cpp)

## Conclusion
* Solution 1. Enumerate each columns and check whether conflict with all previous queens, so slow :rofl:
* Solution 2. Use `pie`, `na`, `col` arrays to record queens' position, enumerate each columns to check conflict with three arrays
* Solution 3. Convert arrays above to integer type :confused: compared to solution2, this would be faster because our computer doesn't need to take time to convert between binary and decimal  
* Solution 4. Don't need to enumerate each columns anymore, use bitwise trick to enumerate the only available seats :yum:

To solve this challenge, you need to get the basic knowledge of:  
* Depth First Search: define each of the levels
* Bitwise trick: more efficient to get the answer you want
