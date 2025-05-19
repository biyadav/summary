### Design Patterns in Java

Design patterns are established solutions to recurring design problems in software development. They provide templates for building robust,
maintainable, and scalable systems. Design patterns are categorized into three types:
	1. Creational Patterns - Deal with object creation. Singleton,Factory Pattern  ,Builder Pattern ,Prototype Pattern
	2. Structural Patterns - Deal with object composition. Decorator Pattern ,Adapter Pattern
	3. Behavioral Patterns - Deal with communication between objects. Observer Pattern ,Command Pattern,Strategy Pattern

## 1. Singleton Pattern (Creational)
Ensures that a class has only one instance and provides a global access point to it.

```
import java.util.Objects;
public class Singleton {
    private static Singleton instance;
private Singleton() {
        // Private constructor to prevent instantiation
    }
public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
public void showMessage() {
        System.out.println("Hello from Singleton!");
    }
}

```

## 2. Factory Pattern (Creational)
Provides an interface for creating objects but allows subclasses to alter the type of objects that will be created.

```	
import java.util.HashMap;
import java.util.Map;
interface Shape {
    void draw();
}
class Circle implements Shape {
    @Override
    public void draw() {
        System.out.println("Drawing a Circle");
    }
}
class Rectangle implements Shape {
    @Override
    public void draw() {
        System.out.println("Drawing a Rectangle");
    }
}
class ShapeFactory {
    public static Shape getShape(String shapeType) {
        return switch (shapeType.toLowerCase()) {
            case "circle" -> new Circle();
            case "rectangle" -> new Rectangle();
            default -> throw new IllegalArgumentException("Unknown shape type");
        };
    }
}

Usage:
Shape shape = ShapeFactory.getShape("circle");
shape.draw();

```

## 3. Builder Pattern (Creational)
Separates the construction of a complex object from its representation, allowing the same construction process to create different representations.

```
class Car {
    private String engine;
    private int wheels;
    private String color;
public static class Builder {
        private String engine;
        private int wheels;
        private String color;
public Builder setEngine(String engine) {
            this.engine = engine;
            return this;
        }
public Builder setWheels(int wheels) {
            this.wheels = wheels;
            return this;
        }
public Builder setColor(String color) {
            this.color = color;
            return this;
        }
public Car build() {
            Car car = new Car();
            car.engine = this.engine;
            car.wheels = this.wheels;
            car.color = this.color;
            return car;
        }
    }
@Override
    public String toString() {
        return "Car [engine=" + engine + ", wheels=" + wheels + ", color=" + color + "]";
    }
}
Usage:

Car car = new Car.Builder()
        .setEngine("V8")
        .setWheels(4)
        .setColor("Red")
        .build();
System.out.println(car);

```

## 4. Prototype Pattern (Creational)
Allows cloning of objects without coupling to their specific classes.

```
interface Prototype extends Cloneable {
    Prototype clone();
}
class ConcretePrototype implements Prototype {
    private String name;
public ConcretePrototype(String name) {
        this.name = name;
    }
@Override
    public Prototype clone() {
        try {
            return (Prototype) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
@Override
    public String toString() {
        return "ConcretePrototype{name='" + name + "'}";
    }
}
Usage:

ConcretePrototype original = new ConcretePrototype("Prototype 1");
ConcretePrototype clone = (ConcretePrototype) original.clone();
System.out.println(original);
System.out.println(clone);

```


## 5. Decorator Pattern (Structural)

Allows behavior to be added to an object dynamically.

```
interface Coffee {
    String getDescription();
    double getCost();
}
class BasicCoffee implements Coffee {
    @Override
    public String getDescription() {
        return "Basic Coffee";
    }
@Override
    public double getCost() {
        return 5.0;
    }
}
class MilkDecorator implements Coffee {
    private final Coffee coffee;
public MilkDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
@Override
    public String getDescription() {
        return coffee.getDescription() + ", Milk";
    }
@Override
    public double getCost() {
        return coffee.getCost() + 1.5;
    }
}
class SugarDecorator implements Coffee {
    private final Coffee coffee;
public SugarDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
@Override
    public String getDescription() {
        return coffee.getDescription() + ", Sugar";
    }
@Override
    public double getCost() {
        return coffee.getCost() + 0.5;
    }
}
Usage:

Coffee coffee = new SugarDecorator(new MilkDecorator(new BasicCoffee()));
System.out.println(coffee.getDescription() + ": $" + coffee.getCost());
```

## 6. Adapter Pattern (Structural)
Allows incompatible interfaces to work together by wrapping an object with a compatible interface.

