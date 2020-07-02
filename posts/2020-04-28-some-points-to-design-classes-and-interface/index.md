# Some Points to design Classes and Interfaces

  
This article would focus on access control of class, importance of mutability, choice between interface and abstrace class. Hope you enjoy it! <!--more-->

## Minimize accessibility and mutability
Most people would use modifier such as `private`, `default`, `protected`, or `public` to control the accessibility of classes or members. Otherwise, using `final` can prevent even the state of public member from being modified. However, there is a potential security hole:  
```java
public static final Type[] values = {...};
```  
The problem is that `values` itself is final, but entries in values are not. **The array with non-zero size is always mutable**. Therefore, giving the accessor for this kind of fields must cause to the security hole. You can use following way to solve this problem:  
```java
private static final Type[] values = {...};
public static final Type[] values(){
    return values.clone();
}
```  
From the example above, we should always make sure the fields referred by `public static final` is **immutable**. For accessibility, public classes should designed to have private fields to follow encapsulation. Before talking about the way to create immutable classes, we should know the reason why immutability is necessary? The reason is that **an immutable instance is thread-safe even accessed from multiple threads**. Following are the five rules to create an immutable class:  
1. Don't provide mutator method  
2. Make sure the class not to be extended
We always like to use `final` to make sure of this point; however, sometimes making class final is too strong. We can use static factory to replace it:  
```java
public class Complex {
    private final double x;
    private final double y;
    private Complex(double x, double y){   // priv constructor
        this.x = x;                         // constructor cannot use static
        this.y = y;
    }
    public static Complex valueOf(double x, double y){  // static factory
        return new Complex(x, y);
    }
}
```  
3. Set all fields final  
4. Set all fields private  
5. Ensure exclusive access to any mutable components  

## Favor composition over inheritance
We all love java, and love the inheritance of java. However, inheritance would break encapsulation. Assume a subclass extends to a superclass. When the version code of superclass is updated and subclass isn't updated accordingly, it would cause error disaster to subclass. For example:  
```java
public class InstrumentedHashSet<E> extends HashSet<E>{
    private int addCount = 0;
    public InstrumentedHashSet(){

    }
    public InstrumentedHashSet(int initCap, float loadFactor){
        super(initCap, loadFactor);
    }
    @Override public boolean add(E e){
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection<? extends E> c){
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount(){
        return addCount;
    }
}
// Consider the client code
InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
s.addAll(List.of("Hi", "HelloWorld"));
```  
We expect to get 2 from `s.getAddCount()`; however, we will get 4 at the end. But why? The reason is the internal working of hashset. Hashset would call `add()` when calling `addAll` (Well...it is not described in the javadoc = =). First of all, `s.addAll()` would set `addCount` as 3 with `addCount += c.size()` then call `HashSet.addAll()`. Secondly, `HashSet.addAll()` would call `add` for two times. However, due to dynamic dispatching, it would invoke `s.add()` and add `addCount` two more times. As a result, `addCount` is 4 at the end.  
You might think we should overriding methods more carefully, for example, don't call `super.addAll()`. However, if we rewrite the whole function body, it would cost much effort and time. Following is the way of using elegant composition to prevent the problem:  
```java
// ForwardingSet
public class ForwardingSet<E> implements Set<E>{
    private final Set<E> s;
    public ForwardingSet(Set<E> s){
        this.s = s;
    }
    public boolean add(E e){
        return s.add(e);
    }
    public boolean addAll(Collection<? extends E> c){
        return s.add(c);
    }
    @Override boolean equals(Object o){
        return s.equals(o);
    }
    // all the other Set.methods
    // IDE would help you to list all the methods need to be implemented
}
// InstrumentedSet
public class InstrumentedSet<E> extends ForwardingSet<E>{
    private int addCount = 0;
    public InstrumentedSet(Set<E> s){
        super(s);
    }
    @Override public boolean add(E e){
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection<? extends E> c){
        addCount += c.size();
        return super.add(c);
    }
    public int getAddCount(){
        return addCount;
    }
}
```  
This example implements the interface of Set instead of HashSet. But, we can pass any kind of set such like HashSet or TreeSet into `InstrumentedSet(new HashSet<>(...))` and `InstrumentedSet(new TreeSet<>(...))`.  
At the end, I would like to mention two interesting exercises for my course:  
In the example of `InstrumentedSet`, does the equals method satisfy the contract?  
The answer is Yes. Assume there are two instances of `InstrumentedSet`. When we try to call `s1.equals(s2)`, it would return true if **s2 is also a set**, **s1 and s2 have the same size**, and **every members in s2 is contained in s1**. The difinition ensures equals method works properly across different implementations of the set interface [2].  
If we replace Set with Collection, does the equals method still satisfy the contract?  
The answer is No. Assume there are two instances of `InstrumentedCollection`. We already know that `Set` and `List` extend to the interface of Collection. However, the contracts for `List.equals` and `Set.equals` state that lists are only equals to other lists, and sets to other sets. Therefore, **a custom equals method for collection class implemented with neither the List nor Set must return false when this collection is compared to any list or set** [3].  
```java
Set<String> r = new HashSet<String>();
r.add("ant"); r.add("bee");

InstrumentedCollection<String> s = new InstrumentedCollection<String>(r);
assertTrue(s.equals(s));   // fail
```
Let's look more deeply into this example. After we pass the hashset into the constructor of `InstrumentedCollection`, `s.equals()` would call equals method in `ForwardingCollection`. Take a look at the function body of `equals`, it returns `s.equals(o)` which means hashset.equals(o) in this example. Therefore, it would check the parameter of `equals()` is also set or not; however, `s` implements the interface of `Collection`. The result must be false.  

