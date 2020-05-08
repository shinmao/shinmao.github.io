# Make it Generic


# Introduction
Since Java 5, generic type has become significant part of Java. Why should we prefer generic type to raw type? Before arise of generic type, we need to convert the type of elements **while reading them in collection**. However, if the conversion fails, it would cause to runtime error. With generic type, we can tell compiler what type should be limited for collection in advance. If there exists an error, get compiler error before it becomes a runtime error is much better, isn't it?<!--more-->
<!--more-->

## Make it generic
```java
public class BoundedQueue {

    protected Object rep[];
    protected int front = 0;
    protected int back = -1;
    protected int size = 0;
    protected int count = 0;

    public BoundedQueue(int size) {
        if (size <= 0){
            throw new IllegalArgumentException("Illegal size!")
        }
        this.size = size;
        rep = new Object[size];
        back = size - 1;
    }

    public boolean isEmpty() {
        return (count == 0);
    }

    public boolean isFull() {
        return (count == size);
    }

    public int getCount() {
        return count;
    }

    public int getSize() {
        return size;
    }

    public void put(Object e) {
        if (e != null && !isFull()) {
            back++;
            if (back >= size)
                back = 0;
            rep[back] = e;
            count++;
        }
    }

    public Object get() {
        Object result = null;
        if (!isEmpty()) {
            result = rep[front];
            rep[front] = null;
            front++;
            if (front >= size)
                front = 0;
            count--;
        }
        return result;
    }

    public static void main(String args[]) {
        BoundedQueue queue = new BoundedQueue(10);
        for (int i = 0; !queue.isFull(); i++) {
            queue.put(Integer.valueOf(i));
            System.out.println("put: " + i);
        }
        while (!queue.isEmpty()) {
            System.out.println("get: " + queue.get());
        }
    }
}
```
In the course of SWE619, we are required to make the class `BoundedQueue` generic (size bounded collection). We also need to eliminate all the compiler warnings, unnecessary instance variable, and add necessary exception handling.  
From the perspective of designer, we don't know which type would be put into collection; therefore, we would use generic type to define class first, e.g. `BoundedQueue <E>`. Based on Bloch28: prefer lists to arrays, we change variable `Object rep[];` to `List<E> rep;` and use ArrayList to implement it in constructor `rep = new ArrayList<E>();`. To change `put()` to a generic method, start from the parameter:  
```java
// Bloch30: generic method
public void put(E e) {
    if (e == null){
        throw new NullPointerException("e should not be null");
    }
    if (isFull()){
        throw new IllegalArgumentException("queue is full");
    }
    rep.add(e);
    count++;
}
```
Due to the fact that we use ArrayList to implement BoundedQueue, we don't need `front` and `back` to help us index anymore. We would also eliminate two variables from above. Next, we are going to change `get()` to a generic method:  
```java
// Bloch30: generic method
public E get() {
    if (isEmpty()) {
        throw new NoSuchElementException("Queue is empty");
    }
    result = rep.get(0);
    rep.remove(0);
    count--;
    
    return result;
}
```  
The change to `get()` is similar to `put()`, so I won't go into the detail. Here, we are required to add two more methods: `putAll()` and `getAll()`. These two methods are different from `put()` and `get()` methods, they would iterate the collection and take action on each elements.  
```java
// Bloch31
public void putAll(Collection<? extends E> list){
    if(this.count + list.size() >= this.size()){
        throw new IllegalArgumentException("Queue is full");
    }
    for(E e : list){
        push(e);
    }
}

public void getAll(Collection<? super E> col){
    if(isEmpty()){
        throw new IllegalArgumentException("Queue is empty");
    }
    while(!isEmpty()){
        col.add(get());
    }
}
```
Here you must be confused, why not use `Collection<E>` as the parameter? First of all, we should get into a question: Can `List<Integer>` be put into `List<Number>` as parameter? Based on our common sense, this should be work because Integer is subtype of Number. However, the answer is **NO** due to the **incovariance** of generic type. Incovariance means that any `List<type1>` would not be subtype or supertype of any `List<type2>`. When we new the class in main function, we should use parameterized type instead of generic type `E`. Although using `Collection<E>` won't cause to the compiler warning, but the best choice is using **bounded wildcard type**. Bounded wildcard type can make it implemented based on the common sense, e.g. we can call `getAll()` to add from `BoundedQueue<Number>` to `Collection<Object>`. Secondly, someone might be interested in the use of `extends` or `super` in bounded wildcard type. The use of `extends` and `super` is based on PECS mnemonic: Producer-Extends Consumer-super. I would use two samples above to explain the rule: In `putAll()`, `list` is the producer of E instance, so we should use `extends`. In `getAll()`, `col` is the consumer to get E instance, so we should use `super`.

## Conclusion
Now we have understood the importance of generic type. We should use generic type instead of raw type to avoid `ClassCastException` of runtime error. Generic type is the way to keep application typesafe. In addition to the example I gave above, I strongly recommend to read through the Chapter5.Generics of Effective-Java-3rd, there are more details worth getting into.

## Reference
* [Effective-Java-3rd]()
* [BoundedQueue.java](https://cs.gmu.edu/~pammann/619/code/BoundedQueue.java)