```
interface MediaPlayer {
    void play(String audioType, String fileName);
}
class AdvancedMediaPlayer {
    public void playMp4(String fileName) {
        System.out.println("Playing MP4 file: " + fileName);
    }
public void playVlc(String fileName) {
        System.out.println("Playing VLC file: " + fileName);
    }
}
class MediaAdapter implements MediaPlayer {
    private final AdvancedMediaPlayer advancedMediaPlayer;
public MediaAdapter(String audioType) {
        advancedMediaPlayer = new AdvancedMediaPlayer();
    }
@Override
    public void play(String audioType, String fileName) {
        if ("mp4".equalsIgnoreCase(audioType)) {
            advancedMediaPlayer.playMp4(fileName);
        } else if ("vlc".equalsIgnoreCase(audioType)) {
            advancedMediaPlayer.playVlc(fileName);
        }
    }
}
Usage:

MediaPlayer player = new MediaAdapter("mp4");
player.play("mp4", "example.mp4");

```

## 7. Composite Pattern (Structural)
Composes objects into tree structures to represent part-whole hierarchies. Allows clients to treat individual objects and compositions uniformly.


```
import java.util.ArrayList;
import java.util.List;
interface Component {
    void showDetails();
}
class Leaf implements Component {
    private final String name;
public Leaf(String name) {
        this.name = name;
    }
@Override
    public void showDetails() {
        System.out.println(name);
    }
}
class Composite implements Component {
    private final List<Component> children = new ArrayList<>();
public void add(Component component) {
        children.add(component);
    }
public void remove(Component component) {
        children.remove(component);
    }
@Override
    public void showDetails() {
        for (Component child : children) {
            child.showDetails();
        }
    }
}
Usage:

Component leaf1 = new Leaf("Leaf 1");
Component leaf2 = new Leaf("Leaf 2");
Composite composite = new Composite();
composite.add(leaf1);
composite.add(leaf2);
composite.showDetails();
```

## 8. Observer Pattern (Behavioral)
Defines a one-to-many dependency between objects, so that when one object changes state, all its dependents are notified.

```
import java.util.ArrayList;
import java.util.List;
interface Observer {
    void update(String message);
}
class ConcreteObserver implements Observer {
    private final String name;
public ConcreteObserver(String name) {
        this.name = name;
    }
@Override
    public void update(String message) {
        System.out.println(name + " received: " + message);
    }
}
class Subject {
    private final List<Observer> observers = new ArrayList<>();
public void addObserver(Observer observer) {
        observers.add(observer);
    }
public void removeObserver(Observer observer) {
        observers.remove(observer);
    }
public void notifyObservers(String message) {
        for (Observer observer : observers) {
            observer.update(message);
        }
    }
}
Usage:

Subject subject = new Subject();
Observer observer1 = new ConcreteObserver("Observer 1");
Observer observer2 = new ConcreteObserver("Observer 2");
subject.addObserver(observer1);
subject.addObserver(observer2);
subject.notifyObservers("Hello Observers!");

```
Advantages of Design Patterns
	• Promote code reusability and readability.
	• Provide best practices for problem-solving.
	• Facilitate communication among developers by using a shared vocabulary.
Mastering these patterns can significantly improve your ability to write clean, maintainable, and scalable Java code.

## 9. Strategy Pattern (Behavioral)
Defines a family of algorithms, encapsulates each one, and makes them interchangeable.

```
interface PaymentStrategy {
    void pay(int amount);
}
class CreditCardPayment implements PaymentStrategy {
    @Override
    public void pay(int amount) {
        System.out.println("Paid $" + amount + " using Credit Card.");
    }
}
class PayPalPayment implements PaymentStrategy {
    @Override
    public void pay(int amount) {
        System.out.println("Paid $" + amount + " using PayPal.");
    }
}
class PaymentContext {
    private PaymentStrategy strategy;
public void setStrategy(PaymentStrategy strategy) {
        this.strategy = strategy;
    }
public void executePayment(int amount) {
        strategy.pay(amount);
    }
}
Usage:

PaymentContext context = new PaymentContext();
context.setStrategy(new CreditCardPayment());
context.executePayment(100);
context.setStrategy(new PayPalPayment());
context.executePayment(200);
```

## 10. Command Pattern (Behavioral)
Encapsulates a request as an object, thereby allowing parameterization of clients with queues, requests, and operations.

```
interface Command {
    void execute();
}
class Light {
    public void turnOn() {
        System.out.println("Light is ON");
    }
public void turnOff() {
        System.out.println("Light is OFF");
    }
}
class TurnOnCommand implements Command {
    private final Light light;
public TurnOnCommand(Light light) {
        this.light = light;
    }
@Override
    public void execute() {
        light.turnOn();
    }
}
class TurnOffCommand implements Command {
    private final Light light;
public TurnOffCommand(Light light) {
        this.light = light;
    }
@Override
    public void execute() {
        light.turnOff();
    }
}
Usage:

java
Copy code
Light light = new Light();
Command turnOn = new TurnOnCommand(light);
Command turnOff = new TurnOffCommand(light);
turnOn.execute();
turnOff.execute();

```

Key Takeaway
* By mastering design patterns, you can solve complex design problems effectively and write clean, maintainable, and reusable code.
* Each pattern serves a specific purpose, and the right choice depends on the use case.


![image](https://github.com/user-attachments/assets/d1f766b4-4953-4b85-80db-9b9d6fca862a)
