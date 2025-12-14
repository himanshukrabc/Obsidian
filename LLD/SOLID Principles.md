## Single Responsibility Principle (SRP)
- Each class should be responsible for a single responsibility -> *Separation of Concerns*
- **Responsibility** - Reason for a class to change.
- Do not create a **God Class** - which is responsible for everything.
```java
class Employee {
    private String name;
    private String email;
    private double salary;
    public void calculateSalary() {
        // Complex salary calculation logic
        // Includes tax calculations
    }
    public void saveToDatabase() {
        // Connect to database
        // Prepare SQL
        // Execute query
    }
    public void generatePayslip() {
        // Format payslip
        // Add company logo
        // Convert to PDF
    }
    public void sendPayslipEmail() {
        // Connect to email server
        // Create email with attachment
        // Send email
    }
}
```
#### Why does SRP matter?
- **Easier to read:** You immediately understand what the class is supposed to do. No surprises.
- **Easier to test:** Smaller responsibilities mean smaller test cases and fewer dependencies.
- **Less brittle:** Changes in one responsibility don’t ripple across unrelated parts of the code.
- **Easier to reuse:** Small, focused classes are more flexible and can be reused in different contexts.
- **Scales better:** Teams can own different parts of the system without stepping on each other’s toes.
#### Applying SRP
```java
class Employee {
    private String name;
    private String email;
    private double baseSalary;
    public Employee(String name, String email, double baseSalary) {
        this.name = name;
        this.email = email;
        this.baseSalary = baseSalary;
    }
    public String getName() { return name; }
    public String getEmail() { return email; }
    public double getBaseSalary() { return baseSalary; }
}
class PayrollCalculator {
    public double calculateNetPay(Employee employee) {
        double base = employee.getBaseSalary();
        double tax = base * 0.2;  // Sample tax logic
        double benefits = 1000;   // Fixed benefit deduction
        return base - tax + benefits;
    }
}
class EmployeeRepository {
    public void save(Employee employee) {
        // Example: JDBC code or ORM logic
        System.out.println("Saving employee " + employee.getName() + " to database...");
    }
}
class PayslipGenerator {
    public String generatePayslip(Employee employee, double netPay) {
        return "Payslip for: " + employee.getName() + "\n" +
               "Email: " + employee.getEmail() + "\n" +
               "Net Pay: ₹" + netPay + "\n" +
               "----------------------------\n";
    }
}
class EmailService {
    public void sendPayslip(Employee employee, String payslip) {
        System.out.println("Sending payslip to: " + employee.getEmail());
        // Simulate email with a print
        System.out.println(payslip);
    }
}
```
#### Mistakes when applying SRP
1. **Over-splitting Responsibilities** - Breaking a class into too many classes leads to unnecessary complexity. Focus should be on cohesion and separation of concerns.
2. **Confusing Methods with Responsibilities** - Assuming each method must be its own class.
	```java
	class EmailService {
	    public void sendWelcomeEmail() {}
	    public void sendPayslipEmail() {}
	}
	```
	Some developers might try to split this into:
	- WelcomeEmailSender
	- PayslipEmailSender
	**Why it’s a problem:**
	- Both methods deal with the same **responsibility**: sending emails
	- Splitting them adds more boilerplate without clear benefits
3. **Ignoring SRP in Small Classes** - 
	- These responsibilities often evolve independently
	- Small changes to one feature might introduce bugs in others
4. **Misunderstanding “Reason to Change”** - 
	- SRP is **not** about who asks for the change, but **what kind of change** is being made.
#### Cons of SRP
- **Bulky code** - May lead to many small classes.
- More classes to test but each class will have 1 responsibility.
## Open-Closed Principle
- Classes should be -
  - **Open for Extension** - The behaviour of the class can be extended as new requirements come in.
  - **Closed for Modification** - Existing code of the class should not be modified
