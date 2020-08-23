# Explore Binary Search


查找一個數組裡面的目標，我們能夠想到最快的方法可能是hash table，然而當數組很大時，內存就會爆掉！第二個選擇則是binary search。Binary search的觀念看似很簡單，然而卻有多種實現方法！最近在做一些題目時(包括面試時)發現binary search的應用面其實很廣，只要dataset有單調性，我們都可以用binary search把時間複雜度縮小到對數等級。特別感謝Leetcode出了Explore篇章幫助我進行摸索。下面我也會針對Explore上的題目做出解析來整理Binary search的訣竅！

## 解題過程
> 所有的元素為unique，尋找唯一的index
### [Explore Binary Search](https://leetcode.com/explore/learn/card/binary-search/138/background/1038/)
```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        if(target < -9999 || target > 9999) return -1;
        int start = 0, end = nums.size() - 1, mid = 0;
        while(start <= end) {
            mid = start + ((end - start) >> 1);   // remember your bracket
            if(nums[mid] == target) return mid;
            if(nums[mid] < target) 
                start = mid + 1;
            else
                end = mid - 1;
        }
        return -1;
    }
};
```
以上我的解法只有哪到66%的成績，但在我審了一遍前段的code之後，我推斷時間複雜度只差在I/O優化！  

### [Find First and Last Position of Element in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/)
我一開始的想法是先找到旋轉sorted array的pivot，再透過這個pivot判斷目標應該在哪邊。獲取了69.9%的成績。
```cpp
int search(vector<int>& nums, int target) {
    if(nums.size() == 1) {
        if(target == nums[0]) return 0;
        else return -1;
    }
    // find rotate pivot
    int rotate = -1;
    for(int i = 1; i < nums.size(); ++i) {
        if(nums[i] < nums[i - 1]) rotate = i;
    }
    int start = 0, end = nums.size() - 1, mid = 0;
    if(rotate != -1) {
        // if rotated, divide by rotate pivot
        if(nums[rotate] == target) return rotate;
        if(target > nums[rotate - 1] || target < nums[rotate]) return -1;
        if(target <= *(--nums.end())) start = rotate;   // remember *(--nums.end()) instead of *(nums.end())...
        else if(target >= *(nums.begin())) end = rotate - 1;
    }
    // BS
    while(start <= end) {
        mid = start + ((end - start) >> 1);
        if(nums[mid] == target) return mid;
        if(nums[mid] < target) start = mid + 1;
        else end = mid - 1;
    }
    return -1;
}
```
看這成績就知道，我的解法肯定可以優化。我認為我在尋找pivot的時候花了多餘的時間:  
```cpp
...
for(int i = 1; i < nums.size(); ++i)
    if(nums[i] < nums[i - 1]) rotate = i;
...
```
事實上，我可以把這段合併進while-loop。每一次iteration選取中點時，中點的左右狀況只可能是 `<升序/混序>`，`<混序/升序>`，或是`<升序/升序>`。而要判斷目前的段落是哪一種序別，我只需要比較兩個端點的大小！
```cpp
...
int start = 0, end = nums.size() - 1, mid = 0;
while(start <= end) {
    mid = start + ((end - start) >> 1);
    if(nums[mid] == target) return mid;
    if(nums[mid] >= nums[start]) {
        // left is in order
        if(target < nums[mid] && target >= nums[start]) {
            // target is in left section
            end = mid - 1;
        }else {
            start = mid + 1;   
        }
    }else {
        // right is in order
        if(target > nums[mid] && target <= nums[end]) {
            // target is in right section
            start = mid + 1;
        }else {
            end = mid - 1;
        }
    }
}
...
```
所以我的思路是  
1. 先選取升序的一邊
2. 判斷target有沒有可能在裡面
3. 如果可能，再用binary search去搜；否則，target就在另外一邊或者根本沒有它
4. 回到第一步...

太棒了！這樣修改之後我的成績高達96.5%。估計是因為少了`O(n)`的時間...

> 元素可不為unique，access當前index和他右邊的鄰居，用右邊的鄰居來選邊  

