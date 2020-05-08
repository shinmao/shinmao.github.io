# Functions You always need to Override


# Introduction
We always can override non-final methods in parent class. However, we also need to pay attention to the general contracts of those methods. Otherwise, they will not work correctly. In this article, I would share how I learn about the way to override specific functions correctly with effective-java-3rd.<!--more-->

# Override equals
It is not necessary to override the equals method each time in child class. But if we really need overriding `equals()` to solve our problems, our `equals()` need to obey following contracts:  
* Relexive: `x.equals(x) = true`  
* Symmetric: `x.equals(y) iff y.equals(x)`  
* Transitive: `if x.equals(y) && y.equals(z) => x.equals(z)`  
* Consistency: `x.equals(y)` should always has a same result  
* For any x not null, `x.equals(null) = false`  

The problematic cases always happen when we try to use equals between different class.  
e.g-1.  
```java
a.equals(b); => true
b.equals(a); => false
```  
I override the `equals()` of a; however, I don't override the `equals()` of b. Then, this violate the symmetric. To solve this kind of problem, how about checking the instance of object which is the parameter of `equals()`? Take a look at the e.g.-2.  

e.g.-2.  
```java
// class a
@Override public boolean equals(o){
    if(!(o instanceof a)){
        return o.equals(this);
    }
}

// class b
@Override public boolean equals(o){
    if(!(o instanceof b)){
        return o.equals(this);
    }
}

// extra sample
if(getClass() != o.getClass()){
    return ...
}
```  
This is an very interesting example because this would have a problem of stackoverflow. First, I would describe the difference between `instanceof` and `getClass`. Here I would assume that a is the parent class of b. The result of `(a instance of b)` is false and `(b instanceof a)` is true while the result of `(a.getClass != b.getClass())`. The reason is that `getClass()` would return the most cloest level of class name. But, if `a` and `b` are the same level child classes, the problem of stackoverflow would happen. `a.equals(b)` will try to call `b.equals(a)`, but `b.equals(a)` will call back to `a.equals(b)` again... then it becomes an infinite call loop!  
Here I would summarize four important points to overriding `equals()` correctly:  
1. use `==` to check object type first. It can return true directly if we get true on this check, because `==` would compare the memory addresses of two objects.  
2. use `instanceof` to check object type  
3. Cast of object to correct type  
4. check all properties match  

The properties in the point four are those **significant field** in class such like some variable or arrays. If the fields are basic type which are not float or double, we can use `==` to check equality.  
If the fields are object, we should recursively call equals on properties of the object.  
If the fields are float or double type, we should call `Float.compare()` or `Double.compare()`.  
Then here comes a sample `equals()` method:  
```java
// ClassA
// For type of parameter, we can and should only use "Object"
@Override public boolean equals(Object o){
    if(o == this){
        // o is completly same as ClassA
        return true;
    }
    if(!(o instanceof ClassA)){
        // o is even not child of ClassA
        return false;
    }
    // if a is child of ClassA, we should cast to ClassA
    ClassA a = (ClassA) o;
    // compare all signfincant fields
    return a.variable1 == variable1 && a.variable2 == variable2 ...;
}
```
At the end, `URL.equals()` might violates the consistency requirement of equals. When connected to the Internet, the `equals()` method follows the steps decribed in java API; when disconnected, it performs a string compare on the two URLS [3]. In addition, you should also remember to override `hashCode()` method after you override the `equals()`.

# Override hashCode
I would only describe an interesting problem in this section:  
```java
Map<Phone, String> map = new HashMap<>();
map.put(new Phone(123), "str");
System.out.println(map.get(new Phone(123)));    // null
```
Why the result of `map.get(new Phone(123))` not `str`? The answer is simple: two key are put into the different hash buckets because two keys have different hashcode! The solution is **override hashCode() method** with matching significant fields.