#### Example
```Java
class PaymentProcessor {
    public void processCreditCardPayment(double amount) {
        System.out.println("Processing credit card payment of $" + amount);
        // Complex logic for credit card processing
    }

    public void processPayPalPayment(double amount) {
        System.out.println("Processing PayPal payment of $" + amount);
        // Logic for PayPal processing
    }
}

class CheckoutService {
    public void processPayment(String paymentType) {
        PaymentProcessor processor = new PaymentProcessor();

        if ("CreditCard".equals(paymentType)) {
            processor.processCreditCardPayment(100.00);
        } else if ("PayPal".equals(paymentType)) {
            processor.processPayPalPayment(100.00);
        }
    }
}
```
- Now suppose you want to implement another payment method through UPI -> you add another method in PaymentProcessor and add another condition in processPayment method.
- Each modification carries the risk of:
	1. **Introducing Bugs:** You might accidentally break the existing credit card or PayPal functionality while adding the new payment method.
	2. **Increased Testing Overhead:** Every time you change the class, you need to re-test _all_ its functionalities, not just the new one.
	3. **Reduced Readability:** The class becomes a monstrous collection of `if-else if` statements or a `switch` case that's hard to navigate and understand.
	4. **Scalability Issues:** Adding new payment types becomes progressively more difficult and error-prone.
#### Implementing OCP
##### 1. Define an Interface
```Java
interface PaymentMethod {
    void processPayment(double amount);
}
```
##### 2. Implement Classes extending Interfaces
```Java
class CreditCardPayment implements PaymentMethod {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing credit card payment of $" + amount);
    }
}
class PayPalPayment implements PaymentMethod {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing PayPal payment of $" + amount);
    }
}
class UPIPayment implements PaymentMethod {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing UPI payment of ₹" + amount * 80); // Assuming a conversion rate for example
    }
}
```
##### 3. Modify PaymentProcessor to use abstraction
```Java
class PaymentProcessor {
    public void process(PaymentMethod paymentMethod, double amount) {
        paymentMethod.processPayment(amount);
    }
}
```
#### Mistakes while applying OCP
- **Premature Abstraction** - Do not apply OCP unless you have a case for it.
- **Misinterpreting "Closed for Modification"** - If there is a bug code modification is necessary.
- **Abstraction Hell** - Too many abstractions can lead to complexity.
- **Forgetting the "Why"** - Apply OCP for maintainability, scalability
- **Not Anticipating the Right Extension Points** - Identifying where your system is likely to change is crucial.
## Liskov Substitution Principle
- If Class S is subType of Class T then any object of Class S must be usable anywhere T is expected without breaking the code.
#### Example
```Java
class Document {
    protected String data;
    public Document(String data) {
        this.data = data;
    }
    public void open() {
        System.out.println("Document opened. Data: " + data.substring(0, Math.min(data.length(), 20)) + "...");
    }
    public void save(String newData) {
        this.data = newData;
        System.out.println("Document saved.");
    }
    public String getData() {
        return data;
    }
}
class ReadOnlyDocument extends Document {
    public ReadOnlyDocument(String data) {
        super(data);
    }
    @Override
    public void save(String newData) {
        throw new UnsupportedOperationException("Cannot save a read-only document!");
    }
}
```
- Suppose you have the Document Class which is in use. Then there comes a requirement where a ReadOnlyDocument is to be implemented. So you extend the Document Class and implement the ReadOnlyDocument Class.
- This is where it goes wrong - A piece of existing code assumes all docs are editable/savable.
```Java
class DocumentProcessor {
    public void processAndSave(Document doc, String additionalInfo) {
        doc.open();
        String currentData = doc.getData();
        String newData = currentData + " | Processed: " + additionalInfo;
        doc.save(newData); // Critical assumption: all Documents are savable
        System.out.println("Document processing complete.");
    }

    public static void main(String[] args) {
        Document regularDoc = new Document("Initial project proposal content.");
        ReadOnlyDocument confidentialReport = new ReadOnlyDocument("Top secret government data.");
        DocumentProcessor processor = new DocumentProcessor();
        System.out.println("--- Processing Regular Document ---");
        processor.processAndSave(regularDoc, "Reviewed by Alice");
        System.out.println("\n--- Processing ReadOnly Document ---");
        try {
            processor.processAndSave(confidentialReport, "Reviewed by Bob");
        } catch (UnsupportedOperationException e) {
            System.err.println("Error: " + e.getMessage());
        }
    }
}
````
- Our subtype (**ReadOnlyDocument**) cannot be seamlessly substituted for its base type (**Document**) without altering the desired behaviour of the program.
#### Implementing LSP
##### 1. Define Behaviour Interface
- Instead of having one base class with assumptions about mutability, **let’s break responsibilities apart**
```Java
interface Document {
    void open();
    String getData();
}
interface Editable {
    void save(String newData);
}
```
##### 2. Implement EditableDocument and ReadOnlyDocument
```java
class EditableDocument implements Document, Editable {
    private String data;
    public EditableDocument(String data) {
        this.data = data;
    }
    @Override
    public void open() {
        System.out.println("Editable Document opened. Data: " + preview());
    }
    @Override
    public void save(String newData) {
        this.data = newData;
        System.out.println("Document saved.");
    }
    @Override
    public String getData() {
        return data;
    }
    private String preview() {
        return data.substring(0, Math.min(data.length(), 20)) + "...";
    }
}
class ReadOnlyDocument implements Document {
    private final String data;
    public ReadOnlyDocument(String data) {
        this.data = data;
    }
    @Override
    public void open() {
        System.out.println("Read-Only Document opened. Data: " + preview());
    }
    @Override
    public String getData() {
        return data;
    }
    private String preview() {
        return data.substring(0, Math.min(data.length(), 20)) + "...";
    }
}
```
##### 3. Refactor the Client Code
```java
class DocumentProcessor {
    public void process(Document doc) {
        doc.open();
        System.out.println("Document processed.");
    }

