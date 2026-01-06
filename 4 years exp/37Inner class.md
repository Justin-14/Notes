# Python Inner class — Interview Guide (30 Questions)

This comprehensive guide covers **Inner Classes (Nested Classes)** in Python Object-Oriented Programming. It is designed for a Python developer with **4 years of experience**, moving beyond basic syntax into architectural decisions, scoping rules, and practical design patterns.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### 1. What is a Python Inner Class and how does it differ primarily from Java Inner Classes?

**Difficulty:** Easy

* **Expected Answer:** An Inner Class is a class defined inside the body of another class. The primary difference from Java is that in Python, an inner class does **not** automatically hold a reference to an instance of the outer class. It is essentially a normal class that lives in the namespace of the outer class.
* **Clear Explanation:** In Python, nesting a class is mostly a namespace organization technique. When you instantiate the inner class, you do not need an instance of the outer class unless you explicitly design it that way (e.g., passing `self` of the outer to the inner).
* **Reasoning:** Python treats classes as first-class objects. Defining a class inside another simply places the inner class object in the outer class's `__dict__`.
* **Common Mistakes:** Assuming the inner class can access `self.outer_variable` directly without passing the outer instance.
* **Why Interviewers Ask This:** To check if the candidate understands Python's specific scoping rules versus generic OOP assumptions from other languages.
* **Sample Code:**

```python
class Outer:
    class Inner:
        def __init__(self):
            self.data = "I am independent"

# Instantiation does not require an Outer instance
inner_obj = Outer.Inner()
print(inner_obj.data)

```

* **Line-by-line:**
* `class Inner:`: Defined inside `Outer`.
* `inner_obj = Outer.Inner()`: Accesses `Inner` via the `Outer` namespace, but doesn't need `Outer()` (instance).



---

### 2. How do you instantiate an Inner Class?

**Difficulty:** Easy

* **Expected Answer:** You access it via the Outer class name: `Outer.Inner()`. If you are inside the Outer class methods, you can use `self.Inner()` or `Outer.Inner()`.
* **Clear Explanation:** Since the inner class is an attribute of the outer class object, it is accessed using dot notation.
* **Reasoning:** It follows standard attribute access rules in Python.
* **Common Mistakes:** Trying to call `Inner()` directly from the global scope without the `Outer` prefix.
* **Why Interviewers Ask This:** To ensure basic syntax fluency.
* **Sample Code:**

```python
class Computer:
    class CPU:
        def process(self):
            return "Processing"

# Method 1: Outside the class
my_cpu = Computer.CPU()

# Method 2: Inside the class
class ComputerTwo:
    class CPU:
        pass
    def create_cpu(self):
        return self.CPU() # or ComputerTwo.CPU()

```

---

### 3. Can an Inner Class access the private members of the Outer Class?

**Difficulty:** Easy

* **Expected Answer:** Not automatically/implicitly. However, Python does not enforce strict private access controls. If the inner class is given a reference to the outer instance, it *can* access private members (name mangled as `_Outer__private`), but it requires explicit passing of the instance.
* **Clear Explanation:** Scopes are distinct. The inner class does not inherit the scope of the outer class.
* **Reasoning:** Python relies on "consenting adults" rules. Access requires possession of the object reference.
* **Common Mistakes:** Thinking `self.private_var` works inside the inner class to access outer state.
* **Why Interviewers Ask This:** To test understanding of Python's "explicit is better than implicit" philosophy regarding scopes.
* **Sample Code:**

```python
class BankAccount:
    def __init__(self):
        self.__balance = 1000  # Private
    
    class Transaction:
        def __init__(self, account):
            self.account = account
        
        def show_balance(self):
            # Must access via the passed object
            return self.account._BankAccount__balance 

acc = BankAccount()
trans = BankAccount.Transaction(acc)
print(trans.show_balance()) # 1000

```

---

### 4. What is the scope visibility of an Inner Class? Can the global scope see it?

**Difficulty:** Easy

