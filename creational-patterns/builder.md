The Builder pattern is a creational pattern that separates the construction of a complex object from its representation. This way, the same construction process can be used to create different representations of the product.

Motivation
Suppose we are tasked with developing software to assemble restaurant meals. Each meal consists of starters, main courses, drinks, and desserts.

At first glance, this might not seem overly complex. We could have a Meal class and simply use a constructor to pass in these four components.

```javascript
class Meal {
	constructor(starter, main, dessert, drink) {
		this.starter = starter;
		this.main = main;
		this.dessert = dessert;
		this.drink = drink;
	}
}

const myMeal = new Meal("Nachos", "Pasta", "Icecream", "Margarita");
// ... further operations with myMeal
```

Unfortunately, this is not a scalable or readable approach.

It's not immediately clear which parameter corresponds to which attribute. Furthermore, each time a new feature is added, we would have to add a new parameter and thus update all references to the constructor.

The attributes of the Meal are also exposed to the client, meaning the client needs to know every single detail. Remember, encapsulation is not just about making the fields private; it is also about hiding the complexity and details.

Using the factory method pattern to create these meals is one option, but it doesn't grant us fine-grained control over the creation of the different meals.

Our goal is to craft meals in a more granular manner, ensuring the ability to add or remove components while adhering to a specific sequence.

Another tempting solution might be to establish a class hierarchy. We could have a base class, Meal, and then create subclasses for each possible combination. However, this approach could also lead to a combinatorial explosion of classes.

The solution
The builder pattern is used to extract out the steps of constructing a product and delegate them to "Builders". This way, we can create multiple variations of a given product.

This allows us to break the creation process down into steps. Clients do not have to implement all of the steps to get a concrete product. Not all of the products will have the same implementation.

In our meal example, a meal like VeganMeal will have different implementation to a HealthyMeal. However, we will still have a common interface, which all builders implement. This way, all builders "build" a product but the specifics are up to the Concrete Builders.

Concrete Builders, which implement the Builder interface, are responsible for implementing the business logic of the construction steps.

It's also important to not confuse the builder pattern with the abstract factory. While the abstract factory is concerned with creating a family of related products with a single call, the builder is more about creating a single complex object using a series and sequences of steps.
Design and Implementation
The builder pattern typically consists of the following components:

1. Product - This is the complex object (end product) that needs to be constructed. It's the end result of the construction process.

2. Builder Interface or the AbstractBuilder - This interface declares the methods for constructing the parts of the product. These methods are used to ensure the necessary steps in the construction process.

It often (but not always) includes a method to return the final product once it's constructed. This is because each Concrete Builder may have a different return type, in which case the method is added in the Concrete Builder.

3. Concrete Builders - This implements the builder interface and provides specific implementations for each method. Each Concrete Builder is responsible for constructing and assembling the product in a particular way.

At the end of the process, the concrete builder typically provides a way to retrieve the constructed product. This method might be called getResult() or build().

4. Director - This is responsible for managing the correct sequence of the construction process. It uses the Builder interface to construct the product and shields the client from the specifics of the product's construction.

In simpler scenarios, clients can interact directly with the builder, and might not need a Director at all. But, the Director is beneficial because it allows flexibility.

5. Client - The client decides which type of product it needs and accordingly chooses the appropriate concrete builder.

It either interacts with the Director to construct the product or might use the builder directly.

Conceptually, the builder can be visualized like the following:

![Builder Pattern](../images/builder-01.avif)

When we apply the above to our scenario, below is the UML class diagram and the code implementation.

![Builder Pattern](../images/builder-02.avif)

