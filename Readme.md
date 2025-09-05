# Ultimate Design Patterns

This is a comprehensive summary of a course on design patterns. Design patterns are proven, reusable solutions to common problems in software design. They help make code more flexible, maintainable, and scalable. We'll start with the SOLID principles, which are foundational guidelines for object-oriented design, then briefly touch on UML (Unified Modeling Language) for visualizing designs, and proceed to explore various behavioral, structural, and creational patterns.

# SOLID

SOLID is an acronym for five key principles of object-oriented programming and design, introduced by Robert C. Martin (Uncle Bob). These are not rigid rules but awesome guideline principles that promote code that's easier to understand, extend, and refactor. They reduce bugs, improve reusability, and make systems more robust to change. Violating them often leads to "spaghetti code" that's tightly coupled and hard to maintain.

## Single Responsibility Principle

**A class should have only one reason to change.**

This principle emphasizes that a class should focus on a single responsibility or concern. I personally think the term "one reason to change" is better than the common paraphrase "a class should do only one thing" because the major advantage of design patterns is to ease refactoring. The phrase "one reason to change" highlights that when requirements evolve (e.g., due to business needs), only one part of the system should need modification, minimizing ripple effects.

In case of violation, refactoring becomes painful: you'll have to change many things because the class now manages a lot of state and is very vulnerable to high coupling of methods, even if they are separated. As long as many functionalities live within the class scope, it becomes a violation—no matter if they are in separated methods or not. This leads to "god classes" that do everything, making testing harder and increasing the risk of bugs.

Pros: Easier testing, lower coupling, better organization.  
Cons: Can lead to more classes if over-applied (class explosion).  
When to use: In services or models where multiple concerns (e.g., data access, logging, and business logic) are mixed.  
Real-world analogy: A chef in a restaurant shouldn't also handle accounting and cleaning; each role has one responsibility.

### TS Example

This is a violation (the `UserService` handles user creation, email sending, and logging—all in one class, leading to tight coupling):

```ts
class UserService {
  createUser(name: string, email: string): void {
    console.log(`User ${name} created.`);

    this.sendEmail(email, "Welcome!", "Thank you for signing up!");
    this.logActivity(name, "User account created.");
  }

  private sendEmail(to: string, subject: string, body: string): void {
    console.log(`Sending email to ${to}: ${subject} - ${body}`);
  }

  private logActivity(user: string, action: string): void {
    console.log(`Logging activity: ${user} - ${action}`);
  }
}

const userService = new UserService();
userService.createUser("Alice", "alice@example.com");
```

