# Adapter
- Bridge the gap between two code pieces without modifying existing functionality
- When working with **incompatible interfaces**, code is written in if..else cases with knowledge of executing class(new or legacy).
- Introduces a wrapper class that sits between the two gateways.
## Example - Payment Gateways
- Suppose you have a paymentService which does payments based on your in-house payment gateway.
```Java
interface PaymentProcessor {
    void processPayment(double amount, String currency);
    boolean isPaymentSuccessful();
    String getTransactionId();
}

class InHousePaymentProcessor implements PaymentProcessor {
    private String transactionId;
    private boolean isPaymentSuccessful;
    @Override
    public void processPayment(double amount, String currency) {
        System.out.println("InHousePaymentProcessor: Processing payment of " + amount + " " + currency);
        transactionId = "TXN_" + System.currentTimeMillis();
        isPaymentSuccessful = true;
        System.out.println("InHousePaymentProcessor: Payment successful. Txn ID: " + transactionId);
    }
    @Override
    public boolean isPaymentSuccessful() {
        return isPaymentSuccessful;
    }
    @Override
    public String getTransactionId() {
        return transactionId;
    }
}

class CheckoutService {
    private PaymentProcessor paymentProcessor;
    public CheckoutService(PaymentProcessor paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }
    public void checkout(double amount, String currency) {
        System.out.println("CheckoutService: Attempting to process order for $" + amount + " " + currency);
        paymentProcessor.processPayment(amount, currency);
        if (paymentProcessor.isPaymentSuccessful()) {
            System.out.println("CheckoutService: Order successful! Transaction ID: " 
                               + paymentProcessor.getTransactionId());
        } else {
            System.out.println("CheckoutService: Order failed. Payment was not successful.");
        }
    }
}

public class ECommerceAppV1 {
    public static void main(String[] args) {
        PaymentProcessor processor = new InHousePaymentProcessor();
        CheckoutService checkout = new CheckoutService(processor);
        checkout.checkout(199.99, "USD");
    }
}
```
- Now you want to integrate another payment gateway which has a different interface.
```Java
class LegacyGateway {
    private long transactionReference;
    private boolean isPaymentSuccessful;
    public void executeTransaction(double totalAmount, String currency) {
        System.out.println("LegacyGateway: Executing transaction for " 
                           + currency + " " + totalAmount);
        transactionReference = System.nanoTime();
        isPaymentSuccessful = true;
        System.out.println("LegacyGateway: Transaction executed successfully. Txn ID: " 
                           + transactionReference);
    }
    public boolean checkStatus(long transactionReference) {
        System.out.println("LegacyGateway: Checking status for ref: " + transactionReference);
        return isPaymentSuccessful;
    }
    public long getReferenceNumber() {
        return transactionReference;
    }
}
```
- So you use a wrapper class that sits between your system and the other gateway.
### Two Types of Adapters
There are two primary ways to implement an adapter, depending on the language and use case:
#### 1. Object Adapter (Preferred in Java)
- Uses **composition**: the adapter holds a reference to the adaptee (the object it wraps).
- Allows flexibility and reuse across class hierarchies.
- This is the most common and recommended approach in Java.
#### 2. Class Adapter (Rare in Java)
- Uses **inheritance**: the adapter inherits from both the target interface and the adaptee.
- Requires **multiple inheritance**, which Java doesn’t support for classes.
- More suitable for languages like **C++**.
## Implementing Adapter
```Java
class LegacyGatewayAdapter implements PaymentProcessor {
    private final LegacyGateway legacyGateway;
    private long currentRef;
    public LegacyGatewayAdapter(LegacyGateway legacyGateway) {
        this.legacyGateway = legacyGateway;
    }
    @Override
    public void processPayment(double amount, String currency) {
        System.out.println("Adapter: Translating processPayment() for " + amount + " " + currency);
        legacyGateway.executeTransaction(amount, currency);
        currentRef = legacyGateway.getReferenceNumber(); // Store for later use
    }
    @Override
    public boolean isPaymentSuccessful() {
        return legacyGateway.checkStatus(currentRef);
    }
    @Override
    public String getTransactionId() {
        return "LEGACY_TXN_" + currentRef;
    }
}

public class ECommerceAppV2 {
    public static void main(String[] args) {
        // Modern processor
        PaymentProcessor processor = new InHousePaymentProcessor();
        CheckoutService modernCheckout = new CheckoutService(processor);
        System.out.println("--- Using Modern Processor ---");
        modernCheckout.checkout(199.99, "USD");
        // Legacy gateway through adapter
        System.out.println("\n--- Using Legacy Gateway via Adapter ---");
        LegacyGateway legacy = new LegacyGateway();
        processor = new LegacyGatewayAdapter(legacy);
        CheckoutService legacyCheckout = new CheckoutService(processor);
        legacyCheckout.checkout(75.50, "USD");
    }
}
```
## Class Diagram
![[Screenshot 2026-01-02 at 11.24.05 PM.png|700]]
- **Target Interface (e.g.,** `PaymentProcessor`**)**: The interface that the client code expects and uses.
- **Adaptee (e.g.,** `LegacyGateway`**)**: The existing class with an incompatible interface that needs adapting.
- **Adapter**: The class that implements the Target interface and uses the Adaptee internally. It translates calls on the Target interface into calls on the Adaptee's interface.
- **Client (e.g.,** `CheckoutService`**)**: The part of your system that uses the Target interface.
## Advantages of Adapter Method
- **Composition Over Inheritance** - Adapter wraps LegacyGateway instead of subclassing it.
- It is loosely coupled, easier to test and more flexible to change.
- **Encapsulation of Legacy Logic** - No need to change anything in the legacy code.
# Facade
- Provides a unified, simplified interface to a complex subsystem.
## Example - OrderService
```Java
class InventoryService {
    void reserve(String productId) {
        System.out.println("Inventory reserved");
    }
}
class PaymentService {
    void charge(double amount) {
        System.out.println("Payment charged");
    }
}
class ShippingService {
    void ship(String productId) {
        System.out.println("Order shipped");
    }
}
class NotificationService {
    void notifyUser() {
        System.out.println("User notified");
    }
}

public class Main {
    public static void main(String[] args) {

        InventoryService inventory = new InventoryService();
        PaymentService payment = new PaymentService();
        ShippingService shipping = new ShippingService();
        NotificationService notification = new NotificationService();

        // client must know correct order
        inventory.reserve("P1");
        payment.charge(1000);
        shipping.ship("P1");
        notification.notifyUser();
    }
}
```
## Problems
- Whenever your logic to order changes, it changes the main class as well.
## Class Diagram
![[Screenshot 2026-01-03 at 4.49.01 PM.png|700]]
## Implementing Facade Pattern
```Java 
class OrderFacade {

    private final InventoryService inventory = new InventoryService();
    private final PaymentService payment = new PaymentService();
    private final ShippingService shipping = new ShippingService();
    private final NotificationService notification = new NotificationService();

    public void placeOrder(String productId, double amount) {
        inventory.reserve(productId);
        payment.charge(amount);
        shipping.ship(productId);
        notification.notifyUser();
        
        // New Code
        /*
        inventory.reserve(productId);
        payment.charge(amount);
        if(payment.isSucessful){
			shipping.ship(productId);
	        notification.notifyUser("Order Shipped");
        }
        else{
	        notification.notifyUser("Payment Failed");
        }
        */
    }
}

public class Main {
    public static void main(String[] args) {
        OrderFacade facade = new OrderFacade();
        facade.placeOrder("P1", 1000);
    }
}
```
## Benefits
- This code is now extensible to new changes. Does not violate the OCP principle.
- Now if the code changes only the Facade class changes. The core logic does not change.
# Decorator
- lets you **dynamically add new behavior or responsibilities to objects** without modifying their underlying code.
## Example - Text Renderer
```Java
interface TextView {
    void render();
}
class BoldTextView extends TextView {
    @Override
    public void render() {
        System.out.print("Rendering bold text");
    }
}
class ItalicTextView extends TextView {
    @Override
    public void render() {
        System.out.print("Rendering italic text");
    }
}
class BoldItalicTextView extends TextView {
    @Override
    public void render() {
        System.out.print("Rendering bold + italic text");
    }
}
```
- For each type of text you want to render you need a new class. -> **Class Explosion**
- **Rigid Design** - You cant turn off bold based on user preference at runtime.
- **Violates OCP** - Every time a new feature is introduced, you need to modify or extend existing classes.
## Implementing Decorator
```Java
interface TextView {
    void render();
}
// Concrete Component
class PlainTextView implements TextView {
    private final String text;
    public PlainTextView(String text) {
        this.text = text;
    }
    @Override
    public void render() {
        System.out.print(text);
    }
}
//Abstract Decorator
abstract class TextDecorator implements TextView {
    protected final TextView inner;
    public TextDecorator(TextView inner) {
        this.inner = inner;
    }
}
class BoldDecorator extends TextDecorator {
    public BoldDecorator(TextView inner) {
        super(inner);
    }
    @Override
    public void render() {
        System.out.print("<b>");
        inner.render();
        System.out.print("</b>");
    }
}
class ItalicDecorator extends TextDecorator {
    public ItalicDecorator(TextView inner) {
        super(inner);
    }
    @Override
    public void render() {
        System.out.print("<i>");
        inner.render();
        System.out.print("</i>");
    }
}
class UnderlineDecorator extends TextDecorator {
    public UnderlineDecorator(TextView inner) {
        super(inner);
    }
    @Override
    public void render() {
        System.out.print("<u>");
        inner.render();
        System.out.print("</u>");
    }
}
public class TextRendererApp {
    public static void main(String[] args) {
        TextView text = new PlainTextView("Hello, World!");

        System.out.print("Plain: ");
        text.render();
        System.out.println();

        System.out.print("Bold: ");
        TextView boldText = new BoldDecorator(text);
        boldText.render();
        System.out.println();
		//Composite Decorators.
        System.out.print("Italic + Underline: ");
        TextView italicUnderline = new UnderlineDecorator(new ItalicDecorator(text));
        italicUnderline.render();
        System.out.println();

        System.out.print("Bold + Italic + Underline: ");
        TextView allStyles = new UnderlineDecorator(new ItalicDecorator(new BoldDecorator(text)));
        allStyles.render();
        System.out.println();
    }
}
```
## Class Diagram
![[Screenshot 2026-01-04 at 3.47.34 PM.png|700]]
## What We Achieved
- **Dynamic layering:** We can add, remove, or combine decorators at runtime
- **Modular design:** Each decorator is focused on one formatting feature
- **No class explosion:** We avoided creating dozens of subclasses for every combination
- **Open/Closed Principle:** New formatting options can be added without modifying existing classes
- **Highly flexible:** Any combination and ordering of features is now possible
### Example - 2 - Coffee
```Java
abstract class Coffee {
    abstract double cost();
}
class SimpleCoffee extends Coffee {
    double cost() {
        return 50;
    }
}
class MilkCoffee extends SimpleCoffee {
    double cost() {
        return super.cost() + 10;
    }
}
class MilkSugarCoffee extends MilkCoffee {
    double cost() {
        return super.cost() + 5;
    }
}
class Coffee {
    boolean milk;
    boolean sugar;
    boolean cream;

    double cost() {
        double cost = 50;
        if (milk) cost += 10;
        if (sugar) cost += 5;
        if (cream) cost += 15;
        return cost;
    }
}
//------------------------------------------------------
interface Coffee {
    double cost();
    String description();
}
class SimpleCoffee implements Coffee {
    public double cost() {
        return 50;
    }
    public String description() {
        return "Simple Coffee";
    }
}
abstract class CoffeeDecorator implements Coffee {
    protected final Coffee coffee;
    protected CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
}
class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    public double cost() {
        return coffee.cost() + 10;
    }
    public String description() {
        return coffee.description() + ", Milk";
    }
}
class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }
    public double cost() {
        return coffee.cost() + 5;
    }
    public String description() {
        return coffee.description() + ", Sugar";
    }
}
public class Main {
    public static void main(String[] args) {
        Coffee coffee = new SimpleCoffee();
        System.out.println(coffee.cost()); 
        // 50
        coffee = new MilkDecorator(coffee);
        coffee = new SugarDecorator(coffee);
        System.out.println(coffee.description());
        // Simple Coffee, Milk, Sugar
        System.out.println(coffee.cost());
        // 65
    }
}

```
# Composite
- lets you **treat individual objects and compositions of objects uniformly**.
- Allows you to create whole-part hierarchies and deal with both single and group of elements in a consistent way.
## Example - File System

