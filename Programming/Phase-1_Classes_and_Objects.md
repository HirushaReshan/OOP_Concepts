# Phase 1: Classes and Objects

**Module:** 5COSC019W Object Oriented Programming
**Before this:** Pre-Phase 0 (you can write variables, `if`, loops, methods and arrays).
**Goal:** by the end you can write a class with fields, a constructor and methods, build several objects from it, and explain exactly what each line does.
**Time needed:** about 3 hours if you type every example.

**Built from your files:** Pure OOP Week 01 Lecture (Sections 02–04 and the reference implementation), Pure OOP Week 01 Lecture Notes (§10–25, §28–34, Appendices A–D), Lecture 1 (Section 03, Classes & Objects), Week 01 Tutorial (Exercise 12).

> **Why this phase matters for the exam.** Task 1 of your practical is "Java OOP implementation" (18 marks). Lecture 6's mock briefing describes it as: add a field, write a constructor, write a method. Everything in that sentence is taught in this guide.

---

## Where we are on the roadmap

| Phase | Title | Status |
| :-- | :-- | :-- |
| Pre-Phase 0 | Absolute Java Basics | Done |
| **Phase 1** | **Classes and Objects** | **You are here** |
| Phase 2 | Encapsulation and UML Class Diagrams | Next |
| Phase 3 | Collections and Generics | |
| Phase 4 | Inheritance | |
| Phase 5 | Polymorphism and Abstraction | |
| Phase 6 | Exceptions | |
| Phase 7 | Dependencies and Layered Design | |
| Phase 8 | Exam Consolidation | |

---

## 1. The problem, one more time

Pre-Phase 0 ended with this. A university lends equipment, and each item needs four pieces of data:

```java
long id1 = 101;   String name1 = "Dell Laptop";    double cost1 = 185000.00;  boolean available1 = true;
long id2 = 102;   String name2 = "Canon Camera";   double cost2 = 120000.00;  boolean available2 = true;
```

Nothing in this code says that `id1`, `name1`, `cost1` and `available1` belong together. Only the digit "1" in the names hints at it.

### 1.1 A first attempt that still fails: parallel arrays

Your lecture notes (§10) show the obvious next idea, using the arrays from Pre-Phase 0:

```java
long[] ids = {101L, 102L};
String[] names = {"Dell Laptop", "Canon Camera"};
double[] costs = {185000.00, 120000.00};
boolean[] available = {true, true};
```

This is called **parallel arrays**: several arrays where position 0 in each one describes the same item. It reduces the number of names, but creates a new danger. If one array gets sorted or has a different length, position 0 no longer describes the same item in all four, and nothing warns you.

### 1.2 What we actually want

We want to be able to write this:

```java
EquipmentItem laptop = new EquipmentItem(101, "Dell Laptop", 185000.00, true);
```

One variable. One thing. All four values travel together inside it. To make that line legal, we have to teach Java what an `EquipmentItem` is. That is what a class does.

---

## 2. Classes: inventing your own type

### 2.1 Definition

In Pre-Phase 0 you used types that Java gave you: `int`, `double`, `String`. A **class** lets you **create a brand-new type of your own**.

A class is a **blueprint**: a written description of what one kind of thing contains (its data) and what it can do (its actions).

- **Analogy: a house blueprint.** A blueprint is a drawing on paper. It says "every house of this design has 3 bedrooms, 1 kitchen and a front door". You cannot live in a blueprint. But a builder can use that one blueprint to build 1 house or 500 houses.
- **Analogy: a cookie cutter.** The cutter is the class. The cookies are the things you make with it. One cutter, many cookies, all the same shape, each one a separate cookie.

### 2.2 Your first real class

Create a **new file** called `EquipmentItem.java` (in IntelliJ: right-click `src` → New → Java Class → type `EquipmentItem`).

```java
public class EquipmentItem {

    long id;
    String name;
    double replacementCost;
    boolean available;

}
```

| Line | Piece | What it means |
| :-- | :-- | :-- |
| 1 | `public` | Visible to all other code. |
| 1 | `class` | "I am defining a new type." |
| 1 | `EquipmentItem` | The name of the new type. Capital first letter, UpperCamelCase. The file must be `EquipmentItem.java`. |
| 1 | `{` | The description of the type starts here. |
| 3 | `long id;` | Every equipment item will have a whole-number `id`. |
| 4 | `String name;` | Every equipment item will have a text `name`. |
| 5 | `double replacementCost;` | Every equipment item will have a decimal `replacementCost`. |
| 6 | `boolean available;` | Every equipment item will have a true/false `available`. |
| 8 | `}` | The description ends. |

### 2.3 Fields

Those four lines are called **fields**.

A **field** is a variable declared **inside a class but outside any method**. It describes one piece of data that every thing built from this class will carry.

