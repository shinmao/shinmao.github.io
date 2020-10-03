# AlgoExpert Longest Peak


This challenge is not difficult; however, I make a mistake when analyzing the time complexity. The reason is that I make a large duplicate and useless calculation in my solution. Take a look into the detail for the challenge "Longest Peak".

## Writeup
{{< youtube s7eoePF8UjQ >}}  

You can see the detail of challenge and my initial solution in this video. However, I can finish this problem in `O(n)` in fact. Why?  
Assume there is ten numbers in array:  
```cpp
a, b, c, d, e, f, g, h, i, j
```
And if I iterate peak from index 1:  
```cpp
   v
a, b, c, d, e, f, g, h, i, j
<  v  >  >
a, b, c, d, e, f, g, h, i, j
```
Ok, our right pointer stops at somewhere. The reason is that **e is bigger than d**; therefore, our right pointer cannot move forward (be careful! according to my source code, right pointer doesn't stop at the end of the peak, but stop at the next position of the end). So, right pointer should points to `e` now.  

But, do you know who is `e` ? **He is the next possible peak**!  

In my original solution in video, I iterate all the numbers in array as potential peak. However, we already know that the potential peaks are (`b`, `e`) instead of (`b`, `c`, `d`, `e`). After I find out this fact, I change my solution as following:  
```cpp
int longestPeak(vector<int> array) {
	int n = array.size();
  if(n < 3)
		return 0;
	int maxL = 0;
	// iterate peak index from 1 to n - 2
	int peak = 1;
	while(peak < n - 1) {
		if(!(array[peak - 1] < array[peak] && array[peak] > array[peak + 1])) {
			peak++;
			continue;
		}
		int left = peak - 2;
		int right = peak + 2;
		while(left >= 0 && array[left] < array[left + 1]) {
			left--;
		}
		while(right < n && array[right] < array[right - 1]) {
			right++;
		}
		maxL = max(maxL, right - left - 1);
		peak = right;
	}
  return maxL;
}
```
Again, do you think time complexity is `O(n^2)`?  
```cpp
<  v  >  <  v  >  >  <  v  >
a, b, c, d, e, f, g, h, i, j
```
The key change in this solution is `peak = right`. In this result, potential peaks would be (`b`, `e`, `i`). And left pointers, right pointers iterate the rest elements for **one time**. Therefore, time complexity should be `O(n)`! It's interesting, isn't it?  

Good luck! Feel free to leave any comments.