```Java
class File {
    private String name;
    private int size;
    public int getSize() {
        return size;
    }
    public void printStructure(String indent) {
        System.out.println(indent + name);
    }
    public void delete() {
        System.out.println("Deleting file: " + name);
    }
}
class Folder {
    private String name;
    private List<Object> contents = new ArrayList<>();
    public int getSize() {
        int total = 0;
        for (Object item : contents) {
            if (item instanceof File) {
                total += ((File) item).getSize();
            } else if (item instanceof Folder) {
                total += ((Folder) item).getSize();
            }
        }
        return total;
    }
    public void printStructure(String indent) {
        System.out.println(indent + name + "/");
        for (Object item : contents) {
            if (item instanceof File) {
                ((File) item).printStructure(indent + " ");
            } else if (item instanceof Folder) {
                ((Folder) item).printStructure(indent + " ");
            }
        }
    }
    public void delete() {
        for (Object item : contents) {
            if (item instanceof File) {
                ((File) item).delete();
            } else if (item instanceof Folder) {
                ((Folder) item).delete();
            }
        }
        System.out.println("Deleting folder: " + name);
    }
}
```
## Problems
- **Repetitive Type checks** - Repeated use of typeOf -> Breaks encapsulation.
- **No Shared Interface to deal with a group of files and folders**
```java
	List<FileSystemItem> items = List.of(file, folder);
	for (FileSystemItem item : items) {
	    item.delete();
	}
```
- **Violation of OCP**
## Implementing Composite
```Java
interface FileSystemItem {
    int getSize();
    void printStructure(String indent);
    void delete();
}
class File implements FileSystemItem {
    private final String name;
    private final int size;
    public File(String name, int size) {
        this.name = name;
        this.size = size;
    }
    @Override
    public int getSize() {
        return size;
    }
    @Override
    public void printStructure(String indent) {
        System.out.println(indent + "- " + name + " (" + size + " KB)");
    }
    @Override
    public void delete() {
        System.out.println("Deleting file: " + name);
    }
}
class Folder implements FileSystemItem {
    private final String name;
    private final List<FileSystemItem> children = new ArrayList<>();
    public Folder(String name) {
        this.name = name;
    }
    public void addItem(FileSystemItem item) {
        children.add(item);
    }
    @Override
    public int getSize() {
        int total = 0;
        for (FileSystemItem item : children) {
            total += item.getSize();
        }
        return total;
    }
    @Override
    public void printStructure(String indent) {
        System.out.println(indent + "+ " + name + "/");
        for (FileSystemItem item : children) {
            item.printStructure(indent + "  ");
        }
    }
    @Override
    public void delete() {
        for (FileSystemItem item : children) {
            item.delete();
        }
        System.out.println("Deleting folder: " + name);
    }
}
public class FileExplorerApp {
    public static void main(String[] args) {
        FileSystemItem file1 = new File("readme.txt", 5);
        FileSystemItem file2 = new File("photo.jpg", 1500);
        FileSystemItem file3 = new File("data.csv", 300);

        Folder documents = new Folder("Documents");
        documents.addItem(file1);
        documents.addItem(file3);

        Folder pictures = new Folder("Pictures");
        pictures.addItem(file2);

        Folder home = new Folder("Home");
        home.addItem(documents);
        home.addItem(pictures);

        System.out.println("---- File Structure ----");
        home.printStructure("");

        System.out.println("\nTotal Size: " + home.getSize() + " KB");

        System.out.println("\n---- Deleting All ----");
        home.delete();
    }
}
```
## Advantages
- **Unified treatment of both types**
- **Clean recursion:** No `instanceof`, no casting
- **Scalability**
- **Maintainability**
- **Extensibility**
#### How does Composite help with the Open/Closed Principle here?
Tomorrow if you need to add a new file type you can add it by implementing the interface and the client code will still treat it as the same.
## Transparent vs Safe Composite
- Transparent proxy will implement all the methods that are implemented by the whole or part in the interfact.
- Safe composite will implement methods which are particular to part in part and like wise for whole.
- Transparent Composite is useful when you have a client that does not need to know which concrete it is interacting with.
# Proxy
- Provides a **placeholder or surrogate** for another object, allowing you to **CONTROL ACCESS** to it.
## Example - Document
```Java
public interface Document {
    void display();
}
public class RealDocument implements Document {
    private final String id;
    public RealDocument(String id) {
        this.id = id;
        //Expensive
        loadFromRemoteServer();
    }
    private void loadFromRemoteServer() {
        System.out.println("Loading document " + id + " from remote server...");
    }
    @Override
    public void display() {
        System.out.println("Displaying document " + id);
    }
}

public class ImageGalleryAppV1 {
    public static void main(String[] args) {
        System.out.println("Application Started. Initializing documents...");
        Document doc1 = new RealDocument("doc1.jpg");
        Document doc2 = new RealDocument("doc2.png");
        System.out.println("User requests to display " + doc1.getId());
        image1.display();
        System.out.println("\nUser requests to display " + image3.getId());
        image2.display();
        System.out.println("\nApplication finished.");
    }
}
```
## Problems
- **Resource-Intensive Initialization** - Documents are loaded as soon as the page is loaded.
- **No Control Over Access** - 
## Implementing Proxy
```Java
public interface Document {
    void display();
}

class PDFDocument implements Document {
    private final String id;
    public PDFDocument(String id) {
        this.id = id;
    }
    private void loadPDF() {
        System.out.println("Loading PDF document with ID: " + id);
    }
    @Override
    public void display() {
        System.out.println("Displaying PDF document");
    }
}

public class ImageGalleryAppV1 {
    public static void main(String[] args) {
        System.out.println("Application Started. Initializing documents...");
        Document doc1 = new RealDocument("doc1.jpg");
        Document doc2 = new RealDocument("doc2.png");
        System.out.println("User requests to display " + doc1.getId());
        image1.display();
        System.out.println("\nUser requests to display " + image3.getId());
        image2.display();
        System.out.println("\nApplication finished.");
    }
}
// Remote Proxy -> requires concrete class dependency
class RemoteProxy implements Document{
    private Document doc;
    private String id;
    public RemoteProxy(String id){
        this.id = id;
    }
    @Override
    public display(){
    // Another instance of lazy -> Do not implement in constructor or lazy violated.
        String res = RESTClient.fetchDocument(id);
        this.doc = new PDFDocument(res);
        this.doc.display();
    }
}

// Lazy -> Remote -> PDF - Requires concrete dependency
class LazyPDFProxy implements Document{
    private String id;
    private Document doc;
    public LazyPDFProxy(String id){
        this.id = id;
    }
    @Override
    public void display(){
        if(doc==null){
            doc = new RemoteProxy(this.id);
        }
        doc.display();
    }
}
// Secure Proxy
class SecurePDFProxy implements Document{
    private User user;
    private Document doc;
    private AccessStrategy strat;

    public SecurePDFProxy(User user, Document doc, AccessStrategy strat){
        this.user = user;
        this.doc = doc;
        this.strat = strat;
    }

    public void setStrategy(AccessStrategy strat){
        this.strat = strat;
    }

    @Override
    public void display(){
        if(strat.canAccess(user, doc)){
            doc.display();
        }
        else{
            throw new Exception("Unauthorized access");
        }
    }
}

// Smart Proxy
class SmartPDFProxy implements Document{
    private Document doc;
    public SecurePDFProxy(Document doc){
        this.doc = doc;
    }

    @Override
    public void display(){
        //Some logic here
        doc.display();
        //Some logic here
    }
}

Document myDoc = new SecurePDFProxy(user, new SmartPDFProxy( new LazyPDFProxy(id)), strat);
```
### Chaining Proxies
- **Protection -> Smart -> Virtual -> Remote**
```Java
Document doc =
    new ProtectionProxy(
        new CachingDocumentProxy(
            new LoggingDocumentProxy(
                new DocumentProxy(
                    new RemoteProxy("document1")
                )
            )
        ),
        user,
        true
    );
```
## Benefits
- Lazy Loading
- No Code Changes to Real Object
- Reusability
## Extending Functionality
```Java
private boolean checkAccess(String userRole) {
    System.out.println("ProtectionProxy: Checking access for role: " + userRole + " on file: " + fileName);
    // Simulate a basic access control rule
    return "ADMIN".equals(userRole) || !fileName.contains("secret");
}
public void display(String userRole) {
    if (!checkAccess(userRole)) {
        System.out.println("ProtectionProxy: Access denied for " + fileName);
        return;
    }
    if (realImage == null) {
        System.out.println("ImageProxy: Loading image for authorized access...");
        realImage = new HighResolutionImage(fileName);
    }
    realImage.display();
}
```
## When to stop with the Proxy
- If there is a lot of logic in the proxy class, you should move the behavioural methods to decorators(like logging).
## Decorator vs Proxy vs Factory
- Proxy controls access. It is always there between client and the concrete class. Factory just creates and vanishes.
- Decorator is used to extend methods. Proxy is used to control access.
# Bridge
- lets you **decouple an abstraction from its implementation**, allowing the two to vary **independently**.
- Used when you have classes which can be extended in multiple orthogonal directions.
- **Client -> Abstrction -> Implementation**
- So you pass implementation into abstraction.
## Example - Shape Renderer
- a **cross-platform graphics library**. It supports rendering **shapes** like circles and rectangles using different rendering approaches:
	- 🟢 **Vector rendering** – for scalable, resolution-independent output
	- 🔵 **Raster rendering** – for pixel-based output
