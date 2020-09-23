# LeetCode 33 Search in Rotated Array


First, I need to come up with the words, 'Damn it'. This belongs to my LeetCode Self Challenge for 30 mins. I almost finish the problem, but I forget to check the search space for binary search, which should be the common sense for it. I will provide the writeup and the optimization here.

## Writeup
{{< youtube WnmgJ6C9at0 >}}  

The programming language in video is C++  

```cpp
int search(vector<int>& nums, int target) {
if(nums.size() == 0)
    return -1;
int n = nums.size();
int start = 0, end = n-1;
if(target >= nums[0]) {
    while(start <= end) {
        int mid = start + ((end - start) >> 1);
        if(nums[mid] == target)
            return mid;
        if(nums[mid] > target)
            end = mid - 1;
        else if(nums[mid] < target) {
            if(nums[mid] >= nums[0]) {
                // mid in left
                start = mid + 1;
            }else {
                // mid in right
                end = mid - 1;
            }
        }
    }
}else if(target <= nums[n - 1]) {
    while(start <= end) {
        int mid = start + ((end - start) >> 1);
        if(nums[mid] == target)
            return mid;
        if(nums[mid] < target)
            start = mid + 1;
        else if(nums[mid] > target) {
            if(nums[mid] >= nums[0]) {
                // mid in left
                start = mid + 1;
            }else {
                // mid in right
                end = mid - 1;
            }
        }
    }
}

return -1;
```
What is different from the video? For the purpose of guarantee the searching space, I change binary search to template which only needs one space to work. So I change from `start + 1 < end` to `start <= end` here. However, just like what I check in my video, this code seems a little redundant; therefore, I can clear my code to become:  
```cpp
if(target >= nums[0]) {
    ...
    if(nums[mid] < target && nums[mid] >= nums[0]) {
        start = mid + 1;
    }else {
        end = mid - 1;
    }
}else if(target <= nums[n-1]) {
    if(nums[mid] > target && nums[mid] < nums[0]) {
        end = mid - 1;
    }else {
        start = mid + 1;
    }
}
```  
However, it has a bad time complexity. Therefore, here comes up with an optimization.

## Optimization
The idea comes here: It's annoying that this array is rotated. For the rotated problem, **how about trying to make it back to the ascending order?** It means that we can find out the index of the smallest number first, and add the previous sequence to the tail of the array, then we can make it back to the ascending order again.  
But how can this help us to solve this problem? Imagine that the rotated array: `[4,5,6,7,0,1,2]` becomes `[4, 5, 6, 7, 0, 1, 2, 4, 5, 6, 7]`. Now, we can search in the sorted array. If we have the mid at 1, it is still not out of the bound of original array, so it's ok. However, if we have the mid at 5, how do we deal with the OFB index? So the first thing to do in this problem is using binary search to find out the rotated index, then we can use `(mid + rotated index) % n`
to shift back to the original index. Here comes the final solution:  
```cpp
int search(vector<int>& nums, int target) {
    int n = nums.size();
    if(n == 0)
        return -1;
    // find the idx of smallest number in array
    // which is the rotated one
    int start = 0, end = n - 1;
    while(start < end) {
        int mid = start + ((end - start) >> 1);
        if(nums[mid] > nums[end])
            start = mid + 1;
        else
            end = mid;
    }
    int smallest = start;
    start = 0, end = n - 1;
    while(start <= end) {
        int mid = start + ((end - start) >> 1);
        int sorted_mid = (mid + smallest) % n;
        if(nums[sorted_mid] == target)
            return sorted_mid;
        if(nums[sorted_mid] < target)
            start = mid + 1;
        else
            end = mid - 1;
    }
    return -1;
}
```
Then it has great time complexity which beats over 96% of submission.

Good luck! Feel free to leave any comments.