* **Expected Answer:** The Inner Class is visible wherever the Outer Class is visible, provided you use the qualified name (`Outer.Inner`). It is not globally available by its short name.
* **Clear Explanation:** The inner class is just a name bound within the Outer class's namespace.
* **Reasoning:** This is identical to how a variable or method inside a class is hidden from the global scope but accessible via the class.
* **Common Mistakes:** Importing `Inner` directly from the module without the `Outer` prefix (unless aliased).
* **Why Interviewers Ask This:** To verify knowledge of Python namespaces.
* **Sample Code:**

```python
class Outer:
    class Inner:
        pass

# print(Inner) # NameError: name 'Inner' is not defined
print(Outer.Inner) # <class '__main__.Outer.Inner'>

```

---

### 5. Why would you use an Inner Class instead of a generic separate class?

**Difficulty:** Easy

* **Expected Answer:** Logical grouping and encapsulation. If a class is only useful in the context of another class (e.g., a `Cursor` for a `DatabaseConnection`), keeping it nested keeps the global namespace clean and indicates the relationship clearly.
* **Clear Explanation:** It signals to other developers: "This class is a helper for the Outer class and likely shouldn't be used independently."
* **Reasoning:** Code organization and readability.
* **Why Interviewers Ask This:** To assess architectural design intent.
* **Comparison:** Similar to `static nested classes` in Java.

---

### 6. Can an Outer Class inherit from its own Inner Class?

**Difficulty:** Easy (Concept) / Medium (Execution)

* **Expected Answer:** No, this causes a `NameError` because the Inner class is not fully defined when the Outer class is declaring its parent.
* **Clear Explanation:** Python processes class bodies from top to bottom. The Inner class doesn't exist until the body of the Outer class is processed, but the base class list is processed *before* the body.
* **Reasoning:** Resolution order of class definitions.
* **Common Mistakes:** Attempting `class Outer(Outer.Inner):` or `class Outer(Inner):`.
* **Why Interviewers Ask This:** To test knowledge of the Python interpreter's class creation lifecycle.

---

### 7. How do you implement a link from Inner Class back to the Outer Class instance?

**Difficulty:** Medium