### [First Bad Version](https://leetcode.com/problems/first-bad-version/)
這個part裡的題目又有了些變化，但是我們還是可以發現單調性。下面是我AC 100%的解法  
```cpp
int firstBadVersion(int n) {
    if(n <= 0) return -1;
    if(n == 1) {
        if(isBadVersion(1)) return 1;
        else return -1;
    }
    int start = 1, end = n, mid = 1;
    while(start < end) {
        mid = start + ((end - start) >> 1);
        if(!isBadVersion(mid)) start = mid + 1;
        else end = mid;
    }
    return (isBadVersion(start) ? start : -1);
}
```
這個解法中最需要注意的是我不能使用**start <= end**，因為這樣會導致死循環。讓我們來看看為何會有死循環: `start`我定義為第一個bad version。悲劇會發生在`start == end`時並且他們都回傳`true`。`start == (end = mid)`的條件會永遠成立因為end會永遠選mid那邊！    

### [Find peak element](https://leetcode.com/problems/find-peak-element/)
下一個挑戰需要注意題目說的`nums[-1] = nums[n] = -∞`，所以題目不管怎樣都至少會有一個peak！那我要如何在每一個binary search選邊蒐呢？我開始分析每一種可能的情況，在選定一個中點時，中點的鄰居只有四種情況:  
1. 升序 -> pivot point -> 升序
2. 降序 -> pivot point -> 升序
3. 降序 -> pivot point -> 降序
4. 升序 -> pivot point -> 降序 => pivot正是peak!

在情況1和2，我會選擇右邊下去搜，或者至少我可以把tail當成peak。在情況3，我會選擇左邊下去搜：  
```cpp
int findPeakElement(vector<int>& nums) {
    int start = 0, end = nums.size() - 1, mid;
    while(start < end) {
        mid = start + ((end - start) >> 1);
        if(nums[mid] < nums[mid + 1]) start = mid + 1;
        else end = mid;
    }
    return start;
}
```  
我的思路: `start`被定義為peak。注意到這行程式碼: `else end = mid`，這行涵蓋了狀況3和4的邏輯。我們也不需要擔心這行會忽略第四種情況的邏輯，因為while loop在`start == end = mid`時就會break了! 我的解法獲取了86.7%的成績。  

### [Find minimum in rotated sorted array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/)
這個challenge裡，我的AC解也只有拿到61%。  
```cpp
int findMin(vector<int>& nums) {
    if(nums.size() == 1) return nums[0];
    if(*(nums.begin()) < *(--nums.end())) return *min_element(nums.begin(), nums.end());
    int start = 0, end = nums.size() - 1, mid;
    while(start < end) {
        mid = start + ((end - start) >> 1);
        if(nums[mid] > nums[end]) start = mid + 1;
        else end = mid;
    }
    return nums[start];
}
```
在我尋找可以優化的地方時，我發現了這行:  
```cpp
if(*(nums.begin()) < *(--nums.end())) return *min_element(nums.begin(), nums.end());
```
最一開始，我用這句去處理沒有rotate的例外情況。然而，他其實不是例外情況，當我們選完邊的時候，那個段落也可能是沒rotate過的，更棒的是如果選取到的段落是升序，那我們直接取這個段落的開頭，就是我們要的最小元素了！因此，這個邏輯可以合併進迴圈並且加速判斷找出我們的最小值。Input肯定是`升序 -> pivot...小降 -> 升序`的形式，而我們的步驟可以整理成下面幾步:  
1. 升序 -> piv..小降 -> 升序
2. 選完邊後段落的三種情況:
升序 -> piv...小降 -> 升序
piv...小降 -> 升序
升序  
在步驟2中，如果是第三種情況，我們可以直接回傳段落的head。其他的狀況的話則繼續用binary search來選邊搜就好。下面是我合併完的code:  
```cpp
...
while(start < end) {
    if(nums[start] < nums[end]) break;
    mid = start + ((end - start) >> 1);
    if(nums[mid] > nums[end]) start = mid + 1;
    else end = mid;
}
...
```
簡直好太多了，AC 94%!