- So Shape here is a abstraction and Render is an implementation.
```Java
abstract class Shape {
    public abstract void draw();
}
class VectorCircle extends Shape {
    public void draw() {
        System.out.println("Drawing Circle as VECTORS");
    }
}
class RasterCircle extends Shape {
    public void draw() {
        System.out.println("Drawing Circle as PIXELS");
    }
}
class VectorRectangle extends Shape {
    public void draw() {
        System.out.println("Drawing Rectangle as VECTORS");
    }
}
```
```java
public class App {
    public static void main(String[] args) {
        Shape s1 = new VectorCircle();
        Shape s2 = new RasterRectangle();

        s1.draw(); // Drawing Circle as VECTORS
        s2.draw(); // Drawing Rectangle as PIXELS
    }
}
```
## Problems
- **Class Explosion** - If you add a new renderer, you will need to add 2 new classes. Or a shape and a renderer you need to add 5 new classes.
- **Tight Coupling** - shape and rendering logic are tightly coupled.
- **Violates OCP** - New rendering - modify/recreate shape classes.
## Bridge Implementation
```Java
interface Renderer{
	void renderCircle(float radius);
	void renderRectangle(float width, float height);
}
class VectorRenderer implements Renderer {
    @Override
    public void renderCircle(float radius) {
        System.out.println("Drawing a circle of radius " + radius + " using VECTOR rendering.");
    }
    @Override
    public void renderRectangle(float width, float height) {
        System.out.println("Drawing a rectangle " + width + "x" + height + " using VECTOR rendering.");
    }
}
class RasterRenderer implements Renderer {
    @Override
    public void renderCircle(float radius) {
        System.out.println("Drawing pixels for a circle of radius " + radius + " (RASTER).");
    }
    @Override
    public void renderRectangle(float width, float height) {
        System.out.println("Drawing pixels for a rectangle " + width + "x" + height + " (RASTER).");
    }
}

abstract class Shape{
	private Renderer renderer;
	public Shape(Renderer renderer){
		this.renderer = renderer;
	}
	public abstract void draw();
}
class Circle extends Shape{
	private float radius;
	public Circle(Renderer renderer, float radius){
		super(renderer);
		this.radius = radius;
	}
    @Override
    public void draw() {
        renderer.renderCircle(radius);
    }
}
class Rectangle extends Shape {
    private final float width;
    private final float height;

    public Rectangle(Renderer renderer, float width, float height) {
        super(renderer);
        this.width = width;
        this.height = height;
    }
    @Override
    public void draw() {
        renderer.renderRectangle(width, height);
    }
}
public class BridgeDemo {
    public static void main(String[] args) {
        Renderer vector = new VectorRenderer();
        Renderer raster = new RasterRenderer();

        Shape circle1 = new Circle(vector, 5);
        Shape circle2 = new Circle(raster, 5);

        Shape rectangle1 = new Rectangle(vector, 10, 4);
        Shape rectangle2 = new Rectangle(raster, 10, 4);

        circle1.draw();     // Vector
        circle2.draw();     // Raster
        rectangle1.draw();  // Vector
        rectangle2.draw();  // Raster
    }
}
```
## Benefits
- **Decoupled abstractions from implementations:** Shapes and renderers evolve independently
- **Open/Closed compliance:** You can add new renderers or shapes without modifying existing ones
- **No class explosion:** Avoided the need for every shape-renderer subclass
- **Runtime flexibility:** Dynamically switch renderers based on user/device context
- **Clean, extensible design:** Each class has a single responsibility and can be composed as needed
## Class Diagram
![[Screenshot 2026-01-05 at 9.29.53 PM.png|700]]

