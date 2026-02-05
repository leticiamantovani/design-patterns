The factory method is a creational pattern that delegates the responsibility of object instantiation to its concrete subclasses.

Motivation
Imagine a drive-thru where customers order meals by number, such as "Combo 1". Customers don't specify ingredients or preparation details; they trust the kitchen to handle that.

But how do we design this from a software perspective, such that we create a flexible and loosely coupled design?

Let's take a look at two candidate solutions:

1. Direct Instantiation by the client: At first glance, one way we can handle this is direct instantiation. Imagine creating burger objects based on multiple conditionals:

```javascript
// ...
let burger;

if (TYPES.CHEESE) {
	burger = new CheeseBurger();
} else if (TYPES.DELUXE) {
	burger = new DeluxeCheeseBurger();
} else if (TYPES.VEGAN) {
	burger = new VeganBurger();
}
// ...
```

This does work, but what happens when we introduce a new burger type or a limited-time special?

Each addition requires us to modify this logic. This isn't just tedious but also risks introducing errors, particularly if this decision-making code sprawls across the application.

We have discussed "program to an interface and not to an implementation" before, meaning that clients should remain unaware of the specific types of objects they use, as long as the objects adhere to the interface that they expect.

While the code above adheres to this principle, we are still limiting ourselves to the concrete implementation when we perform instantiation. This direct instantiation within the client code is brittle. It makes the system less adaptable to change and increases the chances of mistakes.

We also violate the single-responsibility principle because we are handling the logic of cooking, preparing the drink and the side, all together in one class. If we make a mistake updating any of the conditional code, we might end up taking down the entire burger class. In a real kitchen, the cashier does not decide how to cook the burger or what ingredients to use. The same should be the case with our code.

Taking into consideration the pitfalls of this approach, let's address them in the next candidate solution.

2. Encapsulate what varies using a simple factory: We know that the burger menu may evolve over time, but we would prefer not to directly modify all of the parts of code that instantiate a burger.
   Taking the design principle, "encapsulate what varies" into consideration, we can create a class called SimpleBurgerFactory and isolate the burger creation logic to a method called createBurger.

```javascript
const BurgerType = Object.freeze({
	CHEESEBURGER: "CHEESEBURGER",
	DELUXE_CHEESEBURGER: "DELUXE_CHEESEBURGER",
	VEGANBURGER: "VEGANBURGER",
});

class SimpleBurgerFactory {
	createBurger(type) {
		let burger = null;
		switch (type) {
			case BurgerType.CHEESEBURGER:
				burger = new CheeseBurger();
				break;
			case BurgerType.DELUXE_CHEESEBURGER:
				burger = new DeluxeCheeseBurger();
				break;
			case BurgerType.VEGANBURGER:
				burger = new VeganBurger();
				break;
		}
		return burger;
	}
}
```

This is the essence of the simple factory pattern - a class dedicated to producing objects based on conditions.
The above solution still violates the open/closed principle but it also violates the dependency inversion principle. The simple factory has direct dependencies on the concrete classes that it is creating. And while the simple factory is abstracting away the creation process, the product type still needs to be provided to the factory.

The solution
The Factory Method pattern improves upon the simple factory by promoting loose coupling and adhering to the open/closed principle.

1. Building on the Simple Factory

While the simple factory centralized the creation process in one class, the Factory Method pattern defers the instantiation process to subclasses.

If we introduce a new burger type, for instance, we just need to extend our base factory class and provide the creation logic specific to that new type — all without touching existing code. This will make more sense soon.

2. Promoting the Open/Closed Principle

One of the core strengths of the Factory Method is its adherence to the open/closed principle. It allows our system to be open for extension (by adding new burger types) but closed for modification (no need to change existing code).

3. Decoupling and the Power of Abstraction

The Factory Method shines in promoting decoupling. Clients order a burger interacting solely with the BurgerStore abstract class. The specific details of which burger are abstracted away.

This means we can provide different implementations of the BurgerStore and the client code will not need to change. This is known as the dependency inversion principle.

You might have come across another pattern named the abstract factory. The Factory Method is all about creating individual products, delegating the 'how' to subclasses. In contrast, the Abstract Factory is concerned with creating families of related products. Think of the Factory Method as a singular method, while the Abstract Factory is more of an object comprising multiple factory methods.
Design and Implementation
The factory method UML class diagram can be conceptualized like the following:

![Factory Method Pattern](/images/factory-01.avif)

Let's expand on the components that are collaborating together in the above diagram.

1. Creator Interface or Abstract Class

This class/interface provides the declaration of the factory method. This method might be abstract or concrete, depending on the specific scenario.

