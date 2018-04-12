# Java 8 Tutorial

### Default Methods for Interfaces

Java 8 enables us to add non-abstract method implementations to interfaces by utilizing the`default`keyword. This feature is also known as**Extension Methods**. Here is our first example:

```
interface
Formula
{
double
calculate
(
int
 a
)
;
default
double
sqrt
(
int
 a
)
{
return
 Math
.
sqrt
(
a
)
;
}
}
```

Besides the abstract method`calculate`the interface`Formula`also defines the default method`sqrt`. Concrete classes only have to implement the abstract method`calculate`. The default method`sqrt`can be used out of the box.

```
Formula formula 
=
new
Formula
(
)
{
@Override
public
double
calculate
(
int
 a
)
{
return
sqrt
(
a 
*
100
)
;
}
}
;


formula
.
calculate
(
100
)
;
// 100.0

formula
.
sqrt
(
16
)
;
// 4.0
```

The formula is implemented as an anonymous object. The code is quite verbose: 6 lines of code for such a simple calucation of`sqrt(a * 100)`. As we'll see in the next section, there's a much nicer way of implementing single method objects in Java 8.

### Lambda expressions

Let's start with a simple example of how to sort a list of strings in prior versions of Java:

```
List
<
String
>
 names 
=
 Arrays
.
asList
(
"peter"
,
"anna"
,
"mike"
,
"xenia"
)
;


Collections
.
sort
(
names
,
new
Comparator
<
String
>
(
)
{
@Override
public
int
compare
(
String a
,
 String b
)
{
return
 b
.
compareTo
(
a
)
;
}
}
)
;
```

The static utility method`Collections.sort`accepts a list and a comparator in order to sort the elements of the given list. You often find yourself creating anonymous comparators and pass them to the sort method.

Instead of creating anonymous objects all day long, Java 8 comes with a much shorter syntax,**lambda expressions**:

```
Collections
.
sort
(
names
,
(
String a
,
 String b
)
-
>
{
return
 b
.
compareTo
(
a
)
;
}
)
;
```

As you can see the code is much shorter and easier to read. But it gets even shorter:

```
Collections
.
sort
(
names
,
(
String a
,
 String b
)
-
>
 b
.
compareTo
(
a
)
)
;
```

For one line method bodies you can skip both the braces`{}`and the`return`keyword. But it gets even more shorter:

```
Collections
.
sort
(
names
,
(
a
,
 b
)
-
>
 b
.
compareTo
(
a
)
)
;
```

The java compiler is aware of the parameter types so you can skip them as well. Let's dive deeper into how lambda expressions can be used in the wild.

### Functional Interfaces

How does lambda expressions fit into Javas type system? Each lambda corresponds to a given type, specified by an interface. A so called_functional interface_must contain**exactly one abstract method**declaration. Each lambda expression of that type will be matched to this abstract method. Since default methods are not abstract you're free to add default methods to your functional interface.

We can use arbitrary interfaces as lambda expressions as long as the interface only contains one abstract method. To ensure that your interface meet the requirements, you should add the`@FunctionalInterface`annotation. The compiler is aware of this annotation and throws a compiler error as soon as you try to add a second abstract method declaration to the interface.

Example:

```
@FunctionalInterface
interface
Converter
<
F
,
 T
>
{

    T 
convert
(
F from
)
;
}
```

```
Converter
<
String
,
 Integer
>
 converter 
=
(
from
)
-
>
 Integer
.
valueOf
(
from
)
;

Integer converted 
=
 converter
.
convert
(
"123"
)
;

System
.
out
.
println
(
converted
)
;
// 123
```

Keep in mind that the code is also valid if the`@FunctionalInterface`annotation would be ommited.

### Method and Constructor References

The above example code can be further simplified by utilizing static method references:

```
Converter
<
String
,
 Integer
>
 converter 
=
 Integer
:
:
valueOf
;

Integer converted 
=
 converter
.
convert
(
"123"
)
;

System
.
out
.
println
(
converted
)
;
// 123
```

Java 8 enables you to pass references of methods or constructors via the`::`keyword. The above example shows how to reference a static method. But we can also reference object methods:

```
class
Something
{

    String 
startsWith
(
String s
)
{
return
 String
.
valueOf
(
s
.
charAt
(
0
)
)
;
}
}
```

```
Something something 
=
new
Something
(
)
;

Converter
<
String
,
 String
>
 converter 
=
 something
:
:
startsWith
;

String converted 
=
 converter
.
convert
(
"Java"
)
;

System
.
out
.
println
(
converted
)
;
// "J"
```

Let's see how the`::`keyword works for constructors. First we define an example bean with different constructors:

```
class
Person
{

    String firstName
;

    String lastName
;
Person
(
)
{
}
Person
(
String firstName
,
 String lastName
)
{
this
.
firstName 
=
 firstName
;
this
.
lastName 
=
 lastName
;
}
}
```

Next we specify a person factory interface to be used for creating new persons:

```
interface
PersonFactory
<
P 
extends
Person
>
{

    P 
create
(
String firstName
,
 String lastName
)
;
}
```

Instead of implementing the factory manually, we glue everything together via constructor references:

```
PersonFactory
<
Person
>
 personFactory 
=
 Person
:
:
new
;

Person person 
=
 personFactory
.
create
(
"Peter"
,
"Parker"
)
;
```

We create a reference to the Person constructor via`Person::new`. The Java compiler automatically chooses the right constructor by matching the signature of`PersonFactory.create`.



