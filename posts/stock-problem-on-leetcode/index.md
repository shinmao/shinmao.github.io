# Stock Problem on Leetcode


I spend much time on completing a series of stock problem on leetcode. Some problem can be solved with greedy while others can be solved with dynamic programming. However, I ran into many challenges while defining the subproblems. Therefore, I would like to take a conclusion in this article, hope someone also try to solve these problems find this article helpful!<!--more-->  
* [121. Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)  
* [122. Best Time to Buy and Sell Stock II](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/)
* [123. Best Time to Buy and Sell Stock III](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/)
* [188. Best Time to Buy and Sell Stock IV](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/)
* [309. Best Time to Buy and Sell Stock with Cooldown](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)
* [714. Best Time to Buy and Sell Stock with Transaction Fee](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

Pay attention! The writeup shown in this article won't include exception handling!

## 121. Best Time to Buy and Sell Stock
In this problem, we can only make at most one transaction to find the max profit. In this series of problem, if we buy stock on the first day, it's not necessary for us to sell on next day. We can choose one of the following days to get the max profit.  
Idea: If we can only make one transaction, we should try to find biggest price gap which is `prices[n] - prices[m]` for `m = n+1, n+2, ... the last day`. Therefore, we use two variables: `mp` to store the least price so far, and `max` to store the biggest price gap so far.  
```java
int len = prices.length;
int max = 0;
int mp = Integer.MAX_VALUE;

for(int i = 0; i < len; i++){
    if(prices[i] < mp){
        mp = prices[i];
    }
    
    if(prices[i] - mp > max){
        max = prices[i] - mp;
    }
}

return max;
```
If the biggest price gap for current price is bigger than max, then we can change it to be the max value. To the last day, we can get the max profit for making at most one transaction.  
Luckily, this problem is at the easy level, and my writeup beats over 99% of submission on leetcode.

## 122. Best Time to Buy and Sell Stock II
In this problem, we can make many transactions as we can. Before coming up with idea, we need to figure out the strategy for this problem. Following is the visualization of prices:  
```
^                  ^
|      ./          |           .
|     ./           |    .     /
|    ./            |  ./ \   /
|   ./             |      \./
|------------      |---------------
```  
There are two maps for `p1, p2, p3, p4` in above. How can we choose buy and sell for the max profit? If we look at the second picture, the max profit would be `(p2 - p1) + (p4 - p3)` which is bigger than `(p4 - p1)`. If we look at the first picture, the max profit would be `(p2 - p1) + (p3 - p2) + (p4 - p3)` which is equal to `(p4 - p1)`. In other words, we only need to consider one day for each time. If the unit price gap is positive, then we can add to the profit.  
Idea: If the price on the second day is bigger than the first day, buy it on the first day and sell it on the second day. Add the price gap to the max profit.  
```java
int profit = 0;
        
for(int i = 1; i < prices.length; i++){
    if(prices[i] > prices[i-1]){
        profit += prices[i] - prices[i-1];
    }
}

return profit;
```  
Luckily, this problem is also at the easy level, and my writeup beats over 93% of submission on leetcode.  

## 123. Best Time to Buy and Sell Stock III
In this problem, we can make at most two transactions. This is a classic DP problem. Following are several ways to define the subproblem:  
Idea: `dp[i][j]`: the max profit for completing i transactions before day j. There are two cases on the jth day: 1. **We do nothing on the jth day, so the profit remain the same**. 2. **We sell stock on the jth day, the possible day for us to buy such stock would be from 0 to (j-1)**. (We don't need to consider the case of buying on jth day because it must decrease the profit.) Therefore, our max profit can be found from following:
$$ dp[i][j-1] $$
$$ dp[i-1][0] + prices[j] - prices[0] $$
$$ dp[i-1][1] + prices[j] - prices[1] $$
$$ ... $$
$$ dp[i-1][j-1] + prices[j] - prices[j-1] $$
The first one is the case 1, we do nothing on jth day, which means we already complete i transactions on (j-1)th day. From the second one to the last one is for case 2. Following is the source code:  
```java
int[][] dp = new int[3][prices.length];
for(int i = 0; i < prices.length; i++){
    dp[0][i] = 0;
}
for(int i = 0; i < 3; i++){
    dp[i][0] = 0;
}
int maxprofit = 0;

for(int i = 1; i <= 2; i++){
    for(int j = 1; j < prices.length; j++){
        maxprofit = Math.max(0, dp[i][j-1]);
        for(int k = 0; k < j; k++){
            maxprofit = Math.max(
                maxprofit,
                dp[i-1][k] + prices[j] - prices[k]
            );
        }
        dp[i][j] = maxprofit;
    }
}

return dp[2][prices.length-1];
```
However, this solution only beats over 6% of submission on leetcode because of high time complexity: O(n^2k). After reading the discussion board, I think I can swap two loops to make it faster:  
```java
for(... j < prices.length ...){
    for(... i <=2 ...){
        // function body would also be changed
    }
}
```
The reason for being faster is that the first one switches to the next loop each time for O(n) while the next one switches to the next loop each time for O(1). However, I am still not satisfied with the solution.  
{{<admonition>}} If you are still not satisfied with the solution, define more subproblem with lower dimension. Take this problem for example, two dimensions for subproblem of first solution are number of transactions and time. If we try to lower the dimension, it could be `dp[transaction]`, `dp[time]`, or even just a variable! {{</admonition>}}  
Idea: If we define subproblem to be `dp[i]`: maxprofit got until ith day, it would be hard to define first transaction and the second transaction. However, we don't need to define array for maxprofit got until each transactions. There are four states in this problem, **buy 1nd stock -> sell 1nd stock -> buy 2nd stock -> sell 2nd stock**. We eager to get the max profit on each state. For each state, we have two choices, one is remain the same, the other one is do the operation, find the choice for the bigger profit. Following is the source code:   
```java
int buy1 = Integer.MIN_VALUE, sell1 = 0;
int buy2 = Integer.MIN_VALUE, sell2 = 0;

for(int i : prices){
    buy1 = Math.max(buy1, -i);
    sell1 = Math.max(sell1, i + buy1);
    buy2 = Math.max(buy2, sell1 - i);
    sell2 = Math.max(sell2, i + buy2);
}

return sell2;
```
Take a look at the line 5, we eager to find the max profit for buying stock on first transaction. Therefore, our two choices are: 1. remain the same as previous, which is `buy1` 2. `0 - prices[i]`, which is `-i`.  
Interesting, with this solution, we can beat over 92% of submission on leetcode.

## 188. Best Time to Buy and Sell Stock IV
In this problem, we can make at most k transactions. It is very similiar to the previous problem, so I use the same idea: `buy[0]`, `sell[0]`, `buy[1]`, `sell[1]`, ... `buy[k]`, `sell[k]`. Following is the source code:  
```java
int[] buy = new int[k + 1];
int[] sell = new int[k + 1];
buy[0] = 0;
for(int i = 1; i <= k; i++){
    buy[i] = Integer.MIN_VALUE;
}
sell[0] = 0;

for(int i = 0; i < prices.length; i++){
    for(int j = 1; j <= k; j++){
        buy[j] = Math.max(buy[j], sell[j-1] - prices[i]);
        sell[j] = Math.max(sell[j], buy[j] + prices[i]);
    }
}

return sell[k];
```
`buy[0]` and `sell[0]` are defined for the convenience, one reason is for easy understanding, another reason is for convenience. Everything seems perfect; however, there are a trap hidden in this problem. I found that if k is so big, memory time limit would be exceeded. In fact, if `k*2 >= prices.length`, we can reduce our time complexity from `O(n^2)` to `O(n)` because we can make transaction on each day, then the problem is same as <122. Best Time to Buy and Sell Stock II>.  
```java
if(k*2 >= prices.length){
    int profit = 0;
    for(int i = 1; i < prices.length; i++){
        if(prices[i] > prices[i-1]){
            profit += prices[i] - prices[i-1];
        }
    }
    return profit;
}
```
With this solution, we can beat over 95% of submission on leetcode.

## 309. Best Time to Buy and Sell Stock with Cooldown
There is no limit times for transactions, but there must be a cooldown day after each sell operations.  
Idea: The problem is different from the previous one which is limited by k times. Therefore, I define the subproblem as `buy[i]`: max profit for holding stock on ith day and `sell[i]`: max profit for not holding stock on ith day. Following is the source code:  
```java
int[] buy = new int[prices.length];
int[] sell = new int[prices.length];

buy[0] = -prices[0];
sell[0] = 0;

for(int i = 1; i < prices.length; i++){
    buy[i] = Math.max(
        buy[i-1],
        (i > 1 ? sell[i-2] : 0) - prices[i]    
    );
    sell[i] = Math.max(
        sell[i-1],
        buy[i-1] + prices[i]
    );
}
return sell[prices.length-1];
```
First, there are two possibilities for holding stock on today: One is **already holding on yesterday**, which is `buy[i-1]`. Second is **not holding on two days before and buying on today**, which is `sell[i-2] - prices[i]`. However, index need to be larger than 1.  
Second, there are also two possibilities for not holding stock on today: One is **already not holding on yesterday**, which is `sell[i-1]`. Second is **hold on yesterday and sell on today**, which is `buy[i-1] + prices[i]`.  
However, this solution only beats over 61% of submission on leetcode.  
{{<admonition>}}After reading the discussion board, I think changing the name of array from `buy[]` and `sell[]` to `hold[]` and `not_hold[]` can help more people figure it out.{{</admonition>}}  
But, I am still not satisfied with current solution. Dig into the detail of the solution above, and I found that we can just use several variables instead of using arrays. `buy[i]` is only related with `buy[i-1]` and `sell[i-2]`. `sell[i]` is only related with `sell[i-1]` and `buy[i-1]`.  
Idea: For the purpose of better understanding, I would like to change name to `hold` and `not_hold` now. The relationship between `hold[i]` and `hold[i-1]` can be defined as updating the value if bigger, and same for `not_hold`. In addition, we need another variable to store the `sell[i-2]`, which is called as `pre_not_hold`.  
```java
int hold = Integer.MIN_VALUE;
int not_hold = 0;
int pre_not_hold = 0;

for(int price : prices){
    int tmp = not_hold;
    hold = Math.max(hold, pre_not_hold - price);
    not_hold = Math.max(not_hold, hold + price);
    pre_not_hold = tmp;
}

return not_hold;
```  
Following is my explanation for the solution above:  
```
dayi        dayi+1         dayi+2
hold        hold
not hold    not hold
 |_______> pre_not_hold
```
Nice, now our solution beats 100% of submission on leetcode!!

## 714. Best Time to Buy and Sell Stock with Transaction Fee
There is no limit times for transactions, but there is transaction fee with each transaction **after sell the stock**. Therefore, I would combine operation of sell and paying for fee to one state.  
Idea: I use the same idea for array in the previous problem.  
```java
int[] hold = new int[prices.length];
int[] not_hold = new int[prices.length];
hold[0] = -prices[0];
not_hold[0] = 0;

for(int i = 1; i < prices.length; i++){
    // today hold:
    // 1. yesterday already hold and today do nothing
    // 2. yesterday not hold and today buy
    hold[i] = Math.max(hold[i-1], not_hold[i-1] - prices[i]);
    // today not hold:
    // 1. yesterday already not hold and today do nothing
    // 2. yesterday buy and today sell and pay for transaction fee
    not_hold[i] = Math.max(not_hold[i-1], hold[i-1] + prices[i] - fee);
}

return not_hold[prices.length-1];
```
However, I am not satisfied with my solution because it only beats over 53% of submission on leetcode. Therefore, I would make a same change as previous problem to my solution. Following is my source code:  
```java
int not_hold = 0;
int hold = -prices[0];
int pre_hold = -prices[0];

for(int price : prices){
    hold = Math.max(hold, not_hold - price);
    not_hold = Math.max(not_hold, pre_hold + price - fee);
    pre_hold = hold;
}

return not_hold;
```  
Now, my solution beats over 94% of submission on leetcode!

## Conclusion
When we run into a problem, the first thing we should do is to draw a state machine map. How many states does the problem have? What is the relationship between these states? Then how to define subproblem? Which kind of subproblem can lead us to a convenient solution? Don't pursuit the most efficient solution at first, but implement with the solution which is easy to understand at first, and try to simplify our first solution to get higher grade.  
To summarize this series of problems, I find that if number of transactions is limited, we need to define subproblem of each time transaction, e.g. `buy1`, `sell1`, `buy2`, `sell2` ... `buyk`, `sellk`. Also we can turn it back to the problem of arbitrary k if k is bigger then [prices.length]/[2]! If k is arbitrary, we can use `hold1`, `not_hold1` ... `holdi`, `not_holdi` where i is the day of price to define the subproblem, then update the max value of `hold` and `not_hold` on each day.  
When I read through the solution on discussion board, some solution comes up with the most votes; however, it is hard for me to understand. Therefore, I think solution is not absolute, but we should choose presonally the one which is easy to understand for us.  

[This is also a good article to read for this series of stock problem](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/discuss/108870/Most-consistent-ways-of-dealing-with-the-series-of-stock-problems)
