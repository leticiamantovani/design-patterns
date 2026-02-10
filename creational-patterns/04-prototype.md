# Prototype

The Prototype pattern is a creational design approach that streamlines the process of creating complex objects. It enables clients to produce clones of existing objects rather than constructing new instances from the ground up.

This method significantly reduces both the cost and complexity associated with object creation, making it particularly useful in scenarios where there is a frequent need for similar objects.

## Motivation

Imagine being tasked with duplicating a complex document that includes images, various font sizes, styles, and formatting. How would we go about doing this efficiently?

The brute force approach might involve recreating the document step by step, adding each element individually. A good candidate solution for this would be the builder pattern which we already learned about.

It allows us to construct complex objects step by step. Recreating the original document by following the same steps would mean creating all properties from scratch and instantiating that new object with these properties.

The Builder solution would look something like this:


```javascript


class Document {
  constructor(content) {
    this.content = content;
  }

  getContent() {
    return this.content;
  }

  display() {
    console.log("Document Content:", this.content);
  }
}

class DocumentBuilder {
  constructor() {
    this.contentBuilder = "";
  }

  addContent(content) {
    this.contentBuilder += content;
    return this; // Enable method chaining
  }

  build() {
    return new Document(this.contentBuilder);
  }
}

// Using the Builder to create and replicate documents
// Original document
const originalDoc = new DocumentBuilder().addContent("Hello, World!").build();

// Replicating the document using Builder
const copiedDoc = new DocumentBuilder()
  .addContent(originalDoc.getContent())
  .build();

// Display contents of the copied document
copiedDoc.display();

```

This approach works but has many limitations. In the simple example above, the process seems manageable, but real-world documents often include diverse elements such as various fonts, images, and intricate layouts. Each of these elements requires individual construction steps, significantly increasing the complexity of using the Builder pattern for this purpose.

Also, this approach will not scale well when multiple copies of a document are needed. With each duplication, we must reconstruct the entire document from scratch. This process can be resource-intensive, particularly for larger or more complex documents.

Basically, this approach is akin to opening up another Google Document and trying to copy paste and configure everything manually.

All of these problems tell us that we need a more efficient solution, something that doesn't have the overhead of starting from scratch, is readable, and also scales well.

## Solution

The Prototype pattern offers an elegant solution to overcome the limitations of the Builder pattern, particularly in scenarios like duplicating a document. By enabling the cloning of an existing object, this pattern facilitates the creation of object copies without the need to instantiate them from scratch.

In programming, cloning means copying the exact values of an object's properties without writing explicit code for each property.
The beauty of the Prototype pattern lies in its abstraction of the cloning logic within the object itself. Clients can simply invoke a clone() method to duplicate an object, which significantly reduces the likelihood of errors and simplifies the code. This method is far less error-prone compared to the Builder pattern, where each element of a document must be manually constructed.

A major advantage of this pattern is its adherence to the principle of "Program to an Interface, not an implementation." This means clients can interact with different concrete implementations of a document object without altering the underlying code. It provides the flexibility to swap out implementations seamlessly. This will become clearer in a bit.

As you can see, this approach addresses the two primary drawbacks of the Builder pattern: the need to construct complex objects from scratch and the resource-intensive nature of such constructions.

## Design & Implementation

Now that we have the motivation behind the prototype pattern, let's take a look at the components that it comprises of:

1. Prototype (Interface) - The Prototype interface declares the cloning method used to create new objects. All concrete prototype classes must implement this interface, providing their own logic for the clone() method. In languages like Java, this can be achieved by implementing an interface like Cloneable, though our implementation will be custom. This will make more sense in a bit.

2. Concrete Prototype - These classes implement the Prototype interface and define the actual cloning logic.

Each concrete prototype represents a different variation of the clone. This cloning process will involve creating a new instance of the same class by copying over the properties from the original object.

Depending on the requirements, the cloning can be either a shallow copy or a deep copy of the object.

A shallow copy means that a new instance of an object is created but all its properties are reused. The copied properties point to the same memory location as the original document. A deep copy means that all the properties are initialized with new instances. Each cloned property points to a different memory location.
3. Client - This class uses Prototype interface to clone objects. It holds a reference to a Prototype instance and uses it to create new objects as needed. The Client is typically unaware of the Concrete Prototype class of the object it is cloning. The client's primary role is to utilize the clone() method to obtain new instances.

The Prototype pattern differs from the Factory pattern, which is another creational pattern we discussed earlier. While both are used for object creation, their objectives are distinct. The Factory pattern is ideal for creating the initial object, whereas the Prototype pattern excels in making copies of that object, reducing the overhead of instantiation.