### [Find First and Last Position of Element in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/)
這一個問題讓我花了一段時間思考。我剛開始一直固執得想要同時找到第一個位置和最後一個位置。最後才發現實在不行，因為第一個和第二個搜索的方向並不一樣，不過先找到第一個位置，然後再從剩下的段落中進行binary search聽起來也不賴！
```cpp
vector<int> searchRange(vector<int>& nums, int target) {
    vector<int> res;
    if(nums.size() == 0) return vector<int>({-1, -1}); 
    if(target < *(nums.begin()) || target > *(--nums.end())) return vector<int>({-1, -1});
    int start = 0, end = nums.size() - 1, mid;
    while(start <= end) {
        mid = start + ((end - start) >> 1);
        if(nums[mid] < target) start = mid + 1;
        else end = mid - 1;
    }
    if(start < nums.size() && nums[start] == target) res.push_back(start);
    else return vector<int>({-1, -1});
    end = nums.size() - 1;
    while(start <= end) {
        mid = start + ((end - start) >> 1);
        if(nums[mid] > target) end = mid - 1;
        else start = mid + 1;
    }
    res.push_back(end);
    return res;
}
```
遺憾的是，這AC解只有65％！還是跟大家說明一下我的思路，`start`和`end`的定義各別是第一個位置和最後一個位置，而我的search space只有保留一個位置，結束迴圈的條件是`start > end`。大略上的步驟已經在上面說明過了<1. 找出第一個位置>:
pick一個中點，如果中點是小於我們的目標的，則從下一個位置繼續找，若是中點是大於目標的，則我們的結束位置絕對至少是中點的前一個位置，這時候我們或許會疑惑，若是中點等於目標，我這樣處理會不會漏掉解呢？**我們在這裡回傳的是start，end只是來幫我們縮小範圍的**。e.g. 假設我們現今start和end都指到了目標，下一步end比start小就會結束迴圈了，start還是我們的答案！但是這是有例外情況的，由於`start >
end`是我們的while終止條件，儘管我們沒有找到目標，while還是會終止回傳給我們的start！因此我們必須確認：`start`上是不是真的就是我們的目標？為了確保這點，我們還要確保這個index沒有out of bound。如果不是目標，代表我們連第一個位置都沒找到，那我們就可以直接結束這回合囉！<2. 找到最後一個位置>，我們的段落會是長成這副德性:
`<target, target.... , rest of this section>`。所以while中也只有兩種狀況需要我們判斷，一種是大於目標，另一種是等於目標。我們要回傳的是`end`，當中點大於目標的時候，最後一個位置要往前定位，當中點等於目標的時候，我們的`start`則往後定位。跟上面一樣，我們也不用擔心這樣會漏掉`start == end`時的解，`start`比`end`大就會跳出while迴圈，`end`還是我們的答案！  
我又花了一段時間改造成`start < end`的形式，發現找出第一個位置還好說，找出最後一個位置實在太折磨人了 :cold_sweat: ，很容易就造成TLE！後來我學習了九章大神的模板，終於拿到AC 99%了！我後面會再開一個段落去解析這個模板，一下先附上小弟的code：  
```cpp
vector<int> searchRange(vector<int>& nums, int target) {
    vector<int> res;
    if(nums.size() == 0) return vector<int>({-1, -1}); 
    if(target < *(nums.begin()) || target > *(--nums.end())) return vector<int>({-1, -1});
    int start = 0, end = nums.size() - 1, mid;
    while(start + 1 < end) {
        mid = start + ((end - start) >> 1);
        if(nums[mid] == target) end = mid;
        else if(nums[mid] < target) start = mid;
        else end = mid;
    }
    if(nums[start] == target) res.push_back(start);
    else if(nums[end] == target) res.push_back(end);
    if(res.size() < 1) return vector<int>({-1, -1});
    end = nums.size() - 1;
    while(start + 1 < end) {
        mid = start + ((end - start) >> 1);
        if(nums[mid] == target) start = mid;
        else if(nums[mid] < target) start = mid;
        else end = mid;
    }
    if(nums[end] == target) res.push_back(end);
    else if(nums[start] == target) res.push_back(start);
    return res;
}
```
其實把小於target和大於target的情況改成`mid + 1`和`mid - 1`是無任何大礙，不過時間複雜度也沒有差到一個迴圈...  

