A UML class diagram consists of the following building blocks:
## Class Diagram
### Visibility
**+** - Public
**-** - Private
**~** - Package
**#** - Protected
### Mulitplicity
**1** - Exactly 1
**0..1** - 0 or 1
**_ * -** 0 or more
**1..* -** 1 or more
### Sections
##### Name
##### Attributes
- visibility name : type \[multiplicity\] = default value
![[Screenshot 2025-12-22 at 11.32.17 PM.png|400]]
##### Method
- visibility name(parameterList): returnType
![[Screenshot 2025-12-22 at 11.32.47 PM.png|400]]
#### Interfaces
- <<interface\>\>  above name. There are no attributes.
![[Screenshot 2025-12-22 at 11.33.29 PM.png|160]]
#### Abstract Class
- Can have attributes and functions. Add <<abstract\>\> above Name.
![[Screenshot 2025-12-22 at 11.35.11 PM.png|200]]
#### Enumeration
- «enumeration» above name.
![[Screenshot 2025-12-22 at 11.36.48 PM.png|180]]
### Relations
#### 1. Association
![[Screenshot 2025-12-23 at 6.06.39 PM.png|500]]
#### 2. Aggregation
![[Screenshot 2025-12-23 at 6.07.12 PM.png|500]]
#### 3. Composition
![[Screenshot 2025-12-23 at 6.07.54 PM.png|200]]
#### 4. Inheritance
![[Screenshot 2025-12-23 at 6.08.40 PM.png|400]]
#### 5. Realization(Implementation)
![[Screenshot 2025-12-23 at 6.09.30 PM.png|400]]
#### 6. Dependency
![[Screenshot 2025-12-23 at 6.09.58 PM.png|300]]
### Example
![[Screenshot 2025-12-23 at 6.11.05 PM.png]]
## Use Case Diagram
- Indicates how different users will interact with our system.
#### 1. Actors
- An actor represents anything that interacts with the system from outside.
- There are two types of actors:
	- **Primary actors**: Initiates an interaction (e.g., a user logging in)
	- **Secondary actors**: Helps fulfill a use case but don't initiate it (e.g., a payment gateway)
#### 2. Use Cases
- A **use case** is a **functionality or goal** that the system provides to the actor.
- **Notation**: Oval with the name of the use case inside.
![[Screenshot 2025-12-23 at 7.29.22 PM.png|300]]
- Each use case should:
	- Start with a verb (e.g., "Register", "Search", "Book Ticket", “Make Payment“)
	- Represent a complete interaction from the user's point of view
	- Deliver a meaningful result
#### 3. System Boundary
- The **system boundary** defines **what’s inside the system and what’s outside**. This helps clearly define scope.
- **Notation**: A labeled box that encloses al the use cases
![[Screenshot 2025-12-23 at 7.30.26 PM.png|200]]
#### 4. Relationships
Relationships describe how actors and use cases are connected or how different use cases relate to one another. 
There are four main types:
##### a. Association
- **Notation:** Represented by a solid line
![[Screenshot 2025-12-24 at 7.06.22 PM.png|500]]
##### b. Include
- Represents **common functionality** shared between use cases. Think of it as: "Always includes this"
- Example: `Checkout` includes `Validate Payment`
- **Notation**: Dashed arrow with label `<<include>>`

![[Screenshot 2025-12-24 at 7.07.08 PM.png|500]]
##### c. Extend
- Represents **optional or conditional behavior**. Think of it as: "Sometimes adds this"
- Example: `Search` can extend to `Advanced Filter`
- **Notation**: Dashed arrow with label `<<extend>>`
![[Screenshot 2025-12-24 at 7.08.01 PM.png|500]]
##### d. Generalization
- Shows inheritance between actors or use cases. The child actor/use case is an enhancement of the parent use case.
- Example: `Admin` is a specialized `User`
- **Notation**: Directed arrow with a triangle arrowhead from child to parent
![[Screenshot 2025-12-24 at 7.08.57 PM.png|400]]
### 3. How to Draw a Use Case Diagram (Ticket booking system)

#### Identify Actors
- Human users (e.g., Customer, Admin)
- External systems (e.g., Payment Gateway, Notification Service)
- For our example - *Customer*, *Admin*(movie lister), *Payment Gateway*
  ![[Screenshot 2025-12-24 at 7.14.01 PM.png|400]]
