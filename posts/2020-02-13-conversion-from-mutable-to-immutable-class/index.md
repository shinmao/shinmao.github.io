# Conversion from mutable to immutable class


When it comes to immutability of java, it's important for us to know about data type of **String**. Each time we try to modify the string, it would create a new object with new value. When it comes to mutability of java, the sample data type for it would be such like array, and we can modify the elements on specific index. However, it is a more interesting topic in this article: **How to create an immutable class by ourselves?**<!--more-->

## Mutable first
Here comes with a mutable class first
```java
public class Queue<E>{
    private List<E> elements;
    private int size;

    public Queue(){
        this.elements = new ArrayList<E>();
        this.size = 0;
    }

    public void enQueue(E e){
        elements.add(e);
        size++;
    }

    public E deQueue(){
        if(size==0) throw new IllegalStateException("Queue.deQueue");
        E result = elements.get(0);
        elements.remove(0);
        size--;
        return result;
    }

    public static void main(String args[]){
        Queue q = new Queue();
        q.enQueue("cat");
        q.enQueue("dog");
        System.out.println(q.deQueue());
    }
}
```
It is very easy for us to understand the function of the class above. The class would create a list and provide user with choices of adding and removing elements from the list.

## Give me an Immutable Queue
Now, change the client interaction in main function to create an immutable version of class Queue. Let's go back to the object **String** again. Each time we change the string, it would create a new object and refer to it. Therefore, what we need to do here is also **create a new object and assign the value to it**!
```java
IQ q = new IQ();
q = q.enQueue("cat");
q = q.enQueue("dog");
System.out.println(q.head());
q = q.deQueue();
```
Therefore in the whole process, we take interaction with 4 IQ objects.

## Reference
* [Class note of SWE619 Object oriented programming in java](https://cs.gmu.edu/~pammann/619.html)  
* [Minimize accessibility and mutability (Recommend)](https://blog.1pwnch.com/posts/2020-04-28-some-points-to-design-classes-and-interface/#minimize-accessibility-and-mutability)