### [Find K Closest Elements](https://leetcode.com/problems/find-k-closest-elements/)
這題也是很容易就看出了binary search的特性，不過如果不知道**sliding window(滑動窗口)**的思路，這題會變得很複雜。首先附上我第一個解法，第一個算是我仰仗著這個window至少有k的大小，所以從input兩端邊量測距離邊裁減，最後一定會得出size為k的window。這大概只能拿到6%左右的成績，推測複雜度有`O(n)`吧...  
```cpp
vector<int> findClosestElements(vector<int>& arr, int k, int x) {
    vector<int> res(arr);
    while(res.size() > k) {
        if(x - *(res.begin()) > *(--res.end()) - x) res.erase(res.begin());
        else res.pop_back();
    }
    return res;
}
```  
後來又發現這個解法其實沒有想像中那麼糟... 上面是爛在我使用了erase直接把開頭刪掉，程式為了繼續維護vector就把全部的元素又往前移，相當於另外花了`O(n)`的時間。我可以改成用兩個指標去維護答案的開頭和結尾，思路還是一樣的，但成績能提升到85%左右！！  
想要把線性時間降低到對數等級，看來還是需要我們的binary search出馬 :horse: ！原本我應用binary search的思路是先找出一個最靠近`x`的數字，然後從開點向左向右擴展，直到`res`的size為`k`為只。Discussion board上`@lee215`的解法就更神了，他應用了**binary search + sliding window**來解這道題目，以下附上程式碼：  
```cpp
vector<int> findClosestElements(vector<int>& arr, int k, int x) {
    int start = 0, end = arr.size() - k;
    while(start < end) {
        int mid = start + ((end - start) >> 1);
        if(x - arr[mid] > arr[mid + k] - x) start = mid + 1;
        else end = mid;
    }
    return vector<int>(arr.begin() + start, arr.begin() + start + k);
}
```
我滴天啊，簡直簡短俐落！注意這裡的binary search是用來找我們window的開頭。舉個`<1, 2, 3, 4, 5, 6, 7>`然後`k = 3, x = 5`的例子，一開始`mid`為3，那我們要怎麼取決要不要向右移呢？  
```cpp
1, 2, 3, 4, 5, 6, 7
    < 3, 4, 5 >
       < 4, 5, 6 >
         <  5, 6, 7 >
```
其實`<3, 4, 5>`和`<4, 5, 6>`之間的比較可以理解為`<3, 4, 5, 6>`究竟要去頭還是去尾，這點從我的第一個解法就可以看出來。因此我們的5就去跟開頭的3`(mid)`和尾巴的6比較`(mid + k)`，後來發現我們的3比較遠，因此選擇`<4, 5, 6>`。再次強調一下：**我們現在打算用(k + 1)個元素的區間去頭或去尾來縮小搜索開頭的範圍**！再來就是討論各種情況了，`x`可以在我們當前區間內，也有可能不在：  
1. x不在區間內：
x在`<mid, mid + k>`的左邊：刪除尾巴，我們的開頭**至多**只能是目前的mid，不能再往右移了，因此`end = mid`。
x在`<mid, mid + k>`的右邊：刪除開頭，我們的開頭**至少**要是mid + 1，不能再向左移了，因此`start = mid + 1`。
2. x在區間內：
x在`<mid, mid + k>`中靠左：刪除尾巴...上面的第一種情況。
x在`<mid, mid + k>`中靠右：刪除開頭...上面的第二種情況。
x在`<mid, mid + k>`的中間：刪除尾巴...由於題目要求如果tied，則選擇比較小的數字，因此跟第一種情況一樣。

