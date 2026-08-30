# SOLID — A Java Field Guide

A practical guide to the five SOLID principles of object-oriented design, with Java examples, refactoring scenarios, and interview questions.

- **Tutorial:** http://rahmanziaur.github.io/solid
- **Author:** https://github.com/rahmanziaur

## Overview

SOLID is an acronym for five object-oriented design principles introduced by Robert C. Martin. They help make software easier to understand, extend, test, and maintain.

| Principle | Core idea |
|---|---|
| **S — Single Responsibility Principle** | A class should have one responsibility and one reason to change. |
| **O — Open/Closed Principle** | Software should be open for extension but closed for modification. |
| **L — Liskov Substitution Principle** | A subtype should be safely substitutable for its base type. |
| **I — Interface Segregation Principle** | Clients should not be forced to depend on methods they do not use. |
| **D — Dependency Inversion Principle** | High-level code should depend on abstractions rather than concrete implementations. |

This guide uses a single running example, **Ironclad Retail**, to demonstrate each principle through a problematic design and a refactored design.

## Table of Contents

- [S — Single Responsibility Principle](#s--single-responsibility-principle)
- [O — Open/Closed Principle](#o--openclosed-principle)
- [L — Liskov Substitution Principle](#l--liskov-substitution-principle)
- [I — Interface Segregation Principle](#i--interface-segregation-principle)
- [D — Dependency Inversion Principle](#d--dependency-inversion-principle)
- [Quick Reference](#quick-reference)
- [Interview Questions](#interview-questions)
- [Resources](#resources)
- [About the Author](#about-the-author)

---

## S — Single Responsibility Principle

> **A class should have only one job — and therefore only one reason to be modified.**

### Scenario

In Ironclad Retail, the `Invoice` class calculates the order total, prints the invoice, and saves it to a file. These responsibilities change for different reasons.

### Before: SRP Violation

```java
import java.io.FileWriter;
import java.io.IOException;

public class InvoiceBad {
    private String customer;
    private double amount;

    public InvoiceBad(String customer, double amount) {
        this.customer = customer;
        this.amount = amount;
    }

    public double calculateTotal(double taxRate) {
        return amount + (amount * taxRate);
    }

    public void printInvoice(double taxRate) {
        System.out.println("Invoice for: " + customer);
        System.out.println("Total: $" + calculateTotal(taxRate));
    }

    public void saveToFile(double taxRate) throws IOException {
        FileWriter writer = new FileWriter(customer + "_invoice.txt");
        writer.write("Total: $" + calculateTotal(taxRate));
        writer.close();
    }
}
```

### After: Applying SRP

```java
class Invoice {
    private final String customer;
    private final double amount;

    Invoice(String customer, double amount) {
        this.customer = customer;
        this.amount = amount;
    }

    String getCustomer() {
        return customer;
    }

    double getAmount() {
        return amount;
    }
}

class InvoiceCalculator {
    double calculateTotal(Invoice invoice, double taxRate) {
        return invoice.getAmount() + (invoice.getAmount() * taxRate);
    }
}

class InvoicePrinter {
    private final InvoiceCalculator calculator;

    InvoicePrinter(InvoiceCalculator calculator) {
        this.calculator = calculator;
    }

    void print(Invoice invoice, double taxRate) {
        System.out.println("Invoice for: " + invoice.getCustomer());
        System.out.println(
            "Total: $" + calculator.calculateTotal(invoice, taxRate)
        );
    }
}
```

**Why it matters:** Data, calculation, and presentation can now change independently.

### Interview Questions

**What does "a reason to change" mean?**  
It refers to a distinct stakeholder or business concern. If unrelated stakeholders can request changes to the same class, the class may have multiple responsibilities.

**Does splitting classes create unnecessary clutter?**  
Not necessarily. The goal is high cohesion, not simply having more classes. Split responsibilities when they change independently.

**How can an SRP violation be spotted in code review?**  
Look for unrelated responsibilities, mixed levels of abstraction, unrelated imports, and frequent changes to the same class for different features.

---

## O — Open/Closed Principle

> **Software should be open for extension but closed for modification.**

### Scenario

Ironclad Retail introduces new discount tiers regularly. A long `if/else` chain requires modifying existing checkout logic whenever a new discount is introduced.

### Before: OCP Violation

```java
public class DiscountCalculatorBad {

    public double calculateDiscount(String customerType, double amount) {
        if (customerType.equals("Regular")) {
            return amount * 0.05;
        } else if (customerType.equals("Premium")) {
            return amount * 0.15;
        }

        return 0;
    }
}
```

### After: Applying OCP

```java
interface Discount {
    double apply(double amount);
}

class RegularDiscount implements Discount {
    public double apply(double amount) {
        return amount * 0.05;
    }
}

class PremiumDiscount implements Discount {
    public double apply(double amount) {
        return amount * 0.15;
    }
}

class StudentDiscount implements Discount {
    public double apply(double amount) {
        return amount * 0.10;
    }
}

public class Main {
    public static void main(String[] args) {
        Discount[] tiers = {
            new RegularDiscount(),
            new PremiumDiscount(),
            new StudentDiscount()
        };

        for (Discount tier : tiers) {
            System.out.println(
                tier.getClass().getSimpleName() + ": $" + tier.apply(200)
            );
        }
    }
}
```

**Why it matters:** A new discount can be added as a new implementation without changing the existing discount-processing logic.

### Interview Questions

**How can OCP be achieved?**  
Use abstractions such as interfaces or abstract classes together with polymorphism.

**Which design patterns support OCP?**  
Strategy, Decorator, Template Method, and plugin/factory-based designs can support OCP.

**Can OCP be over-applied?**  
Yes. Creating abstractions for changes that are unlikely to occur can add unnecessary complexity.

---

## L — Liskov Substitution Principle

> **If `S` is a subtype of `T`, objects of type `S` should be usable wherever objects of type `T` are expected without breaking correctness.**

### Scenario

A warehouse application uses a mutable `Rectangle`. A `Square` subclass changes the behavior of `setWidth()` and `setHeight()` so that both dimensions remain equal. This violates the assumptions made by code using `Rectangle`.

### Before: LSP Violation

```java
class RectangleBad {
    protected int width;
    protected int height;

    void setWidth(int width) {
        this.width = width;
    }

    void setHeight(int height) {
        this.height = height;
    }

    int getArea() {
        return width * height;
    }
}

class SquareBad extends RectangleBad {
    @Override
    void setWidth(int width) {
        this.width = width;
        this.height = width;
    }

    @Override
    void setHeight(int height) {
        this.width = height;
        this.height = height;
    }
}
```

### After: Applying LSP

```java
interface Shape {
    int getArea();
}

class Rectangle implements Shape {
    private final int width;
    private final int height;

    Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }

    public int getArea() {
        return width * height;
    }
}

class Square implements Shape {
    private final int side;

    Square(int side) {
        this.side = side;
    }

    public int getArea() {
        return side * side;
    }
}

public class Main {
    static void reportArea(Shape shape) {
        System.out.println("Area: " + shape.getArea());
    }

    public static void main(String[] args) {
        reportArea(new Rectangle(5, 10));
        reportArea(new Square(5));
    }
}
```

**Why it matters:** `Rectangle` and `Square` no longer use an inappropriate inheritance relationship. Both satisfy the smaller `Shape` contract.

### Interview Questions

**Why can the Square/Rectangle example violate LSP even though it compiles?**  
LSP concerns behavioral contracts, not just method signatures. A mutable rectangle implies independently changeable width and height, while a square cannot honor that contract.

**What are common LSP warning signs?**  
Unsupported operations in overrides, empty overrides, changed input expectations, and client code that needs `instanceof` checks can indicate a problem.

**Why does the Java compiler not catch every LSP violation?**  
The compiler checks type compatibility and method signatures, but it cannot generally determine whether behavioral contracts are preserved.

---

## I — Interface Segregation Principle

> **Prefer several small, specific interfaces over one large, general-purpose interface.**

### Scenario

A warehouse defines one `WarehouseRobot` interface containing `pack()`, `ship()`, and `refrigerate()`. A packing-only robot is forced to implement operations it does not support.

### Before: ISP Violation

```java
interface WarehouseRobot {
    void pack();
    void ship();
    void refrigerate();
}

class PackingRobot implements WarehouseRobot {
    public void pack() {
        System.out.println("Packing order...");
    }

    public void ship() {
        throw new UnsupportedOperationException(
            "PackingRobot can't ship"
        );
    }

    public void refrigerate() {
        throw new UnsupportedOperationException(
            "PackingRobot can't refrigerate"
        );
    }
}
```

### After: Applying ISP

```java
interface Packable {
    void pack();
}

interface Shippable {
    void ship();
}

interface Refrigeratable {
    void refrigerate();
}

class PackingRobot implements Packable {
    public void pack() {
        System.out.println("Packing order...");
    }
}

class ColdChainRobot implements Packable, Refrigeratable {
    public void pack() {
        System.out.println("Packing frozen order...");
    }

    public void refrigerate() {
        System.out.println("Holding order at -18C...");
    }
}
```

**Why it matters:** Each implementation depends only on the capabilities it actually provides.

### Interview Questions

**How is ISP different from SRP?**  
SRP focuses on a class's reasons to change. ISP focuses on avoiding unnecessarily large interfaces and unnecessary dependencies for clients.

**What is a common warning sign of ISP violations?**  
Implementations containing empty methods or methods that immediately throw exceptions often indicate an overly broad interface.

**Can ISP be taken too far?**  
Yes. Excessive splitting can create unnecessary interfaces. Group methods that genuinely belong to the same client role.

---

## D — Dependency Inversion Principle

> **High-level modules should not depend directly on low-level modules. Both should depend on abstractions.**

### Scenario

`OrderNotifier` directly creates an `EmailSender`. Adding another notification channel requires changing the notifier itself.

### Before: DIP Violation

```java
class EmailSender {
    void send(String message) {
        System.out.println("Email sent: " + message);
    }
}

class OrderNotifierBad {
    private final EmailSender sender = new EmailSender();

    void notifyCustomer(String orderId) {
        sender.send("Order " + orderId + " has shipped!");
    }
}
```

### After: Applying DIP

```java
interface NotificationChannel {
    void send(String message);
}

class EmailChannel implements NotificationChannel {
    public void send(String message) {
        System.out.println("Email sent: " + message);
    }
}

class SmsChannel implements NotificationChannel {
    public void send(String message) {
        System.out.println("SMS sent: " + message);
    }
}

class OrderNotifier {
    private final NotificationChannel channel;

    OrderNotifier(NotificationChannel channel) {
        this.channel = channel;
    }

    void notifyCustomer(String orderId) {
        channel.send("Order " + orderId + " has shipped!");
    }
}

public class Main {
    public static void main(String[] args) {
        OrderNotifier emailNotifier =
            new OrderNotifier(new EmailChannel());

        emailNotifier.notifyCustomer("A1001");

        OrderNotifier smsNotifier =
            new OrderNotifier(new SmsChannel());

        smsNotifier.notifyCustomer("A1002");
    }
}
```

**Why it matters:** `OrderNotifier` depends on the `NotificationChannel` abstraction, making different notification mechanisms interchangeable and easier to test.

### Interview Questions

**What is the difference between Dependency Inversion and Dependency Injection?**  
DIP is a design principle. Dependency Injection is a technique for supplying the required implementation from outside the dependent class.

**Does DIP mean high-level code should never use `new`?**  
No. Concrete objects still need to be created somewhere. The important point is to keep construction of implementation details outside core business logic where practical.

**How does DIP help unit testing?**  
An interface allows tests to inject a fake or test implementation without requiring real external services.

---

## Quick Reference

| Principle | Main Problem | Typical Solution |
|---|---|---|
| **S — Single Responsibility** | A class has multiple unrelated responsibilities. | Separate responsibilities into focused classes. |
| **O — Open/Closed** | New behavior requires modifying existing code. | Use abstractions and polymorphism. |
| **L — Liskov Substitution** | A subtype breaks expectations of its base type. | Model compatible behavior through appropriate abstractions. |
| **I — Interface Segregation** | Clients depend on methods they do not need. | Create focused, role-based interfaces. |
| **D — Dependency Inversion** | High-level code depends directly on implementation details. | Depend on abstractions and inject implementations. |

## Interview Questions

### Are SOLID principles always the right choice?

No. SOLID principles are guidelines for managing change, not a checklist. Applying unnecessary abstractions to small or stable code can create over-engineering.

### How do design patterns relate to SOLID?

Many Gang of Four design patterns provide practical ways to address SOLID-related design concerns. For example, Strategy and Decorator can support OCP, while dependency injection supports DIP.

### Which principles can be difficult to apply?

OCP and LSP often require careful decisions about abstraction boundaries and expected future changes.

### Can good object-oriented code violate SOLID?

Yes. SOLID is a set of design heuristics rather than a strict certification of good code. The appropriate design depends on the system, its expected changes, and its complexity.

## Resources

- [SOLID Principles — Full Tutorial](http://rahmanziaur.github.io/solid)
- [Gang of Four Design Patterns in Java](https://rahmanziaur.github.io/Java/)
- [Java Multi-Threaded Programming](https://rahmanziaur.github.io/Java/)

## About the Author

**Ziaur Rahman, PhD**  
Full Professor, Department of ICT  
Mawlana Bhashani Science & Technology University

Research and teaching interests include Java, object-oriented design, design patterns, and multi-threaded programming.

- [Website](https://rahmanziaur.github.io/)
- [GitHub](https://github.com/rahmanziaur)

---

**SOLID principles by Robert C. Martin · Java examples and explanations by Ziaur Rahman**