https://imagedelivery.net/CLfkmk9Wzy8_9HRyug4EVA/aeda8610-db3d-4c79-88bf-d82a22dae500/sharpen=1

The UML template above can be applied to any problem that uses the prototype pattern.
When we apply the above to our example, we get the following UML diagram and code.

https://imagedelivery.net/CLfkmk9Wzy8_9HRyug4EVA/a14d22b2-d6cd-4c19-1026-80bdfd3bb800/sharpen=1

```javascript

class DocumentPrototype {
  constructor() {
    if (new.target === DocumentPrototype) {
      throw new TypeError("Cannot constructor abstract instances directly!");
    }
  }

  cloneDocument() {
    throw new Error("Must implement the cloneDocument method!");
  }

  display() {
    throw new Error("Must implement the display method!");
  }
}

class Document extends DocumentPrototype {
  #content;
  #images;
  #formatting;
  #annotations;

  constructor(content, images, formatting, annotations) {
    super();
    this.#content = content;
    this.#images = [...images];
    this.#formatting = formatting;
    this.#annotations = [...annotations];
  }

  cloneDocument() {
    return new Document(
      this.#content,
      this.#images,
      this.#formatting,
      this.#annotations
    );
  }

  display() {
    console.log(`Content ${this.#content}`);
    console.log(`Images ${this.#images}`);
    console.log(`Formatting ${this.#formatting}`);
    console.log(`Annotations ${this.#annotations}`);
  }

  addImage(image) {
    this.#images.push(image);
  }

  addAnnotation(annotation) {
    this.#annotations.push(annotation);
  }
}
const images = [];
images.push("Image1.png");
const annotations = [];
annotations.push("Annotation1");

const originalDoc = new Document("Hello, World!", images, "Basic", annotations);

const copiedDoc = originalDoc.cloneDocument();
originalDoc.addImage("Image2.png");
originalDoc.addAnnotation("Annotation2");

console.log("Original Document");
originalDoc.display();
console.log("Copied Document");
copiedDoc.display();

```

In the code above, the client interacts with the DocumentPrototype interface, which is implemented by Document. Both the originalDoc and copiedDoc are instances of Document that conforms to the DocumentPrototype interface. Notice how flexible this design is; it enables the client to work with various types of documents without being tied to a specific implementation. As a result, modifications to originalDoc or copiedDoc do not impact one another, which goes to show the independence of clones generate through the Prototype pattern.

I included properties such as content, images, formatting and annotations to simulate the copying of a realistic document. This variety of properties showcases the Prototype pattern's capability to handle complex objects with multiple attributes. In the cloneDocument() method, we create a deep copy of the document, which grants the cloned document complete independence from the original.

## Limitations & Pitifails

1. Complexity in performing deep copies - Just like our example, if the implementation requires performing a deep copy, it can be tricky and resource-intensive to implement, particularly for objects with nested structures. This is why we need to ensure that a correct and complete deep copy is made so we avoid unintended side effects like shared resources among objects.

2. Limited Applicability - While the prototype pattern is great for mass-producing objects with similar configurations, it has limited applicability in that its mostly beneficial in scenarios where objects require substantial setup or configuration after instantiation.

## Use Cases

1. Prototype-based Programming - In prototype-based languages like JavaScript, objects are primarily created by cloning existing objects rather than through new class-based instantiation. Prototypes in JavaScript have some differences from the traditional Prototype pattern, which you can learn more about in this video.

2. Implementing Undo/Redo Mechanisms - In applications with undo/redo functionality, the Prototype pattern can help capture the state of an object at a particular moment. Cloning the object's state allows the application to revert to or redo a previous state as needed.

3. Graphic Design Applications - If you have ever used graphic design software for creating diagrams with similar configurations, the Prototype pattern allows objects to be cloned and then customized to your liking. This makes the design process more efficient.

4. Game Development - If you have played video games before, you have probably encountered similar collectibles throughout the game. The Prototype pattern can be used to create objects or collectibles that share common traits.

## Closing notes

1. Program to an Interface, Not an Implementation - The pattern centers around an interface that declares a cloning method, enabling concrete classes to implement their own cloning logic. This design abstracts the duplication process behind an interface, ensuring that clients are not bound to specific implementations.

2. Efficient Creation of Complex Objects - By cloning objects through the copying of existing properties, the Prototype pattern eliminates the need to instantiate entirely new objects. This approach significantly reduces the cost and effort involved in creating complex objects.

3. Delegated Cloning Responsibility - The Prototype pattern delegates the responsibility of cloning to the objects themselves, rather than the client. This ensures that the objects control their own cloning logic, leading to a more encapsulated and maintainable design.