## Don't call overridable methods in constructor
Constructor mustn't call overridable method directly or indirectly. Before we take a look at example, we need to make sure you know how to invoke right method:  
```java
public class Super {
    public Super(){
        System.out.println("Super");
        overrideMe();
    }
    public overrideMe(){

    }
}
public class Sub extends Super {
    public Sub(){
        System.out.println("Sub");
    }
    public overrideMe(){
        System.our.println("Hi");
    }
}
// client code
Sub sub = new Sub();
Sub.overrideMe();
```  
When the constructor of `Sub` is called, it would call the constructor of `Super` first because `Sub extends Super`. Then `Super.constructor` call `overrideMe()`; however, it would call `Sub.overrideMe` due to dynamic dispatching. At the end, it would call constructor of `sub` to build up the instance of subclass. Therefore, the example would print out: Super, Hi, Sub, Hi. After we understand this example, we can go to the following example to see why constructor cannot call overridable methods:  
```java
public class Super {
    public Super(){
        overrideMe();
    }
    public void overrideMe(){}
}
public final class Sub extends Super{
    private final Instance instant;

    public Sub(){
        instant = Instance.now();
    }
    @Override public void overrideMe(){
        System.out.println(instant);
    }
}
// With the same example as above
```  
When constructor of `Super` tries to call `overrideMe()` of `Sub`, `Super` cannot find the private `instant`. Therefore, `instant` is a null object for `Super`, the it would print out `null` at the first time. In fact, it would always cause to `NullPointerException`, it can print null here because `println()` accepts null as parameters.  
In addition, `clone()` and `readObject()` should also not call overridable methods because they are simaliar to constructor.  

## Prefer Interface to Abstract class
If you still don't know what the abstract class is, it is the class which includes abstract method. They are the only two things able to have blank function body in Java. The methods in abstract class isn't necessary to be abstract, but abstract funcions should be in abstract class. In addition, abstract class can only be extended but cannot be instantialized. Back to the points of methods, we can define the default behavior of function in abstract class, but Interface prohibit it strictly (However, it is also permitted in Java8...). Here are some reasons for preferring Interface to Abstract class:  
1. Existing class can be easily retrofitted into a new interface, but not for abstract class because we cannot extends to multiple classes at the same time.  
2. We can merge several interfaces into an existing class as optional functionality, but we can't do that for abstract class also because not able to extend to multiple classes at the same time.  

However, if interface provides with so many APIs which are not required for current application, it would cause to the blank implementation in implementer classes. Here comes with a solution: **Abstract Interface** which combines the advantages of both abstract class and interface.  
```java
// pseudo code
Interface HelloWorld {
    method1(){}
    method2(){}
    method3(){}
}
public abstract class Hi implements HelloWorld {
    public abstract void method1(){}   // abstract method: subclass should override
    public abstract void method2(){}
    public void method3(){
        return true;
    }
}
public class H1 extends Hi {
    public void method1(){
        ...
    }
    public void method2(){
        ...
    }
    // H1 doesn't need method3
}
public class H2 extends Hi {
    public void method1(){
        ...
    }
    public void method2(){
        ...
    }
    public void method3(){
        return false
    }
}
```  
With abstract interface, in abstract class which implements interface, we can define the necessary methods as abstract methods because they should be overriden. Otherwise, we can define the default body of the method which is not necessary for some subclasses. We would like to apply this trick to the Interface which provides several methods such like `AbstractList` for `List` interface.


## Reference
1. [Effective-java Item 15~22]()  
2. [@Javadoc Set.equals()](https://docs.oracle.com/javase/7/docs/api/java/util/Set.html#equals(java.lang.Object))  
3. [@Javadoc Collection.equals()](https://docs.oracle.com/javase/7/docs/api/java/util/Collection.html#equals(java.lang.Object))  
4. [IBM: Figuring out abstract class and interface](https://www.ibm.com/developerworks/cn/java/l-javainterface-abstract/index.html)
