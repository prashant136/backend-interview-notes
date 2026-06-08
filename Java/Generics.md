
# What is Generics in Java?

Generics allow you to write classes, interfaces, and methods that work with different data types while maintaining type safety.

# Problem Without Generics

```
List list = new ArrayList(); 

list.add("Prashant");

list.add("Java");

String name = (String) list.get(0);
```

### Problems

- Manual type casting
- Runtime errors possible
- Less readable

---
# With Generics

```
List<String> list = new ArrayList<>();

list.add("Prashant");
list.add("Java");

// list.add(100); // Compile-time Error

String name = list.get(0);
```


Now Java checks types at compile time.

---

# Creating Generic Classes

```
class Box<T> {

   private T item;
   
   public void setItem(T item) {
       this.item = item;

   }

   public T getItem() {
       return item;
   }
}

```

```
Box<String> box = new Box<>();

box.setItem("Java");

String value = box.getItem();

System.out.println(value);

Output:
Java
```


```
Box<Integer> box = new Box<>(); 

box.setItem(100); 

Integer value = box.getItem();

System.out.println(value);

Output: 
100
```

---

# Multiple Generic Types

```
class Pair<K, V> {

   private K key;
   private V value;

   public Pair(K key, V value) {
       this.key = key;
       this.value = value;
   }

   public K getKey() {
       return key;
   }

   public V getValue() {
       return value;
   }

}

// Usage:

Pair<Integer, String> pair = new Pair<>(1, "Java");
System.out.println(pair.getKey());
System.out.println(pair.getValue());

// Output:
1
Java
```


---

# Generic Methods

Sometimes only a method should be generic.

```
class Utility {
   public static <T> void print(T value) {
       System.out.println(value);
   }
}

// Usage:
Utility.print("Java");
Utility.print(100);
Utility.print(true);

// Output:
Java
100
true
```

---

# Bounded Generics

Suppose you want only numeric types.

```
class Calculator<T extends Number> {

   private T number;

   public Calculator(T number) {
       this.number = number;
   }

   public double doubleValue() {
       return number.doubleValue();
   }
}


// Usage:
Calculator<Integer> c1 = new Calculator<>(10);
Calculator<Double> c2 = new Calculator<>(20.5);

```

Not allowed:

```
Calculator<String> c3 =  new Calculator<>("Java");
```

Compile-time error because String is not a Number.


---

# Generic Interface
```
interface Repository<T> {
   
   void save(T data);
   
   T findById(Long id);

}

// Implementation:

class UserRepository implements Repository<User> {

   @Override
   public void save(User user) {
       System.out.println("Saved");
   }

   @Override
   public User findById(Long id) {
       return new User();
   }
}
```

---

# Most Common Interview Question

## Why does List<String> improve performance?

It actually doesn't improve runtime performance. Provide compile-time type safety.
    
---


# Type Erasure (Very Important Interview)

Java Generics exist only at compile time.

```
List<String> names =   new ArrayList<>();
List<Integer> nums =   new ArrayList<>();
```

After compilation:

```
List names =  new ArrayList();
List nums =   new ArrayList();
```

Java removes generic type information.

This process is called **Type Erasure**.

---

# Interview One-Line Definition

**Generics allow classes, interfaces, and methods to operate on different data types while providing compile-time type safety and eliminating explicit type casting.**


---
---


# WildCards ?

# Types of Wildcards

There are 3 types:

```
?
? extends T
? super T
```

---

# 1. Unbounded Wildcard (?)

Syntax:

```
List<?>
```

means:

```
List<String>
List<Integer>
List<Double>
List<User>
```

all are accepted.

---

Example

```
public static void print(List<?> list) {

    for(Object obj : list) {
        System.out.println(obj);
    }
}
```

Usage:

```
print(List.of("Java","Spring"));
print(List.of(1,2,3));
print(List.of(1.5,2.5));
```

Output:

```
Java
Spring
1
2
3
1.5
2.5
```

---

Can we add?

```
List<?> list = new ArrayList<>();list.add("Java");
```

❌ Compile Error

Because Java doesn't know the actual type.

Could be:

```
List<String>
List<Integer>
List<User>
```

So adding is unsafe.



---


# Understand the upper and lower bound
## First Understand the Class Hierarchy

```
Object
   ↑
Number
   ↑
Integer
```

Think of it as a family tree:

```
Object  
  |  
Number  
  |  
Integer
```

- `Object` = Grandfather
- `Number` = Father
- `Integer` = Son

---

# Upper Bound
## Why is it called UPPER BOUND?

Look again:

```
Object
  ↑
Number  ← Bound
  ↑
Integer
```

When we write:

```
List<? extends Number>
```

we are saying:

> Give me Number OR anything BELOW Number.

Meaning:

```
List<Number>
List<Integer>
List<Double>
List<Float>
```

are all allowed.

The **highest limit (upper boundary)** is `Number`.

Nothing above Number is allowed.

```
List<Object>  // ❌
```

because Object is above Number.

That's why it's called:

```
? extends Number
```

Upper Bound = "Maximum allowed parent is Number."