# Flyweight
- create a **large number of similar objects**, but most of their data is **shared or repeated**.
- Storing all object data individually would result in high memory consumption.
## Example - Word Doc
- You need to render characters on a word doc. Each character is an object and has its own size, font, position etc.
- Naive approach - 
```Java
class CharacterGlyph {
    private char symbol;          // e.g., 'a', 'b', etc.
    private String fontFamily;    // e.g., "Arial"
    private int fontSize;         // e.g., 12
    private String color;         // e.g., "#000000"
    private int x;                // position X
    private int y;                // position Y

    public CharacterGlyph(char symbol, String fontFamily, int fontSize, String color, int x, int y) {
        this.symbol = symbol;
        this.fontFamily = fontFamily;
        this.fontSize = fontSize;
        this.color = color;
        this.x = x;
        this.y = y;
    }

    public void draw() {
        System.out.println("Drawing '" + symbol + "' in " + fontFamily +
            ", size " + fontSize + ", color " + color + " at (" + x + "," + y + ")");
    }
}
```
## Problems
- **High memory usage** - Each charGlyph holds large amount of data.
- **Performance Bottleneck**
- **Poor Scalability**
## Implementing Flyweight
- What we need is similar characters to share the same properties. Suppose you render 'A' in two places you need not use 2 different classes in each place.
- Instead of initializing a class for each position, we will keep the same class and render at different positions.
- For this we will need to pass position to draw(). -> Make an interface with draw(). -> Flyweight.
```Java
interface CharacterFlyweight {
    void draw(int x, int y);
}
class CharacterGlyph implements CharacterFlyweight {
    private final char symbol;
    private final String fontFamily;
    private final int fontSize;
    private final String color;
    public CharacterGlyph(char symbol, String fontFamily, int fontSize, String color) {
        this.symbol = symbol;
        this.fontFamily = fontFamily;
        this.fontSize = fontSize;
        this.color = color;
    }
    @Override
    public void draw(int x, int y) {
        System.out.println("Drawing '" + symbol + "' [Font: " + fontFamily +
            ", Size: " + fontSize + ", Color: " + color + "] at (" + x + "," + y + ")");
    }
}
class CharacterFlyweightFactory {
	private final Map<String,CharacterGlyph> flyweightMap = new HashMap<>();

	public CharacterFlyweight getFlyweight(char symbol, String fontFamily, int fontSize, String color) {
        String key = symbol + fontFamily + fontSize + color;
        flyweightMap.putIfAbsent(key, new CharacterGlyph(symbol, fontFamily, fontSize, color));
        return flyweightMap.get(key);
    }
    public int getFlyweightCount() {
        return flyweightMap.size();
    }
}

class TextEditorClient {
    private final CharacterFlyweightFactory factory = new CharacterFlyweightFactory();
    private final List<RenderedCharacter> document = new ArrayList<>();

    public void addCharacter(char c, int x, int y, String font, int size, String color) {
        CharacterFlyweight glyph = factory.getFlyweight(c, font, size, color);
        document.add(new RenderedCharacter(glyph, x, y));
    }

    public void renderDocument() {
        for (RenderedCharacter rc : document) {
            rc.render();
        }
        System.out.println("Total flyweight objects used: " + factory.getFlyweightCount());
    }

    private static class RenderedCharacter {
        private final CharacterFlyweight glyph;
        private final int x, y;

        public RenderedCharacter(CharacterFlyweight glyph, int x, int y) {
            this.glyph = glyph;
            this.x = x;
            this.y = y;
        }

        public void render() {
            glyph.draw(x, y);
        }
    }
}
public class FlyweightDemo {
    public static void main(String[] args) {
        TextEditorClient editor = new TextEditorClient();
        // Render "Hello" with same style
        String word = "Hello";
        for (int i = 0; i < word.length(); i++) {
            editor.addCharacter(word.charAt(i), 10 + i * 15, 50, "Arial", 14, "#000000");
        }
        // Render "World" with different font and color
        String word2 = "World";
        for (int i = 0; i < word2.length(); i++) {
            editor.addCharacter(word2.charAt(i), 10 + i * 15, 100, "Times New Roman", 14, "#3333FF");
        }
        editor.renderDocument();
    }
}
```
- **RenderedCharacter** is the most important class here. Instead of storing the huge glyph for each position, we now store a small lightweight class at each position.
## Benefits
- **Memory efficiency:** Shared formatting data eliminates duplication
- **Improved performance:** Fewer objects = faster rendering and lower GC pressure
- **Separation of concerns:** Formatting logic and position/context are cleanly separated
- **Reusability:** Glyphs for common characters are reused across the document
- **Scalability:** Can handle thousands of characters with minimal memory footprint

