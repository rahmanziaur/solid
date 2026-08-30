<div align="center">

# 🏗️ SOLID — A Java Field Guide

### Five load-bearing principles, one running story, real code you can run in place.

[![Made with](https://img.shields.io/badge/made%20with-Java-1c3f8f?style=flat-square&logo=openjdk&logoColor=white)](#)
[![Format](https://img.shields.io/badge/format-single--page%20HTML-e2590c?style=flat-square)](#)
[![Runs in](https://img.shields.io/badge/code%20execution-Piston%20API-0f7a4a?style=flat-square)](#-run-the-code-in-place)
[![License](https://img.shields.io/badge/license-MIT-64748b?style=flat-square)](#-license)
[![Status](https://img.shields.io/badge/status-active-brightgreen?style=flat-square)](#)

**[📖 See this Tutorial here](http://rahmanziaur.github.io/solid)** · **[🌐 More Java on Design Pattern](https://rahmanziaur.github.io/Java)** · **[🐙 Author on GitHub](https://rahmanziaur.github.io/)**

</div>

<br>

<div align="center">
<img src="https://raw.githubusercontent.com/rahmanziaur/solid/refs/heads/main/solid.png" alt="SOLID Principles in Java — banner" width="820">
</div>

<br>

---
<div align="center">
🏗️ SOLID Principles in Java
Five load-bearing principles for object-oriented design — walked through with one running story and real, runnable Java.
![Language](https://img.shields.io/badge/language-Java-1c3f8f?style=flat-square&logo=openjdk&logoColor=white)
![Topic](https://img.shields.io/badge/topic-OOP%20Design-e2590c?style=flat-square)
![Author](https://img.shields.io/badge/principles%20by-Robert%20C.%20Martin-0f7a4a?style=flat-square)
![Reading time](https://img.shields.io/badge/reading%20time-~15%20min-64748b?style=flat-square)
</div>
<br>
> *"A class should have one, and only one, reason to change."*
> — Robert C. Martin
<br>
📐 What is SOLID?
SOLID is an acronym for five object-oriented design principles introduced by Robert C. Martin ("Uncle Bob"). They exist to keep software changeable without collapsing under its own weight — the way a building code keeps a structure standing as floors get added.
Letter	Principle	One-line rule
S	Single Responsibility Principle	One class, one reason to change.
O	Open/Closed Principle	Open for extension, closed for modification.
L	Liskov Substitution Principle	A subtype must be substitutable for its base type.
I	Interface Segregation Principle	Don't force clients to depend on methods they don't use.
D	Dependency Inversion Principle	Depend on abstractions, not concretions.
This document walks through all five using a single running story, with the full before/after Java code for each principle.
<br>
🧵 The story: Ironclad Retail
> **Ironclad Retail** is a small team shipping an order-processing platform in Java. Their engineer, **Maya**, is about to spend a sprint fighting a codebase that fights back: one class does five jobs, adding a discount means editing code that already works, and a "helpful" subclass quietly breaks the shipping calculator.
>
> None of this is a language problem — it's a *design* problem. SOLID is Robert C. Martin's set of five principles for keeping object-oriented code changeable without it collapsing under its own weight. We'll follow Maya through all five, one refactor at a time, with the actual Java before and after.
<br>
📑 Table of contents
S — Single Responsibility Principle
O — Open/Closed Principle
L — Liskov Substitution Principle
I — Interface Segregation Principle
D — Dependency Inversion Principle
Quick reference
Interview questions that span all of SOLID
<br>
---
🅢 S — Single Responsibility Principle
> **A class should have only one job — and therefore only one reason to be modified.**
🧵 Scenario — Ironclad Retail, Sprint 1
Maya's `Invoice` class calculates the order total and prints it and writes it to a file. When Finance asks for a PDF export, she has to reopen a class that already correctly calculates tax — and risks breaking it while she's in there for an unrelated reason.
<table>
<tr><th>❌ Before — violates SRP</th><th>✅ After — follows SRP</th></tr>
<tr valign="top">
<td>
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

    // Reason to change #1: the tax rules change
    public double calculateTotal(double taxRate) {
        return amount + (amount * taxRate);
    }

    // Reason to change #2: the console format changes
    public void printInvoice(double taxRate) {
        System.out.println("Invoice for: " + customer);
        System.out.println("Total: $" + calculateTotal(taxRate));
    }

    // Reason to change #3: how/where we persist invoices changes
    public void saveToFile(double taxRate) throws IOException {
        FileWriter writer = new FileWriter(customer + "_invoice.txt");
        writer.write("Total: $" + calculateTotal(taxRate));
        writer.close();
    }

    public static void main(String[] args) {
        InvoiceBad invoice = new InvoiceBad("Maya's Cafe", 250.00);
        invoice.printInvoice(0.08);
        // Adding PDF export means opening this same class again.
    }
}
```
</td>
<td>
```java
class Invoice {
    private final String customer;
    private final double amount;

    Invoice(String customer, double amount) {
        this.customer = customer;
        this.amount = amount;
    }

    String getCustomer() { return customer; }
    double getAmount() { return amount; }
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
        System.out.println("Total: $" + calculator.calculateTotal(invoice, taxRate));
    }
}

public class Main {
    public static void main(String[] args) {
        Invoice invoice = new Invoice("Maya's Cafe", 250.00);
        InvoiceCalculator calculator = new InvoiceCalculator();
        InvoicePrinter printer = new InvoicePrinter(calculator);

        printer.print(invoice, 0.08);
        // A future InvoiceExporter (PDF, CSV, ...) slots in beside
        // these classes -- none of them need to change.
    }
}
```
</td>
</tr>
</table>
> 💡 **Why it matters:** `Invoice`, `InvoiceCalculator`, and `InvoicePrinter` each answer to one stakeholder — data, math, and formatting change for different reasons and now change independently.
🎤 Interview questions — SRP
<details>
<summary><b>What does "a reason to change" actually mean?</b></summary>
<br>
It refers to a single stakeholder or business concern. If two different people (say, an accountant and a UI designer) could each ask for a change to the same class for unrelated reasons, that class is carrying two responsibilities.
</details>
<details>
<summary><b>Doesn't splitting classes just create clutter?</b></summary>
<br>
More classes isn't automatically better — the goal is cohesion, not a headcount of files. Split when responsibilities genuinely change at different rates or for different stakeholders; keep things together when they always change in lockstep.
</details>
<details>
<summary><b>How would you spot an SRP violation in code review?</b></summary>
<br>
Look for the word "and" in a class's description, mixed levels of abstraction inside one method, imports pulling in unrelated concerns (e.g. a domain class importing a file-IO or HTTP library), and a change history where unrelated features keep touching the same file.
</details>
<br>
---
🅞 O — Open/Closed Principle
> **You should be able to add new behavior without touching code that already works and is already tested.**
🧵 Scenario — Ironclad Retail, Sprint 2
Every quarter, marketing invents a new discount tier. Maya's `DiscountCalculator` is one long `if/else` chain, so "add a Student discount" really means "edit and re-test a class every other feature already depends on."
<table>
<tr><th>❌ Before — violates OCP</th><th>✅ After — follows OCP</th></tr>
<tr valign="top">
<td>
```java
public class DiscountCalculatorBad {

    public double calculateDiscount(String customerType, double amount) {
        if (customerType.equals("Regular")) {
            return amount * 0.05;
        } else if (customerType.equals("Premium")) {
            return amount * 0.15;
        }
        // Every new tier means another "else if" wedged in here,
        // in a class every checkout flow already relies on.
        return 0;
    }

    public static void main(String[] args) {
        DiscountCalculatorBad calc = new DiscountCalculatorBad();
        System.out.println("Regular: $" + calc.calculateDiscount("Regular", 200));
        System.out.println("Premium: $" + calc.calculateDiscount("Premium", 200));
    }
}
```
</td>
<td>
```java
interface Discount {
    double apply(double amount);
}

class RegularDiscount implements Discount {
    public double apply(double amount) { return amount * 0.05; }
}

class PremiumDiscount implements Discount {
    public double apply(double amount) { return amount * 0.15; }
}

// New tier: a new class, zero edits to existing ones.
class StudentDiscount implements Discount {
    public double apply(double amount) { return amount * 0.10; }
}

public class Main {
    public static void main(String[] args) {
        Discount[] tiers = {
            new RegularDiscount(), new PremiumDiscount(), new StudentDiscount()
        };
        for (Discount tier : tiers) {
            System.out.println(tier.getClass().getSimpleName() + ": $" + tier.apply(200));
        }
    }
}
```
</td>
</tr>
</table>
> 💡 **Why it matters:** `StudentDiscount` is a brand-new class implementing `Discount` — the checkout code that loops over discounts never had to change, so it never had to be re-tested.
🎤 Interview questions — OCP
<details>
<summary><b>How do you achieve OCP without modifying tested code?</b></summary>
<br>
Program to an abstraction (an interface or abstract class) and use polymorphism. New behavior becomes a new implementation of that abstraction, plugged in wherever the abstraction is used — nothing about the existing call sites has to change.
</details>
<details>
<summary><b>Which design patterns embody OCP?</b></summary>
<br>
Strategy (swap the algorithm, as above), Decorator (add behavior by wrapping), Template Method (extend a fixed skeleton), and plugin/factory-based architectures are all direct applications of OCP.
</details>
<details>
<summary><b>Can you over-apply OCP?</b></summary>
<br>
Yes. Building an abstraction layer for a variation that will never actually happen adds indirection and complexity for no benefit. OCP is worth paying for where change is likely (as with discount tiers); it's over-engineering where it isn't.
</details>
<br>
---
🅛 L — Liskov Substitution Principle
> **If `S` is a subtype of `T`, you should be able to swap any `T` for an `S` without breaking the program's correctness.**
🧵 Scenario — Ironclad Retail, Sprint 3
The warehouse's box-packing calculator takes any `Rectangle` and computes its area. A junior dev adds `Square extends Rectangle`, overriding `setWidth`/`setHeight` so both sides stay equal. It compiles fine — and then silently mis-sizes every square box on the floor.
<table>
<tr><th>❌ Before — violates LSP</th><th>✅ After — follows LSP</th></tr>
<tr valign="top">
<td>
```java
class RectangleBad {
    protected int width;
    protected int height;
    void setWidth(int width) { this.width = width; }
    void setHeight(int height) { this.height = height; }
    int getArea() { return width * height; }
}

class SquareBad extends RectangleBad {
    @Override
    void setWidth(int width) { this.width = width; this.height = width; }
    @Override
    void setHeight(int height) { this.width = height; this.height = height; }
}

public class Main {
    static void packBox(RectangleBad box) {
        box.setWidth(5);
        box.setHeight(10);
        System.out.println("Expected area 50, got: " + box.getArea());
    }

    public static void main(String[] args) {
        packBox(new RectangleBad()); // 50 -- correct
        packBox(new SquareBad());    // 100 -- silently wrong box size
    }
}
```
</td>
<td>
```java
interface Shape {
    int getArea();
}

class Rectangle implements Shape {
    private final int width, height;
    Rectangle(int width, int height) { this.width = width; this.height = height; }
    public int getArea() { return width * height; }
}

class Square implements Shape {
    private final int side;
    Square(int side) { this.side = side; }
    public int getArea() { return side * side; }
}

public class Main {
    static void reportArea(Shape shape) {
        System.out.println("Area: " + shape.getArea());
    }

    public static void main(String[] args) {
        reportArea(new Rectangle(5, 10)); // 50
        reportArea(new Square(5));        // 25 -- correct, no surprises
    }
}
```
</td>
</tr>
</table>
> 💡 **Why it matters:** `Square` no longer pretends to be a mutable `Rectangle` it isn't. Both shapes just promise `getArea()`, so any code using `Shape` behaves correctly no matter which one it's handed.
🎤 Interview questions — LSP
<details>
<summary><b>Why does Square-extends-Rectangle break LSP if it compiles fine?</b></summary>
<br>
LSP is about behavioral contracts, not just method signatures. <code>Rectangle</code>'s implicit contract is "width and height vary independently." <code>Square</code> can't honor that without breaking its own invariant, so it changes the meaning of <code>setWidth</code>/<code>setHeight</code> for every caller that trusted the base class.
</details>
<details>
<summary><b>What are common code smells for LSP violations?</b></summary>
<br>
An overridden method that throws <code>UnsupportedOperationException</code>, an override that does nothing, a subclass that narrows an accepted input range or widens what it throws, or client code doing <code>instanceof</code> checks before calling a method are all red flags.
</details>
<details>
<summary><b>Why doesn't the Java compiler catch this?</b></summary>
<br>
The compiler only checks type signatures — that <code>Square</code> has methods matching <code>Rectangle</code>'s. It has no idea about behavioral invariants like "changing width shouldn't change height," so a technically valid override can still violate the principle.
</details>
<br>
---
🅘 I — Interface Segregation Principle
> **Prefer several small, specific interfaces over one large, general-purpose one — clients should only see the methods that are relevant to them.**
🧵 Scenario — Ironclad Retail, Sprint 4
The warehouse automation team defines one fat `WarehouseRobot` interface with `pack()`, `ship()`, and `refrigerate()`. Their cheapest robot only packs — but it's still forced to "implement" the other two, throwing exceptions the moment anyone calls them.
<table>
<tr><th>❌ Before — violates ISP</th><th>✅ After — follows ISP</th></tr>
<tr valign="top">
<td>
```java
interface WarehouseRobot {
    void pack();
    void ship();
    void refrigerate();
}

class PackingRobot implements WarehouseRobot {
    public void pack() { System.out.println("Packing order..."); }
    public void ship() {
        throw new UnsupportedOperationException("PackingRobot can't ship");
    }
    public void refrigerate() {
        throw new UnsupportedOperationException("PackingRobot can't refrigerate");
    }
}

public class Main {
    public static void main(String[] args) {
        PackingRobot robot = new PackingRobot();
        robot.pack();
        robot.ship(); // crashes at runtime
    }
}
```
</td>
<td>
```java
interface Packable { void pack(); }
interface Shippable { void ship(); }
interface Refrigeratable { void refrigerate(); }

class PackingRobot implements Packable {
    public void pack() { System.out.println("Packing order..."); }
}

class ColdChainRobot implements Packable, Refrigeratable {
    public void pack() { System.out.println("Packing frozen order..."); }
    public void refrigerate() { System.out.println("Holding order at -18C..."); }
}

public class Main {
    public static void main(String[] args) {
        PackingRobot packer = new PackingRobot();
        packer.pack();

        ColdChainRobot coldRobot = new ColdChainRobot();
        coldRobot.pack();
        coldRobot.refrigerate();
        // Neither robot is forced to implement ship(), which it never needs.
    }
}
```
</td>
</tr>
</table>
> 💡 **Why it matters:** Each robot class now implements only the roles it actually performs. No exception-throwing stubs, and no risk of a caller invoking a method that was never really supported.
🎤 Interview questions — ISP
<details>
<summary><b>How is ISP different from SRP?</b></summary>
<br>
SRP is about a class's own reasons to change. ISP is about the shape of the contract a class exposes to its clients. A class can have a single, well-defined responsibility internally and still force clients to depend on a bloated interface — that's an ISP problem, not necessarily an SRP one.
</details>
<details>
<summary><b>What's a warning sign of an ISP violation?</b></summary>
<br>
Implementations with empty method bodies or methods that immediately throw an exception are the classic tell — the interface is asking for more than that implementer can honestly provide.
</details>
<details>
<summary><b>Can ISP be taken too far?</b></summary>
<br>
Yes — splitting into one interface per method can cause "interface explosion" and scatter a naturally cohesive role across too many types. Group methods that genuinely belong to the same client role, and split only where different clients need different subsets.
</details>
<br>
---
🅓 D — Dependency Inversion Principle
> **High-level modules shouldn't depend on low-level modules — both should depend on abstractions, and abstractions shouldn't depend on details.**
🧵 Scenario — Ironclad Retail, Sprint 5
`OrderNotifier` builds an `EmailSender` straight inside itself. When Support asks for SMS alerts too, Maya can't just add a channel — she has to reopen the notifier's core logic to wire in a second concrete class.
<table>
<tr><th>❌ Before — violates DIP</th><th>✅ After — follows DIP</th></tr>
<tr valign="top">
<td>
```java
class EmailSender {
    void send(String message) { System.out.println("Email sent: " + message); }
}

class OrderNotifierBad {
    private final EmailSender sender = new EmailSender(); // hard-wired

    void notifyCustomer(String orderId) {
        sender.send("Order " + orderId + " has shipped!");
    }
}

public class Main {
    public static void main(String[] args) {
        OrderNotifierBad notifier = new OrderNotifierBad();
        notifier.notifyCustomer("A1001");
        // Adding SMS means editing OrderNotifierBad's internals directly.
    }
}
```
</td>
<td>
```java
interface NotificationChannel {
    void send(String message);
}

class EmailChannel implements NotificationChannel {
    public void send(String message) { System.out.println("Email sent: " + message); }
}

class SmsChannel implements NotificationChannel {
    public void send(String message) { System.out.println("SMS sent: " + message); }
}

class OrderNotifier {
    private final NotificationChannel channel;
    OrderNotifier(NotificationChannel channel) { this.channel = channel; }

    void notifyCustomer(String orderId) {
        channel.send("Order " + orderId + " has shipped!");
    }
}

public class Main {
    public static void main(String[] args) {
        OrderNotifier emailNotifier = new OrderNotifier(new EmailChannel());
        emailNotifier.notifyCustomer("A1001");

        OrderNotifier smsNotifier = new OrderNotifier(new SmsChannel());
        smsNotifier.notifyCustomer("A1002");
        // OrderNotifier itself never changes, no matter how many channels exist.
    }
}
```
</td>
</tr>
</table>
> 💡 **Why it matters:** `OrderNotifier` now depends on the `NotificationChannel` abstraction, injected through its constructor. Both email and SMS channels depend on that same abstraction — neither depends on the other.
🎤 Interview questions — DIP
<details>
<summary><b>Dependency Inversion vs Dependency Injection — what's the difference?</b></summary>
<br>
DIP is the design principle: depend on abstractions, not concrete classes. Dependency Injection is a technique for satisfying that principle — supplying a concrete implementation from the outside (constructor, setter, or a framework like Spring) instead of the class constructing it itself.
</details>
<details>
<summary><b>Does DIP mean high-level code should never call "new"?</b></summary>
<br>
Not literally everywhere — something has to construct the concrete objects. The point is that your core business logic shouldn't be the thing doing it. That wiring is usually pushed to a "composition root" (a <code>main</code> method, a factory, or a DI framework) so the business classes only ever see interfaces.
</details>
<details>
<summary><b>How does DIP help with unit testing?</b></summary>
<br>
Because <code>OrderNotifier</code> depends on the <code>NotificationChannel</code> interface, a test can inject a fake channel that records calls in memory — no real email/SMS provider, network call, or flakiness required.
</details>
<br>
---
🗂️ Quick reference
	Principle	One-line rule	Typical fix
S	Single Responsibility	One class, one reason to change.	Split mixed concerns into focused classes.
O	Open/Closed	Add new behavior without editing old code.	Code to an interface; add new implementations.
L	Liskov Substitution	A subtype must honor its base type's contract.	Model by shared interface, not forced inheritance.
I	Interface Segregation	Don't force clients to depend on unused methods.	Split fat interfaces into role-based ones.
D	Dependency Inversion	Depend on abstractions, not concrete classes.	Inject an interface instead of constructing one.
<br>
💬 Interview questions that span all of SOLID
<details>
<summary><b>Are the SOLID principles always the right call?</b></summary>
<br>
No — they're a tool for managing change, not a checklist to max out. A one-off script or a class that genuinely never changes doesn't need the same abstraction layers as a core domain model. Applying SOLID where nothing is expected to vary is over-engineering; the skill is judging what's actually likely to change.
</details>
<details>
<summary><b>How do design patterns relate to SOLID?</b></summary>
<br>
Most classic Gang-of-Four patterns are concrete recipes for satisfying one or more SOLID principles: Strategy and Decorator support OCP, Adapter and Dependency Injection support DIP, and interface-based Observer supports both OCP and ISP. Knowing SOLID makes it obvious <i>why</i> those patterns are shaped the way they are.
</details>
<details>
<summary><b>Which principle is hardest to apply correctly, and why?</b></summary>
<br>
OCP and LSP tend to be the trickiest, because both require anticipating how the code will need to change or be extended later — get the abstraction boundary wrong and you either over-build for change that never comes, or under-build and violate LSP the first time someone extends it.
</details>
<details>
<summary><b>Can you have well-designed OOP code that violates SOLID?</b></summary>
<br>
Yes, in the sense that SOLID isn't a certification — it's a set of heuristics for a specific goal: code that's easy to extend and safe to change. Small, stable, rarely-touched code can "violate" SOLID and still be perfectly fine; the principles earn their keep as software grows and changes.
</details>
<br>
---
<div align="center">
🔗 Keep going
📘 More Java notes — rahmanziaur.github.io/java · 🐙 SOLID-Principle-Java on GitHub
<sub>SOLID principles by Robert C. Martin · Java examples and story by this guide</sub>
</div>