這邊另外再討論一下為何比較距離不是用`|x - arr[mid]|`，`|x - arr[mid + k]|`？這在`x`和`arr[mid]`，`arr[mid + k]`距離都ㄧ樣的時候會有很大概率的錯誤。e.g. `<1, 1, 2, 2, 2, 2, 2, 3, 3>`以及`x = 3`和`k = 3`，x在一開始的區間右邊，理當往右移，可是根據絕對值的邏輯，我們卻要向左移！老實說這個case真的很難想到... 總之這裏我們需要知道`x`在區間的哪個方向，因此不能用絕對值來判斷。最後以`O(log(N - k))`的時間就能找到我們的最優開頭，也是這題的最佳解！  
老實說我覺得Leetcode-cn上的解答比較親民，這邊分享一下連結給大家：[排除法（双指针）+ 二分法（Java、Python](https://leetcode-cn.com/problems/find-k-closest-elements/solution/pai-chu-fa-shuang-zhi-zhen-er-fen-fa-python-dai-ma/)  
> 迷思++：還是想不通為何是end = mid嗎？那你可能誤會了一些觀念：我們是用start和end來找mid，mid和mid + k只是幫助我們來模擬區間，真正的開頭還是在mid這邊。刪除尾巴的話，我們的開頭當然還是不動的，畢竟只是回到k的元素的狀態，我們只能限制開頭待在目前的位置。
> 另外我也有思考過，若是用start + 1 < end去找這個區間的最佳中點呢？其實這樣也會把問題複雜化，這個中點究竟是區間的偏左還是偏右呢，指標都不知道要往左往右跨多少，還不如直接指定端點，然後+k即可！

### [Cloest Binary Search Tree value](https://leetcode.com/problems/closest-binary-search-tree-value/submissions/)
這道題目被歸類在easy的等級，但是太過直觀的想法會得到不好的成績。我最初的思路是用binary search直接把tree上的node找過一遍，找最小距離的就好，這樣出來只有beat 69%左右。接下來我換個思路，找upperbound和lowerbound。以下附上小弟的代碼:  
```cpp
int ub = root->val, lb = root->val;
while(root) {
    if(target < root->val) {
        ub = root->val;
        root = root->left;
    }else if(target > root->val) {
        lb = root->val;
        root = root->right;
    }else
        return root->val;
}
return (abs(ub - target) > abs(target - lb) ? lb : ub);
```
注意這邊一定要先把兩個邊界都先初始化為`root`。如果一開始要向左search，就可以先把`root`視為upperbound，接下來的node如果再出現比`target`大的，一律可視為更接近`target`的上界。如何？想想很有道理吧！如果是小於全部input的邊界case，那麼我們的lowerbound就會一路維持在`root`的位置，然而不用擔心，因為target跟root的距離一定比跟現在upperbound的距離來得近。這個思路比上面的好多了，更有達到94%的beat數，然而我目前還無法確切分析出兩個解法為何有差這麼多的時間複雜度...

### [Search in a Sorted Array of Unknown Size](https://leetcode.com/problems/search-in-a-sorted-array-of-unknown-size/submissions/)
首先要先touch一下:rofl:因為好久沒有這麼短的時間解出medium了！這道問題用binary search的想法其實很直觀，就算不知道input的size，直接把他當成`10^4`的input，後面都當是超出範圍的數就好。而且當`reader`返回的結果是超出範圍的，我們可以毫不猶豫的再把search範圍縮小：  
```cpp
int search(const ArrayReader& reader, int target) {
    if(target >= 10000 || target <= -10000) return -1;
    int start = 0, end = 1, mid;
    while(reader.get(end) < target) {
        start = end;
        end = end << 1;
    }
    while(start <= end) {
        mid = (start + end) >> 1;
        if(reader.get(mid) >= 10000) end = end/2;
        if(reader.get(mid) == target) return mid;
        if(reader.get(mid) < target) start = mid + 1;
        else end = mid - 1;
    }
    return (reader.get(mid) == target ? mid : -1);
}
```
這裡給個小建議，前面`start`，`end`不用設`0`和`10000`。我們可以先探測應該可以包住target的範圍，這就是我第一個while在做的事。

## 九章大神模板 - 試解析
這個模板的核心概念是為了少出錯，放棄在while中就找出最優解，縮小search space到`start`和`end`，再從中找出目標。其實這樣時間複雜度也不會差多少，畢竟`O(logn)`跟`O(logn + 1)`沒差多少嘛。
```cpp
if(nums.size() == 0) return -1;
int start = 0, end = nums.size() - 1, mid;
while(start + 1 < end) {
    if(nums[mid] == target) {
        // find any position -> return mid;
        // find first position -> end = mid;
        // find last position -> start = mid;
    } else if(nums[mid] < target) start = mid;
    else end = mid;
}
// find any position, 下面兩種順序都可
// find first position,
if(nums[start] == target) return start;
if(nums[end] == target) return end;
// find last position,
// 反過來就好
```
* 用`start + 1 < end`以避免死循環。search space留下`start`和`end`，所以在迴圈外另外判斷哪一個是答案
* 在小於目標或大於目標時用`mid + 1`或`mid - 1`可能會漏解  

在這裡也附上leetcode的模板解析：[Binary Search Template Analysis](https://leetcode.com/explore/learn/card/binary-search/136/template-analysis/935/)


## 結論
我發現這些題目都有一定的**單調性**，所以我們可以透過binary search去縮小搜尋的範圍。  
* 定義start, end, 或 mid？
* 考慮邊界情況，他也會影響我們選擇`start < end` 或是 `start <= end`.
* 在選完邊後，我們就把他視為子問題，專注於解決當前的情況。  
* 永遠記得：start和end是用來幫助我們找到一個目標！因此題目的目標如果是找到一個區間，binary search可以用來找到該最優區間的**開頭**或**結尾**。

## 參考
1. [Leetcode Explore Binary Search](https://leetcode.com/explore/learn/card/binary-search/)