If this method is declared as abstract, it just means that it is to be implemented by the concrete classes that extend it. This does not mean that the pattern becomes an abstract factory pattern.

Whether we choose to make use of an abstract class or an interface depends on the specific scenario. If there are shared behaviors or attributes among each concrete implementation of the product, an abstract class is a better choice. However, if it is just the methods that need to be implemented, an interface would suffice.

The factory method might have default implementation in some cases. This factory method must return an instance of the Product type - we shall see why.

2. Concrete Creators

These are the subclasses that extend/implement the creator. They will be overriding the factory method to produce concrete products based on some input parameter.

Even though the factory method has a return type Product, we can still return an instance of a concrete product because it implements (or extends) the Product interface/abstract class.

3. Product Interface or Abstract Class

This represents the contract for the types of products the factory will produce.

4. Concrete Products

These are concrete implementations of the product interface or the abstract class. They define the specifics of each product variant. Adding functionality each time is done through extension, rather than modification. This follows the Open/Closed principle closely.

If we apply all of the above components to our scenario, we end up with the following:

![Factory Method Pattern](/images/factory-02.avif)

The above class diagram demonstrates the class diagram applied to our scenario.
The complete implementation of the code is available in the `View Code`` section above.

```javascript
class PrinterService {
	static instance;
	#mode = null;

	constructor() {
		if (PrinterService.instance) {
			return PrinterService.instance;
		}
		PrinterService.instance = this;
	}

	getPrinterStatus() {
		return this.#mode;
	}

	setMode(mode) {
		this.#mode = mode;
		console.log(mode);
	}
}

const worker1 = new PrinterService();
const worker2 = new PrinterService();

console.log(worker1 === worker2); //true

worker1.setMode("GrayScale");
worker2.setMode("Color");

console.log(worker1.getPrinterStatus());
console.log(worker2.getPrinterStatus());
```

Please note that:

The Singleton property in Python is enforced by overriding the new method and making use of a \_unique_instance attribute to store the Singleton instance.
The threading.Lock is used to ensure that the instance creation is thread-safe.
Limitations and Pitfalls

1. Violation of the Single Responsibility Principle

The singleton pattern introduces an additional responsibility to a class: managing its own instantiation. This class now has to do its primary job and also ensure there's only one instance of itself.

This coupling of concerns can be seen as a violation of the Single Responsibility Principle, which also leads to tight coupling.

2. Difficulty in Unit Testing

Because the singleton carries a state which persists for the lifetime of the application, it can lead to bugs when performing unit testing.

Unit tests are supposed to be isolated tests, testing specific parts of the application and one test should not have any adverse effects on the other tests. If one test changes the singleton's state, it could inadvertently affect other tests as well, breaking the principle of unit testing.

3. Global Scope

Singletons are often used to provide a global point of access to a resource in an application. The state of the singleton can also be modified from anywhere in the application, making it difficult to track down where the state was changed.

For this reason, singletons are often referred to as an anti-pattern.

Use Cases

1. Configuration Manager

Many applications and even operating systems require a centralized place to manage configuration settings. Whether these settings are loaded from a file, a database, or an external service, we often want a single point of access to this configuration throughout the application.

A Singleton Configuration Manager ensures that configuration data is loaded only once.

2. Connection Pool

In systems that interact with databases, network services, or other resources, creating and destroying connections repeatedly can be resource-intensive and slow. Connection pools manage and reuse a set of open connections.

Singletons can be used to ensure that connections are managed centrally, optimizing resource usage.

3. Logging Service

Logging is a common requirement in many applications for tracking events, errors, or informational messages. Having a centralized logger can help manage log destinations, formats, and levels.

A Singleton ensures that all log messages are handled consistently and preventing potential issues like multiple log files or duplicated log entries. It also optimizes performance by
reusing resources like file handles or network connections.

4. Hardware Interface Access

This one goes back to our printer example. It's crucial to coordinate access to prevent conflicts or unexpected behavior.

Using a Singleton to represent and manage access to the hardware ensures that all interactions are coordinated through a single access point. This can prevent issues like sending multiple simultaneous commands to a device, which might cause malfunctions or erratic behavior.

Closing Notes
The singleton pattern effectively helps us achieve the following:

1. Reduced memory usage

By creating only one instance of the class, we reduce the overall memory usage of the application. This way, we avoid allocating and deallocating memory for multiple objects of the same type, reducing garbage collection overhead.

2. We are guaranteed one instance

Because there is only a single point of access, we ensure that every class that uses the Singleton uses the same instance that is instantiated by all the other classes before it. If needed, we can implement concurrency controls to ensure that there are no data inconsistencies.