    public void processAndSave(Document doc, String additionalInfo) {
        if (!(doc instanceof Editable editableDoc)) {
            throw new IllegalArgumentException("Document is not editable.");
        }

        doc.open();
        String currentData = doc.getData();
        String newData = currentData + " | Processed: " + additionalInfo;
        editableDoc.save(newData);
        System.out.println("Editable document processed and saved.");
    }
}
```
#### Why does LSP matter?
- **Reliability and Predictability** - New classes are reliable.
- **Reduced Bugs:** 
  LSP violations often lead to conditional logic (e.g., if (obj instanceof ReadOnlyDocument)) in client code to handle subtypes differently. This is a ==**code smell**==. It’s a sign that your design is leaking abstraction. When *client code has to “know” the subtype to behave correctly*, you’ve broken polymorphism.
- **Maintainability and Extensibility:** Well-behaved hierarchies are easier to understand, maintain, and extend.
- **True Polymorphism:** LSP is what makes polymorphism truly powerful. You can write generic algorithms that operate on a base type, confident that they will work correctly with any current or future subtype.
- **Testability:** Tests written for the base class's pass for all its subtypes.
#### Pitfalls when implementing LSP
- **The “Is-A” Linguistic Trap** - Creating subtypes just based on "is-a" relationship is wrong. *Subtyping must be based on behavior, not just taxonomy*. 
- **Overriding Methods to Do Nothing or Throw Exceptions** - 
  If a subclass **cannot meaningfully implement** a method defined in the base class, it’s likely **not a valid subtype**.
- **Violating Preconditions or Postconditions** - 
	- **Precondition violation**: The subtype **requires more** than the base class contract promised.
	- **Postcondition violation**: The subtype **delivers less** than the base class guaranteed.
- **Type Checks in Client Code** - 
	- Whenever client code has to **know the exact subtype** to behave correctly, you’ve violated the principle of substitution.
	- Polymorphism should make the client code unaware of specific subtypes.
- **Restricting or Relaxing Behavior Unexpectedly** - 
	- Subclasses shouldn’t arbitrarily **tighten or loosen** the behavior defined by the base class.
	  For example:
		- Making a **mutable** property in the base class **immutable** in the subclass (or vice versa) can lead to subtle bugs.
		- Changing validation logic in ways that break existing assumptions in client code is another LSP violation.