# 

##### Adapter 
-> Works as a bridge between two implementations.
##### Facade
-> Moves complicated interactions between classes to new facade classes.
```Java fold
class OrderFacade{
    private final InventoryService inventoryService;
    private final PricingService pricingService;
    private final PaymentService paymentService;
    private final ShippingService shippingService;
    private final NotificationService notificationService;

    // High level class is depending on low level classes. Violates Dependency Inversion Principle.
    public OrderFacade(){
        this.inventoryService = new InventoryService();
        this.pricingService = new PricingService();
        this.paymentService = new PaymentService();
        this.shippingService = new ShippingService();
        this.notificationService = new NotificationService();
    }

    public void placeOrder(User user, Item item){
        if(inventoryService.checkAvailability(item)){
            inventoryService.reserve(item);
            double price = pricingService.calculatePrice(item, user); 
            String paymentId = paymentService.initiatePayment(user,price);
            if(paymentService.confirmPayment(paymentId)){
                shippingService.createShipment(new Order(user, item));
                notificationService.notifyUser(user, "Your order has been placed successfully.");
            }
        }
    }
}

public class Main(){
    public static void main(String[] args){
        User user = new User();
        Item item = new Item();
        OrderFacade orderFacade = new OrderFacade();
        orderFacade.placeOrder(user, item);
    }
    
}
```
##### Decorator
-> Used when you have a class where you want to add logic before or after the current logic withhout actually changing the class.
-> As you want to change the functions of a class at runtime, you would pass a different class, so both of them should implement a common interface. The Decorator class will be an abstract class which accepts the concrete implementation of the interface. 
-> The interface should thus contain all the methods you want to decorate
-> In the above code the retry logic is the issue. 
**Decorator vs Facade**
- Decorator is used when you have a single class
- Use facade when multiple.
##### Composite
-> Treats wholes and aprts as the same.
-> Favors uniformity so it is difficult to differentiate between the whole and the part.
-> If a problem arises where you have to distinguish between the whole and the part, you can use an empty interface and differentiate using instanceOf. Here instanceOf does not cause a code smell as it is not breaking encapsulation.
 ```Java file:NotificationSender fold
	interface NotificationSender {
	    void send(String message);
	}
	class EmailNotificationSender implements NotificationSender {
	    public void send(String message) {
	        System.out.println("Sending email: " + message);
	    }
	}
	abstract class NotificationDecorator implements NotificationSender {
	    private NotificationSender sender;
	    public NotificationDecorator(NotificationSender sender){
	        this.sender = sender;
	    }
	    public abstract void send(String message);
	}
	class LoggingDecorator extends NotificationDecorator{
	    public LoggingDecorator(NotificationSender sender){
	        super(sender);
	    }
	    @Override
	    public void send(String message){
	        System.out.println("Sending Message....");
	        this.sender.send(message);
	        System.out.println("Message Sent");
	    }
	}
	class EncryptionDecorator extends NotificationDecorator{
	    private Encryptor encryptor;
	    public EncryptionDecorator(NotificationSender sender,Encryptor encryptor){
	        super(sender);
	        this.encryptor = encryptor;
	    }
	    @Override
	    public void send(String message){
	        encryptor.encrypt(message);
	        this.sender.send(message);
	    }
	}	
	class RetryDecorator extends NotificationDecorator{
	    private int retryCount=0;
	    private int notificationTriggerCount = 0;
	    public RetryDecorator(NotificationSender sender,int retryCount){
	        super(sender);
	        this.retryCount = retryCount;
	    }
	    @Override
	    public void send(String message){
	        try{
	            this.sender.send(message);
	        }
	        catch(Error e){
	            this.notificationTriggerCount++;
	            if(notificationTriggerCount > retryCount){
	                return;
	            }
	            this.send(message);
	        }
	    }
	    public void setRetryCount(int retryCount){
	        this.retryCount = retryCount;
	    }
	}
	```
