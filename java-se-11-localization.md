# Localization

## Table of Contents

- [PICKING A LOCALE](#picking-a-locale)
- [LOCALIZING NUMBERS](#localizing-numbers)
  * [Formatting Numbers](#formatting-numbers)
  * [Parsing Numbers](#parsing-numbers)
  * [Writing a Custom Number Formatter](#writing-a-custom-number-formatter)
- [LOCALIZING DATES](#localizing-dates)
- [SPECIFYING A LOCALE CATEGORY](#specifying-a-locale-category)
- [CREATING A RESOURCE BUNDLE](#creating-a-resource-bundle)
- [PICKING A RESOURCE BUNDLE](#picking-a-resource-bundle)
- [FORMATTING MESSAGES](#formatting-messages)
- [USING THE `PROPERTIES` CLASS](#using-the--properties--class)

`Localization` means actually supporting multiple locales
or geographic regions. You can think of a locale as being like a language and country pairing. Localization includes
translating strings to different languages. It also includes
outputting dates and numbers in the correct format for that
locale.

## PICKING A LOCALE

The `Locale` class is in the `java.util` package. The first useful Locale to find is the user's current locale

```java
Locale locale = Locale.getDefault();
System.out.println(locale);
// OUTPUT: en_US
// en -> Lowercase language code
// US -> Uppercase country code

// use the built‐in constants in the Locale class
System.out.println(Locale.JAPAN); // ja_JP
System.out.println(Locale.JAPANESE); // ja

// Locale is to use the constructors to create a new object
System.out.println(new Locale("vi")); // vi
System.out.println(new Locale("vi", "VN")); // vi_VN

// The builder design pattern
Locale l1 = new Locale.Builder()
            .setLanguage("en")
            .setRegion("US")
            .build(); // en_US
Locale l2 = new Locale.Builder()
            .setRegion("US")
            .setLanguage("en")
            .build(); // en_US
```

```java
// use a Locale other than the default of your computer
System.out.println(Locale.getDefault()); // en_US
Locale locale = new Locale("vi");
Locale.setDefault(locale); // change the default
System.out.println(Locale.getDefault()); // vi
```

## LOCALIZING NUMBERS

### Formatting Numbers

```java
int attendeesPerYear = 3_200_000;
int attendeesPerMonth = attendeesPerYear / 12;

var us = NumberFormat.getInstance(Locale.US);
System.out.println(us.format(attendeesPerMonth)); 
// OUTPUT: 266,666

var gr = NumberFormat.getInstance(Locale.GERMANY);
System.out.println(gr.format(attendeesPerMonth));
// OUTPUT: 266.666

var ca = NumberFormat.getInstance(Locale.CANADA_FRENCH);
System.out.println(ca.format(attendeesPerMonth));
// OUTPUT: 266 666
```

```java
double price = 48;
var myLocale = NumberFormat.getCurrencyInstance();
System.out.println(myLocale.format(price));
// OUTPUT: $48.00
```

### Parsing Numbers

We convert data from a `String` to a structured object or `primitive` value. The `NumberFormat.parse()` method accomplishes this and takes
the locale into consideration.
Note: The `parse()` method, found in various types, declares a
checked exception `ParseException` that must be handled
or declared in the method in which they are called.

```java
String s = "40.45";
var en = NumberFormat.getInstance(Locale.US);
System.out.println(en.parse(s)); 
// OUTPUT: 40.45
var fr = NumberFormat.getInstance(Locale.FRANCE);
System.out.println(fr.parse(s)); 
// OUTPUT: 40

String income = "$92,807.99";
var cf = NumberFormat.getCurrencyInstance();
double value = (Double) cf.parse(income);
System.out.println(value); 
// OUTPUT: 92807.99
```

### Writing a Custom Number Formatter

Create your own number format strings using the `DecimalFormat` class, which extends `NumberFormat`

Symbol | Meaning
-- | --
`#` | Omit the position if no digit exists for it.
`0` | Put a 0 in the position if no digit exists for it.

```java
double d = 1234567.467;
NumberFormat f1 = new DecimalFormat("###,###,###.0");
System.out.println(f1.format(d)); 
// OUTPUT: 1,234,567.5

NumberFormat f2 = new DecimalFormat("000,000,000.00000");
System.out.println(f2.format(d)); 
// OUTPUT: 001,234,567.46700
NumberFormat f3 = new DecimalFormat("$#,###,###.##");
System.out.println(f3.format(d)); 
// OUTPUT: $1,234,567.47
```

## LOCALIZING DATES

Description | Using default `Locale`
-- | --
formatting dates | `DateTimeFormatter.ofLocalizedDate(dateStyle)`
formatting times | `DateTimeFormatter.ofLocalizedTime(timeStyle)`
formatting dates & times | `DateTimeFormatter.ofLocalizedDateTime(dateStyle, timeStyle)` `DateTimeFormatter.ofLocalizedDateTime(dateTimeStyle)`

```java
import java.time.LocalDateTime;
import java.time.Month;
import java.time.format.DateTimeFormatter;
import java.time.format.FormatStyle;
import java.util.Locale;

public class SampleDateTime {
    public static void print(DateTimeFormatter dtf, LocalDateTime dateTime, Locale locale) {
        System.out.println(dtf.format(dateTime) + ", " + dtf.withLocale(locale).format(dateTime));
    }

    public static void main(String[] args) {
        var italy = new Locale("it", "IT");
        var dt = LocalDateTime.of(2020, Month.OCTOBER, 20, 15, 12, 34);
        // 10/20/20, 20/10/20
        print(DateTimeFormatter.ofLocalizedDate(FormatStyle.SHORT), dt, italy);
        // 3:12 PM, 15:12
        print(DateTimeFormatter.ofLocalizedTime(FormatStyle.SHORT), dt, italy);
        // 10/20/20, 3:12 PM, 20/10/20, 15:12
        print(DateTimeFormatter.ofLocalizedDateTime(FormatStyle.SHORT, FormatStyle.SHORT), dt, italy);
    }
}
```

## SPECIFYING A LOCALE CATEGORY

If you require finer‐grained control of the default locale, Java actually subdivides the underlying formatting options into distinct categories, with the `Locale.Category` enum.

Value | Description
-- | --
`DISPLAY` | Category used for displaying data about the locale
`FORMAT` | Category used for formatting dates, numbers, or currencies

## CREATING A RESOURCE BUNDLE

```text
test_us.properties
hello=Hello!
open=Open the door!!!!
```

```java
import java.util.Locale;
import java.util.ResourceBundle;

public class TestResourceBundle {
    public static void printWelcomeMessage(Locale locale) {
        var rb = ResourceBundle.getBundle("test", locale);
        System.out.println(rb.getString("hello") + ", " + rb.getString("open"));
    }

    public static void main(String[] args) {
        var us = new Locale("us");
        var vi = new Locale("vi");
        var jp = new Locale("ja");
        printWelcomeMessage(us);
        printWelcomeMessage(vi);
        printWelcomeMessage(jp);
    }
}
```

Since a resource bundle contains key/value pairs, you can even loop through them to list all of the pairs. The `ResourceBundle` class provides a `keySet()` method to get a set of all keys.

```java
var us = new Locale("en", "US");
ResourceBundle rb = ResourceBundle.getBundle("test", us);
rb.keySet().stream()
    .map(k -> k + ": " + rb.getString(k))
    .forEach(System.out::println);
```

## PICKING A RESOURCE BUNDLE

```java
ResourceBundle.getBundle("name");
ResourceBundle.getBundle("name", locale);
```

```java
// requested locale -> requested with no country 
// -> default locale -> default language with no country 
// -> no locale at all -> MissingResourceException.

Locale.setDefault(new Locale("hi"));
ResourceBundle rb = ResourceBundle.getBundle("Zoo", new Locale("en"));
// They are listed here:
// 1. Zoo_en.properties
// 2. Zoo_hi.properties
// 3. Zoo.properties
```

## FORMATTING MESSAGES

```java
// resource
helloByName=Hello, {0} and {1}

String format = rb.getString("helloByName");
System.out.print(MessageFormat.format(format, "Tammy", "Henry"));
// OUTPUT: Hello, Tammy and Henry
```

## USING THE `PROPERTIES` CLASS

When working with the `ResourceBundle` class, you may also come across the `Properties` class (like `HashMap`) except that it uses `String` values for the keys and values

```java
import java.util.Properties;

public class TestPropertiesClass {
    public static void main(String[] args) {
        var props = new Properties();
        props.setProperty("name", "Our zoo");
        props.setProperty("open", "10am");
        System.out.println(props.getProperty("camel"));
        // null
        System.out.println(props.getProperty("camel", "Bob"));
        // Bob
        props.get("open"); // 10am
        props.get("open", "The zoo will be open soon"); // DOES NOT COMPILE
    }
}
```