| | Local variable (Pre-Phase 0) | Field (new) |
| :-- | :-- | :-- |
| Declared where? | Inside a method | Inside the class, outside methods |
| Lives how long? | Until the method ends | As long as the thing it belongs to exists |
| Starting value if you give none? | None. Compile error if you read it. | A default: `0` for numbers, `false` for `boolean`, `null` for `String` (`null` is explained in [Section 10.5](#105-null-a-name-with-nothing-attached)) |

Other names you will see for fields: **attributes**, **instance variables**, **properties**. They all mean the same thing here.

> **Important.** At this point we have defined a type. We have **not** created any actual equipment item. A blueprint is not a house. (Your lecture slide makes exactly this point.)

---

## 3. Objects: the real things built from the blueprint

### 3.1 Definition

An **object** is one actual thing built from a class. It exists in the computer's memory while the program runs, and it holds its own real values.

Another word for object is **instance**. "An instance of `EquipmentItem`" means "one object built from the `EquipmentItem` class". Building an object is called **instantiation** ("to instantiate" = "to build an instance").

| | Class | Object |
| :-- | :-- | :-- |
| What is it? | A blueprint / a type | One real thing built from the blueprint |
| Where does it exist? | In your `.java` file | In memory, while the program runs |
| How many? | You write it **once** | You can build **as many as you like** |
| Holds real values? | No. It only says which fields exist. | Yes. Its own value for each field. |
| Example | `EquipmentItem` | The particular Dell laptop with id 101 |

### 3.2 Building objects

Create a second file, `Main.java`. Your program now has two files: one describing the type, one using it.

```java
public class Main {
    public static void main(String[] args) {

        EquipmentItem laptop = new EquipmentItem();

        laptop.id = 101;
        laptop.name = "Dell Laptop";
        laptop.replacementCost = 185000.00;
        laptop.available = true;

        EquipmentItem camera = new EquipmentItem();
        camera.id = 102;
        camera.name = "Canon Camera";

        System.out.println(laptop.name);
        System.out.println(camera.name);
    }
}
```

**Output:**

```
Dell Laptop
Canon Camera
```

#### `EquipmentItem laptop = new EquipmentItem();`

| Piece | What it means |
| :-- | :-- |
| `EquipmentItem` (first) | The **type** of the variable. Just like `int age`, but the type is one you invented. |
| `laptop` | The variable's name. |
| `=` | "Becomes." |
| `new` | The keyword that **builds a new object** in memory. |
| `EquipmentItem()` | Which blueprint to build from. The round brackets are for starting values; they are empty for now. |
| `;` | End of statement. |

In plain English: "Build a new equipment item from the blueprint, and let the name `laptop` refer to it."

#### `laptop.name = "Dell Laptop";`

| Piece | What it means |
| :-- | :-- |
| `laptop` | Which object. |
| `.` | The dot: "go inside this object". This is called **dot notation**. |
| `name` | Which field inside it. |
| `= "Dell Laptop"` | Store this text there. |

Read `laptop.name` as "the `name` that belongs to `laptop`".

### 3.3 Each object has its own values

```
   EquipmentItem class (ONE blueprint)
   +-----------------------------+
   | id, name,                   |
   | replacementCost, available  |
   +-----------------------------+
          |                 |
          | new             | new
          v                 v
   +----------------+   +-----------------+
   | laptop object  |   | camera object   |
   | id = 101       |   | id = 102        |
   | name = Dell... |   | name = Canon... |
   +----------------+   +-----------------+
   TWO separate objects
```

Changing `laptop.name` does **nothing** to `camera.name`. They are two separate objects, each with its own copy of every field.

- **Analogy:** two houses built from the same blueprint. Painting one kitchen red does not change the other kitchen.

> **Checkpoint (from Lecture 1).** "If I create 100 `Product` objects, how many `Product` classes exist?" Just **one** class. All 100 objects share the same blueprint.

### 3.4 The problem with building objects this way

Look at `camera` above. We forgot to set `replacementCost` and `available`. Java did not complain. It quietly filled them with defaults: `0.0` and `false`.

A camera that costs nothing is nonsense, but the program runs happily. We need a way to **force** every object to be given all its values at the moment it is built. That tool is the constructor.

---

## 4. Constructors: setting an object up properly

### 4.1 Definition

A **constructor** is a special block of code inside a class that **runs automatically at the moment an object is built with `new`**. Its job is to give the new object its starting values.

- **Analogy: the factory setup step.** When a phone is made, it does not leave the factory blank. A setup step installs the serial number and the model name before it goes in the box. The constructor is that setup step.

### 4.2 Writing one

Add this inside `EquipmentItem`, below the fields:

```java
public class EquipmentItem {

    long id;
    String name;
    double replacementCost;
    boolean available;

    public EquipmentItem(long id, String name,
                         double replacementCost,
                         boolean available) {
        this.id = id;
        this.name = name;
        this.replacementCost = replacementCost;
        this.available = available;
    }
}
```

**The constructor header:** `public EquipmentItem(long id, String name, double replacementCost, boolean available) {`

| Piece | What it means |
| :-- | :-- |
| `public` | Any code is allowed to build these objects. |
| `EquipmentItem` | The constructor's name. **It must be exactly the class name.** That is how Java recognises it as a constructor. |
| *(nothing between `public` and the name)* | **A constructor has no return type. Not even `void`.** This is the second way Java recognises it. |
| `(long id, String name, double replacementCost, boolean available)` | Four **parameters**: the four values that must be handed in when building an object. |
| `{` | The setup instructions begin. |

It is spread over three lines only because it is long. Java ignores line breaks.

**The constructor body:** four lines that all follow the same pattern.

`this.id = id;`

| Piece | What it means |
| :-- | :-- |
| `this.id` | The **field** called `id` that belongs to the object being built right now. |
| `=` | "Becomes." |
| `id` | The **parameter** called `id`: the value that was just handed in. |

In plain English: "Take the value that was handed in, and store it in this object's own field."

### 4.3 Using it

`Main.java` now becomes:

```java
public class Main {
    public static void main(String[] args) {

        EquipmentItem laptop = new EquipmentItem(
                101, "Dell Laptop", 185000.00, true);

        EquipmentItem camera = new EquipmentItem(
                102, "Canon Camera", 120000.00, true);

        System.out.println(laptop.name);
        System.out.println(camera.replacementCost);
    }
}
```

**Output:**

```
Dell Laptop
120000.0
```

The values in the brackets after `new EquipmentItem` are **arguments**. They are matched to the constructor's parameters **by position**:

| Position | Argument | Goes into parameter | Stored in field |
| :-- | :-- | :-- | :-- |
| 1st | `101` | `id` | `this.id` |
| 2nd | `"Dell Laptop"` | `name` | `this.name` |
| 3rd | `185000.00` | `replacementCost` | `this.replacementCost` |
| 4th | `true` | `available` | `this.available` |

Get the order wrong, for example `new EquipmentItem("Dell Laptop", 101, ...)`, and the compiler stops you, because text cannot go into a `long`.

One object, fully set up, in one statement. It is no longer possible to forget the cost.

### 4.4 The three constructor rules

| Rule | Why |
| :-- | :-- |
| 1. Same name as the class, same capitals. | That is how Java knows it is a constructor. |
| 2. **No return type**, not even `void`. | A constructor does not hand a value back; it sets up the object being built. |
| 3. It runs only when `new` is used. | You never call it like a normal method. |

> **The `void` trap (lecture slide, Mistake 1).** If you write `public void EquipmentItem(...)`, Java does not treat it as a constructor. It becomes an ordinary method that happens to be named `EquipmentItem`, and it never runs when you use `new`. Your fields stay at their defaults and you will not see an error. Never put a return type on a constructor.

### 4.5 The disappearing free constructor

In Section 3.2, `new EquipmentItem()` with empty brackets worked even though we had written no constructor. Why?

- If a class has **no constructor at all**, Java silently provides a free empty one that takes no arguments. It is called the **default constructor**.
- **The moment you write any constructor yourself, the free one is taken away.**

So after adding the four-parameter constructor:

```java
EquipmentItem item = new EquipmentItem();   // COMPILE ERROR
```

```
constructor EquipmentItem in class EquipmentItem cannot be applied to given types;
  required: long,String,double,boolean
  found:    no arguments
```

Read the message as your lecturer teaches: Java **required** four values and **found** none. The only constructor that exists needs four arguments, and Java cannot call a constructor that does not exist.

Can a class have both? Yes, you can write a second constructor with empty brackets yourself. A constructor with no parameters is called a **no-argument constructor** (or "no-arg constructor"). Phase 2 covers that and explains why Spring needs one.

---

## 5. The `this` keyword

### 5.1 The problem `this` solves

Inside the constructor there are **two different things called `id`** at the same time:

1. the **field** `id`, which belongs to the object, and
2. the **parameter** `id`, the value handed in.

When two things share a name, Java picks the closest one, which is the parameter. The field is hidden behind it. (The technical word is **shadowing**: the parameter "casts a shadow" over the field.)

**`this` means "the current object": the one being built or used right now.** So `this.id` means "the `id` field of the current object", and that reaches past the parameter to the field.

```
        this.id        =        id;
        -------                 --
           |                     |
   the object's FIELD     the PARAMETER
   (the box inside        (the value that
    the object)            was handed in)
```

- **Analogy:** filling in a form about yourself. Someone tells you "the name is Kamal". You write it into **my** name box on **this** form. `this.name = name` means "this form's name box becomes the name I was just told".

### 5.2 What happens if you forget `this`

```java
public EquipmentItem(long id, String name,
                     double replacementCost, boolean available) {
    id = id;          // WRONG: sets the parameter to itself
    name = name;      // WRONG
}
```

Both sides now mean the parameter. The parameter is copied onto itself, and the field is never touched. The program **compiles and runs**, and your object ends up with `id` equal to `0` and `name` equal to `null`. This is Mistake 3 on your lecture slide, and it is hard to spot because there is no error message.

### 5.3 Do you have to use the same names?

No. This also works:

```java
public EquipmentItem(long itemId, String itemName,
                     double cost, boolean isAvailable) {
    id = itemId;
    name = itemName;
    replacementCost = cost;
    available = isAvailable;
}
```

The names differ, so nothing is hidden and `this` is not needed. But standard Java style, and the style in all your lecture code, is matching names plus `this`. **Use `this.field = parameter;` in the exam.**

### 5.4 `this` changes meaning with each object

`this` is not one fixed object. It means whichever object is being worked on at that moment.

- While `new EquipmentItem(101, "Dell Laptop", ...)` runs, `this` is the laptop.
- While `new EquipmentItem(102, "Canon Camera", ...)` runs, `this` is the camera.

> **Common misconception (notes §31).** `this` does **not** mean "the class". It means the current **object**.

---

## 6. What `new` actually does, step by step

For the line:

```java
EquipmentItem laptop = new EquipmentItem(101, "Dell Laptop", 185000.00, true);
```

Your lecture notes (§15) give this sequence:

| Step | What happens |
| :-- | :-- |
| 1 | Java works out the argument values: `101`, `"Dell Laptop"`, `185000.00`, `true`. |
| 2 | Memory is set aside for one new `EquipmentItem` object. |
| 3 | Its fields get default values: `0`, `null`, `0.0`, `false`. |
| 4 | The constructor runs and overwrites those with the real values. |
| 5 | `new` produces a **reference** to the finished object (explained in Section 10). |
| 6 | That reference is stored in the variable `laptop`. |

Three statements that look similar but do different things:

| Code | Declares a variable? | Builds an object? |
| :-- | :-- | :-- |
| `EquipmentItem x;` | Yes | **No** |
| `new EquipmentItem(101, "A", 1.0, true);` | No | Yes |
| `EquipmentItem x = new EquipmentItem(101, "A", 1.0, true);` | Yes | Yes |

> **Notes question:** after `EquipmentItem firstItem;`, how many objects exist because of that statement? **None.** It only creates a name that is able to refer to an `EquipmentItem` later.

---

## 7. Behaviour: putting methods inside the class

### 7.1 The poor design

So far the object is only a bag of data. All the thinking still happens outside it, in `main`:

```java
EquipmentItem laptop = new EquipmentItem(101, "Laptop", 185000, true);

int daysLate = 4;
double fee = daysLate * 2.50;

if (laptop.replacementCost > 100000) {
    System.out.println("High value item");
}
```

The rules about equipment (the late fee is 2.50 per day; "expensive" means above some amount) are scattered outside the `EquipmentItem` class. If another part of the program needs the same rule, the code gets copied. When the rule changes, you must find every copy.

**The object-oriented answer:** move behaviour that is about an equipment item **into** the `EquipmentItem` class.

### 7.2 Three methods

Add these inside `EquipmentItem`, after the constructor:

```java
public void displayDetails() {
    System.out.println(id + " - " + name
            + " (£" + replacementCost + ")");
}

public double calculateLateFee(int daysLate) {
    return daysLate * 2.50;
}

public boolean isExpensive(double threshold) {
    return replacementCost > threshold;
}
```

#### `public void displayDetails() {`

| Piece | What it means |
| :-- | :-- |
| `public` | Can be called from other classes. |
| `void` | Hands nothing back. It performs an action (printing). |
| `displayDetails` | The method name. |
| `()` | No parameters. It needs no extra information, because **the object already has its own `id`, `name` and `replacementCost`**. |
| `id + " - " + name + ...` | Uses the fields of the current object directly. |

#### `public double calculateLateFee(int daysLate) {`

| Piece | What it means |
| :-- | :-- |
| `double` | Hands back a decimal number. |
| `int daysLate` | One parameter. The number of days changes with every call and is not a permanent fact about the item, so it is passed in. |
| `return daysLate * 2.50;` | Work out the fee and hand it back. |

#### `public boolean isExpensive(double threshold) {`

| Piece | What it means |
| :-- | :-- |
| `boolean` | Hands back `true` or `false`. |
| `double threshold` | The amount to compare against. A **threshold** is a cut-off line. |
| `return replacementCost > threshold;` | The comparison itself produces `true` or `false`, and that is handed straight back. No `if` needed. |

A method that returns a `boolean` answer to a question is sometimes called a **predicate**. By convention their names start with `is`, `has` or `can`.

### 7.3 Spot the difference: no `static`

Compare with Pre-Phase 0:

```java
public static double calculateArea(double radius)     // Pre-Phase 0: has static
public        double calculateLateFee(int daysLate)   // Phase 1: no static
```

| | `static` method | **Instance method** (no `static`) |
| :-- | :-- | :-- |
| Belongs to | The class as a whole | Each individual object |
| How to call it | Just the name, or `ClassName.method()` | **Through an object:** `laptop.method()` |
| Can it use the object's fields? | No. There is no "current object". | Yes. That is the whole point. |
| Can it use `this`? | No | Yes |

An **instance method** is a method that runs "on" one particular object and can read that object's fields.

- **Analogy:** `static` is a notice on the school wall, the same for everyone. An instance method is asking one particular student "what is **your** name?" The answer depends on who you ask.

> **Rule for this module:** methods that describe what an object does are **not** `static`. Only `main` is. Your notes list "every method should be static" as a misconception. If you find yourself adding `static` just to make an error go away, stop: the real fix is almost always to call the method through an object.

### 7.4 Calling instance methods

```java
laptop.displayDetails();
double fee = laptop.calculateLateFee(4);
System.out.println(laptop.isExpensive(100000));
```

`laptop.displayDetails();`

| Piece | What it means |
| :-- | :-- |
| `laptop` | Which object we are asking. Sometimes called the **receiver**, because it receives the request. |
| `.` | "Go inside." |
| `displayDetails()` | The method to run on that object. |

While it runs, `id`, `name` and `replacementCost` inside the method mean **the laptop's** values. If you call `camera.displayDetails()`, the very same code runs, but now those names mean **the camera's** values. Inside the method, `this` is whichever object is to the left of the dot.

Writing `id` inside an instance method is short for `this.id`. You only **have** to write `this.` when a parameter with the same name is hiding the field, as in the constructor.

### 7.5 The method that uses the object's own state

Your lecture notes (§23) show a richer late fee where the rate depends on the item's cost:

```java
public double calculateLateFee(int daysLate) {
    double dailyRate = replacementCost * 0.01;
    return daysLate * dailyRate;
}
```

| Line | What it means |
| :-- | :-- |
| `double dailyRate = replacementCost * 0.01;` | A local variable. The daily rate is 1% of **this object's** replacement cost. |
| `return daysLate * dailyRate;` | Days multiplied by the rate. |

Now `laptop.calculateLateFee(3)` and `camera.calculateLateFee(3)` run identical code and give **different answers**, because each object has a different `replacementCost`. Same method, different object, different result. Hold on to this idea. Phase 5 (polymorphism) is built on it.

Your slides use the simpler `daysLate * 2.50` version, so the rest of this guide uses that one.

### 7.6 Two kinds of method

| Kind | Meaning | Examples |
| :-- | :-- | :-- |
| **Query** | Answers a question. Does not change the object. | `calculateLateFee()`, `isExpensive()` |
| **Command** | Performs an action, and may change the object's fields. | `displayDetails()` (prints); `borrow()`, `returnItem()` come in Phase 2 |

### 7.7 Why does `calculateLateFee` belong inside `EquipmentItem`?

This is a checkpoint question on your slides. Model answer:

> Because calculating a late fee is behaviour of an equipment item and uses its item-related rules. Putting it in the class keeps the rule in one place, next to the data it uses, so other code asks the object instead of copying the calculation.

---

## 8. State, identity and behaviour

Your lecturers want you to use these three words correctly.

| Word | Meaning | In `EquipmentItem` | Human analogy |
| :-- | :-- | :-- | :-- |
| **State** | The data an object holds **right now**: the current values of its fields. | `id=101`, `name="Dell Laptop"`, `replacementCost=185000.0`, `available=true` | Your current age, address, hair colour |
| **Identity** | What makes this object **this particular one** and not another, even if another holds identical values. | The laptop object itself | You, as a person. An identical twin is still a different person. |
| **Behaviour** | What an object can **do**: its methods. | `displayDetails()`, `calculateLateFee()`, `isExpensive()` | Walking, speaking |

Quick test: **fields are state, methods are behaviour.**

```java
EquipmentItem a = new EquipmentItem(101, "Laptop", 185000, true);
EquipmentItem b = new EquipmentItem(101, "Laptop", 185000, true);
```

`a` and `b` have **the same state** but **different identities**. `new` was used twice, so two separate objects were built.

**Why it matters later** (notes §18): encapsulation protects state (Phase 2). Inheritance shares state and behaviour (Phase 4). Polymorphism lets related objects respond differently to the same behaviour request (Phase 5).

---

## 9. The complete reference implementation

This is the full version from your lecture slides. Type it out and run it.

**`EquipmentItem.java`**

```java
public class EquipmentItem {

    long id;
    String name;
    double replacementCost;
    boolean available;

    public EquipmentItem(long id, String name,
                         double replacementCost,
                         boolean available) {
        this.id = id;
        this.name = name;
        this.replacementCost = replacementCost;
        this.available = available;
    }

    public void displayDetails() {
        System.out.println(id + " - " + name
                + " (£" + replacementCost + ")");
    }

    public double calculateLateFee(int daysLate) {
        return daysLate * 2.50;
    }

    public boolean isExpensive(double threshold) {
        return replacementCost > threshold;
    }

    public boolean isAvailableForLoan() {
        return available;
    }
}
```

**`Main.java`**

```java
public class Main {
    public static void main(String[] args) {

        EquipmentItem laptop = new EquipmentItem(
                101, "Dell Laptop", 185000.00, true);

        EquipmentItem camera = new EquipmentItem(
                102, "Canon Camera", 120000.00, true);

        laptop.displayDetails();
        camera.displayDetails();

        System.out.println(laptop.calculateLateFee(4));
        System.out.println(camera.isExpensive(100000));
    }
}
```

**Output:**

```
101 - Dell Laptop (£185000.0)
102 - Canon Camera (£120000.0)
10.0
true
```

| Output line | Where it came from |
| :-- | :-- |
| `101 - Dell Laptop (£185000.0)` | `laptop.displayDetails()` reading the laptop's fields. A `double` always prints with at least one decimal place, so `185000.00` shows as `185000.0`. |
| `102 - Canon Camera (£120000.0)` | The same method, run on the camera. |
| `10.0` | 4 days × 2.50. |
| `true` | Is 120000.0 greater than 100000? Yes. |

If the `£` shows as `?` in your console, that is only the console's character settings. Your code is fine.

The standard layout of a class, top to bottom: **fields → constructor(s) → methods.** Keep to this order in the exam; markers find things faster.

The pattern to remember from the slide: **create object → call behaviour → store or use the return value → print the result.**

---

## 10. References: what a variable really holds

This is the subtlest idea in Phase 1. Go slowly.

### 10.1 A variable does not contain the object

With `int age = 20;`, the box named `age` contains the number 20 itself.

With `EquipmentItem laptop = new EquipmentItem(...);`, the box named `laptop` does **not** contain the object. The object lives elsewhere in memory. The box contains a **reference**: a value that leads to the object.

A **reference** is a value that says "the object is over there". A variable that holds one is a **reference variable**.

- **Analogy: a TV remote.** The object is the television. The variable is a remote control for it. The remote is not the TV, but pressing its buttons controls the TV. The dot (`laptop.name`) is "pressing a button on the remote".
- **Analogy: a contact in your phone.** The contact entry is not the person. It is a way to reach the person.

This is the difference between the two families of types:

| | Primitive types | Reference types |
| :-- | :-- | :-- |
| Which ones | `int`, `long`, `double`, `boolean`, `char` | Every class: `String`, `Scanner`, `EquipmentItem`, arrays |
| The variable holds | The value itself | A reference to an object |
| Spelling clue | all lower case | starts with a capital |

### 10.2 The memory picture: stack and heap

Lecture 1 draws memory as two areas.

| Area | Plain meaning | What lives there |
| :-- | :-- | :-- |
| **Stack** | A small, tidy area for the variables of the method that is running now | Local variables: primitives hold their value; reference variables hold a reference |
| **Heap** | A large open area | Every object built with `new` |

```java
EquipmentItem laptop = new EquipmentItem(101, "Dell Laptop", 185000.00, true);
EquipmentItem camera = new EquipmentItem(102, "Canon Camera", 120000.00, true);
```

```
      STACK (variables)                 HEAP (objects)

      +----------+                +---------------------------+
      |  laptop  | -------------> | EquipmentItem             |
      +----------+                | id = 101                  |
                                  | name = "Dell Laptop"      |
                                  | replacementCost = 185000.0|
                                  | available = true          |
                                  +---------------------------+

      +----------+                +---------------------------+
      |  camera  | -------------> | EquipmentItem             |
      +----------+                | id = 102                  |
                                  | name = "Canon Camera"     |
                                  | replacementCost = 120000.0|
                                  | available = true          |
                                  +---------------------------+
```

The arrows are the references. Your notes add a caution: treat this as a helpful picture, not an exact description of every JVM. And say "reference", not "memory address"; Java deliberately hides real addresses from you.

### 10.3 Aliasing: two names, one object

Predict the output before reading on. This is a slide from your lecture.

```java
EquipmentItem item1 = new EquipmentItem(101, "Laptop", 185000, true);
EquipmentItem item2 = item1;

item2.name = "Updated Laptop";

System.out.println(item1.name);
```

**Output:** `Updated Laptop`

| Line | What happens |
| :-- | :-- |
| 1 | `new` builds **one** object. `item1` refers to it. |
| 2 | **There is no `new` on this line, so no object is built.** The reference inside `item1` is copied into `item2`. Both now lead to the same object. |
| 4 | Through `item2`, the object's `name` is changed. |
| 6 | Through `item1`, we look at the same object. The name has changed. |

```
      +---------+
      |  item1  | ------+
      +---------+       |        +--------------------------+
                        +------> | EquipmentItem            |
      +---------+       |        | id = 101                 |
      |  item2  | ------+        | name = "Updated Laptop"  |
      +---------+                +--------------------------+
                                  ONE object, TWO names
```

Two variables referring to the same object is called **aliasing**. An **alias** is a second name for the same thing.

- **Analogy:** two remote controls paired to the same TV. Change the channel with either one, and the one TV changes.
- **Analogy:** "Mum" and "Mrs Perera" are two names for one person. If Mrs Perera gets a haircut, Mum has a haircut too.

> **Counting rule: count the `new`s to count the objects.** One `new` means one object, no matter how many variables refer to it.

Compare with primitives, where the value itself is copied:

```java
int a = 5;
int b = a;     // b gets its own copy of 5
b = 99;
System.out.println(a);   // still 5
```

### 10.4 `==` on objects compares identity

```java
EquipmentItem a = new EquipmentItem(101, "Laptop", 185000, true);
EquipmentItem b = new EquipmentItem(101, "Laptop", 185000, true);
EquipmentItem c = a;

System.out.println(a == b);   // false
System.out.println(a == c);   // true
```

For reference variables, `==` asks "do these two refer to **the very same object**?" It does not look inside at the field values.

- `a == b` is `false`: two `new`s, two objects, even though the contents match.
- `a == c` is `true`: `c` is an alias of `a`.

**This finally explains the rule from Pre-Phase 0.** `String` is a class, so a `String` variable holds a reference. `==` on two Strings asks "same object?", not "same letters?". `.equals()` is the method that compares the letters. That is why text is compared with `.equals()`.

### 10.5 `null`: a name with nothing attached

```java
EquipmentItem item = null;
```

**`null` is a special value meaning "this reference variable refers to no object at all".**

- **Analogy:** a remote control that is not paired with any TV. You can hold it. Pressing a button does nothing useful.
- **Analogy:** a contact entry with a name but no phone number.

It is legal to store `null`. The trouble comes when you use the dot on it:

```java
EquipmentItem missing = null;
missing.displayDetails();
```

```
Exception in thread "main" java.lang.NullPointerException: Cannot invoke
"EquipmentItem.displayDetails()" because "missing" is null
        at Main.main(Main.java:4)
```

| Part of the message | Meaning |
| :-- | :-- |
| `Exception` | Java's word for "something went wrong while running". Phase 6 covers exceptions fully. |
| `NullPointerException` | You used the dot on a reference that is `null`. Often shortened to **NPE**. |
| `because "missing" is null` | Java names the guilty variable for you. |
| `at Main.main(Main.java:4)` | File `Main.java`, line 4. |

This code **compiles**. The compiler only checks that `missing` has the type `EquipmentItem` and that the type has a `displayDetails` method. Whether a real object is attached is only known when the program runs. So this is a **runtime error**.

> **Model answer (notes Q8), "Why can code compile and still fail at runtime?"** The compiler checks the language and type rules, but the state of the program while it runs can still be invalid, for example using a reference that is `null`.

This also completes the table from Section 2.3: a field of a class type, such as `String name`, starts as `null` if nothing is assigned to it. And it answers row 8 of the Pre-Phase 0 warm-up: `String s = null; s.length();` compiles, then throws `NullPointerException`.

`null` matters all through the module. When you search a list for a product and nothing matches, what do you hand back? Phase 3 (`Optional`) and Phase 6 (exceptions) are both answers to that question.

### 10.6 What happens to objects nobody refers to?

If no variable refers to an object any more, the program can never reach it again. Java notices and reclaims the memory automatically. This is called **garbage collection**. You never delete objects yourself in Java.

---

## 11. `main` and the "static context" error

Now you can understand `public static void main(String[] args)` properly.

`main` is `static`, so it belongs to the class, not to any object. When your program starts, **no objects exist yet**. The JVM needs a starting point it can run without building anything first. That is why `main` must be `static`.

The consequence is one of the most common errors for beginners:

```java
public class Main {

    EquipmentItem item = new EquipmentItem(101, "Laptop", 850.0, true);

    public static void main(String[] args) {
        System.out.println(item.name);   // COMPILE ERROR
    }
}
```

```
non-static variable item cannot be referenced from a static context
```

| Phrase | Meaning |
| :-- | :-- |
| "non-static variable `item`" | `item` is a field. It belongs to a `Main` **object**. |
| "static context" | Inside `main`, which is `static`. No `Main` object exists there, so there is no `this`. |
| Together | "You are asking for a field of an object, but there is no object here." |

**The fix:** declare the variable **inside** `main`, as every example in this guide does.

```java
public class Main {
    public static void main(String[] args) {
        EquipmentItem item = new EquipmentItem(101, "Laptop", 850.0, true);
        System.out.println(item.name);
    }
}
```

Do not fix it by sprinkling `static` on things.

---

## 12. The same ideas in your other lecture: `Product`

Lecture 1 teaches the same concepts with a `Product` class. This is the class the Spring Boot track grows all semester, so recognise it now.

```java
public class Product {

    Long id;
    String name;
    double price;

    public Product(Long id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }
}
```

```java
Product p1 = new Product(1L, "Laptop", 999.99);
Product p2 = new Product(2L, "Headphones", 49.99);

System.out.println(p1.name);    // Laptop
System.out.println(p2.price);   // 49.99
```

It is the same pattern: fields, a constructor using `this.field = parameter`, objects built with `new`.

Two small new details:

| Detail | Meaning |
| :-- | :-- |
| `Long` with a capital L | `long` (small l) is the primitive. `Long` (capital L) is a class version of the same thing, called a **wrapper**: a `long` wrapped inside an object. It behaves almost the same, but because it is a reference type it can be `null`, meaning "no id yet". Phase 3 explains why collections need wrappers. |
| `1L` | A `long` literal. The `L` tells Java "this number is a long, not an int". **With a `Long` parameter the `L` is required**: `new Product(1, "Laptop", 999.99)` is a compile error, because Java will not turn an `int` into a `Long` object automatically. (With a primitive `long` parameter, plain `101` is fine, which is why the `EquipmentItem` examples work without the `L`.) |

> **Exam tip.** The project you are given may use `Long id`. When you build objects in it, write ids as `1L`, `2L`, `3L`.

---

## 13. A simple picture of a class

Before formal UML in Phase 2, your lecture uses a simple three-box sketch. Top: the class name. Middle: the fields (state). Bottom: the constructor and methods (behaviour).

```
+------------------------------------------------+
|                 EquipmentItem                  |
+------------------------------------------------+
| id : long                                      |
| name : String                                  |
| replacementCost : double                       |
| available : boolean                            |
+------------------------------------------------+
| EquipmentItem(id, name, cost, available)       |
| displayDetails() : void                        |
| calculateLateFee(days) : double                |
| isExpensive(threshold) : boolean               |
+------------------------------------------------+
```

In these diagrams the type is written **after** a colon: `name : String`. For methods, what follows the colon is the return type.

The `Circle` from Tutorial Exercise 12 is drawn the same way:

```
+-------------------------------+
|            Circle             |
+-------------------------------+
| - radius : double             |
+-------------------------------+
| + getArea() : double          |
| + getCircumference() : double |
+-------------------------------+
```

The `-` and `+` signs mean "private" and "public". Phase 2 explains them. Here is the Circle as code, with what you know so far:

```java
public class Circle {

    double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    public double getArea() {
        return Math.PI * radius * radius;
    }

    public double getCircumference() {
        return 2 * Math.PI * radius;
    }
}
```

```java
Circle small = new Circle(5);
System.out.println(small.getArea());            // 78.53981633974483
System.out.println(small.getCircumference());   // 31.41592653589793
```

Compare with Pre-Phase 0. There, `calculateArea(radius)` needed the radius passed in every time. Here `small.getArea()` needs **nothing** passed in, because the circle already knows its own radius. That is the shift your tutorial describes as "DATA + OPERATIONS → OBJECT".

---

## 14. Common mistakes

These six are named on your lecture slide.

| # | Mistake | What goes wrong | Fix |
| :-- | :-- | :-- | :-- |
| 1 | Writing `void` before a constructor | It becomes an ordinary method. It never runs on `new`. Fields stay at defaults. | No return type on constructors. |
| 2 | Writing the class but never creating an object | Nothing happens. A blueprint alone does nothing. | Use `new` in `main`. |
| 3 | Forgetting `this` (`id = id;`) | The parameter is assigned to itself. The field stays at its default. No error shown. | `this.id = id;` |
| 4 | Confusing the reference variable with the object | Surprise when changing through one name affects "another" object. | Count the `new`s. |
| 5 | Putting all behaviour in `main` | Rules scattered and copied. | Put the method in the class that owns the data. |
| 6 | Not running after each small change | Ten errors at once, with no idea which change caused them. | Run after every small step. |

### Errors you will meet in this phase

| Message | Kind | Cause |
| :-- | :-- | :-- |
| `constructor X in class X cannot be applied to given types` | Compile | Wrong number, order or types of arguments after `new X(...)`. Compare your arguments with the constructor's parameter list. |
| `cannot find symbol` | Compile | Misspelled field or method name, wrong capitals, or calling a method the class does not have. |
| `incompatible types` | Compile | For example, storing the result of a `void` method, or passing text where a number is needed. |
| `non-static ... cannot be referenced from a static context` | Compile | Using a field or instance method directly from `main` without an object. |
| `NullPointerException` | Runtime | Using the dot on a reference that is `null`. |

**Debugging routine from your notes (§27):** read the first error, click to the line, check spelling and capitals, check the constructor's parameter count, order and types, change one thing, run again.

---

## 15. Practice

### 15.1 Checkpoint questions (from your slides and notes)

Answer each one out loud before reading the model answer.

| # | Question | Model answer |
| :-- | :-- | :-- |
| 1 | What is the difference between a class and an object? | A class defines a type (a blueprint). An object is one instance created from that class. |
| 2 | What does `new` do? | It creates a new object in memory, calls a constructor, and produces a reference to the object. |
| 3 | Why does the constructor have the same name as the class? | That is the rule that identifies it as a constructor. It must match the class name and has no return type. |
| 4 | What does `this.id` mean? | The `id` field inside the current object. |
| 5 | If we create two `EquipmentItem` objects, do they share the same `name` field? | No. Each object has its own independent field values. |
| 6 | Why should `calculateLateFee` belong inside `EquipmentItem`? | It is behaviour of an equipment item and uses its rules, so it lives with the data it relates to. |
| 7 | Does `EquipmentItem item;` create an object? | No. It declares a reference variable that is able to refer to one. |
| 8 | Can two reference variables refer to the same object? | Yes. Assigning one to the other copies the reference, not the object. |
| 9 | What does `null` mean? | The reference currently refers to no object. |
| 10 | Why is a constructor different from a normal method? | It is used during object creation, has the same name as the class, and has no return type. |
| 11 | Give one example of state and one of behaviour in `EquipmentItem`. | State: `replacementCost`. Behaviour: `calculateLateFee()`. |

### 15.2 Predict the output

Assume the `EquipmentItem` class from Section 9.

```java
// A
EquipmentItem x = new EquipmentItem(1, "Tripod", 15000, false);
EquipmentItem y = new EquipmentItem(1, "Tripod", 15000, false);
System.out.println(x == y);

// B
EquipmentItem p = new EquipmentItem(2, "Projector", 110000, true);
EquipmentItem q = p;
p.available = false;
System.out.println(q.available);

// C
EquipmentItem m = new EquipmentItem(3, "Mic", 9000, true);
EquipmentItem n = new EquipmentItem(4, "Speaker", 30000, true);
m.name = "Microphone";
System.out.println(n.name);

// D
EquipmentItem z = null;
System.out.println(z.name);

// E  How many objects exist after these three lines?
EquipmentItem a = new EquipmentItem(5, "Cable", 500, true);
EquipmentItem b = a;
EquipmentItem c = b;
```

**Answers**

| | Output | Reason |
| :-- | :-- | :-- |
| A | `false` | Two `new`s, two objects. Same state, different identity. |
| B | `false` | `q` is an alias of `p`. One object, changed through `p`, seen through `q`. |
| C | `Speaker` | `m` and `n` are separate objects. Changing one does not touch the other. |
| D | `NullPointerException` at runtime | It compiles, but `z` refers to no object. |
| E | **One** object | One `new`. Three names for it. |

### 15.3 Spot the error

```java
// A
public class Book {
    String title;
    public void Book(String title) {
        this.title = title;
    }
}

// B
public class Book {
    String title;
    public Book(String title) {
        title = title;
    }
}

// C   (Book has only the constructor Book(String title))
Book b = new Book();

// D
public class Book {
    String title;
    public Book(String title) { this.title = title; }
    public static void printTitle() {
        System.out.println(title);
    }
}
```

| | Mistake | Result |
| :-- | :-- | :-- |
| A | `void` before the constructor name. | It is now an ordinary method. `new Book("Dune")` is a compile error because no one-argument constructor exists. |
| B | Missing `this.` | Compiles. Every book's `title` stays `null`. |
| C | No no-argument constructor exists once you write your own. | Compile error: "cannot be applied to given types". |
| D | A `static` method trying to read an instance field. | Compile error: "non-static variable title cannot be referenced from a static context". Remove `static`. |

### 15.4 Build it yourself: the `Room` class

This is the after-class practice from your lecture. **Do it without looking at `EquipmentItem`.** Building a second class from scratch is the proof that you understand the first.

Task:

1. Create a class `Room` with `roomNumber`, `capacity` and `available`.
2. Add a constructor using `this`.
3. Add `isLargeRoom(int threshold)` that returns true if capacity is above the threshold.
4. Add `displayRoomDetails()`.
5. In `main`, create three `Room` objects and call both methods.
6. Write down which parts are state and which are behaviour.

Give yourself 20 minutes, then compare.

**`Room.java`**

```java
public class Room {

    String roomNumber;
    int capacity;
    boolean available;

    public Room(String roomNumber, int capacity, boolean available) {
        this.roomNumber = roomNumber;
        this.capacity = capacity;
        this.available = available;
    }

    public boolean isLargeRoom(int threshold) {
        return capacity > threshold;
    }

    public void displayRoomDetails() {
        System.out.println("Room " + roomNumber
                + " | capacity=" + capacity
                + " | available=" + available);
    }
}
```

**`RoomMain.java`**

```java
public class RoomMain {
    public static void main(String[] args) {

        Room lab = new Room("C1.04", 30, true);
        Room hall = new Room("LT1", 200, false);
        Room office = new Room("B2.11", 4, true);

        lab.displayRoomDetails();
        hall.displayRoomDetails();
        office.displayRoomDetails();

        System.out.println(lab.isLargeRoom(50));
        System.out.println(hall.isLargeRoom(50));
    }
}
```

**Output:**

```
Room C1.04 | capacity=30 | available=true
Room LT1 | capacity=200 | available=false
Room B2.11 | capacity=4 | available=true
false
true
```

- **State:** `roomNumber`, `capacity`, `available`.
- **Behaviour:** `isLargeRoom()`, `displayRoomDetails()`.
- `roomNumber` is a `String`, not a number, because values like "C1.04" and "LT1" contain letters.

### 15.5 More exercises (from Appendix B of your notes)

Try each one, then check.

**1. Count the expensive items.** Create three items and count how many are expensive using a threshold of 100000.

```java
int count = 0;
if (laptop.isExpensive(100000)) { count++; }
if (camera.isExpensive(100000)) { count++; }
if (tripod.isExpensive(100000)) { count++; }
System.out.println(count);
```

**2. Add a method that returns text.** `getAvailabilityMessage()` returns `"Available"` or `"Currently unavailable"`.

```java
public String getAvailabilityMessage() {
    if (available) {
        return "Available";
    }
    return "Currently unavailable";
}
```

`if (available)` needs no `== true`, because `available` is already a `boolean`. The second `return` needs no `else`: the method only reaches it when the `if` was false, because `return` leaves the method immediately.

**3. Depreciation.** Add `replacementCostAfterDepreciation(double rate)` that returns the cost after a rate such as `0.10` (10%).

```java
public double replacementCostAfterDepreciation(double rate) {
    return replacementCost * (1 - rate);
}
```

**4. A second small class.** Create `Student` with `id` and `name`, a constructor, and `displayDetails()`.

```java
public class Student {

    long id;
    String name;

    public Student(long id, String name) {
        this.id = id;
        this.name = name;
    }

    public void displayDetails() {
        System.out.println(id + " - " + name);
    }
}
```

**5. Break the constructor on purpose.** Call `new EquipmentItem("Laptop", 101, true, 850.0)` with the arguments in the wrong order. Read the compiler message and explain it in your own words before fixing it.

---

## 16. Glossary

| Term | Plain-English meaning |
| :-- | :-- |
| **Alias / aliasing** | A second variable referring to the same object. |
| **Argument** | An actual value supplied in a method or constructor call. |
| **Attribute** | Another word for field. |
| **Behaviour** | What an object can do: its methods. |
| **Blueprint** | The everyday picture of a class. |
| **Class** | A definition of a new type: which fields its objects hold and which methods they have. |
| **Command** | A method that performs an action and may change state. |
| **Constructor** | A special block, named after the class and with no return type, that runs on `new` to set up a new object. |
| **Default constructor** | The free no-argument constructor Java supplies only when you write none. |
| **Default value** | What a field holds if never assigned: `0`, `0.0`, `false` or `null`. |
| **Dot notation** | Using `.` to reach a field or method of an object: `laptop.name`. |
| **Field** | A variable declared in a class, outside methods; one piece of an object's data. |
| **Garbage collection** | Java automatically reclaiming objects nothing refers to. |
| **Heap** | The memory area where objects live. |
| **Identity** | What makes an object that particular object, regardless of its values. |
| **Instance** | An object. "An instance of `X`" = an object built from class `X`. |
| **Instance method** | A non-static method, called on an object, able to use its fields. |
| **Instantiate** | To build an object from a class. |
| **`new`** | The keyword that builds an object and calls a constructor. |
| **No-argument constructor** | A constructor with no parameters. |
| **`null`** | A reference value meaning "no object". |
| **NullPointerException (NPE)** | The runtime error from using the dot on `null`. |
| **Object** | One real thing built from a class, with its own state and identity. |
| **Parallel arrays** | Several arrays where the same index describes the same item; fragile. |
| **Predicate** | A method that returns `true` or `false`. |
| **Primitive type** | `int`, `long`, `double`, `boolean`, `char`: the variable holds the value itself. |
| **Query** | A method that returns information without changing state. |
| **Receiver** | The object to the left of the dot in a method call. |
| **Reference** | A value that leads to an object (or is `null`). |
| **Reference type** | Any class type; the variable holds a reference. |
| **Shadowing** | A parameter hiding a field of the same name. |
| **Stack** | The memory area where local variables live. |
| **State** | The current values of an object's fields. |
| **`static`** | Belonging to the class as a whole rather than to one object. |
| **`this`** | The current object, inside a constructor or instance method. |
| **Wrapper** | A class version of a primitive, such as `Long` for `long`. |

---

## 17. One-page cheat sheet

```java
// ---------- EquipmentItem.java ----------
public class EquipmentItem {                       // class = blueprint = new type

    // 1. FIELDS (state): one copy per object
    long id;
    String name;
    double replacementCost;
    boolean available;

    // 2. CONSTRUCTOR: same name as class, NO return type, runs on `new`
    public EquipmentItem(long id, String name,
                         double replacementCost, boolean available) {
        this.id = id;                              // this.field = parameter;
        this.name = name;
        this.replacementCost = replacementCost;
        this.available = available;
    }

    // 3. METHODS (behaviour): NOT static, use the object's own fields
    public void displayDetails() {                 // command: does something
        System.out.println(id + " - " + name);
    }

    public double calculateLateFee(int daysLate) { // query: returns a value
        return daysLate * 2.50;
    }

    public boolean isExpensive(double threshold) { // predicate: true / false
        return replacementCost > threshold;
    }
}

// ---------- Main.java ----------
public class Main {
    public static void main(String[] args) {
        EquipmentItem laptop =
            new EquipmentItem(101, "Dell Laptop", 185000.00, true);   // build

        laptop.displayDetails();                    // call through the object
        double fee = laptop.calculateLateFee(4);    // store the return value

        EquipmentItem alias = laptop;               // no new = no new object
        EquipmentItem none = null;                  // refers to nothing
    }
}
```

| Remember | |
| :-- | :-- |
| Class = blueprint, object = thing built from it | One class, many objects |
| Fields are state, methods are behaviour | Each object has its own field values |
| Constructor: class name, no return type | Writing `void` breaks it silently |
| `this.x = x;` | Without `this`, the field is never set |
| Count the `new`s to count the objects | `b = a;` makes an alias, not a copy |
| `==` on objects asks "same object?" | Text is compared with `.equals()` |
| The dot on `null` gives `NullPointerException` | It compiles, then fails at runtime |
| Instance methods are not `static` | Call them as `object.method()` |
| Writing your own constructor removes the free empty one | `new X()` then fails |
| `Long` ids need the `L` | `new Product(1L, "Laptop", 999.99)` |

---

## 18. Am I ready for Phase 2?

This is the exit checklist from your lecture, plus the exit-ticket questions.

- [ ] I can create a Java class with fields. (§2)
- [ ] I can write a constructor with parameters, using `this` to assign them. (§4, §5)
- [ ] I can create several objects with `new` and explain why they are independent. (§3, §6)
- [ ] I can call methods on an object with dot notation. (§7)
- [ ] I can explain state, identity and behaviour using `EquipmentItem`. (§8)
- [ ] I can add a new method to the class and test it from `main`. (§7, §15)
- [ ] I can explain why `EquipmentItem item;` creates no object. (§6)
- [ ] I can explain what happens when two variables refer to one object. (§10.3)
- [ ] I can explain what `null` means and why using it compiles but crashes. (§10.5)
- [ ] I built the `Room` class without copying. (§15.4)

### The question that opens Phase 2

Today's class works, but it is not well designed. This is currently allowed:

```java
EquipmentItem laptop = new EquipmentItem(101, "Dell Laptop", 185000.00, true);

laptop.replacementCost = -5000;
laptop.available = false;
```

Any code anywhere can reach into the object and give it a negative cost. Nothing stops it. **How can an object protect its own state so outside code cannot put it into an invalid condition?**

The answer is `private` fields, `public` methods, validation, and a diagram language called UML.

**Next: Phase 2, Encapsulation and UML Class Diagrams.**