This is the proper application (responsibilities are separated into dedicated services, injected via dependency injection for loose coupling; now, changing email logic doesn't affect user creation):

```ts
class EmailService {
  sendEmail(to: string, subject: string, body: string): void {
    console.log(`Sending email to ${to}: ${subject} - ${body}`);
  }
}

class LoggerService {
  logActivity(user: string, action: string): void {
    console.log(`Logging activity: ${user} - ${action}`);
  }
}

class UserService {
  constructor(
    private emailService: EmailService,
    private loggerService: LoggerService
  ) {}

  createUser(name: string, email: string): void {
    console.log(`User ${name} created.`);

    this.emailService.sendEmail(email, "Welcome!", "Thank you for signing up!");
    this.loggerService.logActivity(name, "User account created.");
  }
}

// Usage
const emailService = new EmailService();
const loggerService = new LoggerService();
const userService = new UserService(emailService, loggerService);

userService.createUser("Alice", "alice@example.com");
```

## Open-Closed Principle

**Classes should be open for extension but closed for modification.**

The idea is: when we refactor or add new features to the code, we shouldn't need to modify existing code; instead, we just add the desired functionality seamlessly through extension (e.g., via inheritance or interfaces). This protects stable code from breaking changes and promotes polymorphism.

This is better demonstrated with an example. Violating it often results in switch statements or if-else chains that grow with every new requirement, making the code fragile.

Pros: Reduces regression bugs, easier to add features.  
Cons: Requires upfront planning; over-abstraction can complicate simple code.  
When to use: In systems with varying behaviors, like payment gateways or UI components.  
Real-world analogy: A smartphone's app store— you extend functionality by adding apps without modifying the OS core.

### Example

The violation (adding a new payment method requires modifying the `PaymentProcessor` class, violating "closed for modification"):

```ts
class PaymentProcessor {
  processPayment(method: string, amount: number): void {
    if (method === "credit_card") {
      console.log(`Processing credit card payment of $${amount}`);
    } else if (method === "paypal") {
      console.log(`Processing PayPal payment of $${amount}`);
    } else {
      console.log("Unknown payment method");
    }
  }
}

// Usage
const processor = new PaymentProcessor();
processor.processPayment("credit_card", 100);
processor.processPayment("paypal", 50);
```

The application (new methods are added via interface implementations without touching `PaymentProcessor`; open for extension):

```ts
interface PaymentMethod {
  process(amount: number): void;
}

class CreditCardPayment implements PaymentMethod {
  process(amount: number): void {
    console.log(`Processing credit card payment of $${amount}`);
  }
}

class PayPalPayment implements PaymentMethod {
  process(amount: number): void {
    console.log(`Processing PayPal payment of $${amount}`);
  }
}

// New payment method added without modifying existing code
class BitcoinPayment implements PaymentMethod {
  process(amount: number): void {
    console.log(`Processing Bitcoin payment of $${amount}`);
  }
}

class PaymentProcessor {
  processPayment(method: PaymentMethod, amount: number): void {
    method.process(amount);
  }
}

// Usage
const processor = new PaymentProcessor();

const creditCard = new CreditCardPayment();
const paypal = new PayPalPayment();
const bitcoin = new BitcoinPayment(); // New method added without modifying `PaymentProcessor`

processor.processPayment(creditCard, 100);
processor.processPayment(paypal, 50);
processor.processPayment(bitcoin, 200);
```

## Liskov Substitution Principle

**Objects of a parent class should be replaceable by objects of a child class without affecting the correctness of the program.**

In other words, subtypes must be substitutable for their base types. This ensures that inheritance hierarchies behave predictably, preventing unexpected errors when using derived classes.

Violations often occur when subclasses override methods in ways that break assumptions made by the base class or clients.

Pros: Promotes reliable polymorphism and code reuse.  
Cons: Can limit flexibility in inheritance if not designed carefully.  
When to use: When building class hierarchies, like in animal simulations or shape calculators.  
Real-world analogy: A USB port—any compatible device (mouse, keyboard) should work without issues.

### Examples

Violation (Penguin inherits from Bird but throws an error on `fly`, breaking substitutability):

```ts
class Bird {
  fly(): void {
    console.log("Flying...");
  }
}

class Sparrow extends Bird {}

class Penguin extends Bird {
  fly(): void {
    throw new Error("Penguins cannot fly!");
  }
}

// Usage
function makeBirdFly(bird: Bird) {
  bird.fly();
}

const sparrow = new Sparrow();
const penguin = new Penguin();

makeBirdFly(sparrow); //  Works fine
makeBirdFly(penguin); //  Throws an error: "Penguins cannot fly!"
```

Application (Separate flying behavior; Penguin can't be passed to flying functions, enforcing type safety):

```ts
class Bird {
  eat(): void {
    console.log("Eating...");
  }
}

class FlyingBird extends Bird {
  fly(): void {
    console.log("Flying...");
  }
}

class Sparrow extends FlyingBird {}

class Penguin extends Bird {
  swim(): void {
    console.log("Penguin is swimming.");
  }
}

// Usage
function makeBirdFly(bird: FlyingBird) {
  bird.fly();
}

const sparrow = new Sparrow();
const penguin = new Penguin();

makeBirdFly(sparrow); //  Works fine
// makeBirdFly(penguin);  Type error: Argument of type 'Penguin' is not assignable to parameter of type 'FlyingBird'.
```

## Interface Segregation Principle

**Classes should not be forced to implement or use interfaces that have members they do not use.**

To fix this, break large interfaces into smaller, more specific ones. Clients should only depend on what they need. Note: The term "interface" here can also refer to abstract classes or contracts in general.

This prevents "fat interfaces" that bloat classes with irrelevant methods, improving cohesion.

Pros: Better modularity, easier to implement partial behaviors.  
Cons: More interfaces to manage.  
When to use: In APIs or libraries where clients have varying needs.  
Real-world analogy: A TV remote shouldn't have buttons for unrelated devices; segregate controls.

### Examples

Violation (Robot must implement `takeBreak`, which doesn't make sense, leading to errors):

```ts
interface Worker {
  work(): void;
  takeBreak(): void;
}

class HumanWorker implements Worker {
  work(): void {
    console.log("Human is working...");
  }

  takeBreak(): void {
    console.log("Human is taking a break...");
  }
}

class RobotWorker implements Worker {
  work(): void {
    console.log("Robot is working...");
  }

  takeBreak(): void {
    throw new Error("Robots do not take breaks!");
  }
}

// Usage
const workers: Worker[] = [new HumanWorker(), new RobotWorker()];

workers.forEach((worker) => {
  worker.work();
  worker.takeBreak(); // Causes an error for RobotWorker
});
```

Application (Split interfaces; robots only implement what's relevant):

```ts
interface Workable {
  work(): void;
}

interface Breakable {
  takeBreak(): void;
}

class HumanWorker implements Workable, Breakable {
  work(): void {
    console.log("Human is working...");
  }

  takeBreak(): void {
    console.log("Human is taking a break...");
  }
}

class RobotWorker implements Workable {
  work(): void {
    console.log("Robot is working...");
  }
}

// Usage
const workers: Workable[] = [new HumanWorker(), new RobotWorker()];
workers.forEach((worker) => worker.work()); // No issue

const breakableWorkers: Breakable[] = [new HumanWorker()];
breakableWorkers.forEach((worker) => worker.takeBreak()); // Only humans take breaks
```

## Dependency Inversion Principle

**High-level modules should not depend on low-level modules; both should depend on abstractions. Abstractions should not depend on details; details should depend on abstractions.**

In practice, high-level (user-facing) modules should depend on interfaces, not concrete implementations. Low-level modules (details) are then **injected** into high-level modules via **dependency injection** (DI). This decouples code, making it easier to swap implementations (e.g., for testing or changes).

Pros: Flexibility, testability (e.g., mock dependencies).  
Cons: Adds indirection, which can feel overkill for simple apps.  
When to use: In layered architectures, like services depending on repositories.  
Real-world analogy: A lamp depends on a socket interface, not a specific bulb type.

### Examples

Violation (Direct dependency on `PayPalPayment`, hard to switch providers):

```ts
class PayPalPayment {
  processPayment(amount: number): void {
    console.log(`Processing PayPal payment of $${amount}`);
  }
}

class OrderService {
  private paymentProcessor = new PayPalPayment(); // Direct dependency

  placeOrder(amount: number): void {
    console.log("Order placed.");
    this.paymentProcessor.processPayment(amount);
  }
}

// Usage
const orderService = new OrderService();
orderService.placeOrder(100);
```

Application (Depend on interface; inject implementations for flexibility):

```ts
interface PaymentProcessor {
  processPayment(amount: number): void;
}

class PayPalPayment implements PaymentProcessor {
  processPayment(amount: number): void {
    console.log(`Processing PayPal payment of $${amount}`);
  }
}

class CreditCardPayment implements PaymentProcessor {
  processPayment(amount: number): void {
    console.log(`Processing Credit Card payment of $${amount}`);
  }
}

class OrderService {
  constructor(private paymentProcessor: PaymentProcessor) {} // Inject dependency

  placeOrder(amount: number): void {
    console.log("Order placed.");
    this.paymentProcessor.processPayment(amount);
  }
}

// Usage
const paypal = new PayPalPayment();
const orderService1 = new OrderService(paypal);
orderService1.placeOrder(100);

const creditCard = new CreditCardPayment();
const orderService2 = new OrderService(creditCard);
orderService2.placeOrder(200);
```

# UML

UML (Unified Modeling Language) is a standardized way to visualize the design of a system using diagrams. It helps in planning, documenting, and communicating software architecture. Common diagrams include class diagrams (showing relationships like inheritance and composition), sequence diagrams (interactions over time), and use case diagrams.

The original content references an image: ![](image-3.png). In practice, UML class diagrams use boxes for classes, arrows for inheritance (solid line with triangle), and diamonds for composition/aggregation.

Pros: Improves team collaboration, identifies issues early.  
Cons: Can be time-consuming to maintain.  
When to use: For complex systems or during design reviews.

# Observer

The Observer pattern defines a one-to-many dependency between objects so that when one object (the Observable or Subject) changes state, all its dependents (Observers) are notified and updated automatically. It's about having an **Observable** (e.g., emailService) that notifies **Observers** (e.g., User) with events that the **Observers** have subscribed to.

This is useful for event-driven systems, like UI updates or pub/sub messaging. Also, who carries the events list is debatable:

- If the Observers (Users) carry the events, the Observable (EmailService) will iterate over millions of users that could be not subscribing to any events at all, wasting resources.

- If the Observable (EmailService) carries the events attached to lists of users, then it will not iterate over millions of users but eventually will carry millions of rows of user pointers and events, which is more efficient for notification but uses more memory.

The approach in this course uses the latter (Observable manages subscribers per event).

Pros: Loose coupling, easy to add/remove observers.  
Cons: Can lead to memory leaks if observers aren't unsubscribed; potential for update cascades.  
When to use: In reactive systems, like stock tickers or notification systems.  
Real-world analogy: A newsletter publisher (Observable) sends updates to subscribers (Observers).

The original content references an image: // v(image-1.png)

## Implementation

### Defining Interfaces

```ts
interface EventInterface {
  name: string;
  payload?: any; // Additional event data
}
```

```ts
interface ObservableInterface {
  attach(observer: ObserverInterface, eventType: string): void;
  detach(observer: ObserverInterface, eventType: string): void;
  notify(event: EventInterface): void;
}
```

```ts
interface ObserverInterface {
  update(event: EventInterface): void;
}
```

Note: The original uses `Observable` and `ObserverInterface`, but I've kept it consistent. `attach/detach` are synonyms for subscribe/unsubscribe.

### Defining Classes

```ts
// Observer (User)
class User implements ObserverInterface {
  constructor(public name: string) {}

  update(event: EventInterface) {
    console.log(
      `User ${this.name} received notification: ${event.name}`,
      event.payload
    );
  }
}
```

```ts
// Notification Service with Event-Based Subscription Management
class NotificationService implements ObservableInterface {
  private observers: Map<string, Set<ObserverInterface>> = new Map();

  attach(observer: ObserverInterface, eventType: string) {
    if (!this.observers.has(eventType)) {
      this.observers.set(eventType, new Set());
    }
    this.observers.get(eventType)!.add(observer);
    console.log(
      `${(observer as User).name} subscribed to ${eventType} notifications.`
    );
  }

  detach(observer: ObserverInterface, eventType: string) {
    if (this.observers.has(eventType)) {
      this.observers.get(eventType)!.delete(observer);
      console.log(
        `${
          (observer as User).name
        } unsubscribed from ${eventType} notifications.`
      );
      if (this.observers.get(eventType)!.size === 0) {
        this.observers.delete(eventType);
      }
    }
  }

  notify(event: EventInterface) {
    if (this.observers.has(event.name)) {
      this.observers
        .get(event.name)!
        .forEach((observer) => observer.update(event));
    }
  }
}
```

```ts
// User Service (Handles user subscriptions and unsubscriptions)
class UserService {
  private users: User[] = [];

  constructor(private notificationService: NotificationService) {}

  createUser(name: string): User {
    const user = new User(name);
    this.users.push(user);
    return user;
  }

  subscribeUserToEvent(user: User, eventType: string) {
    this.notificationService.attach(user, eventType);
  }

  unsubscribeUserFromEvent(user: User, eventType: string) {
    this.notificationService.detach(user, eventType);
  }
}
```

```ts
// Email Service (Triggers email-related events)
class EmailService {
  constructor(private notificationService: NotificationService) {}

  sendEmail(emailContent: string) {
    this.notificationService.notify({ name: "email", payload: emailContent });
  }
}
```

```ts
// SMS Service (Another example observer service)
class SMSService {
  constructor(private notificationService: NotificationService) {}

  sendSMS(smsContent: string) {
    this.notificationService.notify({ name: "sms", payload: smsContent });
  }
}
```

Usage (demonstrates subscription, notification, and unsubscription):

```ts
const notificationService = new NotificationService();
const userService = new UserService(notificationService);
const emailService = new EmailService(notificationService);
const smsService = new SMSService(notificationService);

// Creating Users
const user1 = userService.createUser("Alice");
const user2 = userService.createUser("Bob");

// Subscribing Users to Specific Events
userService.subscribeUserToEvent(user1, "email");
userService.subscribeUserToEvent(user2, "sms");

// Sending Notifications
emailService.sendEmail("New promotional email!");
smsService.sendSMS("Flash Sale: 50% off!");

// Unsubscribing Bob from SMS notifications
userService.unsubscribeUserFromEvent(user2, "sms");

// Sending More Notifications
emailService.sendEmail("Exclusive email offer!");
smsService.sendSMS("New SMS campaign launched!");
```

# Strategy

The Strategy pattern defines a family of algorithms, encapsulates each one, and makes them interchangeable. If a class has a composition relationship with an interface, instead of linking them with hard coding, we can inject the implementation. This allows behavior to be selected at runtime.

It is exactly what we did in the Dependency Inversion principle, promoting flexibility. We can have unlimited implementations of the same strategy.

Pros: Easy to switch behaviors, adheres to open-closed.  
Cons: Clients must know about strategies; adds classes.  
When to use: For varying algorithms, like sorting or compression.  
Real-world analogy: Navigation apps choosing strategies (fastest, shortest, scenic) for routes.

The original content references an image: // v(image-2.png)

## Implementation

Defines a common interface for different pricing strategies.

```typescript
interface PricingStrategy {
  calculatePrice(price: number): number;
}
```

Different pricing strategies implementing `PricingStrategy`.

```typescript
class RegularPricingStrategy implements PricingStrategy {
  calculatePrice(price: number): number {
    return price; // No discount
  }
}

class GoldPricingStrategy implements PricingStrategy {
  calculatePrice(price: number): number {
    return price * 0.9; // 10% discount
  }
}

class PremiumPricingStrategy implements PricingStrategy {
  calculatePrice(price: number): number {
    return price * 0.8; // 20% discount
  }
}
```

Represents a product that uses a pricing strategy to determine its final price.

```typescript
class Product {
  constructor(
    public name: string,
    public price: number,
    private pricingStrategy: PricingStrategy
  ) {}

  calculatePrice(): number {
    return this.pricingStrategy.calculatePrice(this.price);
  }
}
```

Defines a common interface for different payment strategies.

```typescript
interface PaymentStrategy {
  processPayment(amount: number): void;
}
```

Different payment strategies implementing `PaymentStrategy`.

```typescript
class VisaCardPaymentStrategy implements PaymentStrategy {
  processPayment(amount: number): void {
    console.log(`Processing Visa Card payment of $${amount}`);
  }
}

class PaypalPaymentStrategy implements PaymentStrategy {
  processPayment(amount: number): void {
    console.log(`Processing PayPal payment of $${amount}`);
  }
}

class BankTransferPaymentStrategy implements PaymentStrategy {
  processPayment(amount: number): void {
    console.log(`Processing Bank Transfer payment of $${amount}`);
  }
}
```

Handles payment processing using a selected payment strategy.

```typescript
class Checkout {
  constructor(private paymentStrategy: PaymentStrategy) {}

  processPayment(amount: number): void {
    this.paymentStrategy.processPayment(amount);
  }
}
```

Demonstrates how to use the strategies in action.

```typescript
// Create a product with a Gold pricing strategy
const product1 = new Product("Laptop", 1000, new GoldPricingStrategy());

// Calculate the final price
console.log(`Final Price: $${product1.calculatePrice()}`);

// Process payment using PayPal
const checkout = new Checkout(new PaypalPaymentStrategy());
checkout.processPayment(product1.calculatePrice());
```

# Template Method

The **Template Method** design pattern defines the **skeleton of an algorithm** in a base class but lets subclasses **override specific steps** without changing the algorithm's structure. This promotes code reuse for common workflows while allowing customization.

Here’s a **TypeScript implementation** using Markdown under `## Implementation`. It's useful for processes with invariant steps but variable details.

Pros: Reuses code, enforces structure.  
Cons: Subclasses are tightly coupled to the base; hard to change the skeleton.  
When to use: In frameworks, like game loops or report generators.  
Real-world analogy: A recipe template—steps like "prep ingredients" are fixed, but details vary per dish.

## Implementation

Defines the template method and the structure of the algorithm.

```typescript
abstract class OrderProcessor {
  // Template method (final method that defines the process)
  processOrder(): void {
    this.selectProduct();
    this.makePayment();
    this.deliverProduct();
  }

  abstract selectProduct(): void;
  abstract makePayment(): void;

  // Hook method (optional step with default implementation)
  deliverProduct(): void {
    console.log("Delivering product to customer...");
  }
}
```

Subclasses implement specific steps of the algorithm.

```typescript
class OnlineOrderProcessor extends OrderProcessor {
  selectProduct(): void {
    console.log("Selecting product from online store...");
  }

  makePayment(): void {
    console.log("Processing online payment...");
  }
}

class InStoreOrderProcessor extends OrderProcessor {
  selectProduct(): void {
    console.log("Selecting product from physical store...");
  }

  makePayment(): void {
    console.log("Processing cash or card payment...");
  }
}
```

Usage (shows how the template enforces the order while allowing variation):

```typescript
const onlineOrder = new OnlineOrderProcessor();
console.log("Processing Online Order:");
onlineOrder.processOrder();

console.log("\nProcessing In-Store Order:");
const inStoreOrder = new InStoreOrderProcessor();
inStoreOrder.processOrder();
```

# Memento

The Memento pattern allows capturing and restoring an object's internal state without violating encapsulation. A memento class manages previous and next states.

**Think of it like _Ctrl+Z_ (undo) and _Shift+Ctrl+Z_ (redo) buttons** in editors. It's useful for implementing undo/redo functionality.

Pros: Preserves encapsulation, easy history management.  
Cons: Can consume memory with many states.  
When to use: In applications like text editors, games, or databases for transactions.  
Real-world analogy: Browser history—save states to navigate back/forward.

The original content references an image: // v(image-5.png)

## Implementation

```typescript
class Memento {
  constructor(private state: string) {}

  getState(): string {
    return this.state;
  }
}
```

Creates and restores its state using Memento.

```typescript
class Originator {
  private state: string = "";

  setState(state: string): void {
    console.log(`Setting state to: ${state}`);
    this.state = state;
  }

  saveState(): Memento {
    console.log(`Saving state: ${this.state}`);
    return new Memento(this.state);
  }

  restoreState(memento: Memento): void {
    this.state = memento.getState();
    console.log(`Restored state to: ${this.state}`);
  }
}
```

Manages the saved states.

```typescript
class Caretaker {
  private mementos: Memento[] = [];

  saveMemento(memento: Memento): void {
    this.mementos.push(memento);
  }

  getMemento(index: number): Memento | null {
    return this.mementos[index] || null;
  }
}
```

```typescript
const originator = new Originator();
const caretaker = new Caretaker();

originator.setState("State 1");
caretaker.saveMemento(originator.saveState());

originator.setState("State 2");
caretaker.saveMemento(originator.saveState());

originator.setState("State 3");

console.log("\nRestoring previous state:");
const previousState = caretaker.getMemento(1);
if (previousState) originator.restoreState(previousState);

console.log("\nRestoring first saved state:");
const firstState = caretaker.getMemento(0);
if (firstState) originator.restoreState(firstState);
```

## Example Implementation of the Text Editor

This builds on the basic implementation for a more meaningful example, like a text editor with undo/redo.

```typescript
class TextEditorState {
  constructor(private content: string) {}

  getContent(): string {
    return this.content;
  }
}
```

```typescript
class TextEditor {
  private content: string = "";

  setContent(content: string): void {
    this.content = content;
  }

  getContent(): string {
    return this.content;
  }

  save(): TextEditorState {
    return new TextEditorState(this.content);
  }

  restore(state: TextEditorState): void {
    this.content = state.getContent();
  }
}
```

```typescript
class History {
  private prevStates: TextEditorState[] = [];
  private nextStates: TextEditorState[] = [];

  saveHistoryState(state: TextEditorState): void {
    this.prevStates.push(state);
    this.nextStates = []; // Clear redo history when saving a new state
  }

  undo(): TextEditorState | null {
    if (this.prevStates.length === 0) return null;
    const lastState = this.prevStates.pop()!;
    this.nextStates.push(lastState);
    return lastState;
  }

  redo(): TextEditorState | null {
    if (this.nextStates.length === 0) return null;
    const nextState = this.nextStates.pop()!;
    this.prevStates.push(nextState);
    return nextState;
  }
}
```

Usage (demonstrates saving states, undo, and redo):

```typescript
const editor = new TextEditor();
const history = new History();

editor.setContent("Version 1");
history.saveHistoryState(editor.save());

editor.setContent("Version 2");
history.saveHistoryState(editor.save());

editor.setContent("Version 3");
history.saveHistoryState(editor.save());

console.log(`Current Content: ${editor.getContent()}`);

// Undo twice
const undoState1 = history.undo();
if (undoState1) editor.restore(undoState1);
console.log(`After Undo 1: ${editor.getContent()}`);

const undoState2 = history.undo();
if (undoState2) editor.restore(undoState2);
console.log(`After Undo 2: ${editor.getContent()}`);

// Redo once
const redoState = history.redo();
if (redoState) editor.restore(redoState);
console.log(`After Redo: ${editor.getContent()}`);
```

# Visitor

The Visitor pattern allows adding new operations to a class hierarchy without modifying the classes themselves. The idea is to leave a space in the classes to accept 2nd party functionality via a `visit` method.

This separates algorithms from the objects they operate on, useful for operations like reporting or exporting.

Pros: Easy to add new operations; adheres to open-closed.  
Cons: Requires modifying elements if new types are added; can break encapsulation if visitors access private data.  
When to use: When you have stable object structures but frequently add operations (e.g., AST in compilers).  
Real-world analogy: A museum visitor (operation) touring exhibits (objects) without changing the exhibits.

My thoughts: The original critique notes that it may not be useful because **it needs to operate on internal attributes of the class**, thus potentially breaking core principles of OOP (encapsulation, abstraction) and making the code less safe. Additionally, **it abstracts implementation**—if I were to add more implementation, I would refactor the implementation rather than adding a new functionality blindly, which can lead to security vulnerabilities. However, it can be a good pattern in cases of very old legacy code that barely works, or when you need to extend functionality without touching source code (e.g., in libraries).

The original content references an image: // v(image-6.png)

## Example

Interfaces

```ts
// Visitor Interface
interface Visitor {
  visitCircle(circle: Circle): void;
  visitRectangle(rectangle: Rectangle): void;
}
```

```ts
// Element Interface
interface Shape {
  accept(visitor: Visitor): void;
}
```

Our concrete elements

```ts
class Circle implements Shape {
  accept(visitor: Visitor): void {
    visitor.visitCircle(this);
  }
}
```

```ts
class Rectangle implements Shape {
  accept(visitor: Visitor): void {
    visitor.visitRectangle(this);
  }
}
```

### Creating Visitors

This is an example visitor that calculates area based on the **public** members of the shape (to preserve encapsulation; assume shapes expose radius/width/height publicly). We can make as many visitors as needed (e.g., one for drawing, one for serialization).

```ts
class AreaCalculator implements Visitor {
  visitCircle(circle: Circle): void {
    console.log("Calculating area of Circle");
  }

  visitRectangle(rectangle: Rectangle): void {
    console.log("Calculating area of Rectangle");
  }
}
```

### Usage

```ts
const visitor = new AreaCalculator();

const circle = new Circle();

circle.accept(visitor); // it logs: "Calculating area of Circle"
```

# Iterator

The Iterator pattern provides a way to access elements of an aggregate object sequentially without exposing its underlying representation. The idea is to separate iteration logic over objects from the collection itself.

This allows traversing different data structures (arrays, trees, etc.) uniformly.

Pros: Simplifies traversal, supports multiple iterators.  
Cons: Overkill for simple collections; adds complexity.  
When to use: In collections like lists or maps where clients need to iterate without knowing internals.  
Real-world analogy: A TV remote iterating through channels without knowing how the TV stores them.

This is better demonstrated with an example.

The original content references an image: // v(image-7.png)

## Example

Interfaces

```ts
interface Iterator<T> {
  next(): T | null;
  hasNext(): boolean;
}
```

```ts
interface IterableCollection<T> {
  getIterator(): Iterator<T>;
}
```

The iterator

```ts
class ArrayIterator<T> implements Iterator<T> {
  private index = 0;

  constructor(private items: T[]) {}

  next(): T | null {
    if (this.hasNext()) {
      return this.items[this.index++];
    }
    return null;
  }

  hasNext(): boolean {
    return this.index < this.items.length;
  }
}
```

The collection repo

```ts
class NumberCollection implements IterableCollection<number> {
  private numbers: number[];

  constructor(numbers: number[]) {
    this.numbers = numbers;
  }

  getIterator(): Iterator<number> {
    return new ArrayIterator(this.numbers);
  }
}
```

### Usage

```ts
const collection = new NumberCollection([10, 20, 30, 40]);
const iterator = collection.getIterator();

while (iterator.hasNext()) {
  console.log(iterator.next()); // Output: 10, 20, 30, 40
}
```

# Chain of Responsibility (Middlewares)

The Chain of Responsibility pattern avoids coupling the sender of a request to its receiver by giving more than one object a chance to handle the request. It involves having multiple handlers for the same input in a chain, like middlewares in web frameworks (e.g., Express.js).

Each handler decides whether to process the request or pass it to the next.

Pros: Decouples senders/receivers, easy to add handlers.  
Cons: Request might not be handled; debugging chains can be tricky.  
When to use: In logging, authentication, or request pipelines.  
Real-world analogy: A customer service call routed through levels of support.

The original content references an image: // v(image-8.png)

## Example

```ts
abstract class Logger {
  // the class is abstract
  protected next: Logger | null = null; // the next logger, it may exist meaning that there is a next function or may not exist meaning that we are at the last middleware

  setNext(logger: Logger): Logger {
    // this sets up the next middleware chained to the existing middleware
    this.next = logger;
    return logger; // returns the new one
  }

  log(level: string, message: string): void {
    if (this.next) {
      // this is the logging functionality
      this.next.log(level, message);
    }
  }
}
```

### Middlewares

These are three middlewares (each handles a specific log level or passes on).

```ts
class InfoLogger extends Logger {
  log(level: string, message: string): void {
    if (level === "INFO") {
      console.log(`[INFO]: ${message}`);
    } else {
      super.log(level, message);
    }
  }
}

class WarningLogger extends Logger {
  log(level: string, message: string): void {
    if (level === "WARNING") {
      console.log(`[WARNING]: ${message}`);
    } else {
      super.log(level, message);
    }
  }
}

class ErrorLogger extends Logger {
  log(level: string, message: string): void {
    if (level === "ERROR") {
      console.log(`[ERROR]: ${message}`);
    } else {
      super.log(level, message);
    }
  }
}
```

### Usage

Creating classes

```ts
const infoLogger = new InfoLogger();
const warningLogger = new WarningLogger();
const errorLogger = new ErrorLogger();
```

Injecting middlewares (chains them together)

```ts
infoLogger.setNext(warningLogger).setNext(errorLogger);
```

```ts
// Client makes requests
infoLogger.log("INFO", "This is an information message.");
infoLogger.log("WARNING", "This is a warning message.");
infoLogger.log("ERROR", "This is an error message.");
```

This code will loop over all the middleware and only the corresponding one will execute.

# State

The State pattern allows an object to alter its behavior when its internal state changes, appearing as if the object's class has changed. Instead of changing implementation based on state (e.g., via if-else), bind the implementation to the state via delegated classes.

This cleans up complex conditional logic.

Pros: Localizes state behavior, adheres to single responsibility.  
Cons: Can lead to many state classes.  
When to use: In workflows like order processing or UI states.  
Real-world analogy: A traffic light changing behavior based on color state.

This is better demonstrated with an example.

The original content references an image: // v(image-9.png)

## Example

Interfaces

```ts
// State Interface
interface OrderState {
  processOrder(): void;
  shipOrder(): void;
  deliverOrder(): void;
  cancelOrder(): void;
}
```

Order manager

```ts
// Context Class (Order Management)
class OrderManagement {
  private orderState: OrderState;

  constructor() {
    this.orderState = new NewOrderState(this); // Default state
  }

  changeState(state: OrderState) {
    this.orderState = state;
  }

  processOrder() {
    this.orderState.processOrder();
  }

  shipOrder() {
    this.orderState.shipOrder();
  }

  deliverOrder() {
    this.orderState.deliverOrder();
  }

  cancelOrder() {
    this.orderState.cancelOrder();
  }
}
```

Order states (each encapsulates behavior for that state)

```ts
// Concrete State: New Order
class NewOrderState implements OrderState {
  private orderManagement: OrderManagement;

  constructor(orderManagement: OrderManagement) {
    this.orderManagement = orderManagement;
  }

  processOrder() {
    console.log("Order is now being processed.");
    this.orderManagement.changeState(
      new ProcessingOrderState(this.orderManagement)
    );
  }

  shipOrder() {
    console.log("Cannot ship order. Order is still new.");
  }

  deliverOrder() {
    console.log("Cannot deliver order. Order is still new.");
  }

  cancelOrder() {
    console.log("Order is cancelled.");
    this.orderManagement.changeState(new CancelledOrderState());
  }
}

// Concrete State: Processing Order
class ProcessingOrderState implements OrderState {
  private orderManagement: OrderManagement;

  constructor(orderManagement: OrderManagement) {
    this.orderManagement = orderManagement;
  }

  processOrder() {
    console.log("Order is already being processed.");
  }

  shipOrder() {
    console.log("Order is now shipped.");
    this.orderManagement.changeState(
      new ShippedOrderState(this.orderManagement)
    );
  }

  deliverOrder() {
    console.log("Cannot deliver order before shipping.");
  }

  cancelOrder() {
    console.log("Order is cancelled.");
    this.orderManagement.changeState(new CancelledOrderState());
  }
}

// Concrete State: Shipped Order
class ShippedOrderState implements OrderState {
  private orderManagement: OrderManagement;

  constructor(orderManagement: OrderManagement) {
    this.orderManagement = orderManagement;
  }

  processOrder() {
    console.log("Order has already been shipped and cannot be processed.");
  }

  shipOrder() {
    console.log("Order is already shipped.");
  }

  deliverOrder() {
    console.log("Order has been delivered.");
    this.orderManagement.changeState(new DeliveredOrderState());
  }

  cancelOrder() {
    console.log("Cannot cancel. Order has already been shipped.");
  }
}

// Concrete State: Delivered Order
class DeliveredOrderState implements OrderState {
  deliverOrder() {
    console.log("Order is already delivered.");
  }

  processOrder() {
    console.log("Order is already delivered and cannot be processed.");
  }

  shipOrder() {
    console.log("Order is already delivered and cannot be shipped.");
  }

  cancelOrder() {
    console.log("Cannot cancel. Order is already delivered.");
  }
}

// Concrete State: Cancelled Order
class CancelledOrderState implements OrderState {
  deliverOrder() {
    console.log("Cannot deliver. Order is cancelled.");
  }

  processOrder() {
    console.log("Cannot process. Order is cancelled.");
  }

  shipOrder() {
    console.log("Cannot ship. Order is cancelled.");
  }

  cancelOrder() {
    console.log("Order is already cancelled.");
  }
}
```

### Usage

```ts
const order = new OrderManagement();
order.processOrder(); // Transitions to Processing
order.shipOrder(); // Transitions to Shipped
order.deliverOrder(); // Transitions to Delivered
order.cancelOrder(); // Order cannot be cancelled after delivery
```

# Mediator

The Mediator pattern defines an object that encapsulates how a set of objects interact, promoting loose coupling by preventing objects from referring to each other explicitly. The Mediator is a class that does two things:

- Defines communication rules.

- Communicates between classes.

This centralizes complex interactions.

Pros: Reduces dependencies, easier to maintain.  
Cons: Mediator can become a god class.  
When to use: In GUI components or chat apps where objects need to collaborate.  
Real-world analogy: Air traffic control mediating planes.

The original content references an image: // v(image-11.png)

## Example

The original references another image: // v(image-12.png)

Mediator interface

```ts
interface ChatMediator {
  sendDirectMessage(message: string, sender: User, receiver: User): void;
  sendGroupMessage(message: string, sender: User, groupName: string): void;
  registerUserToGroup(user: User, groupName: string): void;
}
```

Mediator implementation

```ts
class ChatManagement implements ChatMediator {
  private groups: Map<string, User[]> = new Map();

  sendDirectMessage(message: string, sender: User, receiver: User): void {
    receiver.receiveDirectMessage(message, sender.getName());
  }

  sendGroupMessage(message: string, sender: User, groupName: string): void {
    const users = this.groups.get(groupName);
    if (!users) {
      console.log(`Group '${groupName}' does not exist.`);
      return;
    }

    users.forEach((user) => {
      if (user !== sender) {
        user.receiveGroupMessage(message, sender.getName(), groupName);
      }
    });
  }

  registerUserToGroup(user: User, groupName: string): void {
    if (!this.groups.has(groupName)) {
      this.groups.set(groupName, []);
    }
    this.groups.get(groupName)?.push(user);
  }
}
```

A user

```ts
class User {
  private name: string;
  private chatMediator: ChatMediator;

  constructor(name: string, chatMediator: ChatMediator) {
    this.name = name;
    this.chatMediator = chatMediator;
  }

  getName(): string {
    return this.name;
  }

  sendDirectMessage(message: string, receiver: User): void {
    console.log(
      `${this.name} sends direct message to ${receiver.getName()}: ${message}`
    );
    this.chatMediator.sendDirectMessage(message, this, receiver);
  }

  sendGroupMessage(message: string, groupName: string): void {
    console.log(
      `${this.name} sends message to group '${groupName}': ${message}`
    );
    this.chatMediator.sendGroupMessage(message, this, groupName);
  }

  receiveDirectMessage(message: string, sender: string): void {
    console.log(
      `${this.name} received direct message from ${sender}: ${message}`
    );
  }

  receiveGroupMessage(
    message: string,
    sender: string,
    groupName: string
  ): void {
    console.log(
      `${this.name} received group message in '${groupName}' from ${sender}: ${message}`
    );
  }
}
```

### Usage

```ts
const chatMediator = new ChatManagement();

const user1 = new User("Alice", chatMediator);
const user2 = new User("Bob", chatMediator);
const user3 = new User("Charlie", chatMediator);

chatMediator.registerUserToGroup(user1, "Developers");
chatMediator.registerUserToGroup(user2, "Developers");
chatMediator.registerUserToGroup(user3, "Developers");

user1.sendDirectMessage("Hello Bob!", user2);
user2.sendGroupMessage("Hey team!", "Developers");
```

# Command

The Command pattern turns a request into a stand-alone object that contains all information about the request. Instead of hardcoding commands and composing them manually, we can create a single entry point called **Command** that follows the same interface and executes multiple actions dynamically.

This allows parameterizing methods with commands, queuing, logging, or undoing them.

Pros: Decouples invokers from receivers, supports undo.  
Cons: Adds classes for each command.  
When to use: In GUIs (e.g., menu actions), transactions, or macros.  
Real-world analogy: A remote control where buttons (commands) trigger device actions.

The original content references an image: // v(image-13.png)

This is better demonstrated with an example.

## Example

### Command Interface

```ts
interface Command {
  execute(): void;
}
```

### Concrete Devices

```ts
class Light {
  turnOn() {
    console.log("Light is turned ON");
  }

  turnOff() {
    console.log("Light is turned OFF");
  }
}

class Tv {
  turnOn() {
    console.log("TV is turned ON");
  }

  turnOff() {
    console.log("TV is turned OFF");
  }
}

class Door {
  lock() {
    console.log("Door is LOCKED");
  }

  unlock() {
    console.log("Door is UNLOCKED");
  }
}
```

### Commands That Run the Concrete Devices

```ts
class TurnOnLightCommand implements Command {
  constructor(private light: Light) {}

  execute(): void {
    this.light.turnOn();
  }
}

class TurnOffLightCommand implements Command {
  constructor(private light: Light) {}

  execute(): void {
    this.light.turnOff();
  }
}

class TurnOnTvCommand implements Command {
  constructor(private tv: Tv) {}

  execute(): void {
    this.tv.turnOn();
  }
}

class TurnOffTvCommand implements Command {
  constructor(private tv: Tv) {}

  execute(): void {
    this.tv.turnOff();
  }
}

class LockDoorCommand implements Command {
  constructor(private door: Door) {}

  execute(): void {
    this.door.lock();
  }
}

class UnlockDoorCommand implements Command {
  constructor(private door: Door) {}

  execute(): void {
    this.door.unlock();
  }
}
```

### Class That Invokes Commands

```ts
class SmartHomeVoiceAssistant {
  private commands: Map<string, Command> = new Map();

  setCommand(name: string, command: Command) {
    this.commands.set(name, command);
  }

  say(commandName: string) {
    const command = this.commands.get(commandName);
    if (command) {
      console.log(`Voice Assistant executing: ${commandName}`);
      command.execute();
    } else {
      console.log(`Command "${commandName}" not found`);
    }
  }
}
```

### Mobile Application That Executes Commands

```ts
class SmartHomeMobileApplication {
  execute(command: Command) {
    console.log("Mobile App executing command...");
    command.execute();
  }
}
```

## Usage

### Create Commands and Device Classes

```ts
const light = new Light();
const tv = new Tv();
const door = new Door();

const turnOnLight = new TurnOnLightCommand(light);
const turnOffLight = new TurnOffLightCommand(light);
const turnOnTv = new TurnOnTvCommand(tv);
const turnOffTv = new TurnOffTvCommand(tv);
const lockDoor = new LockDoorCommand(door);
const unlockDoor = new UnlockDoorCommand(door);
```

### Inject Commands into Executors

```ts
const voiceAssistant = new SmartHomeVoiceAssistant();
voiceAssistant.setCommand("turn on light", turnOnLight);
voiceAssistant.setCommand("turn off light", turnOffLight);
voiceAssistant.setCommand("lock door", lockDoor);
voiceAssistant.setCommand("unlock door", unlockDoor);
```

### Mobile App That Can Execute Any Command

```ts
const mobileApp = new SmartHomeMobileApplication();
```

## Execution

```ts
voiceAssistant.say("turn on light");
mobileApp.execute(lockDoor);
```

# Adapter

The Adapter pattern allows incompatible interfaces to work together by wrapping one in a compatible interface. An adapter enables communication between legacy code and new code that has an unmatching interface.

This is structural, like a converter.

Pros: Reuses existing code, adheres to open-closed.  
Cons: Adds indirection.  
When to use: Integrating third-party libraries or legacy systems.  
Real-world analogy: A power adapter for international plugs.

An example would be:

## Example

### Legacy Code

```ts
interface PaymentMethodAInterface {
  pay(amount: number): void;
}

class ProcessPayment {
  constructor(
    private amount: number,
    private paymentMethod: PaymentMethodAInterface
  ) {}

  processPayment(): void {
    this.paymentMethod.pay(this.amount);
  }
}
```

This legacy code works fine but what if I want to use another payment method with a different interface? I will have to refactor the `ProcessPayment` class without an adapter.

```ts
interface PaymentMethodBInterface {
  pay(amount: string): Promise<void>;
}
```

### New Code

Using an adapter will fix it (converts types and handles async):

```ts
class PaymentMethodBAdapter implements PaymentMethodAInterface {
  constructor(private paymentMethodB: PaymentMethodBInterface) {}

  pay(amount: number): void {
    this.paymentMethodB.pay(amount.toString()).then(() => {
      console.log("Payment processed successfully");
    });
  }
}
```

# Bridge

The Bridge pattern decouples an abstraction from its implementation so that the two can vary independently. In very simple words without BS, the Bridge pattern is just that we want the classes to depend on interfaces, not implementations—that's it, nothing more. It prevents a cartesian product of classes (e.g., no need for RedCircle, BlueCircle if color is bridged).

Pros: Flexibility in changing implementations, avoids class explosion.  
Cons: Increases complexity.  
When to use: When abstractions and implementations need to evolve separately, like devices with remotes.  
Real-world analogy: A remote control (abstraction) working with different TVs (implementations).

The original content references an image: // v(image-15.png)

# Composite

The Composite pattern composes objects into tree structures to represent part-whole hierarchies. It lets you treat individual objects and compositions of objects uniformly.

This is useful for hierarchical structures like file systems or UI components.

Pros: Simplifies client code, easy to add new components.  
Cons: Can make designs overly general.  
When to use: In trees or graphs, like menus or directories.  
Real-world analogy: A company org chart—treat employees and departments the same for operations like payroll.

## Example

The folder and the leaf have the same interface, which facilitates getting reports about a single file (that has size) and many files (through the size of the folder, which is actually the addition of all sizes). We can also make compositions of compositions.

Interfaces

```ts
// Component
interface FileSystem {
  getSize(): number;
}
```

```ts
// Leaf
class File implements FileSystem {
  constructor(private size: number) {}

  getSize(): number {
    return this.size;
  }
}
```

The composing

```ts
// Composite
class Folder implements FileSystem {
  private children: FileSystem[] = [];

  add(child: FileSystem): void {
    this.children.push(child);
  }

  getSize(): number {
    return this.children.reduce((sum, child) => sum + child.getSize(), 0);
  }
}
```

### Application

```ts
const file1 = new File(10);
const file2 = new File(20);
const folder = new Folder();
folder.add(file1);
folder.add(file2);

const file3 = new File(10);
const file4 = new File(20);
const folder2 = new Folder();
folder2.add(file3);
folder2.add(file4);

const folder3 = new Folder();
folder3.add(folder);
folder3.add(folder2);

// we can also add folders to folders
```

# Decorator

The Decorator pattern attaches additional responsibilities to an object dynamically. A decorator adds functionality to methods without touching them, providing a flexible alternative to subclassing.

Pros: More flexible than inheritance, composable.  
Cons: Can lead to many small objects; harder to debug.  
When to use: For adding features like logging or caching without modifying core classes.  
Real-world analogy: Adding toppings to a pizza without changing the base recipe.

## Example Using Basic Syntax

```ts
// Component interface
interface Coffee {
  cost(): number;
  description(): string;
}

// Concrete component
class SimpleCoffee implements Coffee {
  cost(): number {
    return 5;
  }

  description(): string {
    return "Simple Coffee";
  }
}

// Decorator abstract class (implements Coffee interface)
class CoffeeDecorator implements Coffee {
  protected coffee: Coffee;

  constructor(coffee: Coffee) {
    this.coffee = coffee;
  }

  cost(): number {
    return this.coffee.cost();
  }

  description(): string {
    return this.coffee.description();
  }
}

// Concrete decorator
class MilkDecorator extends CoffeeDecorator {
  cost(): number {
    return this.coffee.cost() + 2;
  }

  description(): string {
    return this.coffee.description() + " with Milk";
  }
}

// Another concrete decorator
class SugarDecorator extends CoffeeDecorator {
  cost(): number {
    return this.coffee.cost() + 1;
  }

  description(): string {
    return this.coffee.description() + " with Sugar";
  }
}

// Usage
let coffee: Coffee = new SimpleCoffee();
console.log(coffee.description() + " costs $" + coffee.cost());

coffee = new MilkDecorator(coffee);
console.log(coffee.description() + " costs $" + coffee.cost());

coffee = new SugarDecorator(coffee);
console.log(coffee.description() + " costs $" + coffee.cost());
```

## Example Using Method Decorators

TypeScript supports decorators as a language feature for meta-programming.

```ts
// Base class for coffee
class Coffee {
  cost(): number {
    return 5;
  }

  description(): string {
    return "Simple Coffee";
  }
}

// Method decorator to add Milk
function MilkDecorator(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const originalMethod = descriptor.value;

  descriptor.value = function (...args: any[]) {
    return originalMethod.apply(this, args) + " with Milk";
  };
}

// Method decorator to add Sugar
function SugarDecorator(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const originalMethod = descriptor.value;

  descriptor.value = function (...args: any[]) {
    return originalMethod.apply(this, args) + " with Sugar";
  };
}

// New class that uses decorators on methods
class DecoratedCoffee extends Coffee {
  @MilkDecorator
  description(): string {
    return super.description();
  }

  @SugarDecorator
  descriptionWithSugar(): string {
    return super.description();
  }
}
// Usage
let coffee = new Coffee();
console.log(coffee.description() + " costs $" + coffee.cost());

let decoratedCoffee = new DecoratedCoffee();
console.log(
  decoratedCoffee.description() + " costs $" + decoratedCoffee.cost()
);
```

# Facade

The Facade pattern provides a unified interface to a set of interfaces in a subsystem. A facade is a single entry point that abstracts a lot of implementation details behind a single function call, simplifying complex systems.

Pros: Hides complexity, improves readability.  
Cons: Can become a god object if overused.  
When to use: For libraries or subsystems like APIs.  
Real-world analogy: A car dashboard—hides engine details behind simple controls.

## Example

Without a facade, turning on a home theater might involve multiple steps:

```typescript
class Amplifier {
  on() {
    console.log("Amplifier is ON");
  }
  setVolume(level: number) {
    console.log(`Volume set to ${level}`);
  }
}

class DVDPlayer {
  on() {
    console.log("DVD Player is ON");
  }
  play(movie: string) {
    console.log(`Playing movie: ${movie}`);
  }
}

class Projector {
  on() {
    console.log("Projector is ON");
  }
  setInput(source: string) {
    console.log(`Projector input set to ${source}`);
  }
}
```

Now, using a **Facade**, we can simplify the process:

```typescript
class HomeTheaterFacade {
  private amp: Amplifier;
  private dvd: DVDPlayer;
  private projector: Projector;

  constructor(amp: Amplifier, dvd: DVDPlayer, projector: Projector) {
    this.amp = amp;
    this.dvd = dvd;
    this.projector = projector;
  }

  watchMovie(movie: string) {
    console.log("Setting up home theater...");
    this.amp.on();
    this.amp.setVolume(10);
    this.projector.on();
    this.projector.setInput("DVD");
    this.dvd.on();
    this.dvd.play(movie);
    console.log("Enjoy your movie!");
  }
}

// Usage
const homeTheater = new HomeTheaterFacade(
  new Amplifier(),
  new DVDPlayer(),
  new Projector()
);
homeTheater.watchMovie("Inception");
```

# Proxy

The Proxy pattern provides a surrogate or placeholder for another object to control access to it. A proxy is a middleman that handles communication between two users and interfaces, so it can apply input modification/validation, routing, logging, and more (e.g., virtual proxies for lazy loading, protection proxies for access control).

Pros: Adds control without changing the subject.  
Cons: Can introduce latency.  
When to use: For caching, logging, or remote objects.  
Real-world analogy: A credit card proxying for cash.

# Flyweight

The Flyweight pattern minimizes memory usage by sharing as much data as possible with similar objects. Imagine a game with thousands of trees. Instead of creating separate objects for each tree, we share the common tree type (shape, texture, color) and only store unique details like position. This is great for large numbers of fine-grained objects.

Pros: Reduces memory footprint.  
Cons: Trades time for space (lookup overhead).  
When to use: In graphics, games, or text rendering (e.g., sharing character glyphs).  
Real-world analogy: Shared libraries in OS—code is shared among processes.

## Example

```ts
// Shared tree type (intrinsic state)
class TreeType {
  constructor(
    public name: string,
    public color: string,
    public texture: string
  ) {}

  display(x: number, y: number) {
    console.log(
      `Displaying ${this.name} tree at (${x}, ${y}) with color ${this.color}`
    );
  }
}

// Flyweight Factory to manage shared tree types
class TreeFactory {
  private static treeTypes: Map<string, TreeType> = new Map();

  static getTreeType(name: string, color: string, texture: string): TreeType {
    const key = `${name}_${color}_${texture}`;
    if (!this.treeTypes.has(key)) {
      this.treeTypes.set(key, new TreeType(name, color, texture));
    }
    return this.treeTypes.get(key)!;
  }
}

// Tree object with extrinsic state (position)
class Tree {
  constructor(private x: number, private y: number, private type: TreeType) {}

  display() {
    this.type.display(this.x, this.y);
  }
}

// Usage
const forest: Tree[] = [];

forest.push(new Tree(10, 20, TreeFactory.getTreeType("Oak", "Green", "Rough")));
forest.push(new Tree(15, 25, TreeFactory.getTreeType("Oak", "Green", "Rough"))); // Shares same type
forest.push(
  new Tree(50, 60, TreeFactory.getTreeType("Pine", "Dark Green", "Smooth"))
);

forest.forEach((tree) => tree.display());
```

Here are the simple TypeScript examples for each design pattern:

---

# Singleton

The Singleton pattern ensures that only one instance of a class exists and provides a global access point to it. It's useful for resources like configuration managers or database connections, but use sparingly to avoid global state issues.

Pros: Controlled access, reduces namespace pollution.  
Cons: Hard to test, can hide dependencies.  
When to use: For logging or caching services.

## Example

```ts
class Singleton {
  private static instance: Singleton;

  private constructor() {}

  static getInstance(): Singleton {
    if (!this.instance) {
      this.instance = new Singleton();
    }
    return this.instance;
  }
}

const s1 = Singleton.getInstance();
const s2 = Singleton.getInstance();
console.log(s1 === s2); // true
```

---

# Factory

The Factory pattern provides a method to create objects without specifying the exact class, delegating instantiation to subclasses. It's a creational pattern for object creation logic.

Pros: Hides creation details, promotes loose coupling.  
Cons: Can overcomplicate simple instantiation.  
When to use: When subclasses decide what to create, like document creators.

## Example

```ts
class Car {
  constructor(public model: string) {}
}

class CarFactory {
  static createCar(model: string): Car {
    return new Car(model);
  }
}

const car = CarFactory.createCar("Tesla");
console.log(car.model); // Tesla
```

---

# Abstract Factory

The Abstract Factory pattern creates families of related objects without specifying their concrete classes. It's useful for themes or platforms where consistent object groups are needed.

Pros: Ensures compatibility, easy to swap families.  
Cons: Hard to extend with new products.  
When to use: In cross-platform apps or widget kits.

## Example

```ts
interface Button {
  render(): void;
}

class WindowsButton implements Button {
  render() {
    console.log("Windows Button");
  }
}

class MacOSButton implements Button {
  render() {
    console.log("MacOS Button");
  }
}

interface GUIFactory {
  createButton(): Button;
}

class WindowsFactory implements GUIFactory {
  createButton(): Button {
    return new WindowsButton();
  }
}

class MacOSFactory implements GUIFactory {
  createButton(): Button {
    return new MacOSButton();
  }
}

const factory: GUIFactory = new WindowsFactory();
const button = factory.createButton();
button.render(); // Windows Button
```

---

# Builder

The Builder pattern constructs complex objects step by step, separating construction from representation. It's ideal for objects with many optional parameters.

Pros: Immutable objects, readable code.  
Cons: Requires a separate builder class.  
When to use: For configs or objects like strings with many options.

## Example

```ts
class Car {
  public engine?: string;
  public wheels?: number;
}

class CarBuilder {
  private car = new Car();

  setEngine(engine: string): this {
    this.car.engine = engine;
    return this;
  }

  setWheels(wheels: number): this {
    this.car.wheels = wheels;
    return this;
  }

  build(): Car {
    return this.car;
  }
}

const car = new CarBuilder().setEngine("V8").setWheels(4).build();
console.log(car);
```

# Prototype Pattern

The **Prototype Pattern** allows you to create new objects by copying an existing object (a prototype) instead of creating new instances from scratch. This is useful when object creation is expensive or complex (e.g., database queries for initialization).

Pros: Faster than new; hides complexity.  
Cons: Deep cloning can be tricky.  
When to use: In games for cloning enemies or in configs.  
Real-world analogy: Photocopying a document instead of rewriting it.

## Example

```ts
interface Prototype {
  clone(): Prototype;
}

class Car implements Prototype {
  constructor(public model: string, public color: string) {}

  clone(): Car {
    return new Car(this.model, this.color);
  }
}

const car1 = new Car("Tesla", "Red");
const car2 = car1.clone(); // Creates a new object with the same properties

console.log(car1 === car2); // false (different objects)
console.log(car1.model === car2.model); // true
console.log(car1.color === car2.color); // true
```

This pattern is useful when you want to duplicate objects while preserving their existing structure and state.

# The End

Thanks for reaching so far! 
