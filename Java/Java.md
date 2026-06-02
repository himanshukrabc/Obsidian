- Java is platform independent due to JVM
- **Java Compiler(javac)** converts your java files to bytecode.
- **Bytecode** - .class file which is readable by JVM. 
	- *platform independent* - Bytecode generated anywhere can be read by any JVM.
### Java Development Kit
- Allows you to write and compile Java files. It includes the JRE.
### Java Runtime Environment(JRE)
- It is the environment required to run Java. 
- It contains the JVM and some core java libraries.
### Java Virtual Machine(JVM)
- JVM is not platform independent. Every OS has a different version of JVM implemented.
- *Memory Management* - Uses garbage collector to deallocate unused memory.
- *Security* - Runs the code in a sandbox which prevents unsafe operations.
- *Just In Time(JIT) Compiler* - Converts frequently used bytecode into native machine code for speed
#### JVM Architecture
- **Class Loader** → loads `.class` files
- **Runtime Data Areas** → memory (heap, stack, method area)
- **Execution Engine** → runs bytecode (interpreter + JIT)
- **Garbage Collector** → cleans unused objects
### Java
- Everything in Java is a object.
- There needs to be a main method, which is the entry point for compliation.
- Has the Heap and Stack implemented. 
	- All the declared variables are stored in stack.
	- All the allocated memory including **objects are stored in heap**.
	- Object data is stored in heap. The variable obj is a memory reference to the heap.
```Java
Demo obj = new Demo();
```
#### Strings
- Behind the scenes -> string always calls new String(); =>Strings in java are always initialized in the heap.
```Java
String x = "asdfa"; // is same as new String("asdfa");
```
- Two identical strings are necessarily the same object.
```Java
String s1 = "HK";
String s2 = "HK";
(s1 == s2) -> is always true -> They both reference the same object in the Heap.
```
- **String Constant Pool** - 
	- Any new string is created in the pool.
	- Same string to created -> the pool returns reference to the already created string.
	- *Strings are immutable* - Since they reference an object, strings in java are immutable.
```Java
String s1="str1"; -> Object "str1" created
s1="str2"; -> Object "str2" created. "str1" is marked for garbage collection.
```
- **Mutable Strings**
	- StringBuffer is threadsafe -> Mostly used when you want a string to be shared across threads.
	- StringBuilder is usable in all other cases.
```Java
StringBuffer sb = new StringBuffer("base -");
sb.append("hello");

StringBuilder sb = new StringBuilder("base -");
sb.append("hello");
```
### static
- **static variables** are common to all objects. Can be directly accessed from the class.
	- Any object can modify this.
- **static methods** - Directly called by the class. No need to reference from a object.
	- You can only use static variables(other vars are not initialized)
- **static block** - Used for initialization of static variables. 
	- static block is called before constructor call.
	- Run only the first time an instance of the class is created.
	- *Run by class loader*- Whenever it loads a class, it runs its static block.
