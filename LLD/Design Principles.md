## 1. DRY - Don't Repeat Yourself
- Every piece of knowledge must have a single, unambiguous, authoritative representation within a system.
- Applies to Code, Configuration, Business Rules, Data Models, Documentation and Tests.
#### Why is repetition a problem?
- **Harder to maintain** - Updation to duplicated code, requires update at all places. Single miss -> inconsistent behaviour
- **Higher risk of bugs** - Inconsistent duplication or missed updation may result in a lot of bugs.
- **Bloated Codebase** - Duplicate code takes up unnecessary space. Tests are required for each duplication which causes further bloating.
- **Poor test coverage** - Tests also need to duplicated.
#### How to apply DRY
##### Create a Utility Class
```java
public class EmailValidator {
    public static boolean isValid(String email) {
        return email != null &&
               email.contains("@") &&
               email.contains(".") &&
               (email.endsWith(".com") || email.endsWith(".org"));
    }
}
```
##### Use It Across Modules
```java
if (EmailValidator.isValid(user.getEmail())) {
    // Proceed with business logic
}
```

#### When It Is Okay to Repeat
- **Avoid Premature Abstractions** - Let the conditions arise for duplication, then only create the utility class.
- **Keep Tests Readable** - Repeating a bit of test code improves clarity. Tests should be easy to read and understand.
- **Simple code may be repeated** - code is extremely simple and unlikely to change, it may be better to repeat it.
## 2. KISS - Keep it Simple, Stupid
#### Why Complexity Is Dangerous
- **Code is hard to read and understand**
- **More Places for Bugs to Hide**
- **Slower Onboarding** - New developers take longer to ramp up.
- **Poor Debuggability**
#### Signs You’re Violating KISS
- You added an interface before you needed multiple implementations
- You used reflection for something a method call could handle.
- You introduced an extra layer “just in case” you might need it later
- Your method has five optional parameters and deeply nested conditionals
- You use recursion when a loop would be simpler
##### Reflection
- Ability of an object to inspect, analyse and modify its structure and behaviour at runtime.
```C
class User {
    private String name;
    public User(String name) {
        this.name = name;
    }
    private void greet() {
        System.out.println("Hello " + name);
    }
}
User u = new User("Alice");
Class<?> cls = u.getClass();
System.out.println(cls.getName());   // User
for (Method m : cls.getDeclaredMethods()) {
    System.out.println(m.getName());
}
Method method = cls.getDeclaredMethod("greet");
method.setAccessible(true);
method.invoke(u);   // Hello Alice
```
#### How to Apply the KISS Principle
- **Focus on readability** while writing code
- Avoid **Premature Abstraction**
- **Composition over Inheritance**
- **Keep Functions Short**
- **Use Familiar Constructs** - Do not use DS which are not so recognised when normal DS can do the job.
#### When Not to Simplify
- **Do not oversimplify critical systems.** Sometimes, a little complexity is necessary to meet performance, scalability, or security requirements.
- **Avoid duplicating logic** - If an abstraction prevents repetition, it’s usually worth it.
- **Using a design pattern might be more understandable than a "simplified" custom approach**
## 3. YAGNI - You aren't gonna need it
- Adding features to your code that you thought you would need but don't need them yet.
- Do not apply abstractions prematurely.
- Leads to - Higher Cost, Complexity, Delayed Value and Wasted Effort.
#### When to Violate YAGNI
- **Security and Compliance requirements**
- **Architecture with known long-term constraints** - Some abstraction may be necessary due to type of architecture
## 4. Law of Demeter
- Avoid chaining of methods
```Java
class OrderService{
	public void displayFirstItemPrice(Customer customer) {
	    Money price = customer.getShoppingCart().getItems().get(0).getProduct().getPrice();
	    System.out.println("Price of the first item: " + price.getAmount());
	}
}
```
#### Issues
##### High Coupling
- displayFirstItemPrice class id **deeply coupled** with the structure of the customer class.
- If ShoppingCart start using Map or renames getProduct() or if Product pricing is obtained from a different method, the method will fail.
##### Encapsulation Violation
- For this to work - 
	Customer class exposes ShoppingCart class
	ShoppingCart exposes Items list and Product class