abstract class Employee {
    

abstract class Employee {
	protected String name;
    protected double salary;

    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    public double getSalary() {
        return salary;
    }

    public void printHierarchy(String indent) {
        System.out.println(indent + name + " : " + salary);
    }

    public void add(Employee e) {
        throw new UnsupportedOperationException();
    }
}
class IndividualContributor extends Employee implements CanBeManaged {
    public IndividualContributor(String name, double salary) {
        super(name, salary);
    }
}
class SeniorManager extends Employee implements CanBeManaged {
    protected List<Employee> subordinates = new ArrayList<>();

    public SeniorManager(String name, double salary) {
        super(name, salary);
    }

    @Override
    public void add(Employee e) {
        subordinates.add(e);
    }

    @Override
    public double getSalary() {
        double total = salary;
        for (Employee e : subordinates) {
            total += e.getSalary();
        }
        return total;
    }

    @Override
    public void printHierarchy(String indent) {
        System.out.println(indent + name + " (Senior Manager)");
        for (Employee e : subordinates) {
            e.printHierarchy(indent + "    ");
        }
    }
}
class TeamLead extends Employee {
    private List<Employee> subordinates = new ArrayList<>();

    public TeamLead(String name, double salary) {
        super(name, salary);
    }

    @Override
    public void add(Employee e) {
        if (!(e instanceof CanBeManaged)) {
            throw new IllegalArgumentException("Cannot manage this employee type");
        }
        subordinates.add(e);
    }

    @Override
    public double getSalary() {
        double total = salary;
        for (Employee e : subordinates) {
            total += e.getSalary();
        }
        return total;
    }

    @Override
    public void printHierarchy(String indent) {
        System.out.println(indent + name + " (Team Lead)");
        for (Employee e : subordinates) {
            e.printHierarchy(indent + "    ");
        }
    }
}
```
##### Proxy

