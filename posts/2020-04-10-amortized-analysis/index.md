# Amortized analysis


# Why we need amortized analysis?
We always use worst case analysis (big-O) to calculate time complexity. However, have you ever thought that we would overestimate the cost? In specific algorithm, there is a sequence of operations with different costs. You might said that time complexity would be `number of operations * the largest cost of operations`. For example, there are three kinds of operations for stack: push, pop, and multipop(n) which would pop out n elements. Based on knowledge of worst case analysis, the ith operation takes $ O(n) $ of time complexity in worst case, then the sequence of n operations takes $ O(n^2) $ time. But how if there is only one $ O(n) $ and the rest of them are all $ O(1) $? We have overestimate the cost of the case! Now, we know that the worst case bound is not tight because the expensive operation might not occur frequently. In amortized analysis, if sequence of operations have different costs, we can distribute the high one to the low ones. Following sections are three methods of amortized analysis.<!--more-->

> Amortized analysis helps us to obtain a more accurate bound of worst case...

## Aggregate method
Sum up the all cost of operations $ T(n) $ and divide it by number of operations $ n $. Take the stack for example,  
$ Npop + Nmultipop $ takes at most: $ O(Npush) $  
total cost: $ T(n) = Npush * O(1) + O(Npush) = O(n) $ ... linear time  
per cost: $ O(n) / n = O(1) $ ... constant time  
In aggregate method, we would get same amortized cost on each operations.

## Accounting method
Save credits from the operations which takes less cost and used for future operation that take more cost. Take the stack for example,  
`actual cost:`
$$ push: 1 $$
$$ pop: 1 $$
$$ multipop: min(|S|, k) $$
`assign each operation a valid amortized cost`
$$ push: 2 $$
$$ pop: 0 $$
$$ multipop: 0 $$
`validity check: credit always sufficient`  
$$ c1' + c2' + c3' + ... + cn' >= c1 + c2 + c3 + ... + cn $$
if you want to pop or multipop, you need to push first. so you won't run into bankruptcy forever:)  
**you should explain this**  
`calculate total amortized cost`  
$ O(n) * 2 = O(n) $ ... linear time   (The largest time of push is n)  
$ O(n) / n = O(1) $  
Be careful, check that you won't run into bankruptcy no matter which situation. If amortized cost is bigger than actual lost, the difference becomes a credit, otherwise it would become a withdrawl. What is different from aggregate method, each operation can have different amortized cost.

## Potential method
The core of potential method is potential function $ \Phi(Di) $. Do: the initial state of data structure, Di: the state after ith operation, ci: the actual cost of ith operation, ci': the amortized cost of ith operation which is $ ci + \Phi(Di) - \Phi(Di-1) $. Take the stack for example:  
` design potential function `
$$ \Phi(Di) $$ `the number of elements on stack after ith operation`  
` validity check `  
$$ Sum(ci') = Sum(ci + \Phi(Di) - \Phi(Di-1)) $$
$$ = Sum(ci) + Φ(Dn) - Φ(Dn-1) + Φ(Dn-1) - Φ(Dn-2) + Φ(Dn-2) - Φ(Dn-3) + ... Φ(D1) - Φ(D0) $$
$$ = Sum(ci) + Φ(Dn) - Φ(D0) $$
$$ => Φ(Di) - Φ(D0) >= 0 $$
`we usually make it Φ(D0) = 0, and Φ(Di) >= 0`

`In this example, Φ(D0) = 0 elements, and Φ(Di) >= 0 elements. Great!`  

`Calculate each operation amortized cost:` $ ci + \Phi(Di) - \Phi(Di-1) $
$$ push: 1 + (|s| + 1) - (|s|) = 2 $$
$$ pop: 1 + (|s| - 1) - (|s|) = 0 $$
$$ multipop: min(|s|, k) + (|s| - min(|s|, k)) - (|s|) = 0 $$
Calculate total amortized cost, all operations has O(1) amortized cost, total O(n).  

What's different from accounting method, the credit is assigned to data structure itself.  

> Is it possible for amortized cost to be negative? Yes, it is possible for single operation to have negative amortized cost. But for n operations, total amortized cost should always be non-negative.

## Reference
* [Amortized Cost - can it be negative?](http://ds09.wikidot.com/forum/t-194836/amortized-cost-can-it-be-negative)
