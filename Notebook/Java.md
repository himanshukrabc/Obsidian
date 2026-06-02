It is platform independent, object oriented.  
C/C++ are not platform independent. ⇒ different outputs for different kernels.  
If you have .class file, you can use it anywhwere.

  

Architecture Neutral : doesnot change based on wether it is 32bit or 64bit system. In C/C++, int occupies 2/4 bits based on the system.

  

### static
```SQL
public static void main(String[] args){}
```
- static methods can be directly called by the class, no object declaration is required.  
- static ⇒ remains in the memory.  
- If we dont give static and keep on compiling, we get multiple class files. Now if we call the class, it wont compile due to multiple files. If we use static the main function will have the latest copy stored in the memory.
### Naming Convention

For a file with multiple classes, save with the name of the public class always. In a file, there can be only one public class.

### Memory Alloctions in Java

- Heap ⇒ Runtime init

- Stack ⇒ Compile time init.

- Class ⇒

- Program Counter Memory ⇒ Each thread has this memory associated with it.

- Native Method Stack Memory ⇒This area is implemented using languages other than java. With the creation of new threads, memory is allocated in this area for each created thread. The size of the native area can be fixed or dynamic.

### Compile using ⇒ javac xyz.java ⇒ xyz.class ⇒ javac xyz

When you compile a .java file, memory is allocated for all class properties during compilation. This memory is allocated in the stack.

When you execute the .class file, where an object is initialized, there will be another memory allocation for the object, in the heap. This allocation includes a copy of the properties stored in the stack. ⇒ Thus object is the instance of a class.

  

### Garbage Collector

As soon as the object completes its task, the memory is deallocated by the GC

  

### this keyword

references the instace which called the method/constructor.

  

### Encapsulation

The data in properties is protected within the class and cannot be accessed outside the class. This is done using private keyword.

### Abstraction

Data **abstraction** is the process of hiding certain details and showing only essential information to the user.

> [!important] It is possible to have multiple constructor. ⇒ overloading in action.

### Copy Constructor:

```Java
// Used when we need to do create deep copies of the objects.
public Employee(Employee e){
	this.p1 = e.p1;
	this.p2 = e.p2;
}


// This constructor is called when::
Employee e5 = new Employee(12,"Route");///parameterized constructor called
Employee e6 = e5;//This creates a shallow copy of the object. 
//==> e6 is a reference to e5;
Employee e7 = new Employee(e5); //This creates a deep copy of the object;
```

  

### Polymorphism

- **Compiletime(Overloading) ⇒**

- **Runtime(Overriding) ⇒** dynamic method dispatch.

  

### Access Specifiers

- private ⇒ within class

- default ⇒ by any class in the package.

- protected ⇒ by child classes as well..

- public ⇒ anywhere

  

### Importing:

Packages :

1. Used to remove class naming conflicts.

1. Used to segregate classes based on type/uses.

1. increases securities based on encapsulation identifiers

Declare the package in the first line of your java file.

Java classes are named as follows packageName.subPackageName.className ==> fully classified class name

How to access classes from different packages.  
Use of import keyword is required. This is done in three ways.

```Plain
   1) Importing Using fully classified class name
       import packageName.subPackageName.className; // imports a single class
       import packageName.subPackageName.; // imports the entire package

       className cls = new className();

   2) just Using fully classified class name without importing
       packageName.subPackageName.className cls = new packageName.subPackageName.className();

   3) Importing from different location using class path.
       classpath is an environment variable.

       i) Using temporary classpath ==> works for the current session.
           set classpath = path1;path2;path3;. ==> ; is a separator in paths, .==current path
           ==> use this command in command prompt before compiling the file.

       ii) altering environment variable of the pc.
```

### Distribution of packages created:

we just need to distribute certain classes and certain files. This can be done by creating .jar files. Use "jar -h" in command prompt

```Plain
   jar -cvf mylib.jar Test.class,Test.java
   jar -cvf mylib.jar .class //all .class files
   jar -cvf mylib.jar  // all files

   specifying jar file in the classpath lets you access all the files/classes.
```

  

  

### Inheritance

⇒ models “is a” relationship

The process by which a child class aquires some/all properties and methods of another class.

- Single

- Multi level - A->B->C

- Hierarchical - Multiple child of a parent

- Multiple - class inherits from multiple classes ==> Not in Java

- Hybrid

- Cyclic

### Aggregation

⇒ implements "has a” relationship

It means that a class has an object of another class as its property.

  

### Upcasting

```Java
//When a variable of parent class refers to the object of the child class
// class Child extends Parent{} 
Parent p = new Child();
//p has all the properties of Child class.
```

### super keyword

super ⇒ refers to the immidiate parent of the obj

super.color ⇒ color attr in parent class  
super() ⇒ constructor of parent class

  

### Interfaces

It is the blue print of the class.

All the methods are declared in interface but their implementation is not shown.  
Implementation stays in the class.

We cant inherit multiple classes in Java. To get multiple inheritance, we use interfaces.

```Java
public interface inter{
	void printing(String str);
}

class Myclass implements inter{
	@Override
	public void printing(){
		System.out.println("Heyyy!");
		inter m = new inter(){//Anonymous class
        public void printing(String str){
            System.out.print(str);
        }
    };
	}
}
```

