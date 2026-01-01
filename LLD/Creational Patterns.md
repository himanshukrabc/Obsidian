# Singleton
- creational design pattern that guarantees **a class has only one instance** and provides a global point of access to it.
##### Uses
Logging, Connection Pools, Cache Objects etc.
## UML Diagram
![[Screenshot 2025-12-25 at 5.49.30 AM.png|400]]
## Implementations
### Lazy Initialization
- Private consturctor -> object cannot be initialized.
- **Not thread safe**
```Java
class LazySingleton {
    private static LazySingleton instance;
    private LazySingleton() {} // Private consturctor -> object cannot be initialized.
    public static LazySingleton getInstance() {
        if (instance == null) {
            instance = new LazySingleton();
        }
        return instance;
    }
}
```
### Thread Safe Singleton
- Make getInstance() synchronized.
- synchronized is an overkill here. As when initializing, even if the object is initialized multiple times(because of the race condition at instance\=\=null) the code still works. Plus synchronized has enormous overhead.
```Java
class ThreadSafeSingleton {
    private static ThreadSafeSingleton instance;
    private ThreadSafeSingleton() {}
    public static synchronized ThreadSafeSingleton getInstance() {
        if (instance == null) {
            instance = new ThreadSafeSingleton();
        }
        return instance;
    }
}
```
### Double Checked Locking
- If the first check (`instance == null)` passes, we synchronize on the class object.
```Java
class DoubleCheckedSingleton {
    private static volatile DoubleCheckedSingleton instance;
    private DoubleCheckedSingleton() {}
    public static DoubleCheckedSingleton getInstance() {
        // First check (not synchronized)
        if (instance == null) {
            // Synchronize on the class object
            synchronized (DoubleCheckedSingleton.class) {
                // Second check (synchronized)
                if (instance == null) {
                    instance = new DoubleCheckedSingleton();
                }
            }
        }
        return instance;
    }
}
```
### Eager Initialization
- Problem if the instance is never used. It takes up memory and processing time unnecessarily.
- static ensures there is only one instance shared across classes and final ensures that it is never reassigned.
```Java
class EagerSingleton {
    // The single instance, created immediately
    private static final EagerSingleton instance = new EagerSingleton();
    private EagerSingleton() {}
    public static EagerSingleton getInstance() {
        return instance;
    }
}
```
### Bill Pugh Singleton
- Define a class inside the og class.
- When the getInstance() is called, it initializes the inner class and creates the instance of singleton.
```Java
class BillPughSingleton {
    private BillPughSingleton() {}

    // Static inner class that holds the instance
    private static class SingletonHelper {
        private static final BillPughSingleton INSTANCE = new BillPughSingleton();
    }

    public static BillPughSingleton getInstance() {
        return SingletonHelper.INSTANCE;
    }
}
```
### Static Block Initialization
- Similar to eager initialization. However it allows the opportunity to catch any errors during initialization.
- Has same issues as eager initializaiton.
```Java
class StaticBlockSingleton {
    private static StaticBlockSingleton instance;
    private StaticBlockSingleton() {}
    static {
        try {
            instance = new StaticBlockSingleton();
        } catch (Exception e) {
            throw new RuntimeException("Exception occurred in creating singleton instance");
        }
    }
    public static StaticBlockSingleton getInstance() {
        return instance;
    }
}
```
### Enum Singleton
- Singleton is declared as an enum rather than a class. Hence we have only one instance.
```Java
enum EnumSingleton {
    INSTANCE;
    // Public method
    public void doSomething() {
    }
}
```
## Pros and Cons of Singleton Pattern
![[Screenshot 2025-12-25 at 2.35.40 PM.png|1000]]
# Factory Method
- useful in situations where **the exact type of object** **to be created isn't known until runtime.**
- implements **Open/Closed Principle** 
- Allows the main classes to own the workflow while delegating the object creation logic which is most likely to change.
- So the workflow remains safe even if you try to add new cases/options.
- Ideally you want *a single class which would be responsible **only** for object creation.*
## Example - Notification Service.
```java
interface Notification {
    void send(String msg);
}
class EmailNotification implements Notification {
    public void send(String msg) {
        System.out.println("Email: " + msg);
    }
}
class SlackNotification implements Notification {
    public void send(String msg) {
        System.out.println("Slack: " + msg);
    }
}
class NotificationService {
    void send(Notification notification, String msg) {
        notification.send(msg);
    }
}
public class Main {
    public static void main(String[] args) {
        User user = new User(true);

        Notification notification;
        if (user.getPreference()=="slack") {
            notification = new SlackNotification();
        } else {
            notification = new EmailNotification();
        }

        NotificationService service = new NotificationService();
        service.send(notification, "Hello");
    }
}
```
- The problem here is the creation logic is embedded in the main flow. Now, if any new notification service is added(say Whatsapp notification), you need to modify the main flow.
### Solution 1 -
```Java
class NotificationService {
    void notify(User user, String msg) {
        Notification notification;

        if (user.prefersSlack()) {
            notification = new SlackNotification();
        } else {
            notification = new EmailNotification();
        }

        notification.send(msg);
    }
}
public class Main {
    public static void main(String[] args) {
        User user = new User(true);
        NotificationService service = new NotificationService();
        service.notify(user, "Hello");
    }
}
```
- The NotificationService is dependent on concrete classes. Adding Whatsapp will modify this class.
### Simple Factory
```Java
class NotificationFactory {
    Notification create(User user) {
        if (user.prefersSlack()) {
            return new SlackNotification();
        }
        return new EmailNotification();
    }
}
class NotificationService {
    private final NotificationFactory factory;

    NotificationService(NotificationFactory factory) {
        this.factory = factory;
    }

    void notify(User user, String msg) {
        Notification notification = factory.create(user);
        notification.send(msg);
    }
}
public class Main {
    public static void main(String[] args) {
        NotificationFactory factory = new NotificationFactory();
        NotificationService service = new NotificationService(factory);

        User user = new User(true);
        service.notify(user, "Hello");
    }
}
```
- Now the NotificationFactory class is responsible for object creation and only this class has to be modified for adding new notification service.
- However this still violates the Open Close Principle. -> The NotificationFactory class needs to change.
### Factory Method
- Instead of using the SimpleFactory to create objects, you delegate the job to new creator classes.
```Java
abstract class NotificationCreator {
    // Factory Method
    public abstract Notification createNotification();

    // Common logic using the factory method
    public void send(String message) {
        Notification notification = createNotification();
        notification.send(message);
    }
}
class EmailNotificationCreator extends NotificationCreator {
    @Override
    public Notification createNotification() {
        return new EmailNotification();
    }
}
class SMSNotificationCreator extends NotificationCreator {
    @Override
    public Notification createNotification() {
        return new SMSNotification();
    }
}
public class Main {
    public static void main(String[] args) {
        User user = new User(true);

        NotificationService service;
        if (user.prefersSlack()) {
            service = new SlackNotificationService();
        } else {
            service = new EmailNotificationService();
        }

        service.notify("Hello");
    }
}
```
## Class Diagram
![[Screenshot 2025-12-25 at 10.58.52 PM.png|900]]
1. **Product (e.g., Notification):** An interface or abstract class for the objects the factory method creates.
2. **ConcreteProduct (e.g., EmailNotification, SMSNotification):** Concrete classes that implement the Product interface.
3. **Creator (e.g., NotificationCreator):** An abstract class (or an interface) that declares the factory method, which returns an object of type Product. It might also define a default implementation of the factory method. The Creator can also have other methods that use the product created by the factory method.
4. **ConcreteCreator (e.g., EmailNotificationCreator, SMSNotificationCreator):** Subclasses that override the factory method to return an instance of a specific ConcreteProduct.
# Abstract Factory
### Where the Factory Method Breaks
- Suppose instead of just the notification object, you had to create multiple objects. Eg - 
	- Email → SMTP client + Email formatter
	- Slack → Slack API client + Slack formatter
	- SMS -> SMS Formatter
- In this case the creator interface will have multiple factory methods and for some of the concrete creators will not have all the methods implemented.
- Any new addition will modify the class -> **violation of the Open Close Principle**
```Java
abstract class NotificationService {
    public final void notify(String msg) {
        Notification n = createNotification();
        Formatter f = createFormatter();
        Client c = createClient(); 
        n.send(f.format(msg), c);
    }
    protected abstract Notification createNotification();
    protected abstract Formatter createFormatter();
    protected abstract Client createClient();
}

class EmailService extends NotificationService {
    protected Notification createNotification() {
        return new EmailNotification();
    }
    protected Formatter createFormatter() {
        return new EmailFormatter();
    }
    protected Client createClient() {
        return new SmtpClient();
    }
}

class SmsService extends NotificationService {
    protected Notification createNotification() {
        return new SmsNotification();
    }
    protected Formatter createFormatter() {
        return new PlainTextFormatter();
    }
    protected Client createClient() {
        return null;
    }
}
```
#### Issues with Factory Method
- Base class keeps growing.
- Subclasses forced to implement methods which are not required -> return null/error. -> Violates ISP.
### Abstract Factory
```Java
interface NotificationFactory {
    Notification createNotification();
    Formatter createFormatter();
    Client createClient();
}
class EmailFactory implements NotificationFactory {
    public Notification createNotification() {
        return new EmailNotification();
    }
    public Formatter createFormatter() {
        return new EmailFormatter();
    }
    public Client createClient() {
        return new SmtpClient();
    }
}
class SlackFactory implements NotificationFactory {
    public Notification createNotification() {
        return new SlackNotification();
    }
    public Formatter createFormatter() {
        return new SlackFormatter();
    }
    public Client createClient() {
        return new SlackApiClient();
    }
}
class SmsFactory implements NotificationFactory {
    public Notification createNotification() {
        return new SmsNotification();
    }
    public Formatter createFormatter() {
        return new PlainTextFormatter();
    }
    public Client createClient() {
        return null;
    }
}

class NotificationService {

    private final NotificationFactory factory;

    NotificationService(NotificationFactory factory) {
        this.factory = factory;
    }

    void notify(String msg) {
        Notification n = factory.createNotification();
        Formatter f = factory.createFormatter();
        Client c = factory.createClient();
        n.send(f.format(msg), c);
    }
}
```
- The pro here is **Notification service no longer changes**. Notification service class is part of core logic.
### When to use abstract factory?
- If you need a single class that varies across ClassTypes, use factoryMethod.
- If you need a family of products that varies across ClassTypes, use Abstract Factory.
### Class Diagram
![[Screenshot 2026-01-01 at 4.32.55 PM.png|700]]
- AbstractFactory -> NotificationFactory
  ConcreteFactories -> SmsFactory, SlackFactory, EmailFactory
  AbstractProducts -> Notification, Formatter, Client
  ConcreteProducts -> EmailNotification, EmailFormatter, EnmailClient etc.
# Builder
- Used when you have 
	- a class with constructor having optional parameters
	- To create an object, you need to follow certain steps.
- Usually this is done using constructors with many parameters or setters for every field. This is error prone and violates Single Responsibility Principle.
## Example - HttpRequestObject
```Java
class HttpRequestTelescoping {
    private String url;
    private String method;
    private Map<String, String> headers;
    private Map<String, String> queryParams;
    private String body;
    private int timeout;
    public HttpRequestTelescoping(String url) {
        this(url, "GET");
    }
    public HttpRequestTelescoping(String url, String method) {
        this(url, method, null);
    }
    public HttpRequestTelescoping(String url, String method, Map<String, String> headers) {
        this(url, method, headers, null);
    }
    public HttpRequestTelescoping(String url, String method, Map<String, String> headers,
                                  Map<String, String> queryParams) {
        this(url, method, headers, queryParams, null);
    }
    public HttpRequestTelescoping(String url, String method, Map<String, String> headers,
                                  Map<String, String> queryParams, String body) {
        this(url, method, headers, queryParams, body, 30000);
    }
    public HttpRequestTelescoping(String url, String method, Map<String, String> headers,
                                  Map<String, String> queryParams, String body, int timeout) {
        this.url = url;
        this.method = method;
        this.headers = headers == null ? new HashMap<>() : headers;
        this.queryParams = queryParams == null ? new HashMap<>() : queryParams;
        this.body = body;
        this.timeout = timeout;

        System.out.println("HttpRequest Created: URL=" + url +
            ", Method=" + method +
            ", Headers=" + this.headers.size() +
            ", Params=" + this.queryParams.size() +
            ", Body=" + (body != null) +
            ", Timeout=" + timeout);
    }
    // Getters could be added here
}
```
### Problems with this - 
- **Low Readability**
- **Error Prone** - Easy to mix up parameters.
- **Forced to pass null**.
- **Not Scalable**
## Example with Builder implementation
```Java
class HttpRequest {
    // Required
    private final String url;
    private final String method;
    private final Map<String, String> headers;
    private final Map<String, String> queryParams;
    private final String body;
    private final int timeout;
    // Private constructor
    private HttpRequest(Builder builder) {
        this.url = builder.url;
        this.method = builder.method;
        this.headers = builder.headers;
        this.queryParams = builder.queryParams;
        this.body = builder.body;
        this.timeout = builder.timeout;
    }
    // Getters (optional)
    public String getUrl() { return url; }
    public String getMethod() { return method; }
    public Map<String, String> getHeaders() { return headers; }
    public Map<String, String> getQueryParams() { return queryParams; }
    public String getBody() { return body; }
    public int getTimeout() { return timeout; }
    @Override
    public String toString() {
        return "HttpRequest{" +
               "url='" + url + '\'' +
               ", method='" + method + '\'' +
               ", headers=" + headers +
               ", queryParams=" + queryParams +
               ", body='" + body + '\'' +
               ", timeout=" + timeout +
               '}';
    }
    
	public static class Builder {
        private final String url; // required
        private String method = "GET";
        private Map<String, String> headers = new HashMap<>();
        private Map<String, String> queryParams = new HashMap<>();
        private String body;
        private int timeout = 30000;
        public Builder(String url) {
            this.url = url;
        }
        public Builder method(String method) {
            this.method = method;
            return this;
        }
        public Builder addHeader(String key, String value) {
            this.headers.put(key, value);
            return this;
        }
        public Builder addQueryParam(String key, String value) {
            this.queryParams.put(key, value);
            return this;
        }
        public Builder body(String body) {
            this.body = body;
            return this;
        }
        public Builder timeout(int timeout) {
            this.timeout = timeout;
            return this;
        }
        public HttpRequest build() {
            return new HttpRequest(this);
        }
    }
}


public class HttpAppBuilderPattern {
    public static void main(String[] args) {
        HttpRequest request1 = new HttpRequest.Builder("https://api.example.com/data")
            .build();

        HttpRequest request2 = new HttpRequest.Builder("https://api.example.com/submit")
            .method("POST")
            .body("{\"key\":\"value\"}")
            .timeout(15000)
            .build();

        HttpRequest request3 = new HttpRequest.Builder("https://api.example.com/config")
            .method("PUT")
            .addHeader("X-API-Key", "secret")
            .addQueryParam("env", "prod")
            .body("config_payload")
            .timeout(5000)
            .build();

        System.out.println(request1);
        System.out.println(request2);
        System.out.println(request3);
    }
}
```

## Class Diagram
![[Screenshot 2026-01-01 at 5.21.03 PM.png|700]]
# Prototype
- Lets you create new objects by **cloning existing ones**.
- Default approach -> create new obj, manually copy fields over.
- Problems -
	- **Encapsulation** - some fields are not accessible.
	- **Concrete Dependencies** - Need to know the concrete class of the object if you used abstract classes.
### Example
```Java
interface EnemyPrototype {
    EnemyPrototype clone();
}
class Enemy implements EnemyPrototype {
    private String type;
    private int health;
    private double speed;
    private boolean armored;
    private String weapon;
    public Enemy(String type, int health, double speed, boolean armored, String weapon) {
        this.type = type;
        this.health = health;
        this.speed = speed;
        this.armored = armored;
        this.weapon = weapon;
    }
    @Override
    public Enemy clone() {
        return new Enemy(type, health, speed, armored, weapon);
    }
    public void setHealth(int health) {
        this.health = health;
    }
    public void printStats() {
        System.out.println(type + " [Health: " + health +
                           ", Speed: " + speed +
                           ", Armored: " + armored +
                           ", Weapon: " + weapon + "]");
    }
}
```
### Creating a Prototype Registry
```Java
class EnemyRegistry {
    private Map<String, Enemy> prototypes = new HashMap<>();

    public void register(String key, Enemy prototype) {
        prototypes.put(key, prototype);
    }

    public Enemy get(String key) {
        Enemy prototype = prototypes.get(key);
        if (prototype != null) {
            return prototype.clone();
        }
        throw new IllegalArgumentException("No prototype registered for: " + key);
    }
}
public class Game {
    public static void main(String[] args) {
        EnemyRegistry registry = new EnemyRegistry();

        // Register prototype enemies
        registry.register("flying", new Enemy("FlyingEnemy", 100, 12.0, false, "Laser"));
        registry.register("armored", new Enemy("ArmoredEnemy", 300, 6.0, true, "Cannon"));

        // Clone from registry
        Enemy e1 = registry.get("flying");
        Enemy e2 = registry.get("flying");
        e2.setHealth(80); // maybe this one was spawned with less HP

        Enemy e3 = registry.get("armored");

        // Print stats to verify
        e1.printStats();
        e2.printStats();
        e3.printStats();
    }
}
```