# Implementation of PLA


In this article, I would show the by-hand implementation of PLA. I won't get into the detail of the algorithm, but just help someone wants some practical experience. Also, I recommend everyone to implement it because only do it will you find the points you still not figure out about it.  

## Dataset
Here we have ten training sets and one test set. In datasets, we have two features and classification results in `(0, 1)`.  
```
1.513166 3.895627 0.000000 
1.499273 -1.702793 0.000000 
7.397526 6.808569 1.000000 
...
```
According to the dataset, we can formulate our PLA:  
$$ y = w1*x1 + w2*x2 + bias $$  
Now, let's start on program.

## PLA
First part is data processing.  
```py
def data_process(path):
    rawdata = open(path).readlines()
    size = len(rawdata)
    cols = len(rawdata[0].strip().split(' '))
    
    dataset = [[0.0]*cols for i in range(size)]
    idx = 0
    for line in rawdata:
        tmp = line.strip().split(' ')
        dataset[idx][0] = float(tmp[0])
        dataset[idx][1] = float(tmp[1])
        dataset[idx][2] = float(tmp[2])
        idx += 1
    return dataset
```  
In array of dataset, from index 0 to 2 are x1, x2 and labels. The goal is to get what we want easily. Next step with Core function `PLA`:  
```py
def pla(dataset):
    iteration = 0
    lr = 0.01
    w = rand_list(3)
    while True:
        finished = True
        for i in range(0, len(dataset)):
            x = dataset[i][:-1]
            y = w[0] + w[1]*x[0] + w[2]*x[1]
            if siggn(y) == dataset[i][2]:
                continue
            else:
                finished = False
                w[0] += lr*(dataset[i][2] - siggn(y))
                w[1] += lr*(dataset[i][2] - siggn(y))*x[0]
                w[2] += lr*(dataset[i][2] - siggn(y))*x[1]
        iteration += 1
        if finished == True:  # work until there is no misclassification
            break
    return w, iteration
```
With PLA, we would start from randomized parameters. Therefore, I use `rand_list` to initialize `w1`, `w2`, and `bias` in range of `[0, 1)`. PLA will correct these three parameters again and again until all results of labels become correct (Here I use `finished` to track whether we got all corrected). In the process, PLA would modify the parameters with **the difference between its output y and the correct label**.  
$$ w' = w + (label - y)*x $$  
It's said that `w^T` can decide the decision boundary, so here we change `w` by rotating toward the data points with wrong results. What is in my `siggn(y)`?  
```py
def siggn(y):
    sign = 0.0
    if y >= 0.0:
        sign = 1.0
    else:
        sign = 0.0
    return sign
```
In other words, the difference between our output and the label could only be 0.0 or 1.0.

## Conclusion
Yes, the implementation is so simple. However, there are still some details you can find it interesting or you need to be care about.  
I use each `w` retrieved from training set to work on test set:  
```py
def check_test(testset, w):
    count = 0
    for i in range(0, len(testset)):
        y = w[0] + w[1]*testset[i][0] + w[2]*testset[i][1]
        if siggn(y) == testset[i][2]:
            continue
        else:
            count += 1
    return count
```

First, I have recorded down misclassification rates and iterations time with different initial value of `w`:  
if I restrict `w` to be in range between 0 and 1:  
```
For Set1 - 
Iteration times:  8
Misclassification rate:  0.0205
For Set2 - 
Iteration times:  11
Misclassification rate:  0.005
For Set3 - 
Iteration times:  14
Misclassification rate:  0.039
For Set4 - 
Iteration times:  7
Misclassification rate:  0.005
For Set5 - 
Iteration times:  8
Misclassification rate:  0.0035
```  
However, if I don't restrict it or init in range between 0 and 100:  
```
Iteration times:  420
Misclassification rate:  0.0425
Iteration times:  448
Misclassification rate:  0.001
Iteration times:  535
Misclassification rate:  0.007
Iteration times:  169
Misclassification rate:  0.0005
Iteration times:  322
Misclassification rate:  0.0225
```
Apparently, it takes more time on iterations.  

One another thing I consider it to be important is **we will restrict our iteration time in fact**. If dataset is not linear separable, `while loop` in PLA will not stop forever. So to implement PLA, most people will set up a threshold of iterations, which show that the dataset is not linear separable if iterations time has crossed over this threshold. (We should switch to pocket PLA in this case, but this is another story :)