#### Identify Use Cases
- List out what the actors want to do, i.e, use cases. These are from the actor's perspective.
- Use clear, action-oriented.
For the movie booking system:
- Browse Movies
- Book Ticket
- Cancel Booking
- Make Payment
- Add/Edit Movie Listings
#### Define the System Boundary
- Draw a box around all the use cases. 
- For our example: Movie Ticket Booking System
![[Screenshot 2025-12-24 at 7.14.30 PM.png|300]]
#### Connect Actors to Use Cases
- Link each actor to the relevant use cases using solid lines (associations).
	- The Customer is connected to most of the features
	- The Admin is only connected to the movie management use case
	- The Payment Gateway interacts only with the payment flow
#### Model Relationships Between Use Cases
##### a. Include
This means a use case always includes another use case.
It helps you:
- Reuse common functionality across multiple use cases
- Keep your diagram DRY (Don’t Repeat Yourself)
Example: Whenever someone books a ticket, they must make a payment.
![[Screenshot 2025-12-24 at 7.15.09 PM.png|500]]
##### b. Extend
Used when a use case has optional or conditional behavior.
Example: While browsing movies, the user might choose to filter by genre—but it's not mandatory.
![[Screenshot 2025-12-24 at 7.15.33 PM.png|500]]
##### c. Generalization
Use when actors or use cases share common behavior but differ slightly.
Example: A `Registered User` and `Guest User` both act like a `Customer`, but with slight differences. Use generalization to reflect that.
![[Screenshot 2025-12-24 at 7.15.52 PM.png|300]]
![[Screenshot 2025-12-24 at 7.16.08 PM.png|900]]

## Sequence Diagram
- **Map out interactions between objects over time** -> a storyboard for how things happen in a system.
- **order of messages exchanged** between different components or actors to achieve a particular task or use case.
### Building Blocks of Sequence Diagram
#### Actors
- External Entities(User or External Systems)
#### Objects(Participants)
- Each object is represented in a rectangle.
#### Lifeline
- Vertical dashed line representing the lifetime of an object or actor.
 ![[Screenshot 2025-12-24 at 7.31.22 PM.png|300]]
#### Activation Bars
- Rectangular bars on lifelines which state when the object is processing a message.
### Types of messages in sequence diagrams
#### Synchronous Messages
- Sender sends a message and waits for receiver's response.
- Solid line with a solid arrow head.
![[Screenshot 2025-12-24 at 7.34.35 PM.png|300]]
#### Asynchronous Messages
- Sender sends a message and does not wait for the response.
- Solid line with an arrow.
![[Screenshot 2025-12-24 at 7.35.40 PM.png|400]]
#### Return Message
- Reciever's response
- Dashed line with arrow head.
![[Screenshot 2025-12-24 at 7.36.22 PM.png|400]]
#### Self-Message
- Object calls on its own method
![[Screenshot 2025-12-24 at 7.37.31 PM.png|300]]
#### Create Message
- Marks the creation of a new object.
- ``<<create>>``  on a solid line. Below is the method name.
![[Screenshot 2025-12-24 at 7.38.12 PM.png|300]]
#### Destroy Message
- Optional. Marks the destruction of an object.
- Solid line marked with **X**.
![[Screenshot 2025-12-24 at 7.40.10 PM.png|200]]
## Activity Diagram
- An Activity Diagram models the workflow of a process, showing the sequence of activities, decision points, parallel execution, and the flow of control from start to finish.
### Components of Activity Diagram
- **Initial Node (Start):** A solid black circle that indicates where the workflow begins.
- **Action / Activity:** A rounded rectangle representing a specific task or step in the process.
- **Control Flow:** Solid arrows that show the direction of the process from one step to the next.
- **Decision Node:** A diamond shape where the path splits based on a condition (e.g., "Yes/No").
- **Merge Node:** A diamond shape where multiple alternative paths come back together.
- **Fork:** A thick horizontal or vertical bar used to split one path into multiple **parallel** (concurrent) activities.
- **Join:** A thick bar used to synchronize multiple parallel paths back into one.
- **Final Node (End):** A "bullseye" (solid circle inside a hollow circle) marking the completion of the entire process.
### Control Flow
- **Sequential Flow:** Activities execute one after another
- **Conditional Flow:** Flow takes different paths based on a condition.
- **Parallel Flow:** Multiple activities execute simultaneously.
- **Loop Flow:** Activities repeat until a condition is met.
### Swimlanes (Partitions)
Swimlanes divide the diagram into columns or rows, showing **who** is responsible for each activity.
**Benefits of Swimlanes:**
- Clear responsibility assignment
- Easy to see handoffs between actors
- Identifies integration points
- Helps spot bottlenecks	
![[Screenshot 2025-12-25 at 3.52.03 AM 1.png|600]]