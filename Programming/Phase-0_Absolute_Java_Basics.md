# Pre-Phase 0: Absolute Java Basics

**Module:** 5COSC019W Object Oriented Programming
**Who this is for:** someone who has never written a line of code.
**Goal:** by the end you can read and write small Java programs, so that Phase 1 (classes) makes sense.
**Time needed:** about 3 to 4 hours if you type every example yourself.

**Built from your files:** Week 01 Tutorial (Java Refresher, Exercises 1–12), Pure OOP Week 01 Lecture (Sections 01–02), Pure OOP Week 01 Lecture Notes (§5–9, §25–27), Lecture 1 (Section 02, Java Foundations Recap).

---

## How to study this guide

1. **Read a section, then type the code yourself.** Do not copy and paste. Typing is how your fingers learn where the brackets go.
2. **Predict before you run.** Before pressing Run, say out loud what you think will appear. Your lecturer uses this exact habit ("predict first") in every session.
3. **Break it on purpose.** Delete a semicolon, misspell a word, and read the error message. In the exam you will see error messages, and they must not scare you.
4. **Every new word is defined the first time it appears**, and again in the [Glossary](#16-glossary) at the end.

---

## The full roadmap: every phase, in order

You do not need to understand the words in this table yet. It is a map so you know where you are going.

| Phase | Title | What you learn | Your files it covers |
| :-- | :-- | :-- | :-- |
| **Pre-Phase 0** *(this guide)* | Absolute Java Basics | What Java is, variables, types, operators, input, `if`, loops, methods, arrays, reading errors | Week 01 Tutorial; Pure OOP Week 01 Lecture §01–02; Lecture Notes §5–9, §25–27; Lecture 1 §02 |
| **Phase 1** | Classes and Objects | Class vs object, fields, constructors, `new`, `this`, state / identity / behaviour, references, `null`, memory (stack and heap), methods inside a class | Pure OOP Week 01 Lecture §02–04; Lecture Notes §10–24, §28–34; Lecture 1 §03; Week 01 Tutorial Ex 12 |
| **Phase 2** | Encapsulation and UML Class Diagrams | `private` / `public` / `protected` / default, getters and setters, the JavaBean naming rule, full and no-argument constructors, immutability, one class containing another (Customer has an Address), drawing UML | Lecture 2; Tutorial 2 |
| **Phase 3** | Collections and Generics | `List`, `ArrayList`, the `<T>` in angle brackets, `Map`, the for-each loop, `stream().filter(...)`, `Optional`, cohesion and coupling, why logic moves into a "service" class | Lecture 3; Tutorial 3 |
| **Phase 4** | Inheritance | `extends`, `super(...)`, what a child class gets from its parent, `protected`, IS-A versus HAS-A, UML hollow triangle | Lecture 4; Tutorial 4 |
| **Phase 5** | Polymorphism and Abstraction | Overriding, `@Override`, runtime dispatch, reference type vs actual type, casting and `instanceof`, `abstract` classes, interfaces, UML for both | Lecture 5; Tutorial 5 |
| **Phase 6** | Exceptions | The exception family tree, `try` / `catch` / `finally`, `throw`, writing your own exception class, checked vs unchecked | Lecture 6 §01–04 |
| **Phase 7** | Dependencies and Layered Design | What a dependency is, constructor injection, why interfaces make swapping parts possible, the repository pattern, controller → service → repository | Lecture 7 |
| **Phase 8** | Exam Consolidation | The exam's shape (4 code tasks + functional checks, 100 minutes), a full timed walk-through, checklists, common mark-losing mistakes | Student Assessment Briefing; Lecture 6 §07 (Mock Lab Exam Briefing) |

**The Spring Boot track (your "enterprise / design" lecture, for the other chat):** Lecture 1 §04–07, Tutorial 1, the Spring halves of Tutorials 2–5, Lecture 6 §05–06 (`@ControllerAdvice`, HTTP status codes), and the Spring annotations in Lecture 7.

> **Exam warning.** Your briefing says Task 4 (Spring REST, 20 marks) and Section B (running-program checks, 20 marks) both need Spring. That is 40 of 100 marks. Do not leave the Spring track to the last evening.

---

## 1. What is a program? What is Java?

### 1.1 A program

A **program** is a list of instructions that a computer follows, one after another, exactly as written.

- **Analogy: a recipe.** A recipe says "crack two eggs, stir, bake for 20 minutes". The cook follows each step in order. A program is a recipe, and the computer is a cook who is extremely fast but has zero common sense. It does exactly what you wrote, even if what you wrote is silly.

**Programming** (also called **coding**) means writing those instructions.

### 1.2 A programming language

Computers do not understand English. A **programming language** is a small, strict language that both humans and computers can work with. It has a tiny vocabulary and very strict grammar.

**Java** is one programming language. Your whole module uses it. It is popular for big business systems (banks, shops, booking sites).

### 1.3 Code and syntax

- **Code** (or **source code**) is the text you type in a programming language.
- **Syntax** is the grammar of the language: the rules about where words, brackets and punctuation must go.

- **Analogy: grammar.** In English, "Dog the barked" breaks the grammar rules. A human still understands it. A computer does not forgive anything. One missing semicolon and Java refuses to continue.

---

## 2. How the computer reads Java

Your computer's processor only understands very low-level number codes. It cannot read Java directly. So Java code goes through two steps before anything happens.

```
 YOU TYPE            STEP 1: COMPILE            STEP 2: RUN
+--------------+     +---------------+     +----------------------+
| Week01.java  | --> | Week01.class  | --> | The JVM follows the  |
| (source code)|     | (bytecode)    |     | instructions         |
+--------------+     +---------------+     +----------------------+
 human-readable       translated form       things actually happen
```

| Word | Plain-English meaning |
| :-- | :-- |
| **Source code** | The text you type. Saved in a file ending in `.java`. |
| **Compile** | To translate your whole source code into a form the machine side can use. The translating is done before the program runs. |
| **Compiler** | The tool that does the translating. While translating, it also checks your grammar (syntax) and refuses to finish if it finds a mistake. Java's compiler is called `javac`. |
| **Bytecode** | The translated result. Saved in a file ending in `.class`. You never read or edit it. |
| **JVM** (Java Virtual Machine) | A program that reads bytecode and carries out the instructions. "Virtual machine" means "a pretend computer made of software". |
| **Run** / **execute** | To actually carry out the instructions. |
| **JDK** (Java Development Kit) | The installable package that contains the compiler, the JVM and Java's built-in tools. |
| **IDE** (Integrated Development Environment) | A program for writing code comfortably. Yours is **IntelliJ IDEA**. It colours your code, underlines mistakes in red, and its green **Run** button does both steps (compile, then run) with one click. |
| **Console** | The text panel at the bottom of IntelliJ where your program's printed messages appear. |

- **Analogy: a translator and an actor.** You write a play in English (source code). A translator turns the whole script into French and hands it back, refusing if your English grammar is wrong (compiling). Then a French actor performs it on stage (the JVM running it).

### 2.1 Two moments when things can go wrong

This idea comes up in every lecture you were given, so learn it now.

| Name | When it happens | Example | Does the program start? |
| :-- | :-- | :-- | :-- |
| **Compile-time error** | During translation (step 1) | A missing semicolon; a misspelled word | No. Nothing runs at all. |
| **Runtime error** | While the program is running (step 2) | Dividing a whole number by zero | Yes, then it crashes part-way. |

"**Compile time**" means "the moment of translating". "**Runtime**" means "the period while the program is running".

A compile-time error is the friendlier kind. The compiler catches the mistake for you before any harm is done.

---

## 3. Your first program, line by line

This is Exercise 1 of your Week 01 Tutorial.

```java
public class Week01 {
    public static void main(String[] args) {
        System.out.println("Welcome to Object-Oriented Programming");
        System.out.println("Java Refresher - Week 01");
    }
}
```

**What appears in the console when you run it:**

```
Welcome to Object-Oriented Programming
Java Refresher - Week 01
```

**How to run it in IntelliJ:** File → New → Project → choose Java → Create. Right-click the `src` folder → New → Java Class → type `Week01`. Type the code. Click the green triangle next to `main`.

### 3.1 Line 1: `public class Week01 {`

| Piece | What it means |
| :-- | :-- |
| `public` | "Visible to everyone." It is a permission word. For now, always write it here. Phase 2 explains permissions fully. |
| `class` | A **class** is a named container that holds code. Every piece of Java code must live inside one. Think of it as a labelled folder. (In Phase 1 you will learn a class is much more powerful than a folder. For today, "container" is enough.) |
| `Week01` | The name you chose for the class. Rule: **the file must be called exactly the same**, so this lives in `Week01.java`. Class names start with a capital letter. |
| `{` | An opening **curly brace**. It means "the contents of the class start here". |

### 3.2 Line 2: `public static void main(String[] args) {`

This line is the **starting point** of every Java program. When you press Run, the JVM looks for a line that looks exactly like this and begins there.

- **Analogy: the front door.** A house can have many rooms, but you enter through one front door. `main` is the front door.

| Piece | What it means |
| :-- | :-- |
| `public` | Visible to everyone, so the JVM is allowed to find and start it. |
| `static` | "Belongs to the class itself." It means the JVM can start this without building anything first. Phase 1 explains this properly; today, treat it as required wording. |
| `void` | "Gives nothing back." When `main` finishes, it does not hand a result to anyone. (Section 11 explains "giving back".) |
| `main` | The name. It must be spelled `main`, all lower case, because that is the name the JVM searches for. |
| `(` `)` | **Round brackets** (parentheses). They hold the inputs that this block of code receives. |
| `String[] args` | The input `main` receives: a list of pieces of text that could be passed in when the program is launched. `String` means text, `[]` means "a list of", and `args` is just the list's name (short for "arguments"). You will almost never use it, but it must be written. |
| `{` | "The instructions of `main` start here." |

A named block of instructions like `main` is called a **method**. Section 11 teaches you to write your own.

> **Tutorial question: "What is the purpose of `public static void main(String[] args)`?"**
> Model answer: it is the entry point of a Java program. The JVM starts executing here. `public` lets the JVM access it, `static` means no instance has to be created first, `void` means it returns nothing, and `String[] args` receives any command-line text.

### 3.3 Line 3: `System.out.println("Welcome to Object-Oriented Programming");`

This line prints a message in the console.

| Piece | What it means |
| :-- | :-- |
| `System` | A built-in Java toolbox that represents the computer system your program runs on. |
| `.` | The **dot** means "go inside" or "use the thing that belongs to". Read `A.B` as "the B that belongs to A". |
| `out` | The "output" channel inside `System`. It is connected to the console. |
| `.` | "Go inside" again. |
| `println` | Short for "print line". It prints the text, then moves to a new line (like pressing Enter). |
| `(` | Opens the round brackets that hold what we want to print. |
| `"Welcome to ..."` | The text to print. Text wrapped in **double quotes** is called a **String**. The quotes are not printed; they only mark where the text starts and ends. |
| `)` | Closes the round brackets. |
| `;` | A **semicolon**. It ends the instruction, like a full stop ends a sentence. |

One complete instruction ending in `;` is called a **statement**.

- **Analogy:** `System.out.println("Hi")` reads like "Building → loudspeaker → announce this: Hi".

`System.out.print(...)` (without `ln`) prints the text but stays on the same line.

### 3.4 Lines 5 and 6: `}` and `}`

Each closing curly brace `}` ends the block that the matching `{` opened. The first `}` ends `main`. The second ends the class.

**Rule: every `{` needs exactly one `}`.** A missing brace is one of the most common beginner errors.

---

## 4. Syntax ground rules

| Rule | Detail | Example |
| :-- | :-- | :-- |
| **Statements end with `;`** | A semicolon ends an instruction. | `int age = 20;` |
| **Blocks use `{ }`** | A **block** is a group of statements wrapped in curly braces. Lines that open a block (`class`, `main`, `if`, `for`) do **not** end in `;`. | `if (x > 0) { ... }` |
| **Java is case-sensitive** | Capital and small letters are different letters. `System` works; `system` is an error. `String` works; `string` is an error. | |
| **Text goes in double quotes** | `"Hello"` is text. `Hello` without quotes is treated as a name Java should already know. | |
| **Brackets come in pairs** | `( )`, `{ }`, `[ ]`, `" "` must each be opened and closed. | |
| **Spaces and blank lines are free** | Java ignores extra spaces and line breaks. Use them to keep code readable. | |

### 4.1 Indentation

**Indentation** means pushing a line to the right with spaces. Each time you go inside a `{`, indent by 4 spaces. Java does not require it, but humans do. Without it you cannot see which `}` closes which `{`. IntelliJ fixes it for you with **Ctrl + Alt + L**.

### 4.2 Comments

A **comment** is a note for humans. The compiler skips it completely.

```java
// This is a single-line comment. Everything after the two slashes is ignored.

/* This is a
   multi-line comment. */
```

Your exam starter files mark the places you must fill in with comments such as `// TODO`.

### 4.3 Naming rules

An **identifier** is any name you invent (for a class, a variable, a method).

- Letters, digits, `_` and `$` only. No spaces. Cannot start with a digit.
- Cannot be a **keyword**. A keyword is a word Java has reserved for itself, such as `class`, `public`, `int`, `if`, `for`, `return`.
- **Conventions** (habits every Java programmer follows; your lecturers expect them):
  - Class names: `UpperCamelCase`, e.g. `EquipmentItem`, `ProductController`.
  - Variable and method names: `lowerCamelCase`, e.g. `averageMark`, `calculateArea`.
  - "Camel case" means each new word starts with a capital, like the humps of a camel.

---

## 5. Variables and data types

### 5.1 What is a variable?

A **variable** is a named storage space in the computer's memory that holds one value. A **value** is a single piece of data, such as the number `20` or the text `"Kamal"`.

- **Analogy: a labelled box.** A variable is a box with a label on the outside (its name) and something inside (its value). The box is built to hold only one kind of thing (its type).

### 5.2 What is a data type?

A **data type** (or just **type**) says what kind of value a box can hold. Java makes you choose the type when you create the box, and the box can never hold anything else. Your lecture notes call this **static typing**: the compiler checks every type before the program runs.

| Type | Holds | Example values | Notes |
| :-- | :-- | :-- | :-- |
| `int` | Whole numbers (no decimal point) | `10`, `0`, `-5` | Short for "integer". Range roughly ±2 billion. |
| `long` | Very large whole numbers | `101L` | Write a capital `L` after the number. Your module uses this for ID numbers. |
| `double` | Numbers with a decimal point | `19.99`, `0.5`, `67.0` | |
| `boolean` | Only `true` or `false` | `true`, `false` | A yes/no switch. Not 1 or 0. |
| `char` | Exactly one character | `'B'`, `'7'`, `'?'` | Uses **single** quotes. |
| `String` | Text of any length | `"Kamal"`, `"A101"`, `""` | Uses **double** quotes. Capital `S`. |

`int`, `long`, `double`, `boolean` and `char` are called **primitive types**. "Primitive" means "simple and built-in": the box holds the plain value itself. `String` is a different kind (a "reference type"); Phase 1 explains the difference. Today you only need one consequence of it, covered in [Section 7.5](#75-comparing-text-use-equals).

> **Tutorial Exercise 2 questions: which type would you use?**
>
> | Data | Type | Why |
> | :-- | :-- | :-- |
> | Student name | `String` | It is text. |
> | Number of students | `int` | You cannot have half a student. |
> | Module mark | `double` | A mark can be 67.5. |
> | Whether a student has passed | `boolean` | Only yes or no. |
> | A single grade such as A | `char` | One character. |

### 5.3 Creating and using a variable

```java
int age;
age = 20;
double averageMark = 67.5;
averageMark = 70.0;
```

| Line | Piece | What it means |
| :-- | :-- | :-- |
| 1 | `int` | The type. "This box holds whole numbers." |
| 1 | `age` | The name on the box. |
| 1 | `;` | End of statement. The box now exists but is empty. This step is called a **declaration** (announcing that a variable exists). |
| 2 | `age` | Which box we are talking about. |
| 2 | `=` | The **assignment operator**. It means "put the value on the right into the box on the left". It does **not** mean "equals" as in maths. Read it as "becomes". |
| 2 | `20` | The value going in. Putting a value in is called **assignment**. The very first assignment is called **initialisation**. |
| 3 | `double averageMark = 67.5;` | Declaration and initialisation in one line. This is the usual way. |
| 4 | `averageMark = 70.0;` | A later assignment. The old value `67.5` is thrown away and replaced. Notice there is **no type word** this time, because the box already exists. |

**Rules to remember:**

- You must declare a variable before using it.
- You write the type **once only**, when declaring. Writing `int age = 20;` twice is an error ("variable already defined").
- A variable declared inside a method (a **local variable**) must be given a value before you read it. Otherwise: compile-time error.
- A number written directly in the code, like `20` or `67.5`, is called a **literal**.

### 5.4 Types are strict

```java
int price = 19.99;      // COMPILE ERROR: a decimal does not fit in a whole-number box
String name = 42;       // COMPILE ERROR: a number is not text
double cost = 19;       // Fine. Java safely widens 19 to 19.0
```

Java will not silently squeeze a decimal into an `int` box, because the `.99` would be lost. Your Lecture 1 slide puts it this way: type mismatches being caught at compile time is "a feature, not a bug".

### 5.5 Full program: Tutorial Exercise 2

```java
public class Exercise2 {
    public static void main(String[] args) {
        String studentName = "Kamal";
        int age = 20;
        double averageMark = 67.5;
        char grade = 'B';
        boolean passed = true;

        System.out.println("Student Name: " + studentName);
        System.out.println("Age: " + age);
        System.out.println("Average Mark: " + averageMark);
        System.out.println("Grade: " + grade);
        System.out.println("Passed: " + passed);
    }
}
```

**Output:**

```
Student Name: Kamal
Age: 20
Average Mark: 67.5
Grade: B
Passed: true
```

Breakdown of `System.out.println("Age: " + age);`

| Piece | What it means |
| :-- | :-- |
| `"Age: "` | A piece of text, including the space after the colon. |
| `+` | Between text and anything else, `+` means "glue together". This gluing is called **concatenation**. |
| `age` | No quotes, so Java looks inside the box named `age` and uses its value, `20`. |
| Result | The glued text `Age: 20` is printed. |

If you wrote `"age"` with quotes, Java would print the three letters a-g-e, not the value.

---

## 6. Printing and joining text: the two traps

An **expression** is any piece of code that works out to a value, such as `3 + 4` or `"Age: " + age`. An **operator** is a symbol that does a job on values, such as `+`. The values it works on are called **operands**.

Java works through a chain of `+` from **left to right**. The moment a String is involved, everything after it is glued as text.

```java
System.out.println("Total: " + 3 + 4);   // prints  Total: 34
System.out.println(3 + 4 + " items");    // prints  7 items
```

| Line | Step by step |
| :-- | :-- |
| 1 | `"Total: " + 3` is text + number, so glue: `"Total: 3"`. Then `"Total: 3" + 4` is text + number, so glue: `"Total: 34"`. |
| 2 | `3 + 4` is number + number, so real addition: `7`. Then `7 + " items"` is number + text, so glue: `"7 items"`. |

**Fix when you want real addition after text:** use round brackets, which are always worked out first.

```java
System.out.println("Total: " + (3 + 4));  // prints  Total: 7
```

---

## 7. Operators

### 7.1 Arithmetic operators (maths)

| Operator | Meaning | Example | Result |
| :-- | :-- | :-- | :-- |
| `+` | Add | `7 + 2` | `9` |
| `-` | Subtract | `7 - 2` | `5` |
| `*` | Multiply | `7 * 2` | `14` |
| `/` | Divide | `7 / 2` | `3` (see below) |
| `%` | Remainder (also called "modulo") | `7 % 2` | `1` |

### 7.2 Integer division: the most famous trap

```java
int answer = 5 / 2;
System.out.println(answer);   // prints 2, not 2.5
```

**Why?** When **both** numbers are `int`, Java does **integer division**: it gives a whole-number answer and throws the decimal part away. It does not round. `5 / 2` is `2`, and `7 / 2` is `3`.

- **Analogy:** sharing 5 sweets between 2 children without cutting any. Each child gets 2. The leftover 1 is what `%` gives you: `5 % 2` is `1`.

If **either** number is a `double`, you get the real answer:

```java
System.out.println(7 / 2);     // 3
System.out.println(7 / 2.0);   // 3.5
```

> This is the answer to the Tutorial Exercise 4 question, and to rows 1 and 2 of the warm-up in your lecture notes.

`%` is useful for questions like "is this number even?" A number is even when `number % 2` is `0`.

### 7.3 Order of working out

Same as school maths: brackets first, then `*` `/` `%`, then `+` `-`, left to right.
`2 + 3 * 4` is `14`. `(2 + 3) * 4` is `20`.

### 7.4 Shortcuts for changing a variable

| Shortcut | Same as | Meaning |
| :-- | :-- | :-- |
| `count++;` | `count = count + 1;` | Add 1. Called **increment**. |
| `count--;` | `count = count - 1;` | Subtract 1. Called **decrement**. |
| `total += 5;` | `total = total + 5;` | Add 5 to total. |
| `total -= 5;` | `total = total - 5;` | Subtract 5 from total. |

`count = count + 1;` looks impossible as maths. Remember `=` means "becomes": work out the right side using the old value, then store the answer back in the box.

### 7.5 Comparison operators (asking a yes/no question)

Also called **relational operators**. Each one produces a `boolean`: `true` or `false`.

| Operator | Question it asks | Example | Result |
| :-- | :-- | :-- | :-- |
| `>` | Greater than? | `5 > 3` | `true` |
| `<` | Less than? | `5 < 3` | `false` |
| `>=` | Greater than or equal to? | `40 >= 40` | `true` |
| `<=` | Less than or equal to? | `41 <= 40` | `false` |
| `==` | Equal to? | `5 == 5` | `true` |
| `!=` | Not equal to? | `5 != 5` | `false` |

> **`=` versus `==`.** One `=` **stores** a value. Two `==` **compares** two values. Mixing them up is a classic mistake.

#### Comparing text: use `.equals()`

For `String` values, do **not** use `==`. Use `.equals(...)`.

```java
String a = "Java";
System.out.println(a.equals("Java"));   // true
System.out.println(a.equals("java"));   // false: capital J is not small j
```

| Piece | What it means |
| :-- | :-- |
| `a` | The text we start with. |
| `.equals` | "Use the `equals` ability that belongs to this text." It checks whether the characters match exactly. |
| `("java")` | The text to compare against. |

Why not `==`? For text, `==` checks whether two names point to the very same stored item in memory, not whether the letters match. It sometimes appears to work and sometimes does not. Phase 1 shows the full reason. The rule for now: **numbers use `==`, text uses `.equals()`**.

### 7.6 Logical operators (combining yes/no answers)

| Operator | Name | True when... | Example | Result |
| :-- | :-- | :-- | :-- | :-- |
| `&&` | AND | **both** sides are true | `true && false` | `false` |
| `\|\|` | OR | **at least one** side is true | `true \|\| false` | `true` |
| `!` | NOT | flips the value | `!true` | `false` |

- **Analogy for `&&`:** "You may enter if you have a ticket AND you are over 18." Both needed.
- **Analogy for `||`:** "You get a discount if you are a student OR a pensioner." One is enough.

From your Pure OOP Week 01 warm-up slide:

```java
int quantity = 3;
double price = 19.99;
String name = "Camera";
boolean available = true;

System.out.println(name + " x " + quantity);          // Camera x 3
System.out.println(quantity * price);                 // 59.97
System.out.println(price > 20);                       // false
System.out.println(available && quantity > 0);        // true
```

| Line | Reasoning |
| :-- | :-- |
| 1 | Text glued to text glued to a number: `Camera x 3`. |
| 2 | `int * double` gives a `double`: 3 × 19.99 = `59.97`. |
| 3 | Is 19.99 greater than 20? No: `false`. |
| 4 | `quantity > 0` is worked out first: `true`. Then `true && true` is `true`. |

### 7.7 A warning about decimals

```java
System.out.println(0.1 + 0.2);   // prints 0.30000000000000004
```

This is not a bug in Java. Computers store decimals in binary (base 2), and some decimals, like 0.1, cannot be stored exactly, the same way 1/3 cannot be written exactly as a decimal (0.3333...). Your lecture notes mention that real money software uses a special type called `BigDecimal`; your module uses `double` to keep things simple.

---

## 8. Keyboard input with Scanner

So far the values were fixed in the code. To let a person type values in while the program runs, Java provides a ready-made tool called **`Scanner`**.

```java
import java.util.Scanner;

public class Exercise3 {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);

        System.out.print("Enter your name: ");
        String name = input.nextLine();

        System.out.print("Enter your age: ");
        int age = input.nextInt();

        System.out.print("Enter your mark: ");
        double mark = input.nextDouble();

        System.out.println("Name: " + name);
        System.out.println("Age: " + age);
        System.out.println("Mark: " + mark);
        System.out.println(name + " obtained " + mark + " marks.");
    }
}
```

**Example run** (what the person types is shown after each question):

```
Enter your name: Nimal
Enter your age: 21
Enter your mark: 72.5
Name: Nimal
Age: 21
Mark: 72.5
Nimal obtained 72.5 marks.
```

### Line 1: `import java.util.Scanner;`

| Piece | What it means |
| :-- | :-- |
| `import` | "Bring in a tool I want to use." Java has thousands of ready-made tools; you name the ones you need at the very top of the file, above the class. |
| `java.util` | The **package** where the tool lives. A package is a folder of related tools. `java.util` means "Java's utilities folder". |
| `Scanner` | The tool's name. |

- **Analogy:** fetching a specific tool from a specific drawer of a big toolbox before you start work.

### Line 5: `Scanner input = new Scanner(System.in);`

| Piece | What it means |
| :-- | :-- |
| `Scanner` (first) | The type of the variable. "This box holds a keyboard reader." |
| `input` | The name you chose for it. Any name works. |
| `=` | "Becomes". |
| `new` | "Build a brand-new one." Phase 1 is largely about this word. |
| `Scanner(...)` (second) | Which kind of thing to build. |
| `System.in` | Where it should read from. `System.in` is the keyboard (the partner of `System.out`, the console). |

In plain English: "Build a new keyboard reader connected to the keyboard, and call it `input`."

### Reading values

| Code | What it does |
| :-- | :-- |
| `input.nextLine()` | Waits for the person to type and press Enter, then gives back the whole line as a `String`. |
| `input.nextInt()` | Reads the next whole number as an `int`. |
| `input.nextDouble()` | Reads the next decimal number as a `double`. |

Breakdown of `int age = input.nextInt();` → "Use the `nextInt` ability of `input`. The program pauses until a number is typed. Store that number in a new `int` box called `age`."

Note `System.out.print` (no `ln`) is used for the questions, so the typing cursor stays on the same line as the question.

> **Two traps.**
> 1. If the person types `hello` when `nextInt()` expects a number, the program crashes with `InputMismatchException`. This is a runtime error.
> 2. If you call `nextLine()` straight **after** `nextInt()` or `nextDouble()`, it seems to be skipped. The number-readers leave the Enter keypress behind, and `nextLine()` reads that leftover. Fix: add one extra `input.nextLine();` after the number to clear it.

---

## 9. Making decisions: `if`, `else if`, `else`

Programs need to choose. "If the mark is 40 or more, print Pass; otherwise print Fail." Choosing between paths is called **selection**, and the general idea of deciding what runs next is called **control flow**.

- **Analogy: a fork in the road** with a signpost. The signpost asks a yes/no question, and you take one path or the other, never both.

### 9.1 `if` / `else`

```java
if (mark >= 40) {
    System.out.println("Pass");
} else {
    System.out.println("Fail");
}
```

| Piece | What it means |
| :-- | :-- |
| `if` | Keyword: "check the following question". |
| `(mark >= 40)` | The **condition**: a yes/no question in round brackets. It must work out to `true` or `false`. |
| `{` | Start of the block that runs **only when the condition is true**. |
| `System.out.println("Pass");` | The instruction inside that block. |
| `}` | End of the "true" block. |
| `else` | Keyword: "otherwise". Has no condition of its own. |
| `{ ... }` | The block that runs **only when the condition is false**. |

There is **no semicolon** after `if (mark >= 40)`. Writing `if (mark >= 40);` is a silent bug: the `;` ends the `if` immediately, and the block below it then always runs.

### 9.2 `else if` for more than two paths

Tutorial Exercise 6 classification:

| Mark | Result |
| :-- | :-- |
| 70–100 | Distinction |
| 60–69 | Merit |
| 40–59 | Pass |
| Below 40 | Fail |

```java
if (mark >= 70) {
    System.out.println("Distinction");
} else if (mark >= 60) {
    System.out.println("Merit");
} else if (mark >= 40) {
    System.out.println("Pass");
} else {
    System.out.println("Fail");
}
```

**How Java reads this:** top to bottom. It checks each condition in turn and runs the block of the **first** one that is true, then skips all the rest.

Trace it with `mark = 65`:

1. `65 >= 70`? No. Move on.
2. `65 >= 60`? Yes. Print `Merit`. **Stop. Skip everything below.**

Because of that, `else if (mark >= 60)` does not need to say "and below 70". Reaching that line already proves the mark is below 70.

**Order matters.** If you put `mark >= 40` first, a mark of 85 would print `Pass` and stop.

### 9.3 Validation (Tutorial Exercise 7)

**Validation** means checking that data is sensible before using it. A mark must be between 0 and 100.

```java
if (mark < 0 || mark > 100) {
    System.out.println("Invalid mark");
} else if (mark >= 70) {
    System.out.println("Distinction");
} else if (mark >= 60) {
    System.out.println("Merit");
} else if (mark >= 40) {
    System.out.println("Pass");
} else {
    System.out.println("Fail");
}
```

`mark < 0 || mark > 100` reads: "the mark is below zero OR the mark is above one hundred". Either one makes it invalid. The bad-data check goes **first**, so rubbish never reaches the real logic.

> **Why this matters for your exam:** Task 3 is "Services, validation and exceptions" (20 marks). Validation there is this exact pattern: an `if` that catches bad data first.

Mini-task from your Pure OOP Week 01 lecture:

```java
double replacementCost = 85000.00;

if (replacementCost > 100000) {
    System.out.println("High value");
} else {
    System.out.println("Standard");
}
```

With `85000.00` it prints `Standard`. Change the value to `125000.00` and it prints `High value`.

---

## 10. Repeating things: loops

A **loop** repeats a block of code. One pass through the block is called an **iteration**.

- **Analogy:** "Stir the pot 10 times" (you know the count) versus "Stir until it thickens" (you keep checking a condition).

### 10.1 The `for` loop: when you know how many times

```java
for (int i = 1; i <= 10; i++) {
    System.out.println(i);
}
```

Prints `1` to `10`, each on its own line.

The round brackets hold **three parts separated by semicolons**:

| Part | Code | Name | What it means |
| :-- | :-- | :-- | :-- |
| 1 | `int i = 1` | Initialisation | Create a counter box called `i`, starting at 1. Runs **once**, at the very start. (`i` is the traditional name for a counter.) |
| 2 | `i <= 10` | Condition | The question checked **before every pass**. True: do another pass. False: the loop ends. |
| 3 | `i++` | Update | Add 1 to `i`. Runs **after every pass**. |

**The order Java follows:**

```
initialise  ->  check condition  ->  run block  ->  update  ->  check condition  ->  run block  ->  update ...
                      |
                      false -> leave the loop
```

Trace with a shorter loop, `for (int i = 1; i <= 3; i++)` printing `"Check " + i`:

| Step | `i` | Condition `i <= 3` | Printed |
| :-- | :-- | :-- | :-- |
| Start | 1 | true | `Check 1` |
| After update | 2 | true | `Check 2` |
| After update | 3 | true | `Check 3` |
| After update | 4 | **false** | nothing; loop ends |

**Even numbers 2 to 20:**

```java
for (int i = 2; i <= 20; i += 2) {
    System.out.println(i);
}
```

The only change is the update: `i += 2` jumps by 2 each time.

**Multiplication table (the challenge):**

```java
int number = 5;
for (int i = 1; i <= 10; i++) {
    System.out.println(number + " x " + i + " = " + (number * i));
}
```

The brackets around `(number * i)` force the multiplication to happen before the gluing.

> **Off-by-one errors.** The most common loop bug is repeating one time too many or too few. `i < 5` starting from 0 runs 5 times (0, 1, 2, 3, 4). `i <= 5` starting from 0 runs 6 times. Always trace the first and last pass in your head.

### 10.2 The `while` loop: when you do not know how many times

```java
int remaining = 3;
while (remaining > 0) {
    System.out.println("Remaining: " + remaining);
    remaining--;
}
```

Output: `Remaining: 3`, `Remaining: 2`, `Remaining: 1`.

| Piece | What it means |
| :-- | :-- |
| `while` | Keyword: "keep repeating as long as...". |
| `(remaining > 0)` | The condition, checked before every pass. |
| `{ ... }` | The block to repeat. |
| `remaining--;` | Subtract 1. **Something inside the loop must eventually make the condition false**, or the loop never ends. A loop that never ends is called an **infinite loop** (stop it with the red square in IntelliJ). |

**Tutorial Exercise 9 with the total challenge:**

```java
import java.util.Scanner;

public class Exercise9 {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        int total = 0;

        System.out.print("Enter number: ");
        int number = input.nextInt();

        while (number != 0) {
            total += number;
            System.out.print("Enter number: ");
            number = input.nextInt();
        }

        System.out.println("Program ended.");
        System.out.println("Total: " + total);
    }
}
```

| Line | What it does |
| :-- | :-- |
| `int total = 0;` | A running total, starting at zero. A variable that collects a sum like this is called an **accumulator**. |
| First `input.nextInt()` | Read the first number **before** the loop, so the condition has something to check. |
| `while (number != 0)` | Keep going while the number is not zero. |
| `total += number;` | Add the number just entered to the total. |
| `number = input.nextInt();` | Read the next number. No `int` in front, because the box already exists. |
| After the loop | We only reach here when `0` was typed. |

Typing 5, 10, 7, 0 gives `Total: 22`.

### 10.3 Which loop?

| Situation | Use |
| :-- | :-- |
| "Do this 10 times" / "for each position 0 to N" | `for` |
| "Keep going until something happens" | `while` |
| "For every item in a list" | the **for-each** loop ([Section 12.4](#124-the-for-each-loop)) |

---

## 11. Methods

### 11.1 What is a method?

A **method** is a named block of instructions that does one clear job. You write it once and can use it as many times as you like.

- **Analogy: a vending machine.** You put something in (money and a button choice), it does its work inside, and something comes out (a drink). You do not need to know how it works inside. You only need to know what to put in and what you get back.

| Word | Plain-English meaning | Vending machine |
| :-- | :-- | :-- |
| **Define** a method | Write the method (build the machine). | Building the machine |
| **Call** a method (also "invoke") | Use the method (press the button). | Using the machine |
| **Parameter** | A named input slot in the method's definition. | The coin slot |
| **Argument** | The actual value you pass in when calling. | The actual coin |
| **Return value** | The result the method hands back. | The drink |
| **Return type** | The type of the result. | "This machine gives drinks" |

### 11.2 Anatomy of a method

From Tutorial Exercise 10:

```java
public static double calculateArea(double radius) {
    return Math.PI * radius * radius;
}
```

| Piece | What it means |
| :-- | :-- |
| `public` | Usable from anywhere. |
| `static` | Belongs to the class itself, so `main` (which is also `static`) can call it directly. Phase 1 explains when to drop this word. |
| `double` | The **return type**: this method promises to hand back a decimal number. |
| `calculateArea` | The method's name. Method names are usually verbs, in lowerCamelCase. |
| `(double radius)` | The **parameter list**. One parameter: a `double` that will be called `radius` inside the method. A parameter is a variable that receives its value from whoever calls the method. |
| `{` | Start of the method **body** (its instructions). |
| `return` | Keyword: "stop the method right here and hand this value back to the caller". |
| `Math.PI` | Java's built-in value of π (3.141592653589793). `Math` is a built-in toolbox of maths helpers. |
| `* radius * radius` | Multiply by the radius twice: π × r × r. |
| `;` then `}` | End of the statement; end of the method. |

The first line, `public static double calculateArea(double radius)`, is called the **method header**. The name plus the parameter types is called the **signature**.

### 11.3 Calling a method

```java
double area = calculateArea(5.0);
```

| Step | What happens |
| :-- | :-- |
| 1 | Java sees `calculateArea(5.0)` and pauses the current line. |
| 2 | The **argument** `5.0` is copied into the **parameter** `radius`. |
| 3 | The method body runs: `Math.PI * 5.0 * 5.0` is `78.53981633974483`. |
| 4 | `return` hands that value back. The method call is replaced by the value. |
| 5 | The line becomes `double area = 78.53981633974483;` and the value is stored. |

### 11.4 Full program: Tutorial Exercises 5 and 10

```java
import java.util.Scanner;

public class CircleCalculator {

    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);

        System.out.print("Enter radius: ");
        double radius = input.nextDouble();

        double area = calculateArea(radius);
        double circumference = calculateCircumference(radius);

        System.out.println("Area: " + area);
        System.out.println("Circumference: " + circumference);

        System.out.printf("Area: %.2f%n", area);
        System.out.printf("Circumference: %.2f%n", circumference);
    }

    public static double calculateArea(double radius) {
        return Math.PI * radius * radius;
    }

    public static double calculateCircumference(double radius) {
        return 2 * Math.PI * radius;
    }
}
```

**Run with 5:**

```
Enter radius: 5
Area: 78.53981633974483
Circumference: 31.41592653589793
Area: 78.54
Circumference: 31.42
```

Things to notice:

- **Methods sit side by side inside the class**, never inside one another. `calculateArea` is written after `main`'s closing `}` but before the class's closing `}`.
- The order you write methods in does not matter. The program always starts at `main`.
- The `radius` in `main` and the `radius` in `calculateArea` are **two different boxes** that happen to share a name. The value is copied from one to the other at the call. A variable only exists inside the `{ }` where it was declared; that region is called its **scope**.

**The two-decimal-places challenge:** `System.out.printf("Area: %.2f%n", area);`

| Piece | What it means |
| :-- | :-- |
| `printf` | "Print formatted." Prints text with values slotted into it. |
| `"Area: %.2f%n"` | The template. |
| `%.2f` | A placeholder: "put a decimal number here, rounded to 2 decimal places". `f` stands for floating-point (decimal) number. |
| `%n` | "New line." |
| `, area` | The value that fills the placeholder. |

### 11.5 `void` methods: doing without returning

```java
public static void printSeparator() {
    System.out.println("--------------------");
}
```

| Piece | What it means |
| :-- | :-- |
| `void` | In the return-type position, `void` means "this method hands nothing back". It performs an action instead. |
| `()` | Empty brackets: no parameters. The brackets are still required. |

Call it as a statement on its own: `printSeparator();`. You cannot write `int x = printSeparator();` because there is nothing to store.

**Printing is not returning.** `System.out.println` shows text to a human on the console. `return` hands a value back to the code that called the method so the program can keep using it. They are different things.

### 11.6 Several parameters

From your Pure OOP Week 01 lecture:

```java
public static double calculateLateFee(int daysLate, double dailyRate) {
    return daysLate * dailyRate;
}
```

Call: `double fee = calculateLateFee(4, 2.50);` gives `10.0`.

Arguments are matched to parameters **by position**: first to first, second to second. `4` goes into `daysLate`, `2.50` goes into `dailyRate`. The types must match too: `calculateLateFee("three", 2.50)` is a compile-time error, because text cannot go where an `int` is required.

> **Tutorial Exercise 10 questions, model answers.**
> 1. *What is a method?* A named, reusable block of code that performs one task.
> 2. *What is a parameter?* A variable in the method header that receives a value when the method is called.
> 3. *What is a return value?* The result a method sends back to the code that called it.
> 4. *Why are methods better than putting everything in `main`?* No repeated code, each piece has a clear name, a bug is fixed in one place, and each piece can be tested separately.

---

## 12. Arrays

### 12.1 What is an array?

An **array** is one variable that holds a fixed-size, numbered row of values that are all the same type.

- **Analogy: an egg carton.** One carton, several slots, each slot holds one egg. Or a row of numbered lockers.

```java
double[] marks = {65, 72, 48, 81, 56};
```

| Piece | What it means |
| :-- | :-- |
| `double[]` | The type: "an array of doubles". The square brackets `[]` mean "a row of". |
| `marks` | The array's name. |
| `=` | "Becomes". |
| `{65, 72, 48, 81, 56}` | The starting values, in order, separated by commas. Java counts them and makes exactly 5 slots. |

### 12.2 Positions start at zero

Each slot has a position number called an **index**. **The first index is 0, not 1.**

```
index:     0      1      2      3      4
        +------+------+------+------+------+
marks:  | 65.0 | 72.0 | 48.0 | 81.0 | 56.0 |
        +------+------+------+------+------+
```

| Code | Meaning | Value |
| :-- | :-- | :-- |
| `marks[0]` | The first slot | `65.0` |
| `marks[4]` | The fifth (last) slot | `56.0` |
| `marks[2] = 50;` | Replace the value in the third slot | |
| `marks.length` | How many slots there are (no brackets after `length`) | `5` |
| `marks[5]` | **Runtime error**: `ArrayIndexOutOfBoundsException`. There is no sixth slot. | |

The last valid index is always `length - 1`.

To make an empty array of a given size: `double[] numbers = new double[5];` (5 slots, each starting at `0.0`).

### 12.3 Looping through an array

```java
for (int i = 0; i < marks.length; i++) {
    System.out.println(marks[i]);
}
```

`i` goes 0, 1, 2, 3, 4. Each pass prints the slot at that index. Note `i < marks.length`, **not** `<=`. With `<=`, `i` would reach 5 and crash.

### 12.4 The for-each loop

A shorter loop for "do this for every item":

```java
for (double mark : marks) {
    System.out.println(mark);
}
```

| Piece | What it means |
| :-- | :-- |
| `double mark` | A fresh variable that holds the current item on each pass. |
| `:` | Read the colon as "**in**". |
| `marks` | The array to walk through. |

Read it aloud: "for each `mark` in `marks`". You will use this loop constantly from Phase 3 onwards.

### 12.5 Full program: Tutorial Exercise 11

```java
public class Exercise11 {
    public static void main(String[] args) {
        double[] marks = {65, 72, 48, 81, 56};

        double total = 0;
        double highest = marks[0];
        int passCount = 0;

        for (double mark : marks) {
            System.out.println(mark);
            total += mark;

            if (mark > highest) {
                highest = mark;
            }

            if (mark >= 40) {
                passCount++;
            }
        }

        double average = total / marks.length;

        System.out.println("Total: " + total);
        System.out.println("Average: " + average);
        System.out.println("Highest Mark: " + highest);
        System.out.println("Passed: " + passCount);
    }
}
```

**Output:**

```
65.0
72.0
48.0
81.0
56.0
Total: 322.0
Average: 64.4
Highest Mark: 81.0
Passed: 5
```

| Line | Idea |
| :-- | :-- |
| `double total = 0;` | Accumulator starts at zero. |
| `double highest = marks[0];` | Assume the first mark is the highest so far. Starting from the first real value is safer than starting from 0. |
| `total += mark;` | Add each mark to the running total. |
| `if (mark > highest) { highest = mark; }` | If this mark beats the best so far, it becomes the new best. |
| `passCount++;` | Count one more pass. A variable that counts like this is called a **counter**. |
| `total / marks.length` | `total` is a `double`, so this is real division, not integer division. |

These three patterns (running total, highest-so-far, counter) appear in almost every program you will ever write.

---

## 13. Reading error messages without panic

### 13.1 The method

1. Decide: **compile-time** (red underlines, the program never started) or **runtime** (it started, then stopped with red text in the console)?
2. Read the **first** error only. Later errors are often side effects of the first.
3. Find the **line number**. IntelliJ lets you click it.
4. Read what Java **expected** versus what it **found**.
5. Fix one thing. Run again.

### 13.2 Common errors

| Message | Kind | Usual cause |
| :-- | :-- | :-- |
| `';' expected` | Compile | Missing semicolon on that line or the line above. |
| `cannot find symbol` | Compile | A name Java does not recognise: misspelled, wrong capital letters, or never declared. |
| `incompatible types` | Compile | The value's type does not match the box's type, e.g. `int x = "hello";`. |
| `variable x is already defined` | Compile | You wrote the type word twice for the same variable. |
| `variable x might not have been initialized` | Compile | You read a local variable before giving it a value. |
| `missing return statement` | Compile | A method promises a value but some path does not `return` one. |
| `reached end of file while parsing` | Compile | A closing `}` is missing. |
| `non-static ... cannot be referenced from a static context` | Compile | `main` is trying to use something that is not `static`. Phase 1 explains this fully. |
| `ArithmeticException: / by zero` | Runtime | Dividing an `int` by zero. |
| `ArrayIndexOutOfBoundsException` | Runtime | An index below 0 or above `length - 1`. |
| `InputMismatchException` | Runtime | The person typed text where `nextInt()` wanted a number. |
| `NullPointerException` | Runtime | Using something that was never given a real value. Phase 1 explains `null`. |

A runtime crash prints a **stack trace**: a list of lines showing where the program was when it failed. Read from the top and look for the first line that mentions **your** file name. Your exam briefing lists "diagnose stack traces" in its preparation checklist.

---

## 14. The bridge: why this is not enough

Here is the problem that the rest of the module solves. This comes from Section 02 of your Pure OOP Week 01 lecture.

Suppose the university lends out equipment. One item needs four pieces of data:

```java
long id = 101;
String name = "Dell Laptop";
double replacementCost = 185000.00;
boolean available = true;
```

That works for one item. Now three items:

```java
long id1 = 101;   String name1 = "Dell Laptop";    double cost1 = 185000.00;  boolean available1 = true;
long id2 = 102;   String name2 = "Canon Camera";   double cost2 = 120000.00;  boolean available2 = true;
long id3 = 103;   String name3 = "Tripod";         double cost3 = 15000.00;   boolean available3 = false;
```

**The problems** (from your slide):

- The names become artificial: `name1`, `name2`, `name3`...
- Nothing in the code says "these four boxes belong together as one item". You know it. Java does not.
- A method that works on an item needs all four values passed in separately.
- What about 500 items? 2,000 variables.
- The compiler cannot protect the group. Nothing stops you pairing `name1` with `cost3` by accident.

The same thing happened with the circle in your tutorial: the data (`radius`) sat in one place and the operations (`calculateArea`, `calculateCircumference`) sat somewhere else, connected only by you remembering to pass the right value.

**The idea that fixes it:** bundle the data and the operations that belong together into **one single unit**. That bundled unit is called an **object**. This is the first time this guide has used that word on purpose.

```
        DATA  +  OPERATIONS
               |
               v
             OBJECT
```

| Bundle | Its data (called **state**) | Its operations (called **behaviour**) |
| :-- | :-- | :-- |
| A circle | radius | calculate area, calculate circumference |
| An equipment item | id, name, replacement cost, available | display details, calculate late fee, is it expensive? |

And the "labelled folder" from Section 3, the **class**? Its real job is to be the **blueprint** that describes what such a bundle contains, so you can build as many as you need from one description.

That is exactly where **Phase 1** begins.

---

## 15. Practice

Cover the answers. Predict first.

### 15.1 Predict the output

From §9 of your Week 01 Lecture Notes:

| # | Code | Your prediction |
| :-- | :-- | :-- |
| 1 | `System.out.println(7 / 2);` | |
| 2 | `System.out.println(7 / 2.0);` | |
| 3 | `System.out.println("Total: " + 3 + 4);` | |
| 4 | `System.out.println(3 + 4 + " items");` | |
| 5 | `System.out.println(10 % 3);` | |
| 6 | `System.out.println(0.1 + 0.2);` | |
| 7 | `System.out.println("Java".equals("java"));` | |

**Answers**

| # | Output | Reason |
| :-- | :-- | :-- |
| 1 | `3` | Both are `int`, so integer division drops the `.5`. |
| 2 | `3.5` | One side is a `double`, so real division. |
| 3 | `Total: 34` | Text comes first, so everything after is glued. |
| 4 | `7 items` | `3 + 4` is added first, then glued to the text. |
| 5 | `1` | 10 ÷ 3 is 3 remainder 1. |
| 6 | `0.30000000000000004` | Decimals are stored approximately. |
| 7 | `false` | `equals` is case-sensitive. |

(Row 8 in your notes uses `null`; that is covered in Phase 1.)

### 15.2 Spot the error

Each snippet has one mistake. Find it and say whether it is compile-time or runtime.

```java
// A
int count = 10
System.out.println(count);

// B
string city = "Colombo";

// C
int total = 50;
int total = 60;

// D
int[] numbers = {4, 8, 15};
System.out.println(numbers[3]);

// E
double half = 1 / 2;
System.out.println(half);
```

**Answers**

| | Mistake | Kind |
| :-- | :-- | :-- |
| A | Missing `;` after `10`. | Compile-time |
| B | `string` must be `String` with a capital S. | Compile-time (`cannot find symbol`) |
| C | `total` declared twice. The second line should be `total = 60;`. | Compile-time |
| D | Valid indexes are 0, 1, 2. Index 3 does not exist. | Runtime (`ArrayIndexOutOfBoundsException`) |
| E | No crash, but a **logic error**: `1 / 2` is integer division, so it prints `0.0`. Write `1 / 2.0`. A logic error is when the program runs but gives the wrong answer. | Neither; wrong result |

### 15.3 Final mini challenge (from your tutorial), with a full solution

Task: ask for five numbers, store them in an array, calculate the total with one method and the average with another, find the highest, and say whether the average is above 50.

Try it yourself for 20 minutes before reading on.

```java
import java.util.Scanner;

public class MiniChallenge {

    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        double[] numbers = new double[5];

        for (int i = 0; i < numbers.length; i++) {
            System.out.print("Enter number " + (i + 1) + ": ");
            numbers[i] = input.nextDouble();
        }

        double total = calculateTotal(numbers);
        double average = calculateAverage(numbers);
        double highest = findHighest(numbers);

        System.out.println("Total: " + total);
        System.out.println("Average: " + average);
        System.out.println("Highest: " + highest);

        if (average > 50) {
            System.out.println("The average is greater than 50.");
        } else {
            System.out.println("The average is not greater than 50.");
        }
    }

    public static double calculateTotal(double[] values) {
        double total = 0;
        for (double value : values) {
            total += value;
        }
        return total;
    }

    public static double calculateAverage(double[] values) {
        return calculateTotal(values) / values.length;
    }

    public static double findHighest(double[] values) {
        double highest = values[0];
        for (double value : values) {
            if (value > highest) {
                highest = value;
            }
        }
        return highest;
    }
}
```

**Run with 40, 60, 75, 30, 55:**

```
Total: 260.0
Average: 52.0
Highest: 75.0
The average is greater than 50.
```

Points worth noticing:

| Line | Why it is written that way |
| :-- | :-- |
| `new double[5]` | Five empty slots, filled in by the loop. |
| `"Enter number " + (i + 1)` | `i` starts at 0, but humans count from 1. The brackets make it real addition. |
| `numbers[i] = input.nextDouble();` | Store what was typed into slot `i`. |
| `calculateTotal(double[] values)` | A method can take a whole array as a parameter. |
| `calculateAverage` calls `calculateTotal` | Methods can call other methods. No repeated code. |

---

## 16. Glossary

| Term | Plain-English meaning |
| :-- | :-- |
| **Argument** | The actual value passed into a method when calling it. |
| **Array** | One variable holding a fixed-size, numbered row of same-type values. |
| **Assignment** | Putting a value into a variable using `=`. |
| **Block** | Statements grouped inside `{ }`. |
| **boolean** | A type with only two values: `true` and `false`. |
| **Bytecode** | The translated form of your code that the JVM runs (`.class` file). |
| **Call / invoke** | To use a method. |
| **Case-sensitive** | Capital and small letters count as different. |
| **Class** | For now: a named container for code. From Phase 1: a blueprint. |
| **Comment** | A note for humans that the compiler ignores. |
| **Compile** | Translate source code into bytecode, checking for mistakes on the way. |
| **Compiler** | The tool that compiles. |
| **Compile-time error** | A mistake found during compiling; the program never starts. |
| **Concatenation** | Gluing text together with `+`. |
| **Condition** | An expression that is `true` or `false`, used by `if` and loops. |
| **Console** | The panel where printed output appears. |
| **Control flow** | The order in which statements run, including decisions and loops. |
| **Declaration** | Announcing a variable's type and name. |
| **Expression** | Code that works out to a value. |
| **IDE** | The program you write code in (IntelliJ IDEA). |
| **Identifier** | A name you invent. |
| **Import** | Bringing a ready-made tool into your file. |
| **Index** | A position number in an array, starting at 0. |
| **Initialisation** | Giving a variable its first value. |
| **Iteration** | One pass through a loop. |
| **JDK** | The package containing the compiler, the JVM and built-in tools. |
| **JVM** | The program that runs bytecode. |
| **Keyword** | A word reserved by Java. |
| **Literal** | A value written directly in code, like `20` or `"Hi"`. |
| **Local variable** | A variable declared inside a method. |
| **Logic error** | The program runs but gives a wrong answer. |
| **Loop** | Code that repeats a block. |
| **Method** | A named, reusable block of instructions. |
| **Operand** | A value an operator works on. |
| **Operator** | A symbol that does a job on values (`+`, `>`, `&&`). |
| **Package** | A folder of related code. |
| **Parameter** | A named input slot in a method's definition. |
| **Primitive type** | A simple built-in type: `int`, `long`, `double`, `boolean`, `char`. |
| **Return** | To hand a value back from a method to its caller. |
| **Return type** | The type of value a method hands back; `void` for none. |
| **Runtime** | The period while the program is running. |
| **Runtime error** | A failure that happens while running. |
| **Scope** | The region of code where a variable exists. |
| **Signature** | A method's name plus its parameter types. |
| **Source code** | The text you type (`.java` file). |
| **Stack trace** | The list of lines printed when a program crashes. |
| **Statement** | One complete instruction, usually ending in `;`. |
| **Static typing** | Every variable's type is fixed and checked before the program runs. |
| **String** | A type that holds text. |
| **Syntax** | The grammar rules of the language. |
| **Validation** | Checking that data is sensible before using it. |
| **Variable** | A named storage box holding one value. |
| **void** | "Returns nothing." |

---

## 17. One-page cheat sheet

```java
import java.util.Scanner;                      // bring in a tool (top of file)

public class Name {                            // file must be Name.java
    public static void main(String[] args) {   // the program starts here

        // variables
        int whole = 10;
        long bigId = 101L;
        double decimal = 19.99;
        boolean flag = true;
        char letter = 'A';
        String text = "Hello";

        // output
        System.out.println("Value: " + whole);        // print, then new line
        System.out.printf("%.2f%n", decimal);         // two decimal places

        // input
        Scanner input = new Scanner(System.in);
        int n = input.nextInt();

        // decision
        if (n < 0 || n > 100) {
            System.out.println("Invalid");
        } else if (n >= 40) {
            System.out.println("Pass");
        } else {
            System.out.println("Fail");
        }

        // loops
        for (int i = 0; i < 5; i++) { }               // known number of times
        while (n > 0) { n--; }                        // until a condition fails

        // arrays
        double[] marks = {65, 72, 48};
        for (double m : marks) { }                    // for each m in marks

        // calling a method
        double result = add(2.5, 4.0);
    }

    // defining a method:  access  static  returnType  name(parameters)
    public static double add(double a, double b) {
        return a + b;
    }
}
```

| Remember | |
| :-- | :-- |
| `=` stores, `==` compares | Text is compared with `.equals()` |
| `int / int` drops the decimal | `7 / 2` is `3`; `7 / 2.0` is `3.5` |
| `%` gives the remainder | `10 % 3` is `1` |
| Text `+` anything is gluing | `"Total: " + 3 + 4` is `Total: 34` |
| Array indexes start at 0 | Last index is `length - 1` |
| Every `{` needs a `}` | Every statement needs a `;` |
| `void` returns nothing | `return` hands a value back |

---

## 18. Am I ready for Phase 1?

Tick each one honestly. If you cannot do it, reread that section.

- [ ] I can explain what "compile" and "run" mean, and the difference between a compile-time and a runtime error. (§2)
- [ ] I can write a program with `main` that prints two lines, from memory. (§3)
- [ ] I can pick the right type out of `int`, `double`, `String`, `boolean`, `char` for a piece of data. (§5)
- [ ] I can explain why `5 / 2` is `2` and why `"Total: " + 3 + 4` prints `Total: 34`. (§6, §7)
- [ ] I can write an `if / else if / else` chain that validates first. (§9)
- [ ] I can write a `for` loop from 1 to 10 and trace it by hand. (§10)
- [ ] I can write a method with two parameters that returns a value, and call it. (§11)
- [ ] I can say which word is the return type, the name, and the parameters in a method header. (§11)
- [ ] I can loop through an array and work out a total and a highest value. (§12)
- [ ] I can explain why separate loose variables stop working when a program grows. (§14)

**Next: Phase 1, Classes and Objects.**
