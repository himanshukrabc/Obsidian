## Association
- Reflects a is-a or has-a relationship.
- One object uses/**references** another.
- Objects are **loosely coupled** and can exist **independent of each other**.
- ==Example== - 
  Student **has a** Teacher. Teacher **has a** Student. Teacher and Student both can exist without each other.

### UML Representation
- **Solid line (—):** Represents an association between classes.
##### Directionality
- **Arrowhead (→):** Indicates **directionality** (who knows whom).
- **No arrowhead:** Implies a **bidirectional** association.
##### Multiplicity
- Multiplicity defines **how many instances** of one class can be associated with another. 
- It is written near the class ends in UML diagrams

| **Symbol** | **Meaning**            | **Example Scenario**                     |
| ---------- | ---------------------- | ---------------------------------------- |
| `1`        | Exactly one            | Each `User` has one `Profile`            |
| `0..1`     | Zero or one (optional) | An `Employee` may have a `Manager`       |
| `*`        | Many (zero or more)    | A `Project` can have many `Tasks`        |
| `1..*`     | At least one           | Each `Course` has one or more `Students` |

### Types of Association 
#### Based on Directionality
##### Unidirectional
- Only one object is aware of/**references** the other.
- Order object uses a Payment Gateway.
![[Screenshot 2025-12-07 at 6.56.52 PM.png|700]]
```Java
class PaymentGateway {
    void processPayment(double amount) {}
}
class Order {
    private PaymentGateway gateway;
    public Order(PaymentGateway gateway) {
        this.gateway = gateway;
    }
    public void checkout() {
        gateway.processPayment(100.0);
    }
}
````
##### Bidirectional
- Both classes are aware of each other.
![[Screenshot 2025-12-07 at 7.11.02 PM.png|700]]
```java
class Issue {
    private Project project;
    public void setProject(Project project) {
        this.project = project;
    }
}
class Project {
    private List<Issue> issues = new ArrayList<>();
    public void addIssue(Issue issue) {
        issues.add(issue);
        issue.setProject(this);
    }
}
```

#### Based on Multiplicity
##### One to One
- Each object can be linked to exactly one object of another class.
![[Screenshot 2025-12-07 at 7.09.16 PM.png|700]]
```java
class Profile {
    private User user;
    public void setUser(User user) {
        this.user = user;
    }
}
class User {
    private Profile profile;

    public void setProfile(Profile profile) {
        this.profile = profile;
        profile.setUser(this);
    }
}
```
##### One to Many
- One object is referenced by many objects of another class.
  ![[Screenshot 2025-12-07 at 7.11.02 PM.png|700]]
```java
class Issue {
    private Project project;
    public void setProject(Project project) {
        this.project = project;
    }
}
class Project {
    private List<Issue> issues = new ArrayList<>();
    public void addIssue(Issue issue) {
        issues.add(issue);
        issue.setProject(this);
    }
}
```
##### Many to Many
- Multiple objects from one class are associated with **multiple objects** from another class.
![[Screenshot 2025-12-07 at 7.11.48 PM.png|700]]

```java
class User {
    private List<Group> groups = new ArrayList<>();
}
class Group {
    private List<User> users = new ArrayList<>();
}
```

## Aggregation
- Aggregation is a **weaker form of the whole–part relationship**
- Whole class contains references to other part classes, but the parts can **exist independently** of the whole.
- The **whole** does not own the **part**. **Part** can be shared.
- Eg - Book and Library. Book is a part of library but some books can still exist without the library.
### UML Representation
- Uses a **hollow diamond** on the end of a straight line. Diamond is on the side of the **whole**.
![[Screenshot 2025-12-07 at 7.30.30 PM.png|700]]
- Dev is a part of a team but can still exist without the Team.
### Why Aggregation?
- **Promotes Reusability:** 
  "Part" components (like a `Developer` or a `Microservice`) are independent 
  can be reused across multiple "whole" objects (`Team`s or `ApiGateway`s).
- **Improves Flexibility:** 
  The relationship is loose, which **reduces coupling** between classes. 
  You can modify the `Team` class without affecting the `Developer` class, and vice versa.
- **Reflects Real-World Relationships:** Many real-world systems (teams, projects, organizations) naturally exhibit aggregation
### Bad → Good → Great Example:
#### Bad
- A `Team` class has a method `createNewDeveloper()`, creating and destroying `Developer` objects internally. This creates tight coupling, making it behave like composition.
 ```Java
 class Developer {
    private String name;
    public Developer(String name) { this.name = name; }
}
class Team {
    private List<Developer> developers = new ArrayList<>();
    // BAD: Team is responsible for creating Developers
    public void createNewDeveloper(String name) {
        Developer dev = new Developer(name);
        developers.add(dev);
    }
    // BAD: Team might internally delete Developers
    public void removeDeveloper(Developer dev) {
        developers.remove(dev);  // Team controls lifecycle
    }
}
```
#### Good
- A `Team` class holds a reference to `Developer` instances that are created elsewhere and passed to it. This is standard aggregation.
```Java
class Developer {
    private String name;
    public Developer(String name) { this.name = name; }
}
class Team {
    private List<Developer> developers = new ArrayList<>();
    // GOOD: Developer is created outside and passed in
    public void addDeveloper(Developer developer) {
        developers.add(developer);
    }
    public List<Developer> getDevelopers() {
        return developers;
    }
}
// Somewhere else:
Developer d1 = new Developer("Alice");
Developer d2 = new Developer("Bob");

Team team = new Team();
team.addDeveloper(d1);
team.addDeveloper(d2);
```
#### Great
- A `Team`'s dependencies (the list of `Developer`s) are provided via its constructor or a setter method (**Dependency Injection**). 
- This is the most flexible approach, promoting high modularity and making the `Team` class easy to test with mock `Developer` objects.
~~~Java
class Developer {
    private String name;
    public Developer(String name) { this.name = name; }
}
class Team {
    private final List<Developer> developers;
    // GREAT: Developers injected via constructor (Dependency Injection)
    public Team(List<Developer> developers) {
        this.developers = developers;
    }
    public List<Developer> getDevelopers() {
        return developers;
    }
}
// Example usage:
List<Developer> initialDevs = List.of(
    new Developer("Alice"),
    new Developer("Bob")
);
Team team = new Team(initialDevs);
~~~
## Composition
- **Part-whole** relationship where the part cannot exist without the whole.
- Whole class has ownership and determines the life cycle of the part.
- Parts are not shared across different classes.
- Example - A `House` is made of `Rooms` and `Rooms` cannot exist without a `House`.
### UML Representation
- Uses a **solid diamond** on the end of a straight line. Diamond is on the side of the **whole**.
![[Screenshot 2025-12-08 at 6.35.49 PM.png|700]]
#### Code Example
```java
class Room {
    private String name;
    public Room(String name) {
        this.name = name;
    }
    public void describe() {
        System.out.println("This is the " + name);
    }
}
class House {
    private List<Room> rooms;
    public House() {
        rooms = new ArrayList<>();
        rooms.add(new Room("Living Room"));
        rooms.add(new Room("Kitchen"));
        rooms.add(new Room("Bedroom"));
    }
    public void showHouseLayout() {
        for (Room room : rooms) {
            room.describe();
        }
    }
}
```
- The `House` **creates**, **manages**, and **owns** its `Room` objects.
- The `Room` objects **do not exist independently** outside the context of the `House`.
- No external class should reuse or manage these `Room` instances.
- If the `House` is deleted (e.g., garbage collected), the `Room`s are destroyed too.
### Composition vs Inheritance
- Inheritance is inflexible. Classes need to predefine which classes it has access to.
- Composition is flexible. Classes can be composed at runtime by using abstract classes.
```Java
class Car{
	Engine e;
	public Car(Engine e){
		this.e = e;
	}
}
Engine e1 = new PetrolEngine();
Engine e2 = new DieselEngine();
Car petrolCar = new Car(e1);
Car dieselCar = new Car(e1);
```
- If we used Inheritance we would have to define 2 classes called petrolCar and dieselCar who would inherit from petrolEngine and dieselEngine respectively.
### Comparison

| Feature          | **Association**          | **Aggregation**               | **Composition**                    |
| ---------------- | ------------------------ | ----------------------------- | ---------------------------------- |
| **Ownership**    | ❌ None                   | ❌ Weak — has-a                | ✅ Strong — owns-a                  |
| **Lifecycle**    | ❌ Independent            | ❌ Independent                 | ✅ Dependent — part dies with whole |
| **Tightness**    | Loose coupling           | Moderate coupling             | Tight coupling                     |
| **Multiplicity** | Flexible (1:1, 1:N, N:N) | Whole can group many parts    | Whole composed of integral parts   |
| **Reusability**  | High — parts reusable    | Moderate — parts often reused | Low — parts not reused outside     |
| **UML Symbol**   | Solid Line               | Hollow Diamond (◊)            | Filled Diamond (◆)                 |
- **Association** is a general connection: two classes simply know about each other.
- **Aggregation** is a _grouping:_ the whole and parts can exist independently.
- **Composition** is an _ownership:_ the part’s existence is bound to the whole.
## Dependencies
- A **Dependency** exists when **one class relies on another**, *without retaining a permanent reference* to it.
- This typically happens when:
	- A class **accepts another class as a method parameter**.
	- A class **instantiates or uses another class inside a method**.
	- A class **returns an object of another class** from a method.
### Key Characteristics of Dependency
- **Short-lived** - only during method execution
- **No ownership** - does **not store** the class in a state
- **"Uses-a" relationship** - The other class is used to **accomplish a task**, but does not retain it.
- Example - Chef uses a Knife but does not own it. The knife is used when chopping and then returned.
### UML Diagram
![[Screenshot 2025-12-08 at 6.54.04 PM.png|700]]
- `Printer` temporarily **uses** `Document`, but **does not own** or **associate** with it in a structural sense.
### Code Example
```java
class Document {
    private String content;
    public Document(String content) {
        this.content = content;
    }
    public String getContent() {
        return content;
    }
}
class Printer {
    public void print(Document document) { // dependency: Document
        System.out.println("Printing: " + document.getContent());
    }
}
```
### Dependency Injection
- Some methods may require some other classes. These are dependencies.
- Storing these dependencies as states creates a tight coupling. Makes code inflexible and difficult to test.
- **DI** - pass the dependency as a parameter to the method/constructor.
```Java
interface Sender {
    void send(String message);
}
class NotificationService {
    private final Sender sender; // Interface
    public NotificationService(Sender sender) {
        this.sender = sender; // Injected dependency
    }
    public void notifyUser(String message) {
        sender.send(message); // Uses sender temporarily
    }
}
```
- **Sender** class is received as a parameter. 
- **NotificationService** does not care _how_ messages are sent, it just *depends on* a **Sender** interface.