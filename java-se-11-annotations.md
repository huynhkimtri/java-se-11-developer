# Annotations

## Table of Contents

- [PURPOSE OF ANNOTATIONS](#purpose-of-annotations)
- [CREATING AN ANNOTATION](#creating-an-annotation)
- [SPECIFYING A REQUIRED ELEMENT](#specifying-a-required-element)
- [PROVIDING AN OPTIONAL ELEMENT](#providing-an-optional-element)
- [SELECTING AN ELEMENT TYPE](#selecting-an-element-type)
- [APPLYING ELEMENT MODIFIERS](#applying-element-modifiers)
- [ADDING A CONSTANT VARIABLE](#adding-a-constant-variable)
- [USING ANNOTATIONS IN DECLARATIONS](#using-annotations-in-declarations)
- [MIXING REQUIRED AND OPTIONAL ELEMENTS](#mixing-required-and-optional-elements)
- [CREATING A VALUE() ELEMENT](#creating-a-value---element)
- [PASSING AN ARRAY OF VALUES](#passing-an-array-of-values)
- [LIMITING USAGE WITH `@TARGET`](#limiting-usage-with---target-)
- [STORING ANNOTATIONS WITH `@RETENTION`](#storing-annotations-with---retention-)
- [GENERATING JAVADOC WITH `@DOCUMENTED`](#generating-javadoc-with---documented-)
- [INHERITING ANNOTATIONS WITH `@INHERITED`](#inheriting-annotations-with---inherited-)
- [SUPPORTING DUPLICATES WITH `@REPEATABLE`](#supporting-duplicates-with---repeatable-)
- [MARKING METHODS WITH `@OVERRIDE`](#marking-methods-with---override-)
- [DECLARING INTERFACES WITH `@FUNCTIONALINTERFACE`](#declaring-interfaces-with---functionalinterface-)
- [RETIRING CODE WITH `@DEPRECATED`](#retiring-code-with---deprecated-)
- [IGNORING WARNINGS WITH `@SUPPRESSWARNINGS`](#ignoring-warnings-with---suppresswarnings-)
- [PROTECTING ARGUMENTS WITH `@SAFEVARARGS`](#protecting-arguments-with---safevarargs-)
- [JAVABEAN VALIDATION](#javabean-validation)


## PURPOSE OF ANNOTATIONS

The purpose of an annotation is to assign metadata attributes
to classes, methods, variables, and other Java types

```java
// Annotations Rules

// annotations function a lot like interfaces. 
// In this example, annotations allow us to mark a class as a ZooAnimal
// without changing its inheritance structure
public class Mammal {}
public class Bird {}
@ZooAnimal public class Lion extends Mammal {}
@ZooAnimal public class Peacock extends Bird {}

// annotations establish relationships that make it easier to
// manage data about our application
public class Veterinarian {
    @ZooAnimal(habitat="Infirmary") private Lion sickLion;
    @ZooAnimal(habitat="Safari") private Lion healthyLion;
    @ZooAnimal(habitat="Special Enclosure") private Lion blindLion;
}

// an annotation ascribes custom information on the
// declaration where it is defined
// Lion.java
public class Lion {
    @ZooSchedule(hours={"9am","5pm","10pm"}) void feedLions() {
        System.out.print("Time to feed the lions!");
    }
}
// Peacock.java
public class Peacock {
    @ZooSchedule(hours={"4am","5pm"}) void cleanPeacocksPen() {
        System.out.print("Time to sweep up!");
    }
}
```

## CREATING AN ANNOTATION

```java
public @interface Exercise {}
```

Let's apply `@Exercise` to some classes.

```java
@Exercise() public class Cheetah {}
@Exercise public class Sloth {}
@Exercise
public class ZooEmployee {}
```

## SPECIFYING A REQUIRED ELEMENT

An annotation element is an attribute that stores values about
the particular usage of an annotation


```java
public @interface Exercise {
    int hoursPerDay();
    // looks a lot like an abstract method, 
    // although we're calling it an element (or attribute)
}

// Let's see how this new element changes our usage:
@Exercise(hoursPerDay=3) public class Cheetah {}
@Exercise hoursPerDay=0 public class Sloth {} // DOES NOT COMPILE -> missing parentheses
@Exercise public class ZooEmployee {} // DOES NOT COMPILE -> the hoursPerDay field is required
// -> When declaring an annotation, any element without a default value is considered required.
```

## PROVIDING AN OPTIONAL ELEMENT

```java
public @interface Exercise {
    int hoursPerDay();
    int startHour() default 6;
}

@Exercise(startHour=5, hoursPerDay=3) public class Cheetah {}
@Exercise(hoursPerDay=0) public class Sloth {}
@Exercise(hoursPerDay=7, startHour="8") // DOES NOT COMPILE
public class ZooEmployee {}

// the default value of an annotation must be a non‐ null constant expression
public @interface BadAnnotation {
    String name() default new String(""); // DOES NOT COMPILE -> not a constant expression
    String address() default "";
    String title() default null; // DOES NOT COMPILE -> null is not permitted
}
```

## SELECTING AN ELEMENT TYPE

Must be a `primitive` type, a `String`, a `Class`, an `enum`, another annotation, or an `array` of any of these types.

Note: only `public` & `abstract` are permitted and must be initialized

```java
public class Bear {}
public enum Size {SMALL, MEDIUM, LARGE}
public @interface Panda {
    Integer height(); // DOES NOT COMPILE -> wrapper classes not allow
    String[][] generalInfo(); // DOES NOT COMPILE -> String[] is supported
    Size size() default Size.SMALL;
    Bear friendlyBear(); // DOES NOT COMPILE -> type of friendlyBear() is Bear (not Class)
    Exercise exercise() default @Exercise(hoursPerDay=2);
}
```

## APPLYING ELEMENT MODIFIERS

annotation elements are implicitly `abstract` and `public`, whether you declare them that way or not.

```java
public @interface Material {}
public @interface Fluffy {
    int cuteness();
    public abstract int softness() default 11;
    protected Material material(); // DOES NOT COMPILE
    private String friendly(); // DOES NOT COMPILE
    final boolean isBunny(); // DOES NOT COMPILE
}
```

## ADDING A CONSTANT VARIABLE

Annotations can include constant variables that can be
accessed by other classes without actually creating the
annotation.

```java
// like interface variables, annotation variables are
// implicitly public, static, and final
public @interface ElectricitySource {
    public int voltage();
    int MIN_VOLTAGE = 2;
    public static final int MAX_VOLTAGE = 18;
}
```

## USING ANNOTATIONS IN DECLARATIONS

```java
@FunctionalInterface // applied to interface
interface Speedster {
    void go(String name);
}

@LongEars
@Soft
@Cuddly
public class Rabbit { // applied to class
    @Deprecated
    public Rabbit(@NotNull Integer size) { // applied to constructor
    }

    @Speed(velocity = "fast")
    public void eat(@Edible String input) { // applied method declarations
        @Food(vegetarian = true) // applied to a local variable
        String m = (@Tasty String) "carrots";

        Speedster s1 = new @Racer Speedster() { // applied the anonymous class declaration
            public void go(@FirstName @NotEmpty String name) {
                System.out.print("Start! " + name);
            }
        };
        // applied to a lambda expression parameter
        Speedster s2 = (@Valid String n) -> System.out.print(n);
    }
}
```

## MIXING REQUIRED AND OPTIONAL ELEMENTS

```java
public @interface Swimmer {
    int armLength = 10;
    String stroke(); // required
    String name(); // required
    String favoriteStroke() default "Backstroke";
}

// which of the following compile?
// DOES NOT COMPILE -> missing the required elements stroke() and name()
@Swimmer
class Amphibian {}

// DOES NOT COMPILE -> missing the required elements stroke()
@Swimmer(favoriteStroke="Breaststroke", name="Sally") 
class Tadpole {}

// COMPLIES
@Swimmer(stroke="FrogKick", name="Kermit") 
class Frog {}

// DOES NOT COMPILE -> armLength is a contanst, not an element, cannot be included in an annotation
@Swimmer(stroke="Butterfly", name="Kip", armLength=1) 
class Reptile {}

// COMPLIES
@Swimmer(stroke="", name="", favoriteStroke="") 
class Snake {}
```

## CREATING A VALUE() ELEMENT

```java
@Injured("Broken Tail") public class Monkey {}
```

```java
public @interface Injured {
    String veterinarian() default "unassigned";
    String value() default "foot"; // may be optional or required
    int age() default 1;
}

public abstract class Elephant {
    @Injured("Legs") public void fallDown() {}
    @Injured(value="Legs") public abstract int trip();
    @Injured String injuries[];
}
```

## PASSING AN ARRAY OF VALUES

```java
public @interface Music {
    String[] genres();
}

public class Giraffe {
    @Music(genres={"Rock and roll"}) String mostDisliked;
    @Music(genres="Classical") String favorite;
}

public class Reindeer {
    @Music(genres="Blues","Jazz") String favorite; // DOES NOT COMPILE
    @Music(genres=) String mostDisliked; // DOES NOT COMPILE
    @Music(genres=null) String other; // DOES NOT COMPILE
    @Music(genres={}) String alternate;
}

// COMBINING SHORTHAND NOTATIONS
public @interface Rhythm {
    String[] value();
}

public class Capybara {
    @Rhythm(value={"Swing"}) String favorite;
    @Rhythm(value="R&B") String secondFavorite;
    @Rhythm({"Classical"}) String mostDisliked;
    @Rhythm("Country") String lastDisliked;
}
```

## LIMITING USAGE WITH `@TARGET`

the `@Target` annotation takes an array of `ElementType` enum values as its `value()` element

`ElementType` Type | Applies to
---- | ----
`TYPE` | Classes, interfaces, enums, annotations
`FIELD` | Instance and static variables, enum values
`METHOD` | Method declarations
`PARAMETER` | Constructor, method, and lambda parameters
`CONSTRUCTOR` | Constructor declarations
`LOCAL_VARIABLE` | Local variables
`ANNOTATION_TYPE` | Annotations
`PACKAGE *` | Packages declared in package‐info.java
`TYPE_PARAMETER` | Parameterized types, generic declarations
`TYPE_USE` | Able to be applied anywhere there is a Java type declared or used
`MODULE *` | Modules

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Target;

@Target({ ElementType.METHOD, ElementType.CONSTRUCTOR })
public @interface ZooAttraction {}

// which of the following lines of code will compile?
@ZooAttraction // DOES NOT COMPILE -> cannot applied to a class type
class RollerCoaster {
}

class Events {
    @ZooAttraction
    String rideTrain() {
        return (@ZooAttraction String) "Fun!"; // DOES NOT COMPILE -> not permitted on a cast operation
    }

    @ZooAttraction // permitted on a method declaration
    Events(@ZooAttraction String description) { // DOES NOT COMPILE -> not marked for use in a constructor parameter
        super();
    }

    @ZooAttraction // DOES NOT COMPILE -> cannot applied to fields or variables
    int numPassengers;
}
```

## STORING ANNOTATIONS WITH `@RETENTION`

By default, annotations are not present at runtime

`RetentionPolicy` value | Description
-- | --
`SOURCE` | Used only in the source file, discarded by the compiler
`CLASS` | Stored in the `.class` file but not available at runtime (default compiler behavior)
`RUNTIME` | Stored in the `.class` file and available at runtime

```java
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.CLASS) @interface Flier {}
@Retention(RetentionPolicy.RUNTIME) @interface Swimmer {}
```

## GENERATING JAVADOC WITH `@DOCUMENTED`

the generated Javadoc will include annotation information defined on Java
types. Because it is a marker annotation, it doesn't take any values; therefore, using it is pretty easy.

```java
// Hunter.java
import java.lang.annotation.Documented;
@Documented 
public @interface Hunter {}

// Lion.java
@Hunter public class Lion {}

// the @Hunter annotation would be published
// with the Lion Javadoc information because it's marked with the @Documented annotation.
```

## INHERITING ANNOTATIONS WITH `@INHERITED`

When this annotation is applied to a class, subclasses will inherit the annotation information found in the parent class.

```java
// Vertebrate.java
import java.lang.annotation.Inherited;
@Inherited public @interface Vertebrate {}

// Mammal.java
@Vertebrate public class Mammal {}

// Dolphin.java
public class Dolphin extends Mammal {}

// the @Vertebrate annotation will be applied to
// both Mammal and Dolphin objects. Without the @Inherited
// annotation, @Vertebrate would apply only to Mammal instances.
```

## SUPPORTING DUPLICATES WITH `@REPEATABLE`

The `@Repeatable` annotation is used when you want to specify an annotation
more than once on a type. Generally, you use repeatable annotations when you want to apply the same annotation with different values.

```java
// First rule: without the @Repeatable annotation, 
// an annotation can be applied only once
public class Zoo {
    public static class Monkey {}
    @Risk(danger="Silly") // DOES NOT COMPILE
    @Risk(danger="Aggressive",level=5) // DOES NOT COMPILE
    @Risk(danger="Violent",level=10) // DOES NOT COMPILE
    private Monkey monkey;
}

public @interface Risk {
    String danger();
    int level() default 1;
}

// next rule: to declare a @Repeatable annotation,
// you must define a containing annotation type value
import java.lang.annotation.Repeatable;

@Repeatable // DOES NOT COMPILE -> annotation is not declared correctly.
public @interface Risk {
    String danger();
    int level() default 1;
}

// declaration is a containing annotation type for our Risk annotation
public @interface Risks {
    Risk[] value();
}

// specify the containing annotation class
import java.lang.annotation.Repeatable;
@Repeatable(Risks.class)
public @interface Risk {
    String danger();
    int level() default 1;
}
```

## MARKING METHODS WITH `@OVERRIDE`

The `@Override` is a marker annotation that is used to indicate a method is overriding an inherited method, whether it be inherited from an interface or parent class

```java
public interface Intelligence {
    int cunning();
}
public class Canine implements Intelligence {
    @Override 
    public int cunning() { return 500; }
    void howl() { System.out.print("Woof!"); }
}
public class Wolf extends Canine {
    @Override 
    public int cunning() { return Integer.MAX_VALUE; }

    @Override 
    void howl() { System.out.print("Howl!"); }
}
```

## DECLARING INTERFACES WITH `@FUNCTIONALINTERFACE`

The `@FunctionalInterface` marker annotation can be applied to any valid functional interface.

```java
@FunctionalInterface
public interface Intelligence {
    int cunning();
}
```

```java
// DOES NOT COMPILE -> applied only to interfaces
@FunctionalInterface 
abstract class Reptile {
    abstract String getName();
}

// DOES NOT COMPILE -> not contain any abstract methods
@FunctionalInterface 
interface Slimy {}

// COMPILE -> contains exactly one abstract method
@FunctionalInterface 
interface Scaley {
    boolean isSnake();
}

// DOES NOT COMPILE -> contains two abstract methods, one of which it inherits from Scaley
@FunctionalInterface 
interface Rough extends Scaley {
    void checkType();
}

// COMPILE -> one matches the signature of a method java.lang.Object
@FunctionalInterface 
interface Smooth extends Scaley {
    boolean equals(Object unused);
}
```

## RETIRING CODE WITH `@DEPRECATED`

The `@Deprecated` annotation is similar to a marker
annotation, in that it can be used without any values, but it
includes some optional elements. The`@Deprecated` annotation
can be applied to nearly any Java declaration, such as classes,
methods, or variables.

```java
/**
* Design and plan a zoo.
* @deprecated Use ParkPlanner instead.
*/
@Deprecated
public class ZooPlanner { /* DO SOMETHING */ }

// Note: Whenever you deprecate a method, 
// you should add a Javadoc annotation to instruct users on 
// how they should update their code.
```

support two optional values: `String since()` and `boolean forRemoval()`
```java
/**
* Method to formulate a zoo layout.
* @deprecated Use ParkPlanner.planPark(String… data)
instead.
*/
@Deprecated(since="1.8", forRemoval=true)
public void plan() {}
```

## IGNORING WARNINGS WITH `@SUPPRESSWARNINGS`

Enter `@SuppressWarnings`. Applying this annotation to a class,
method, or type basically tells the compiler, “I know what I am
doing; do not warn me about this.”

Value | Description
-- | --
`deprecation` | Ignore warnings related to types or methods marked with the `@Deprecated` annotation
`unchecked` | Ignore warnings related to the use of raw types, such as `List` instead of `List<String>`

```java
import java.util.*;
class SongBird {
    @Deprecated 
    static void sing(int volume) {}
    static Object chirp(List<String> data) { 
        return data.size(); 
    }
}

public class Nightingale {
    public void wakeUp() {
        SongBird.sing(10);
    }

    public void goToBed() {
        SongBird.chirp(new ArrayList());
    }

    public static void main(String[] args) {
        var birdy = new Nightingale();
        birdy.wakeUp();
        birdy.goToBed();
    }
}

// This code compiles and runs but produces two compiler warnings
// Nightingale.java uses or overrides a deprecated API.
// Nightingale.java uses unchecked or unsafe operations

// Now our code compiles, and no warnings are generated.
@SuppressWarnings("deprecation") 
public void wakeUp() {
    SongBird.sing(10);
}

@SuppressWarnings("unchecked") 
public void goToBed() {
    SongBird.chirp(new ArrayList());
}
```

## PROTECTING ARGUMENTS WITH `@SAFEVARARGS`

The `@SafeVargs` marker annotation indicates that a method
does not perform any potential unsafe operations on its
varargs parameter. It can be applied only to constructors or
methods that cannot be overridden (aka methods marked
`private`, `static`, or `final`).

```java
public class TestSafeVargs {
    final int thisIsUnsafe(List<Integer>... carrot) { // -> complier warning
        Object[] stick = carrot;
        stick[0] = Arrays.asList("nope!");
        return carrot[0].get(0); // ClassCastException at runtime
    }

    public static void main(String[] a) {
        var carrot = new ArrayList<Integer>();
        new TestSafeVargs().thisIsUnsafe(carrot); // -> complier warning
    }
}
// This code compile but it generates two complier warning:
// Type safety: Potential heap pollution via varargs parameter carrot
// Type safety: A generic array of List<Integer> is created for a varargs parameter
// Output: Exception in thread "main" java.lang.ClassCastException: class java.lang.String cannot be cast to class java.lang.Integer (java.lang.String and java.lang.Integer are in module java.base of loader 'bootstrap')

// -> We can remove both compiler warnings by adding the @SafeVarargs annotation
// It still throws a ClassCastException at runtime. 
// However, we made it so the compiler won't warn us about it anymore.
public class TestSafeVargs {
    @SafeVarargs
    final int thisIsUnsafe(List<Integer>... carrot) {
        Object[] stick = carrot;
        stick[0] = Arrays.asList("nope!");
        return carrot[0].get(0); // ClassCastException at runtime
    }

    public static void main(String[] a) {
        var carrot = new ArrayList<Integer>();
        new TestSafeVargs().thisIsUnsafe(carrot);
    }
}
```

```java
@SafeVarargs
public static void eat(int meal) {} // DOES NOT COMPILE -> missing a varargs parameter

@SafeVarargs
protected void drink(String… cup) {} // DOES NOT COMPILE -> not marked static, final, or private.
@SafeVarargs 
void chew(boolean food) {} // DOES NOT COMPILE -> not marked static, final, or private.
```

## JAVABEAN VALIDATION

If you've ever used `JavaBeans` to transmit data, then you've probably written code to validate it. While this can be cumbersome for large data structures, annotations allow you to mark private fields directly. The following are some useful `javax.validation` annotations:


- `@NotNull`: Object cannot be null
- `@NotEmpty`: Object cannot be null or have size of 0
- `@Size(min=5,max=10)`: Sets minimum and/or maximum
sizes
- `@Max(600)` and `@Min(‐5)`: Sets the maximum or minimum
numeric values
- `@Email`: Validates that the email is in a valid format