# Override clone
Here comes the contract we should obey while overriding `clone()` method:  
* `x.clone() != x`  
* `x.clone().getClass() == x.getClass()`  
* `x.clone().equals(x) == true`  

**Always overriding clone() with super.clone()**!! In addition, immutable object shoud not provide `clone()`.  
```java
@Override public PublicNum clone(){
    try {
        return (PublicNum) super.clone();
    } catch (CloneNotSupportedException e){
        throw new AssertionError();
    }
}
```  
Here we should make sure we return a precise object type instead of `public Object clone()`. This way can help client not need to cast again in client side. But, if object with mutable references, **deep copies** are required:  
```java
private Object[] elements;

@Override public Stack clone(){
    try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportedException e){
        throw new AssertionError();
    }
}
```  
However, if there are more mutable reference inside the mutable reference, we would need **deeper copies**:  
```java
public class HashTable implements Cloneable {
    private Entry[] buckets;
    private static class Entry {
        final Object key;
        Object value;
        Entry next;
        Entry (Object key, Object value, Entry next){
            this.key = key;
            this.value = value;
            this.next = next;
        }
        Entry deepCopy(){
            return new Entry(key, value, next == null ? null : next.deepCopy());
        }
    }
    ...
    @Override public HashTable clone(){
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];
            for(int i = 0; i < buckets.length(); i++){
                if(buckets[i] != null){
                    result.buckets[i] = buckets[i].deepCopy();
                }
            }
            return result;
        } catch (CloneNotSupportedException e){
            throw new AssertionError();
        }
    }
}
```
However, there are much better ways to clone a object, the ways are called **Copy Constructor** and **Copy Factory**.  
e.g. Copy Constructor  
```java
Class A {
    public A(){

    }
    // copy constructor
    public A(A copiedA){
        // shallow copy or deep copy
    }
}
```  
e.g. Copy Factory  
```java
public static A newA(A copiedA){
    return new A(copiedA);
}
```  

> I was confused about the use of result.elements = elements.clone(); in example. The elements is the private property of class. Why can we access to the private property of other instance? The answer is the private modifier does not mean that only the same instance can access to the field, but only objects of the same class can access to it!! I think this is very important fact each developers should understand.

# Exercise
Here is the class exercise of `IntSet`
```java
public class IntSet implements Cloneable{
    private List<Integer> els;

    @Override public boolean equals(Object obj){
        if (obj == this) {
            return true;
        }
        if(!(obj instanceof IntSet)){
            return false;
        }
        IntSet result = (IntSet) obj;
        return result.els.equals(this.els);
    }

    @Override public int hashCode(){
        int result = 0;
        for(Integer i : els){
            result = result * 31 + Integer.hashCode(i);
        }

        return result;
    }

    public IntSet (){
        els = new ArrayList<Integer>();
    }

    private IntSet (List<Integer> list){
        els = list;
    }
    
    public List<Integer> getEls(){
    	return els;
    }
    
    public void setEls(List<Integer> els) {
    	this.els = els;
    }
 
    @Override public IntSet clone(){
        try{
            IntSet result = (IntSet) super.clone();
            result.els = new ArrayList<Integer>(els);
            return result;
        } catch (CloneNotSupportedException e){
            throw new AssertionError();
        }
    }
```
Someone would implement `clone()` with `return new IntSet(new ArrayList<Integer>(els));`. However, it is wrong way. I assume `IntChild` is a subclass extending to `IntSet`. If `IntChild` doesn't override the `clone()` method and I try to call `IntChild.clone()`, it would call `IntSet.clone()` and return the wrong type of object.

# Reference
1. [Effective-java-3rd: Chapter10~13]()  
2. [IntSet.java](https://cs.gmu.edu/~pammann/619/inClass/IntSet.java)  
3. [MET08-J. Preserve the equality contract when overriding equals method](https://wiki.sei.cmu.edu/confluence/display/java/MET08-J.+Preserve+the+equality+contract+when+overriding+the+equals%28%29+method)