- All of this is a violation of encapsulation
##### Maintenance Nightmare
- If you now want to change the Money Wrapper from Decimal to BigDecimal for more accuracy, you need to go and change every getPrice() implementation.
##### Testability Issues
- You need to mock everything -> customer with a shopping cart with items of products with price.
### Law of Demeter
- An object should only call methods on:
	- Itself
	- Its own fields
	- Its method parameters
	- Objects it creates
#### Refactoring Above Code
```java
class ShoppingCart {
    public Money getFirstItemPrice() {
        if (items.isEmpty()) return Money.ZERO;
        return items.get(0).getProduct().getPrice();
    }
}
class Customer {
   
    public Money getFirstCartItemPrice() {
        return shoppingCart.getFirstItemPrice();
    }
}
void displayFirstItemPrice(Customer customer) {
    Money price = customer.getFirstCartItemPrice();
    System.out.println("Price of the first item: " + price.getAmount());
}
```
- OrderService now does not care about ShoppingCart at all.
## 5. Separation of Concern
- Each responsibility must be handled separately by a component.
- Eg - In an e-commerce system, a class tracking the user auth should not be responsible for storing the order history.
#### Why Separation of Concern is important?
- **Modularity** - Breakdown of complex system into modular pieces.
- **Maintainability** - Due to smaller pieces which is each responsible for own task-> Any change needs to be done at one place.
- **Scalability** -  New concerns can be addressed by adding or modifying individual modules without changes to other parts of the system.
- **Reusability** - A module can be reused at multiple places as it addresses a single concern.(also promotes DRY)
- **Parallel Development**
#### Successful SoC
```Java
public class ShoppingCart  {  
    private List<Item> items = new List<Item>();  
    public void AddItem(Item item)  {  
        items.Add(item);  
    }  
    public void RemoveItem(Item item)  {  
        items.Remove(item);  
    }  
    public decimal CalculateTotal()  {  
        decimal total = 0;  
        foreach (var item in items) {  
            total += item.Price;  
        }  
        return total;  
    }  
}  
public class Item  {  
    public string Name { get; set; }  
    public decimal Price { get; set; }  
}
```
#### Unsuccessful SoC
```Java
public class User {
    private String username;
    private String password;
    private List<Item> shoppingCart = new ArrayList<>();
    public void addItemToCart(Item item) {
        shoppingCart.add(item);
    }

    public void removeItemFromCart(Item item) {
        shoppingCart.remove(item);
    }

    public boolean authenticate(String username, String password) {
        return this.username.equals(username) && this.password.equals(password);
    }

    public double calculateTotal() {
        double total = 0.0;
        for (Item item : shoppingCart) {
            total += item.getPrice();
        }
        return total;
    }
}
```
- This class also implements authenticate method which makes this class handle 2 concerns - auth and shopping cart.
## 6. Coupling and Cohesion
#### Coupling
- **Coupling** refers to the degree of interdependence between software modules. 
- **High coupling** means that modules are closely connected and changes in one module may affect other modules. 
```java
class OrderService {
    public void placeOrder() {
        MySQLDatabase db = new MySQLDatabase(); // tightly coupled
        db.connect();
        db.save();
    }
}
```
- **Low coupling** means that modules are independent, and changes in one module have little impact on other modules.
```java
class OrderService {
	//Some coupling but still fine.
    private Database database;
    public OrderService(Database database) {
        this.database = database;
    }
    public void placeOrder() {
        database.save();
    }
    //Least Coupled
    public void placeOrder(Database db){
	    db.save();
    }
}
```
#### Cohesion
- **Cohesion** refers to the degree to which elements within a class work together to fulfil a single, well-defined purpose.
- **High cohesion** means that elements are closely related and focused on a single purpose 
```java
class AuthService {
    void authenticateUser() {}
}
class EmailService {
    void sendEmail() {}
}
class ReportService {
    void generateReport() {}
}
```
- **Low cohesion** means that elements are loosely related and serve multiple purposes.
```java
class UserManager {
    void authenticateUser() {}
    void sendEmail() {}
    void generateReport() {}
}
```