---

# Example 1: Zoo Animals

```
class Animal {}
class Dog extends Animal {}
class Cat extends Animal {}
```

Hierarchy:

```
Animal
  ↑
 Dog
 Cat
```

Method:

```
public void printAnimals(
        List<? extends Animal> animals) {
}
```

Allowed:

```
List<Dog>
List<Cat>
List<Animal>
```

Not allowed:

```
List<Object>
```

because Object is above Animal.

---

# Lower Bound
## Why is it called LOWER BOUND?

Now look:

```
Object
  ↑
Number
  ↑
Integer ← Bound
```

When we write:

```
List<? super Integer>
```

we are saying:

> Give me Integer OR anything ABOVE Integer.

Allowed:

```
List<Integer>
List<Number>
List<Object>
```

Not allowed:

```
List<Double>
```

because Double is not a parent of Integer. The lowest type allowed is Integer.
That's why it's called:

```
Lower Bound
```

---

# Example 2: Animal Hierarchy

```
Animal
  ↑
Dog
```

Method:

```
void addDogs(List<? super Dog> dogs)
```

Allowed:

```
List<Dog>
List<Animal>
List<Object>
```

Not allowed:

```
List<Cat>
```

---

# Concepts - If you want to go deep dive ----

----

# Step 1: Read vs Write Means What?

Consider:

```
List<Integer> integers = List.of(10, 20, 30);
```

### Reading

Taking data OUT:

```
Integer n = integers.get(0);
```

```
List --> Variable
```

Data moves OUT of collection.

---

### Writing

Putting data INTO collection:

```
integers.add(40);
```

```
Variable --> List
```

Data moves INTO collection.

---

# Step 2: Understanding `extends`

Suppose:

```
List<? extends Number> list
```

Java knows:

```
Could be:
List<Integer>
List<Double>
List<Float>
List<Long>
```

but Java DOES NOT KNOW WHICH ONE.

---

Imagine runtime object is:

```
List<Integer>
```

but compiler only sees:

```
List<? extends Number>
```

---

# Reading is Safe

Suppose first element is:

```
10
```

Compiler thinks:

```
Could be Integer
Could be Double
Could be Float
```

But all of them are:

```
Number
```

Therefore:

```
Number n = list.get(0);
```

is always safe.

---

Visualize:

```
List<Integer>
      |
      v
   Integer
      |
      v
   Number
```

Compiler says:

> Whatever comes out is at least a Number.

Safe.

---

# Why Writing Fails?

Now you try:

```
list.add(10);
```

Compiler thinks:

```
Maybe List<Integer>
Maybe List<Double>
Maybe List<Float>
```

---

If actual list is:

```
List<Double>
```

then this happens:

```
list.add(10);
```

which means:

```
Double List ← Integer
```

Not safe.

Therefore compiler blocks ALL writes.

---

Think like Java:

```
I don't know the exact child type.
Therefore I won't let you insert anything.
```

---

# Simple Rule

For `extends`

```
List<? extends Number>
```

Compiler knows:

```
What comes OUT = safe
What goes IN = unsafe
```

Therefore:

```
Number n = list.get(0); // OK
list.add(10);           // ERROR
```

---

# Step 3: Understanding `super`

Now:

```
List<? super Integer> list
```

Compiler knows:

```
 Could be:
List<Integer>
List<Number>
List<Object>
```

---

Imagine actual object:

```
List<Object>
```

or

```
List<Number>
```

---

# Writing is Safe

Can Integer go into:

```
List<Integer>
```

Yes.

Can Integer go into:

```
List<Number>
```

Yes.

Can Integer go into:

```
List<Object>
```

Yes.

Therefore:

```
list.add(10);
```

Always safe.

---

Visualize:

```
         Object
            ↑
         Number
            ↑
         Integer
```

Anything above Integer can hold Integer.

Therefore write is safe.

---

# Why Reading Fails?

Now:

```
Dog d = list.get(0);
```

(Using Integer example)

Compiler thinks:

```
Maybe List<Object>  
Maybe List<Number>  
Maybe List<Integer>
```

Suppose actual object is:

```
List<Object>
```

Inside it could contain:

```
"Java"
true
100
```

Can compiler guarantee Integer?

No.

Therefore:

```
Integer i = list.get(0);
```

Not safe.

Only this is safe:

```
Object obj = list.get(0);
```

---


# Ultimate Memory Trick

Forget PECS for a second.

Think:

### `extends`

```
Unknown Child Type
```

Examples:

```
Integer
Double
Float
```

Since child type is unknown:

```
Read = Safe
Write = Dangerous
```

---

### `super`

```
Unknown Parent Type
```

Examples:

```
Number
Object
```

Since parent types can hold Integer:

```
Write = Safe
Read = Dangerous
```

---
---
---
---
---
---
---

# Question:  Comparator vs Comparable? Which one provides natural sorting? 

In Java, both Comparable and Comparator interfaces are used for sorting objects. The main difference between Comparable and Comparator is:

- **Comparable**: It is used to define the natural ordering of the objects within the class.
- **Comparator**: It is used to define custom sorting logic externally.

# 1. Comparable

Used when the class itself knows how it should be sorted.

It provides **natural ordering**.

### Method

```
public interface Comparable<T> {
    int compareTo(T o);
}
```

### Example

Sort Employee by id.

```
class Employee implements Comparable<Employee> {

    int id;
    String name;

    Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public int compareTo(Employee e) {
        return this.id - e.id;
    }

    @Override
    public String toString() {
        return id + " " + name;
    }
}
```

```
List<Employee> list = Arrays.asList(
        new Employee(3, "John"),
        new Employee(1, "Mike"),
        new Employee(2, "David")
);

Collections.sort(list);
System.out.println(list);
```

Output:

```
1 Mike
2 David
3 John
```

---

# 2. Comparator

Comparator allows to create custom (multiple) sorting strategies without modifying class.


### Question: Implement same for employee [id, name, salary, age] to sort in descending order of salary?

```
Employee
---------
id
name
salary
age
```

# Employee Class

```
class Employee {

    private int id;
    private String name;
    private double salary;
    private int age;

    public Employee(int id, String name, double salary, int age) {
        this.id = id;
        this.name = name;
        this.salary = salary;
        this.age = age;
    }

    public double getSalary() {
        return salary;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    @Override
    public String toString() {
        return id + " " + name + " " + salary + " " + age;
    }
}
```

---

# Sample Data

```
List<Employee> employees = Arrays.asList(
        new Employee(1, "Rahul", 80000, 28),
        new Employee(2, "Aman", 50000, 25),
        new Employee(3, "Zoya", 90000, 30),
        new Employee(4, "Priya", 70000, 27)
);
```

---

# 1. OLD WAY (Anonymous Class)

Before Java 8

```
Comparator<Employee> salaryComparator = new Comparator<Employee>() {

	@Override
	public int compare(Employee e1, Employee e2) {

		if(e1.getSalary() < e2.getSalary())
			return 1;

		if(e1.getSalary() > e2.getSalary())
			return -1;

		return 0;
	}
};
```

Use:

```
Collections.sort(employees, salaryComparator);
```

Output:

```
Zoya   90000
Rahul  80000
Priya  70000
Aman   50000
```

---

# 2. Lambda Expression

Java 8+

```
Comparator<Employee> salaryComparator =
        (e1, e2) ->
            Double.compare(e2.getSalary(),
                           e1.getSalary());
```

Use:

```
employees.sort(salaryComparator);
```

---

Ascending:

```
Double.compare(e1.getSalary(), e2.getSalary());
```

Descending:

```
Double.compare(e2.getSalary(), e1.getSalary());
```

-----
-----

## 3. Modern Comparator API (Most Preferred in Interviews)

Ascending:

```
employees.sort(
        Comparator.comparingDouble(Employee::getSalary)
);
```

Descending:

```
employees.sort(
        Comparator.comparingDouble(Employee::getSalary)
                  .reversed()
);
```

This is what you'll see in production code most often.

---

# Comparator Syntax Cheat Sheet

## Old Way

```
Comparator<Employee> comparator =
        new Comparator<Employee>() {

            @Override
            public int compare(Employee e1,
                               Employee e2) {

                return e1.getAge() - e2.getAge();
            }
        };
```

---

## Lambda Way

```
Comparator<Employee> comparator =
        (e1, e2) ->
            e1.getAge() - e2.getAge();
```

---

## Method Reference Way

```
Comparator<Employee> comparator =
        Comparator.comparingInt(Employee::getAge);
```

---

# Sort by Salary Descending

### Old Way

```
Collections.sort(employees,
        new Comparator<Employee>() {

            @Override
            public int compare(Employee e1,
                               Employee e2) {

                return Double.compare(
                        e2.getSalary(),
                        e1.getSalary()
                );
            }
        });
```

---

### Lambda Way

```
employees.sort(
        (e1, e2) ->
                Double.compare(
                        e2.getSalary(),
                        e1.getSalary()
                )
);
```

---

### Modern Way

```
employees.sort(
        Comparator.comparingDouble(Employee::getSalary)
                  .reversed()
);
```

---

# Interview Pattern (Very Important)

Sort by age ascending:

```
employees.sort(
    Comparator.comparingInt(Employee::getAge)
);
```

Sort by age descending:

```
employees.sort(
    Comparator.comparingInt(Employee::getAge)
              .reversed()
);
```

Sort by salary ascending:

```
employees.sort(
    Comparator.comparingDouble(Employee::getSalary)
);
```

Sort by salary descending:

```
employees.sort(
    Comparator.comparingDouble(Employee::getSalary)
              .reversed()
);
```

Sort by name:

```
employees.sort(
    Comparator.comparing(Employee::getName)
);
```

Sort by salary then name:

```
employees.sort(
    Comparator.comparingDouble(Employee::getSalary)
              .thenComparing(Employee::getName)
);
```

This `comparing...()`, `reversed()`, and `thenComparing()` combination is the one most commonly asked and used in Java 8+ interviews.