* **Expected Answer:** You must explicitly pass the outer instance (usually `self`) to the inner class's `__init__` method and store it.
* **Clear Explanation:** Unlike Java, there is no magic `Outer.this`. You implement the relationship manually via composition.
* **Reasoning:** Explicit reference management prevents circular reference surprises and memory leaks (though Python's GC handles cycles well, explicit is better).
* **Time/Space:** O(1) space for the reference.
* **Sample Code:**

```python
class Head:
    def __init__(self):
        self.brain = self.Brain(self) # Pass 'self' (Outer instance)

    class Brain:
        def __init__(self, head_instance):
            self.head = head_instance # Store back-reference

```

---

### 8. Explain how Inner Classes affect Pickling (Serialization).

**Difficulty:** Medium

* **Expected Answer:** Prior to Python 3.3/3.4, pickling inner classes was difficult or impossible because `pickle` relies on the `__qualname__` (or import path) to reconstruct the object. Modern Python supports it, but the inner class must be accessible globally via its qualified name.
* **Clear Explanation:** `pickle` serializes objects by name reference. It needs to know how to find the class definition to unpickle it.
* **Reasoning:** `Outer.Inner` is a valid path in modern Python, allowing pickling to work.
* **Common Mistakes:** Defining inner classes dynamically inside functions (closures) and trying to pickle them (fails because they have no import path).
* **Why Interviewers Ask This:** Critical for backend systems using Redis/Caching or Multiprocessing.

---

### 9. What is the difference between `@staticmethod` inside an Inner Class vs. Instance method?

**Difficulty:** Medium

* **Expected Answer:** It behaves exactly as it does in a top-level class. A static method in an inner class does not receive `self` (inner) or the outer instance. It is purely a utility function namespaced twice.
* **Clear Explanation:** Nesting does not change method semantics.
* **Reasoning:** Scope isolation.
* **Why Interviewers Ask This:** To see if candidates get confused by the double nesting.
* **Sample Code:**

```python
class Outer:
    class Inner:
        @staticmethod
        def say_hello():
            return "Hello"

print(Outer.Inner.say_hello()) # Works without instantiation

```

---

### 10. Can an Inner Class be hidden/private?

**Difficulty:** Medium

* **Expected Answer:** Yes, by prefixing the class name with a double underscore (`class __Inner:`).
* **Clear Explanation:** Python performs name mangling. The class becomes `_Outer__Inner`. It signals strong intent that this class is for internal implementation details only.
* **Reasoning:** Encapsulation.
* **Common Mistakes:** Thinking it is truly invisible. It is just renamed.
* **Why Interviewers Ask This:** To test knowledge of Python's access control conventions.

---

### 11. How do Inner Classes interact with Class Variables of the Outer Class?

**Difficulty:** Medium

* **Expected Answer:** The Inner Class cannot access Outer Class variables directly by name. It must access them via `Outer.variable_name`.
* **Clear Explanation:** The Inner class scope does not search the Outer class scope for names. It searches its own scope, then the global scope.
* **Reasoning:** LEGB rule (Local, Enclosing, Global, Built-in). "Enclosing" refers to functions, not classes. Class scopes in Python do not nest for variable lookup.
* **Sample Code:**

```python
class Config:
    TIMEOUT = 500
    
    class Connector:
        def connect(self):
            # return TIMEOUT # NameError
            return Config.TIMEOUT # Correct

```

---

### 12. When would you use a Dictionary vs. an Inner Class for configuration data?

**Difficulty:** Medium

* **Expected Answer:** Use an **Inner Class** when you need structure, type hints, methods, or validation (properties). Use a **Dictionary** for simple key-value pairs where structure is dynamic or derived from JSON.
* **Reasoning:** Classes provide attribute access (`obj.prop`), which is cleaner than dict access (`obj['prop']`) and supports IDE autocompletion.
* **Comparison:** Inner Class = Schema-enforced; Dict = Schema-less.

---

### 13. How does `__qualname__` differ for Inner Classes?

**Difficulty:** Medium

* **Expected Answer:** `__name__` returns just the class name (e.g., `'Inner'`). `__qualname__` returns the dotted path from the module level (e.g., `'Outer.Inner'`).
* **Clear Explanation:** `__qualname__` was introduced (PEP 3155) specifically to help with introspection and pickling of nested objects.
* **Why Interviewers Ask This:** Deep introspection knowledge.

---

### 14. Can you inherit from an Inner Class defined in another library?

**Difficulty:** Medium

* **Expected Answer:** Yes, as long as the Inner Class is public (accessible).
* **Syntax:** `class MyCustomInner(LibraryClass.InnerHelper): ...`
* **Reasoning:** Inner classes are just attributes. If you can import the Outer class, you can access the Inner class (attribute) and inherit from it.
* **Developer Scenario:** Extending a specific configuration class inside a library like Django or SQLAlchemy (e.g., `class Meta:`).

---

### 15. What are the memory implications of defining an Inner Class?

**Difficulty:** Medium

* **Expected Answer:** The Inner Class definition is executed once when the Outer Class is defined. It creates a Class Object in memory. It does *not* create a new class object for every instance of Outer.
* **Reasoning:** Class definition happens at module load time (usually).
* **Common Mistakes:** Thinking that `Outer()` instantiation re-defines `Inner`. It does not.
* **Why Interviewers Ask This:** Performance and memory usage awareness.

---

### 16. How do Inner Classes relate to the "Flyweight Pattern"?

**Difficulty:** Medium

* **Expected Answer:** Inner classes can be used to hold shared state (intrinsic state) while the outer class holds unique state (extrinsic), or vice versa.
* **Reasoning:** If the Inner Class is designed to be immutable or shared, it can optimize memory usage.

---

### 17. Can you use `super()` inside an Inner Class?

**Difficulty:** Medium

* **Expected Answer:** Yes, but it refers to the parent of the *Inner* class, not the Outer class.
* **Clear Explanation:** `super()` follows the MRO (Method Resolution Order) of the class it is called in. It has no relation to the nesting container.
* **Why Interviewers Ask This:** To confuse the candidate about inheritance vs. nesting.

---

### 18. What is the "Closure" difference between a Nested Function and a Nested Class?

**Difficulty:** Medium

* **Expected Answer:** A nested **function** can access variables from the enclosing function (closure). A nested **class** does **NOT** capture variables from the enclosing class definition scope.
* **Clear Explanation:** This is a specific quirk of Python.
* **Sample Code:**

```python
def outer_func():
    x = 10
    def inner_func():
        return x # Works (Closure)

class OuterClass:
    x = 10
    class InnerClass:
        # print(x) # Fails (NameError) - No class-scope closure
        pass

```

---

### 19. How do Inner Classes facilitate the Builder Pattern in Python?

**Difficulty:** Medium

* **Expected Answer:** The Inner Class can act as the Builder, accumulating parameters, validating them, and then instantiating the private Outer class constructor.
* **Real App Usage:** Complex object construction where you want `Car.Builder().setEngine().setWheels().build()`.

---

### 20. Is it possible to define an Inner Class inside a method? (Local Class)

**Difficulty:** Medium

* **Expected Answer:** Yes. These are called Local Classes.
* **Limitations:** They are not accessible outside the method. They are harder to pickle.
* **Use Case:** Returning a specialized object from a factory method that doesn't need to exist globally.
* **Why Interviewers Ask This:** To see if the candidate knows classes are executable statements in Python.

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### 1. Basic Inventory Item (Easy)

**Question:** Create a class `Product` that contains an inner class `Dimensions`. The `Product` should be initialized with name, price, and dimension values (length, width, height). The `Dimensions` inner class should have a method to calculate volume.

**Solution:**

```python
class Product:
    def __init__(self, name, price, l, w, h):
        self.name = name
        self.price = price
        # Instantiate Inner class
        self.dimensions = self.Dimensions(l, w, h)

    class Dimensions:
        def __init__(self, length, width, height):
            self.length = length
            self.width = width
            self.height = height

        def volume(self):
            return self.length * self.width * self.height

# Usage
box = Product("Box", 10.99, 10, 5, 2)
print(f"Product: {box.name}, Volume: {box.dimensions.volume()}")

```

* **Line-by-line:**
* `self.dimensions = self.Dimensions(...)`: The outer instance creates an inner instance.
* `def volume(self)`: Encapsulated logic inside `Dimensions`.


* **Complexity:** Time O(1), Space O(1).
* **Why Interviewers Ask This:** To test basic composition syntax.

---

### 2. Student Transcript (Easy)

**Question:** Create a `Student` class. Inside it, define an `Address` class. The Student should hold a list of addresses. Demonstrate adding two addresses to a student.

**Solution:**

```python
class Student:
    def __init__(self, name):
        self.name = name
        self.addresses = []

    def add_address(self, city, street):
        new_addr = self.Address(city, street)
        self.addresses.append(new_addr)

    class Address:
        def __init__(self, city, street):
            self.city = city
            self.street = street
        
        def __repr__(self):
            return f"{self.street}, {self.city}"

# Usage
s1 = Student("Alice")
s1.add_address("NY", "5th Ave")
s1.add_address("LA", "Sunset Blvd")
print(s1.addresses)

```

* **Scenario:** Managing user profiles with multiple shipping addresses (Amazon).

---

### 3. Encapsulated Constants (Easy)

**Question:** Create a class `APIClient`. Use an inner class `Routes` to store API endpoint strings as constants. A method in `APIClient` should use these constants to print a URL.

**Solution:**

```python
class APIClient:
    class Routes:
        LOGIN = "/api/v1/login"
        LOGOUT = "/api/v1/logout"
        DASHBOARD = "/api/v1/dashboard"

    def __init__(self, base_url):
        self.base_url = base_url

    def get_login_url(self):
        return f"{self.base_url}{self.Routes.LOGIN}"

# Usage
client = APIClient("https://example.com")
print(client.get_login_url())

```

* **Why Interviewers Ask This:** Checking if you know how to access static inner attributes (`self.Routes.LOGIN`).

---

### 4. Linked List Node Encapsulation (Medium)

**Question:** Implement a `LinkedList` class. Define the `Node` class *inside* the `LinkedList` class to hide it from the global namespace. Implement an `append` method.

**Solution:**

```python
class LinkedList:
    # Inner class Hidden from casual global usage
    class Node:
        def __init__(self, data):
            self.data = data
            self.next = None

    def __init__(self):
        self.head = None

    def append(self, data):
        new_node = self.Node(data) # Accessing Inner Class
        if not self.head:
            self.head = new_node
            return
        
        current = self.head
        while current.next:
            current = current.next
        current.next = new_node

    def display(self):
        elements = []
        cur = self.head
        while cur:
            elements.append(cur.data)
            cur = cur.next
            
        return elements

# Usage
ll = LinkedList()
ll.append(10)
ll.append(20)
print(ll.display())

```

* **Line-by-line:**
* `class Node:` defined inside `LinkedList` keeps the namespace clean.
* `new_node = self.Node(data)`: Creates instance using the class attribute.


* **Complexity:** Append is O(N) (unless tail pointer used).
* **Scenario:** Standard Data Structures library implementation.

---

### 5. The "Deck of Cards" Iterator (Medium)

**Question:** Create a `Deck` class representing 52 cards. Implement an inner class `DeckIterator` that works as a custom iterator. The `Deck` class should implement `__iter__` returning an instance of `DeckIterator`.

**Solution:**

```python
class Deck:
    def __init__(self):
        self.cards = [f"{v} of {s}" for s in ['S', 'H', 'D', 'C'] 
                      for v in range(1, 14)]

    def __iter__(self):
        return self.DeckIterator(self.cards)

    class DeckIterator:
        def __init__(self, cards):
            self._cards = cards
            self._index = 0

        def __iter__(self):
            return self

        def __next__(self):
            if self._index < len(self._cards):
                card = self._cards[self._index]
                self._index += 1
                return card
            raise StopIteration

# Usage
my_deck = Deck()
for card in my_deck: # Uses Inner Class Iterator implicitly
    print(card)
    break # Just print first for brevity

```

* **Why Interviewers Ask This:** Combining Iterators and Inner Classes is a strong OOP pattern.
* **Real App Usage:** Streaming large datasets where the iterator maintains state separate from the collection.

---

### 6. Computer Builder Pattern (Medium)

**Question:** Implement a `Computer` class with a private inner class `Builder`. The `Computer` cannot be instantiated directly (raise error). Users must use `Computer.Builder().set_cpu().set_ram().build()` to get an instance.

**Solution:**

```python
class Computer:
    def __init__(self, builder):
        self.cpu = builder.cpu
        self.ram = builder.ram
        self.storage = builder.storage

    def __str__(self):
        return f"CPU: {self.cpu}, RAM: {self.ram}, Storage: {self.storage}"

    class Builder:
        def __init__(self):
            self.cpu = "Generic"
            self.ram = "8GB"
            self.storage = "256GB"

        def set_cpu(self, cpu):
            self.cpu = cpu
            return self # Fluent interface

        def set_ram(self, ram):
            self.ram = ram
            return self

        def set_storage(self, storage):
            self.storage = storage
            return self

        def build(self):
            # Pass the builder instance to the Outer class
            return Computer(self)

# Usage
my_pc = Computer.Builder()\
    .set_cpu("Intel i9")\
    .set_ram("32GB")\
    .build()

print(my_pc)

```

* **Common Mistakes:** Forgetting to return `self` in setter methods prevents method chaining.
* **Scenario:** Constructing complex objects like API Requests or SQL Queries.

---

### 7. Outer-Inner Communication (The Chat Room) (Medium)

**Question:** Create a `ChatRoom` class. Inside it, a `User` class. When a `User` sends a message, it should append the message to the `ChatRoom`'s history list. This requires the Inner `User` to reference the Outer `ChatRoom`.

**Solution:**

```python
class ChatRoom:
    def __init__(self, title):
        self.title = title
        self.history = []

    def create_user(self, name):
        return self.User(name, self) # Pass 'self' (chatroom) to user

    class User:
        def __init__(self, name, chatroom_instance):
            self.name = name
            self.chatroom = chatroom_instance # Explicit link

        def send_message(self, text):
            full_msg = f"[{self.chatroom.title}] {self.name}: {text}"
            self.chatroom.history.append(full_msg)

# Usage
room = ChatRoom("Python Devs")
user1 = room.create_user("Dev1")
user2 = room.create_user("Dev2")

user1.send_message("Hello World")
user2.send_message("Hi there")

print(room.history)

```

* **Key Concept:** Coupling. The User is tightly coupled to the Room instance.
* **Real App Usage:** WebSocket connections where a client session is tied to a specific server hub/room.

---

### 8. Serialization (DTO) Helper (Medium)

**Question:** You have a `User` model. You want to serialize it to JSON, but only specific fields. Create a method `to_json` that uses an inner class `Schema` to define which fields are allowed, and return the filtered dictionary.

**Solution:**

```python
class User:
    def __init__(self, user_id, name, password, email):
        self.user_id = user_id
        self.name = name
        self.password = password # Should not be serialized
        self.email = email

    class Schema:
        # Define allowed fields for public API
        ALLOWED_FIELDS = {'user_id', 'name', 'email'}

    def to_dict(self):
        data = {}
        for field in self.Schema.ALLOWED_FIELDS:
            data[field] = getattr(self, field)
        return data

# Usage
u = User(1, "John", "secret123", "john@example.com")
print(u.to_dict()) # Password is excluded

```

* **Developer Scenario:** Rest API responses. Keeping the serialization logic close to the model but separated namespaces.

---

### 9. State Memento Pattern (Medium)

**Question:** Implement a `TextEditor` class. It should have a `save()` method that returns a `Memento` object (Inner Class). The `Memento` stores the state of the editor. Implement `restore(memento)` to revert state.

**Solution:**

```python
class TextEditor:
    def __init__(self):
        self._content = ""

    def write(self, text):
        self._content += text

    def show(self):
        return self._content

    def save(self):
        return self.Memento(self._content)

    def restore(self, memento):
        self._content = memento.get_state()

    # Inner class acts as a frozen snapshot
    class Memento:
        def __init__(self, state):
            self._state = state
        
        def get_state(self):
            return self._state

# Usage
editor = TextEditor()
editor.write("Hello")
saved_state = editor.save() # Snapshot

editor.write(" World")
print(editor.show()) # Hello World

editor.restore(saved_state)
print(editor.show()) # Hello

```

* **Why Interviewers Ask This:** Tests understanding of state encapsulation. The Memento is an opaque object to the outside world.

---

### 10. Neural Network Layer (Medium)

**Question:** Simulate a `NeuralNetwork` class. Use an Inner Class `Layer` (with random weights). The Network initializes with a list of layer sizes (e.g., `[2, 3, 1]`) and creates the corresponding `Layer` objects internally.

**Solution:**

```python
import random

class NeuralNetwork:
    class Layer:
        def __init__(self, n_inputs, n_neurons):
            # Create simple weights matrix (random float)
            self.weights = [[random.random() for _ in range(n_inputs)] 
                            for _ in range(n_neurons)]
        
        def __repr__(self):
            return f"Layer(Neurons: {len(self.weights)}, Inputs: {len(self.weights[0])})"

    def __init__(self, layer_sizes):
        self.layers = []
        for i in range(len(layer_sizes) - 1):
            n_in = layer_sizes[i]
            n_out = layer_sizes[i+1]
            # Instantiate Inner Class
            self.layers.append(self.Layer(n_in, n_out))

    def describe(self):
        for idx, layer in enumerate(self.layers):
            print(f"L{idx}: {layer}")

# Usage
nn = NeuralNetwork([2, 4, 1]) 
# Creates: 
# Layer 1: 2 inputs -> 4 neurons
# Layer 2: 4 inputs -> 1 neuron
nn.describe()

```

* **Real App Usage:** Machine Learning libraries (PyTorch/TensorFlow) where layers are often components of a larger Model class.
* **Complexity:** Construction O(N*M) where N is layers and M is weights per layer.
