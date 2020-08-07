# The Skyline Problem


[UVa 105 - The Skyline Problem](https://onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=41)  

This problem tests the application of priority queue which is implemented with min-heap. Min-heap takes `O(logn)` to insert a new node. It takes `O(1)` to extract current minimum value and `O(logn)` to sort again. With this data structure, I can iterate the boundaries of building in ascending order. My idea is recording all boundaries and associated heights in `priority queue` and current existing height in `multiset`. Poping out from priority queue for boundaries in ascending order, if the boundary is **start boundary**, I would insert into the `multiset`; if **end boundary**, I would delete the height from the set. Each time reach the boundaries of building, if current height is different from the previous height, then we can print out the boundary and height.  

## Writeup
I ignored a case at first and got WA for several times: **Same start boundaries with different heights**. For example, in the case of `1 11 5 1 10 5...`, if we use `greater comparator` for `priority queue`, it would sort keys in ascending order; if keys are same, it would sort value in ascending order. Iterating key in ascending order is for the purpose of iterating boundaries, but if the start boundaries are same, we should print the largest height, which should be **descending order**. Therefore, I customize the comparator: if boundaries are same, make descending order.  
Another easy trick: For the purpose of separating start and end boundaries, some people would wrap pair with pair, e.g. `pair<boundary, pair<1 for start and -1 for end, h>>`. However, the easier way is `height * 1` for start and `height * -1` for end. Then we can use `pair<boundary, height>` to implement it.  
Following is the source code in C++:  
```cpp
#include <bits/stdc++.h>
#define _ ios_base::sync_with_stdio(0);cin.tie(0);
#define F first
#define S second
#define mp make_pair
using namespace std;
typedef pair<int, int> pp;

struct cmp {
	bool operator()(pp a, pp b) {
		if(a.first == b.first) return a.second < b.second;    // descending with second
		return a.first > b.first;       // ascending with first
	}
};

int main() { _
    int l, h, r, prevh;
    multiset<int> s;
    s.insert(0);
    priority_queue<pp, vector<pp>, cmp> pq;
    while(cin >> l >> h >> r) {
    	pq.push(mp(l, h));
        pq.push(mp(r, h * -1));
    }
    prevh = 0;
    while(!pq.empty()) {
    	auto hd = pq.top();
    	pq.pop();
    	if(hd.S > 0)
    		s.insert(hd.S);
    	else
    		s.erase(s.find(hd.S * -1));
    	if(pq.empty())
    		cout << hd.F << " 0" << '\n';
		else if(*(--s.end()) != prevh) {
			cout << hd.F << ' ' << *(--s.end()) << ' ';
			prevh = *(--s.end());
		}
	}
    return 0;
}
```

## Feedback
I don't know why I get segmentation fault again and again on TIOJ 1202... However, I can get AC on UVa. If you want to take this problem, take care of the output format!