```javascript
const Starter = Object.freeze({
	SALAD: "Salad",
	SOUP: "Soup",
	BRUSCHETTA: "Bruschetta",
	VEGGIE_STICKS: "Veggie Sticks",
	CHICKEN_WINGS: "Chicken Wings",
});

const Dessert = Object.freeze({
	FRUIT_SALAD: "Fruit Salad",
	ICE_CREAM: "Ice Cream",
	CHOCOLATE_CAKE: "Chocolate Cake",
	VEGAN_PUDDING: "Vegan Pudding",
	CHEESECAKE: "Cheesecake",
});

class Meal {
	constructor() {
		this.starter = null;
		this.main = null;
		this.dessert = null;
		this.drink = null;
	}

	getStarter() {
		return this.starter;
	}

	getMain() {
		return this.main;
	}

	getDessert() {
		return this.dessert;
	}

	getDrink() {
		return this.drink;
	}

	setStarter(starter) {
		this.starter = starter;
	}

	setMain(main) {
		this.main = main;
	}

	setDessert(dessert) {
		this.dessert = dessert;
	}

	setDrink(drink) {
		this.drink = drink;
	}
}

class Builder(ABC):
    @abstractmethod
    def add_starter(self, starter):
        pass

    @abstractmethod
    def add_main_course(self, main):
        pass

    @abstractmethod
    def add_dessert(self, dessert):
        pass

    @abstractmethod
    def add_drink(self, drink):
        pass


class VeganMealBuilder {
    constructor() {
        this.meal = new Meal();
    }

    addStarter() {
        this.meal.setStarter(Starter.SALAD);
    }

    addMainCourse() {
        this.meal.setMain(Main.VEGGIE_STIR_FRY);
    }

    addDessert() {
        this.meal.setDessert(Dessert.VEGAN_PUDDING);
    }

    addDrink() {
        this.meal.setDrink(Drink.VEGAN_SHAKE);
    }

    build() {
        return this.meal;
    }
}

class Director {
    constructVeganMeal(builder) {
        builder.addStarter();
        builder.addMainCourse();
        builder.addDessert();
        builder.addDrink();
    }

    constructHealthyMeal(builder) {
        builder.addStarter();
        builder.addMainCourse();
        builder.addDessert();
        builder.addDrink();
    }
}

const director = new Director();

const veganBuilder = new VeganMealBuilder();
director.constructVeganMeal(veganBuilder);
const veganMeal = veganBuilder.build();
console.log("Vegan Meal constructed:");
console.log("Starter: " + veganMeal.getStarter());
console.log("Main: " + veganMeal.getMain());
console.log("Dessert: " + veganMeal.getDessert());
console.log("Drink: " + veganMeal.getDrink());

// Constructing a healthy meal
const healthyBuilder = new HealthyMealBuilder();
director.constructHealthyMeal(healthyBuilder);
const healthyMeal = healthyBuilder.build();
console.log("Healthy Meal constructed:");
console.log("Starter: " + healthyMeal.getStarter());
console.log("Main: " + healthyMeal.getMain());
console.log("Dessert: " + healthyMeal.getDessert());
console.log("Drink: " + healthyMeal.getDrink());
```

Limitations and Pitfalls

1. Complexity - As we saw, the builder pattern requires a lot of code. Every product needs its own class and every single one of those products require their own builder. When we are dealing with multiple products, the code can grow quickly and become harder to manage.

2. Error-Prone - In our builder pattern, the builder can allow for partial or step-wise construction and there might be situations where the end product is missing some attributes or is in an inconsistent state, leading to potential errors or unexpected behaviors.

Use Cases

1. Protocol Buffers - In data serialization, the Builder pattern is employed to construct complex data structures. Protocol buffers define structured data in a simpler, yet efficient way compared to XML or JSON.

This enables step-by-step construction of objects, especially handy for nested data structures and handling optional or repeated fields. More on Protobufs.

2. Meal Order Systems - As shown in our example, different meals might have different components (starters, mains, drinks, desserts) and preparation steps is another example.

If we generalized our interface a bit more, we could end up with an even more flexible builder. 3. Video Game Character Creation - The builder pattern is popular in the context of video games which allow crafting a custom character.

Consider a role-playing game where a player can choose a character's weapons, armor, accessories, and abilities.

The builder pattern can offer a sequence of methods that let you construct a character.

Closing Notes
The builder pattern effectively achieves the following:

1. Encapsulate what varies: One of the primary advantages of the Builder pattern is that it can hide the internal representation and construction process of a product from the client, ensuring that only valid products are constructed.

2. Separation of Concerns: The pattern separates the construction from the representation. The client does not need to know the intricate details of how an object is put together.

3. Single Responsibility Principle: Every single one of the builders has one responsibility - building a particular representation of the product. If there are different ways or sequences to build different products, we can have a different builder for each variation.

Finally, remember that design patterns, including the Builder pattern, are guidelines and not strict rules. They can be adapted and modified to fit the specific needs and constraints of a project.
