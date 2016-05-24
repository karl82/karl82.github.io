---
title: Unit testing `equals` and `hashCode`
category: java
tags: [java, lombok, unit tests]
---

I had to add few fields into existing class wchih was simple POJO, but was used
in collections. And had `equals` and `hashCode` defined. I forgot to update
`hashCode`. _Which I discovered when I was checking my change before sending it to code
review. World is saved. Go sleep._

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

```java
new EqualsTester()
    .addEqualityGroup(
        new Foobar("hello"),
        new Foobar("hello"))
    .addEqualityGroup(
        new Foobar("bye"))
    .testEquals();
```

#Unit testing `hashCode()` is hard

Wait. It is easy too! Use
[EqualsTester](https://static.javadoc.io/com.google.guava/guava-testlib/19.0/com/google/common/testing/EqualsTester.html)
too. According to javadoc:

> This tests:
> * ...
> * the hash codes of any two equal objects are equal

That's great, because you will follow the contract. And that's probably the only
thing that you can test about hash code. You cannot really test when 2 objects
are not equal, then hash code shouldn't be equal too. Maybe for simple classes
where hash is taken from id on integer range.

The problem is that if you forget to update `hashCode`, you will live in
unconsciousness that your `hashCode` method is ineffective and producing a lot
of collisions during insertions into hashed collections. You will very likely
discover it during integration tests when the time needed to execute test will
dramatically increase. The worse scenario is to discover such mistake in
production...

#How to deal with `equals` and `hashCode` invariant

To ensure that you covered all fields in `equals` and `hashCode` you can:

* Use code generator, like [Project Lombok](https://projectlombok.org/) and it's
  [@EqualsAndHashCode](https://projectlombok.org/features/EqualsAndHashCode.html)
  annotation.
* Write your own tester to which you put list of fields which should participate
  on the equality and hash codes. It will analyze bytecode of the methods if
  fields are read.

##@EqualsAndHashCode

I'm big fan of this approach. If your class is annotated just with
`@EqualsAndHashCode`, Lombok will generate `equals` and `hashCode` from all
non-static, non-transient fields. You will never miss any new field! You have to
rely on another framework, but it will save a lot of pain and makes your code
readable. You should definitely use `EqualsTester` to ensure correctness of
generated code!

If you do not want to cover all fields, you can use `exclude` or `of` attributes
to define your own set of fields which should be used for `equals` and
`hashCode` generation. The advantage is that you have to manage just only one
list of fields, clearly visible, not hidden somewhere in the class body and it's
methods in two places!

```java
@EqualsAndHashCode(of={"foobar1"})
public class MyFoobar {
    private final int foobar1;
    private final int foobar2;

    public MyFoobar(int foobar1, int foobar2) {
       this.foobar1 = foobar1;
       this.foobar2 = foobar2;
    }
}

...

public class MyFoobarTest {
    @Test
    public void equalsAndHashCode() {
        new EqualsTester()
            .addEqualityGroup(
                new MyFoobar(1, 2),
                new MyFoobar(1, 999))
            .addEqualityGroup(
                new MyFoobar(999, 1))
                new MyFoobar(999, 999))
            .testEquals();
    }
}
```

##Checking legacy code with bytecode analyses

This idea came to my mind during writing this post. What if you do not want to
invest into rewriting your code base into Lombok? You would be happy if there is
some tool which can analyse bytecode of existing classes and it will report
discrepancies in usage of fields between `equals` and `hashCode`. Is it even
possible?

I checked [FindBugs](http://findbugs.sourceforge.net/) if they provide such
functionality and didn't find it.

So I quickly implemented PoC
[here](https://github.com/karl82/equalshashcodereporter). It is using
[ASM](http://asm.ow2.org/) for bytecode analyses. Usage is very simple as
feature set.

* Currently detects only direct access to fields which are assigned in
  constructors
* If field is `final` and value is known during compilation, no `GETFIELD`
  operand is generated :(

```java
package cz.rank.tests;

import org.junit.Test;

import java.util.Objects;

public class EqualsHashCodeReporterTest {
    @Test
    public void emptyEqualsAndHashCodeProduceNoReport() throws Exception {
        new EqualsHashCodeReporter(Object.class).report();
    }

    @Test(expected = AssertionError.class)
    public void onlyInHashCodeProducesException() throws Exception {
        new EqualsHashCodeReporter(OnlyHashCodeObject.class).report();
    }

    @Test(expected = AssertionError.class)
    public void onlyInEqualsProducesException() throws Exception {
        new EqualsHashCodeReporter(OnlyEqualsObject.class).report();
    }

    @Test(expected = AssertionError.class)
    public void differentFieldsProduceException() throws Exception {
        new EqualsHashCodeReporter(EqualsAndHashCodeDifferentObject.class).report();
    }

    private static class OnlyHashCodeObject {
        private final int field;

        private OnlyHashCodeObject(int field) {
            this.field = field;
        }

        @Override
        public int hashCode() {
            return Objects.hash(field);
        }
    }

    private static class OnlyEqualsObject {
        private final int field;

        private OnlyEqualsObject(int field) {
            this.field = field;
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (!(o instanceof OnlyEqualsObject)) return false;
            OnlyEqualsObject that = (OnlyEqualsObject) o;
            return field == that.field;
        }
    }

    private static class EqualsAndHashCodeDifferentObject {
        private final int field;
        private final int field2;

        private EqualsAndHashCodeDifferentObject(int field) {
            this.field = field;
            this.field2 = field << 2;
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (!(o instanceof EqualsAndHashCodeDifferentObject)) return false;
            EqualsAndHashCodeDifferentObject that = (EqualsAndHashCodeDifferentObject) o;
            return field == that.field;
        }

        @Override
        public int hashCode() {
            return Objects.hash(field, field2);
        }
    }
}
```
