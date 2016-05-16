---
title: Unit testing `equals` and `hashCode`
category: java
tags: [java, lombok, unit tests]
---

I had to add few fields into existing class wchih was simple POJO, but was used
in collections. And had `equals` and `hashCode` defined. I forgot to update
`hashCode` (which I discovered when I read my change before sending it to code
review).

#General contract for `equals` and `hashCode`

From
[Object.equals](https://docs.oracle.com/javase/7/docs/api/java/lang/Object.html#equals%28java.lang.Object%29)
> The `equals` method implements an equivalence relation on non-null object
> references:
> * It is _reflexive_: for any non-null reference value `x`, `x.equals(x)` should
>   return `true`.
> * It is _symmetric_: for any non-null reference values `x` and `y`,
>   `x.equals(y)` should return `true` if and only if `y.equals(x)` returns
>   `true`.
> * It is _transitive_: for any non-null reference values `x`, `y`, and `z`, if
>   `x.equals(y)` returns `true` and `y.equals(z)` returns `true`, then `x.equals(z)` should
>   return `true`.
> * It is _consistent_: for any non-null reference values `x` and `y`, multiple
>   invocations of `x.equals(y)` consistently return `true` or consistently return
>   `false`, provided no information used in `equals` comparisons on the objects is
>   modified.
> * For any non-null reference value `x`, `x.equals(null)` should return `false`.

From
[Object.hashCode](https://docs.oracle.com/javase/7/docs/api/java/lang/Object.html#hashCode%28%29)
> The general contract of `hashCode` is:
> * Whenever it is invoked on the same object more than once during an execution
>   of a Java application, the `hashCode` method must consistently return the same
>   integer, provided no information used in `equals` comparisons on the object is
>   modified. This integer need not remain consistent from one execution of an
>   application to another execution of the same application.
> * If two objects are equal according to the `equals(Object)` method, then
>   calling the `hashCode` method on each of the two objects must produce the same
>   integer result.
> * It is not required that if two objects are unequal according to the
>   `equals(java.lang.Object)` method, then calling the `hashCode` method on each
>   of the two objects must produce distinct integer results. However, the
>   programmer should be aware that producing distinct integer results for unequal
>   objects may improve the performance of hash tables. 

It's not really easy to meet at requirements in the contract.  Very good article
[How to Write an Equality Method in
Java](http://www.artima.com/lejava/articles/equality.html]) written by Odersky
is diving deep into problems of `equals` and `hashCode`, so I will skip this.

I would rather focus on unit testing of equality.

#Unit testing `equals(Object)` is easy

You can write your own unit tests(and tons of code around it) or use
[EqualsTester](https://static.javadoc.io/com.google.guava/guava-testlib/19.0/com/google/common/testing/EqualsTester.html)
from guava (which is the preferable way).

#Unit testing `hashCode()` is hard

Wait. It is easy too! Use
[EqualsTester](https://static.javadoc.io/com.google.guava/guava-testlib/19.0/com/google/common/testing/EqualsTester.html)
too. According to javadoc:

> This tests:
>
> &lt;snip&gt;
> * the hash codes of any two equal objects are equal

That's great, because you will follow the contract.

The problem is that if you forget to 