Interfaces are used to maintain a common set of methods across a set of classes. This is helpful in automation.

> [!important] Whenever you implement an interface, you need to implement all the methods in the interface. You can add extra methods as well.

> [!important] An empty interface is known as marker interface.

### final keyword

1) for variables ⇒ value cannot be changed  
final int num = 100; ⇒ num cannot be assigned new values.

2) for methods ⇒ method caannot be overridden  
final void func1 ( ) { … }

3) for classes ⇒ this class cannot be extended.

  

### abstract keyword

1) for methods ⇒ Such methods are meant to be overridden.  
abstract void display ( ) { … }

2) for classes ⇒ Any class with abstract methods must be declared with abstract keyword.  
Such classes cannot be instantiated. They are meant for inheriting.  
abstract class MyClass{ … }

  

### Abstract classes vs Interfaces :

Interfaces have 100% abstraction. In abstract classes there is partial abstraction.  
Abstraction ⇒ The funtionality is not visible to the user.

  

### static keyword :

It is used for memory management. There is no extra memory allocation in objects for static attributes. Objects only get references to these attributes.  
Such vars/methods can be called directly from classname.  
1) for variables ⇒ The variable refers to a common property. This variable is shared by all the objects of the class. Even if one object changes it,it get changed for all the objects.

2) for methods ⇒ The method is common to all.

```Java
public class Prod{
	static int name="Electronics";
	static void print(){
		System.out.println("Printing");	
	}
}

System.out.println(Prod.name);
Prod.print();
```

3) for classes ⇒ same as normal class but cannot be instantiated.  

### Generics

These are parameterized types.

We need to use wrapper classes for primitive datatypes for such initializations. Wrapper classes help generics to use the datatypes.

```Java
public class Box<T> {
    T container; // now container can store anything

    public Box(T container){
        this.container = container;
    }

    public T getVal(){
        return this.container;
    }

}

public class Gen{
	public static void main(String[] args){
		Box<Integer> b = new Box<Integer>(123);
		System.out.println(b.getVal());
		Box<Double> b = new Box<Double>(12.3);
		System.out.println(b.getVal());
	}
}

```

### Wrapper Classes

These are the classes declared for converting datatypes into classes.  
Auto Boxing ⇒ conversion from primitive to wrapper classes  
Unboxing ⇒ conversion from wrapper classes to primitive

> [!important] All wrapper classes have Object as their Parent Class

[https://www.javatpoint.com/wrapper-class-in-java](https://www.javatpoint.com/wrapper-class-in-java)

  

### Exception Handling

Exception ⇒ Errors thrown by compiler during **runtime** of a program.

Use of try…catch, throw, throws, finally keywords.

### throw and throws:

throw is used to throw custom exceptions.  
throws is used to state that a method throws an Exception. This is beneficial for unchecked exceptions.

> [!important]
> 
> e.printStackTrace(); ⇒ Prints where the exception occours for debugging.

```Java
class AgeInvalidException extends Exception {
    AgeInvalidException(){
        super("Age is Invalid!!!");
    }
    AgeInvalidException(String msg){
        super(msg);
    }
}

class myClass{
    public static void main(String args[]){
        try{
            int n1 = Integer.parseInt(args[0]);
            int n2 = Integer.parseInt(args[1]);
            int res = n1/n2;
            System.out.println(res);
            if(n2<10)throw new AgeInvalidException("Age is "+n2);
						if(n1<10)throw new ArithmeticException("Age Invalid");
        }
        catch(ArithmeticException e){
            System.out.println(e.getMessage());
        }
        catch(AgeInvalidException e){
            System.out.println(e.getMessage());
        }
        catch(Exception e){///catches all other exceptions.
            System.out.println(e.getMessage());
						e.printStackTrace();// Prints where the exception occours
        }
        finally{
            System.out.println("finally Block");
        }
    }
		public void func1() throws FileNotFoundException(){
				FileReader file = new FileReader();
				BufferReader bf = new BufferReader();
		}
}
```

  

[![](https://static.javatpoint.com/core/images/exception-vs-error-in-java2.png)](https://static.javatpoint.com/core/images/exception-vs-error-in-java2.png)

  

### Runtime Stack

When a method is called, an entry is made in the runtime stack. Each thread has its own stack. Every entry in the run-time stack is known as stack frame or activation record.

Once all the functions have been called, the runtime stack will be empty and is destroyed. Then the thread is also terminated.

  

### Functional Interface

⇒ an interface with just one function and some Datatype properties.

### Anonymous Functions, Anonymous Classes, Lamda Expr

```Java
// Anonymous Functions
int sum=(int a,int b){
	return a+b;
}
```

```Java
// Anonymous Classes
@FunctionalInterface
interface message{
    void show(String str);
}

public class myCls implements message{
    public static void main(String[] args) {
				//Anonymous Class
				string name="HK";
        message m = new message(){
            @Override
            public void show(String str){
                System.out.print(str+"123"+name);
            }
        };
        m.show("000");
    }
}
```

```Java
//Lambda Expr
@FunctionalInterface
interface message{
    String show(String str);
}

public class myCls implements message{
    public static void main(String[] args) {
				//Anonymous Class
				string name="HK";
        message m = (String str){
            return (str+"123"+name);
        };
        System.out.println(m.show("000"));
    }
}
```