```Java
class Main{
	public static x;
	static{
		x=10;
	}
	public static void print(){
		System.out.println("Value -" + x);
	}
}
```
### Packages
- Folder structure in Java is called a package.
- Specify the structure in the file to import.
``` Java
package database.orms.prisma;

class Main{
}

//Structure
location - /database/orms/prima;
-----------


import database.orms.prisma.Main;
class New extends Main{
}
```
### Dynamic Method Dispatch
- Type can be of parent but object is of a child class.
- When the child class implements the same method as the parent(Overriding), which method to call is decided at runtime. THis is called Dynamic Method Dispatch.
```Java
class A{
	public void show(){
		System.out.println("in A");
	}
}
class B extends A{
	public void show(){
		System.out.println("in B");
	}
}
class C extends A{
	public void show(){
		System.out.println("in C");
	}
}
class Main{
	public static void main(String[] args){
		A obj = new A();
		obj.show();
		obj = new B();
		obj.show();
		obj = new C();
		obj.show();
	}
}
// Ouput -> in A in B in C -> Dynamic.
```
### final Keyword
- **final variables** -> Cannot change value
- **final class** -> Cannot inherit
- **final method** -> cannot be overridden
### Object Class
- Every class in java extends the Object class.
```Java
class Laptop{
	public void show(){
		System.out.println("in C");
	}
}
class Main{
	public static void main(String[] args){
		A obj = new A();
		System.out.println(A);
	}
}

A.toString() -> Class.name+"@"+hashCode();
```
- hashCode() - returns hash of all the variables in the class. Two object of the same class may have different hashcodes. Used in hash collections.
- .equals() - compares locations.
```Java
Map<User, String> m = new HashMap<User, String>();
--> Here the key is the hashcode of the User class -> Identify which bucket String should ggo to.
--> Never change the internal values of the key. If you do the hashcode changes -> Old bucket is invalid.
```
### Upcasting and Downcasting
```Java
Class A{
}
Class B extends A{}

A obj = new B();  ==> Upcasting
B obj2 = (B)obj ==> Downcasting - has to be explicit.
```
### Inner Classes
```Java
class A{
	public A(){
		system.out.println("In A consturctor");
	}
	public void show(){
		system.out.println("Showing A");	
	}
	class B{
		public B(){
			system.out.println("In B consturctor");
		}
	}
	static class C{
		public C(){
			system.out.println("In C consturctor");
		}
	}
}

A obj_A = new A();
B obj = new B();//Wrong
A.B obj = new obj_A.B(); // Correct
A.C obj2 = new A.C(); // Correct
```
### Anonymous Inner Class
```Java
A obj = new A(){
	system.out.println("Showing new A");	
};
A.show(); // Prints => Showing new A 
```
### Enums
```Java
enum Laptop{
	Mac(2000), XBox(500), Surface(1500);
	private int price;
	private Laptop(int price){
		this.price = price;
	}
	public void getPrice(){
		return this.price;
	}
}
```
### Functional Interface and Lambda Exressions
- An Interface with just one method.
```Java
@FunctionalInterface
interface A{
	void show();
}
```
- Can be used with functional interfaces.
```Java
A obj = ()->{
	System.out.println("In A");
};
obj.show();
```
### Collections API
- Collection Interface -> Implemented by all the collecitons(Set, Queue, Stack etc.)
```Java
import java.util.*;

class Practice {
    public static void main(String[] args) {

        // ===== COLLECTION BASICS =====
        Collection<Integer> nums = new ArrayList<>();
        nums.add(1); nums.add(2);

        for (int num : nums) System.out.println(num); // foreach

        // ===== LIST (ordered, allows duplicates) =====
        List<String> names = new ArrayList<>();
        names.add("John"); names.add("Jane");
        System.out.println(names.get(0)); // index access

        // ===== SET =====
        Set<String> unique = new HashSet<>(); // unordered, no duplicates
        unique.add("John"); unique.add("Jane"); unique.add("John");

        Set<String> sorted = new TreeSet<>(); // sorted, no duplicates
        sorted.add("John"); sorted.add("Jane");

        // ===== ITERATOR =====
        Iterator<String> it = names.iterator();
        while (it.hasNext()) System.out.println(it.next());

        // ===== MAP (key-value) =====
        Map<String, Integer> map = new HashMap<>();
        map.put("John", 30);
        map.put("John", 35); // overwrite

        for (Map.Entry<String, Integer> e : map.entrySet())
            System.out.println(e.getKey() + " " + e.getValue());

        // ===== THREAD-SAFE MAP =====
        Map<String, List<String>> table = new Hashtable<>();
        table.put("John", Arrays.asList("Reading", "Swimming"));

        // ===== SORTING =====
        Collections.sort(names); // natural order

        // custom comparator (length desc)
        Comparator<String> lengthDesc = new Comparator<String>() {
            public int compare(String a, String b) {
                return b.length() - a.length(); // return a-b if +ve a>b else b>a
            }
        };
        Collections.sort(names, lengthDesc);
        Collections.sort(names, (a, b) -> b.length() - a.length());

        // ===== CUSTOM OBJECT SORT =====
        List<Student> students = new ArrayList<>();
        students.add(new Student("John", 20));
        students.add(new Student("Jane", 25));

        Collections.sort(students); // uses compareTo
    }

    // ===== COMPARABLE =====
    static class Student implements Comparable<Student> {
        String name;
        int age;

        Student(String n, int a) {
            name = n;
            age = a;
        }

        public int compareTo(Student o) {
            return this.age - o.age; // sort by age
        }
    }
}
```
### Stream API
```Java
import java.util.*;
import java.util.function.Consumer;
import java.util.stream.Stream;

class Practice {
    public static void main(String[] args) {
        List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5);

        // forEach takes an object of type Consumer, which is a functional interface that has a single method accept(T t)
        Consumer<Integer> printNum = new Consumer<>(){
            @Override
            public void accept(Integer num) {
                System.out.println("Number: ");
                System.out.println(num);
            }
        };
        Consumer<Integer> printNumLambda = (num) -> {
            System.out.println("Number: ");
            System.out.println(num);
        };
        nums.forEach(printNum);
        nums.forEach(printNumLambda);
        nums.forEach(num ->{
            System.out.println("Number: ");
            System.out.println(num);
        });

        // Stream APIs
        // Stream is an interface.
        // Stream can be run only once, it is not reusable.
        // Stream values do not affect the original collection, it is immutable.
        Stream<Integer> numStream = nums.stream();
        numStream.forEach(num -> {
            System.out.println("Stream Number: ");
            System.out.println(num);
        });
        
        // filter -> returns a new stream that contains only elements that match the given predicate.
        numStream = nums.stream();
        // Takes in a Predicate
        Predicate<Integer> isEven = new Predicate<>(){
            @Override
            public boolean test(Integer num) {
                return num % 2 == 0;
            }
        };
        numStream.filter( num -> num%2==0 ).forEach(num -> System.out.println(num));

        // map -> returns a new stream that contains the results of applying the given function to the elements of the original stream.
        numStream = nums.stream();
        // Takes in a Function
        Function<Integer, String> square = new Function<>(){
            @Override
            public String apply(Integer num) {
                return "Square of " + num + " is " + String.valueOf(num * num);
            }
        };
        numStream.map(num -> num*num).forEach(num -> System.out.println(num));

        // reduce -> returns a single value that is the result of applying a binary operator to the elements of the stream.
        // reduce( identity, accumulator ) -> identity = index to start from, accumulator is a BinaryOperator that takes in two values and returns a single value.
        // Takes in a BinaryOperator
        BinaryOperator<Integer> sumOperator = new BinaryOperator<>(){
            @Override
            public Integer apply(Integer a, Integer b) {
                return a + b;
            }
        };
        numStream = nums.stream();
        int sum = numStream.reduce(0, (a, b) -> a + b);
        System.out.println("Sum: " + sum);
    }
}
```
### Annotations
- Annotations are metadata added to Java classes, methods, or fields
- Frameworks use reflection to read annotation and perform certain actions based on them.
