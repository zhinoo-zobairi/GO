# Variables
- Variable Declaration outside a function, globally only:
```GO
var newVar int
newVar = 12
```
- Variable Declaration **inside** a function, more concise with walrus operator:
```GO
newVar := 12
```
- The above operator is not allowed with the `const` keyword. Constants can be primitive types, not more complex types. Their valued can't be changed after declaration.
- So, if you don't use `var x string`, you either use the **walrus operator** or the `const`. 
- Declaring a constant if its value is computed at run-time, is not allowed, unlike JS:
```GO
const newVar = time.Now()
// the current time can only be known when the program is running
```
- Multiple variable declaration on the same line:
```GO
newVar, oldVar := 12, "Toyota"
```
- GO can infer the type by the value -> type inference!
- GO is statically typed (variable types are known before the code runs.)
- GO can be compiled: The Go compiler's job is to take Go code and produce machine code, an `.exe` file on Windows or a standard *executable* on Mac/Linux. CPU doesn't understand anything other than binary.
- Each GO program includes a small amount of extra code that's included in the *executable binary* called the **Go Runtime**, which **manages memory**; for example it cleans up unused memory at runtime.
- GO has garbage collection, which automatically frees up memory, that's no longer in use. That's why it uses up more memory in comparison to C or Rust.
- `package main` lets the Go compiler know that we want this code to compile and run as a standalone program, as opposed to being a library that's imported by other programs.
- We don't download the original source code, when we want to play a game on our PC. We just download the executable file (the compiled code). We may not have the compiler at all. We just have the executable that runs independantly. That is the convinient point about compiled languages: We just add the pre-compiled binary to the server and start it up. In interpreted languages however, we need both the interpreter (runtime language dependencies) and the source code. Because interpreter reads the high-level code and at run-time converts it to binary code.
- Generally speaking, languages that compile directly to machine code produce programs that are faster than interpreted programs.
- GO doesn't run as fast as its compiled counterparts but compiles faster than them.
- Casting a float to an integer truncates the floating point portion:
```GO
temperatureFloat := 88.26
temperatureInt := int64(temperatureFloat)
```
- The only reason to deviate from the default types is to squeeze out every last bit of performance when you are writing an application that is resource-constrained. 
# Conditionals
- `if` statements in Go do not use parentheses around the condition:
- Unlike other languages, you must put the opening brace on the same line as the condition and not on a new line.
- The variable(s) created in the initial statement are only defined within the scope of the if body. The code looks more concise this way.
```GO
if INITIAL_STATEMENT; CONDITION {
}

// For Example, length isn't available in the parent scope:
if length := getLength(email); length < 1 {
    fmt.Println("Email is invalid")
}

// Instead of:
length := getLength(email)
if length < 1 {
    fmt.Println("Email is invalid")
}
```
- Switch statements are a way to compare a value against multiple options and are used when the number of options is more than 2.
```GO
func getCreator(os string) string {
    var creator string
    switch os {
        case "linux":
            creator = "Linus Torvalds"
        case "windows":
            creator = "Bill Gates"
    
    // all three of these cases will set creator to "Jobs"
        case "mac":
            fallthrough
        case "macOS":
            fallthrough
        case "Mac OS X":
            creator = "Jobs"
        default:
            creator = "Unknown"
    }
    return creator
}
```
- Variables declared inside a block (with := or var) only live inside that block and its nested blocks.
    - if { ... }
    - for { ... }
    - switch { ... }
    - Plain { ... } braces

# Functions
- `func sub(x int, y int) int` is known as the **function signature** and accepts two integer parameters and returns another integer.
- When multiple arguments are of the same type, and are next to each other in the function signature, the type only needs to be declared after the last argument:
```Go
func addToDatabase(hp, damage int, name string, level int) {
  // ?
}
```
- Go's declarations are clear, you just read them left to right, just like you would in English:
```Go
f func(func(int,int) int, int) int
```
`var name type` still holds here:
- `name` = f
- `type` = func(func(int,int) int, int) int
    - Now, let's read the function type itself:
    ```Go
    func(               // f is a function that takes:
        func(int,int) int,  // 1st parameter: a function taking (int,int) and returning int
        int                 // 2nd parameter: an int
    ) int               // and returns an int
    ```
- So, `f` is a variable whose type is **“function that takes (function, int) and returns int”**.
- Variables in Go are **passed by value**.
- A function can **return a value** that the **caller doesn't care about**. We can explicitly ignore variables by using an **underscore** = the **blank identifier _**.
```Go
func getPoint() (x int, y int) {
    return 3, 4
}

// ignore y value
x, _ := getPoint()
```
- The Go compiler will throw an error if you have any unused variable declarations in your code, so you need to ignore anything you don't intend to use.

- A return statement without arguments returns the named return values. This is known as a **"naked"** return. In other words, when you choose to omit return values, it's called a **naked return**. Naked returns should only be used in short and simple functions.
```Go
func getCoords() (x, y int) {
	// x and y are initialized with zero values

	return // automatically returns x and y
}
```
- `x` and `y` are the return values. At the end of the function, we could simply write `return` to return the values of those two variables, rather than writing `return x,y`. It is the same as:
```Go
func getCoords() (int, int) {
	var x int
	var y int
	return x, y
}
```
- Named return parameters are particularly important in longer functions with many return values.
- **Guard Clauses** leverage the ability to `return` early from a function (or `continue` through a loop) to make nested conditionals one-dimensional.
- When writing code, it’s important to try to **reduce the cognitive load** on the reader by reducing the number of entities they need to think about at any given time. With the **one-dimensional structure** offered by guard clauses, it’s as simple as stepping through each case in order -> **Linear approach to logic trees**
- **Anonymous functions** are true to form in that they have no name. They're useful when defining a function that will only be used once or to create a quick **closure**:
```GO
func double(a int) int {
    return a + a
}

func main() {
    // using a named function
	newX, newY, newZ := conversions(double, 1, 2, 3)

    // using an anonymous function
	newX, newY, newZ = conversions(func(a int) int {
	    return a + a
	}, 1, 2, 3)
}
```
- The `defer` keyword is a fairly unique feature of Go. It allows a function to be **executed automatically** just **before its enclosing function returns**. The deferred call's arguments are evaluated immediately, but the function call is not executed until the surrounding function returns. Deferred functions are typically used to clean up resources that are no longer being used. Often to close database connections, file handlers and the like:
```Go
func GetUsername(dstName, srcName string) (username string, err error) {
	// Open a connection to a database
	conn, _ := db.Open(srcName)

	// the `conn.Close()` function is not called here:
	defer conn.Close()

	username, err = db.FetchUser()
	if err != nil {
		// The defer statement is auto-executed if we return here:
		return "", err
	}

	// Or here:
	return username, nil
}
```
- So if you want a function to be called **just before** your main function returns, you defer it.
- Unlike Python, Go is not function-scoped, it's **block-scoped**. Variables declared inside a block are only accessible within that block (and its nested blocks). Blocks are defined by curly braces `{}`:
```Go
package main

import fmt

func main() {
    {
        age := 19
        // this is okay
        fmt.Println(age)
    }

    // this is not okay
    // the age variable is out of scope
    fmt.Println(age)
}
```
- A **closure** is a function that references variables from outside its own function body. The function may access and assign to the referenced variables:
    - Create an enclosed sum value inside the `adder()` function.
    - Return a function from the `adder()` function that adds its input (an `int`) to the sum and returns the new value of sum. (In other words, it keeps a running total of the sum variable within a closure.)
    ```Go
    package main

    func adder() func(int) int {
        var sum int
        return func(a int) int{
            sum += a
            return sum
        }
    }
    ```
    - sum lives inside adder.
    - The returned anonymous function “closes over” sum and keeps using (and mutating) the same sum each time it’s called:
        - “The inner function remembers and can use variables from the outer function, even after the outer function has finished running.”
    - Nowhere does the function call itself, so there’s no recursion.
- Function **currying** is a concept from **functional programming** and involves **partial application of functions**. It allows a function with multiple arguments to be transformed into a sequence of functions, each taking a single argument.

# Struct
- Go is not an object-oriented language. 
- A struct is:
```Go
type car struct {
	brand      string
	model      string
	doors      int
	mileage    int
}
```

- How to check if a string is empty in GO? 

    - if `mToSend.recipient.name == ""` or `len(mToSend.recipient.name) > 0`
- An **anonymous struct** is just like a normal struct, but it is defined without a name and therefore cannot be referenced elsewhere in the code.
    -  How? Just **instantiate the instance** immediately using a second pair of brackets after declaring the type:
    ```Go
    myCar := struct {
    brand string
    model string
    } {
    brand: "Toyota",
    model: "Camry",
    }
    ```
- You can even nest anonymous structs as fields within other structs:
```Go
type car struct {
  brand string
  model string
  doors int
  mileage int
  // wheel is a field containing an anonymous struct
  wheel struct {
    radius int
    material string
  }
}

var myCar = car{
  brand:   "Rezvani",
  model:   "Vengeance",
  doors:   4,
  mileage: 35000,
  wheel: struct {
    radius   int
    material string
  }{
    radius:   35,
    material: "alloy",
  },
}
```

- Keep in mind, Go doesn't support classes or inheritance in the complete sense, but **embedded structs** are a way to elevate and share fields between struct definitions:
```Go
type car struct {
  brand string
  model string
}
type truck struct {
  // "car" is embedded, so the definition of a "truck" now also additionally contains all of the fields of the car struct
  car
  bedSize int
}
```
- The difference to nested structs here is: we can directly access the top-level struct, without needing to mention the top-level struct again:
 ```Go
lanesTruck := truck{
  bedSize: 10,
  car: car{
    brand: "Toyota",
    model: "Tundra",
  },
}
fmt.Println(lanesTruck.brand) // Without saying lanesTruck.car.brand, we directly access model and brand, as if it were a field from lanesTruck directly
fmt.Println(lanesTruck.model) // Instead of lanesTruck.car.model
```
- While Go is not object-oriented, it does support **methods** that can be defined on structs. Methods are just functions that have a **receiver**. A receiver is just **a special kind of function parameter** that syntactically **goes before the name of the function** `func (receiver) name(params) type {}`
```Go
type rect struct {
  width int
  height int
}
// area has a receiver of (r rect)
// rect is the struct
// r is the placeholder
func (r rect) area() int {
  return r.width * r.height
}
var r = rect{
  width: 5,
  height: 10,
}
fmt.Println(r.area())
// prints 50
```
- When you define a method with a receiver, that method automatically has access to all the fields of that specific instance
- Field names in structs follow the Go convention: fields whose name starts with a **lower** case letter are **only visible to code** in the same package, whereas those whose name starts with an **upper** case letter are **visible in other packages**.
## Memory Layout
- In Go, structs sit in memory in a contiguous block, with fields placed one after another as defined in the struct. The order of fields in a struct can have a big impact on memory usage. If you order the fields poorly (from smallest to largest), Go adds some **padding** to make up for the size difference. It's done for execution speed, but it can lead to increased memory usage. If you have a specific reason to be concerned about memory usage, aligning the fields by size **(largest to smallest)** can help.
- Empty structs are used in Go as a unary value:
```Go
// anonymous empty struct type
empty := struct{}{}

// named empty struct type
type emptyStruct struct{}
empty := emptyStruct{}
```

![alt text](memory_usage.png)
## Instantiation
### Instantiation Type 1: Struct Literals with Field Names
This is the recommended and most common way to instantiate a struct when you want to set specific values for its fields.

`Syntax: varName := MyStruct{FieldName1: value1, FieldName2: value2}`

Key Characteristics:
- Clarity: You explicitly name each field, making the code very readable and easy to understand which value goes to which field.
Order doesn't matter: You can list the fields in any order.
- Partial Initialization: You can choose to initialize only a subset of the fields. Any fields you omit will automatically be assigned their zero value (e.g., 0 for int, "" for string, false for bool).
- Flexibility: Great for creating a new struct value with specific initial data.
```Go
Example:

type Person struct {
    Name string
    Age  int
    City string
}

// Full initialization
p1 := Person{Name: "Alice", Age: 30, City: "New York"}

// Partial initialization (City will be "")
p2 := Person{Name: "Bob", Age: 25}
```

### Instantiation Type 2: Struct Literals without Field Names (Positional Initialization)

This method is less common and can be error-prone. It relies on the order of fields.

`Syntax: varName := MyStruct{value1, value2, value3}`
Key Characteristics:
- Order is critical: The values must be provided in the exact order the fields are declared in the struct definition.
All fields required: You must provide a value for every field in the struct.
- Error-Prone: If you change the order of fields in the struct definition, or if you forget a value, your code will compile with an error. This is exactly what you discovered! If you try to provide only one value when there are multiple fields, Go doesn't know which field it corresponds to, hence the error.
Example:
```Go
type Item struct {
    ID    int
    Name  string
    Price float64
}

// Correct usage (all fields in order)
item1 := Item{101, "Sword", 50.0}

// Incorrect usage (will cause a compile-time error)
// item2 := Item{202, "Shield"} // Error: too few values in struct literal
```

## Nested and Embedded instantiations

Consider these structs:
```Go
package main

import "fmt"

type Address struct {
	Street  string
	City    string
	ZipCode string
}

type Person struct {
	Name    string
	Age     int
	Address Address // Nested struct
	Contact // Embedded struct (anonymous field)
}

type Contact struct {
	Email string
	Phone string
}
```

### 1. Nested Struct Instantiation (e.g., Address inside Person)
You use a struct literal for the Person, and then inside that literal, you use another struct literal for the Address field.
```Go
func main() {
	// Instantiation using Struct Literals with Field Names (Type 1)
	p1 := Person{
		Name: "Alice",
		Age:  30,
		Address: Address{ // <--- Here's the nested struct instantiation
			Street:  "123 Magic Lane",
			City:    "Enchanted Forest",
			ZipCode: "90210",
		},
		// Contact fields will be zero-valued for now
	}
	fmt.Printf("Person with nested struct: %+v\n", p1)

	// You could also partially initialize the nested struct:
	p2 := Person{
		Name: "Bob",
		Address: Address{ // Only setting Street, City and ZipCode will be zero-valued
			Street: "456 Wizard Way",
		},
	}
	fmt.Printf("Person with partially initialized nested struct: %+v\n", p2)
}
```

### 2. Embedded Struct Instantiation (e.g., Contact inside Person)
This is where it gets interesting because of how Go's embedding works. When you embed a struct, its fields are "**promoted**" to the outer struct. This gives you two main ways to initialize them:

#### a. Initializing the Embedded Struct Directly within the Outer Struct Literal
You treat the embedded struct as a field with its type name.
```Go
func main() {
	p3 := Person{
		Name: "Charlie",
		Age:  40,
		Address: Address{
			Street: "789 Spell Rd",
			City:   "Mystic Town",
		},
		Contact: Contact{ // <--- Initializing the embedded struct directly
			Email: "charlie@example.com",
			Phone: "555-1111",
		},
	}
	fmt.Printf("Person with embedded struct initialized directly: %+v\n", p3)
}
```

#### b. Initializing Promoted Fields of the Embedded Struct
Because the fields of Contact (Email, Phone) are promoted to Person, you can initialize them *as if they were direct fields of Person.*
```Go
func main() {
	p4 := Person{
		Name:  "Dana",
		Age:   25,
		Email: "dana@example.com", // <--- Initializing promoted field directly
		Phone: "555-2222",     // <--- Initializing promoted field directly
		// Address fields will be zero-valued for now
	}
	fmt.Printf("Person with promoted fields initialized: %+v\n", p4)

	// What if you use both? The explicit 'Contact' field takes precedence
	p5 := Person{
		Name:    "Eve",
		Email:   "eve_promoted@example.com",
		Contact: Contact{Email: "eve_direct@example.com", Phone: "555-3333"},
	}
	// The Email from 'Contact' will be "eve_direct@example.com"
	fmt.Printf("Person with both promoted and direct initialization: %+v\n", p5)
	fmt.Printf("Eve's email: %s\n", p5.Email) // Accesses the promoted field
}
```

Key Takeaways:

- For both nested and embedded structs, you're fundamentally still using the **struct literal syntax ({Field: Value})** to create and populate them.
- The main difference for embedded structs is the added convenience of being able to initialize their fields directly through the "outer" struct **as if they were its own fields**, thanks to **field promotion**.
- The "Type 2" **(positional) instantiation** is still an option for nested/embedded structs if you know the order of their fields, but it carries the same risks of being brittle and hard to read.

# Interfaces
- Interfaces are just **collections of method signatures**.
- Allows you to focus on what a type does, rather than how it's built.
- A type **implements** an interface if it has methods that match the interface's method signatures.
- **Interface** is ONLY used for **polymorphism** and doesn't provide any features
- This is different from **abstraction**, where we write an abstract class (with abstract methods), we, as the **super-class** can also provide a few features for the sub-class -> **Inheritance**:
`abstract class Test1{};` which includes something like `abstract public void meth1()` -> `class Test2 extends Test1{};` is now **concrete**, because it overwrite all methods from super-class, otherwise it will also become abstract.
- We can have a super-class reference and an object of the sub-class `Test1 t = new Test2();`
- Although this is an inheritance, it has only achieved **polymorphism**. So, we could instead only use an **Interface**:
`interface Test1{};` which includes something like `void meth1()` – No need to write abstract public, because that's what they already are! -> `class Test2 implements Test1{};` now overwrites all the methods. 
- In Go, unlike Java, we don't need the "implements" keyword. We just take the interfaces inside our method and complete them, as if they were a method for our struct: `func (receiver) name(params) type {}`. Implicit interfaces decouple the definition of an interface from its implementation. **Decouple** means:
    - The person who writes the type doesn’t need to know about the interface.
    - The person who writes the interface doesn’t need to touch the type.
- When **a type implements an interface**, it can then be **used as that interface type**: For example, you have an interface called `X` and you implement it inside struct `Y`. You can from now on, refer to your struct type `Y` as type `X`. The `X` interface says: "Any type that has all my methods can be an `X`".
- A type "implements" an interface if it has **all** of the methods of the given interface defined on it.
- A type can implement **any number** of interfaces in Go. For example, the **empty interface**, **interface{}**, is always implemented by **every type** because it has **no requirements**.
## Type Assertion
- When working with interfaces in Go, every once-in-awhile you'll need **access to the underlying type of an interface value**. You can **cast** an interface to its underlying type using a **type assertion**:
```Go
func getExpenseReport(e expense) (string, float64) {
    // At this point, e could be email, sms, OR invalid
    // You need to figure out WHICH ONE it actually is
    
    em, ok := e.(email)  // Is it an email?
    if ok {
        return em.toAddress, em.cost()  // Only email has toAddress
    }
    
    s, ok := e.(sms)  // Is it an sms?
    if ok {
        return s.toPhoneNumber, s.cost()  // Only sms has toPhoneNumber
    }
    
    // If it's neither, it must be invalid (or some other type)
    return "", 0.0
}
```
- The syntax `em, ok := e.(email)` is called a **type assertion** in Go. Here's what each part does:
    - `e` - This is your interface value (in your case, an expense)
    - `.(email)` - This is the type assertion, asking "is e actually an email?"
    - `em` - This receives the underlying email value if the assertion succeeds
    - `ok` - This is a boolean that tells you whether the assertion succeeded (true) or failed (false)
    1. Go checks if the concrete type stored in the interface `e` is actually an `email`
    2. If it is an `email`
        - `em` gets the actual email value (so you can access fields like `em.toAddress`)
        - `ok` is set to `true`
    3. If it is not an `email`:
        - `em` gets the zero value of the `email` type
        - `ok` is set to `false`
## Type Switches
- A type switch makes it easy to do several type assertions in a series. A type switch is similar to a regular switch statement, but the cases specify **types** instead of values:
```Go
func getExpenseReport(e expense) (string, float64) {
	switch v := e.(type) {
		case email:
			return v.toAddress, v.cost()
		case sms:
			return v.toPhoneNumber, v.cost()
		default:
			return "", 0.0
	}
}
```
## Embedded Interface
```Go
type firetruck interface {
	car
	HoseLength() int
}
```
It inherits the required methods from `car` as an embedded interface and adds one additional required method to make the `car` a `firetruck`.
# Errors
- Go programs express errors with `error` values. An `Error` is any type that implements the simple built-in `error` interface:
```Go
type error interface {
    Error() string
}
```
- When something can go wrong in a function, that function should return an `error` as its **last return value**.
- Any code that calls a function that can return an `error` should handle errors by testing whether the error is `nil`.
- Because errors are just interfaces, we can build our own custom types that implement the `error` interface. 
- Keep in mind that the way Go handles errors is fairly unique. Most languages treat errors as something special and different. For example, Python raises exception types and JavaScript throws and catches errors. In Go, an `error` is just another value that we handle like any other value - however we want! There aren't any special keywords for dealing with them.

## custom error type vs errors.New
### Approach 1: Custom error type

```Go
type divideError struct {
	dividend float64
}
```
This is just a normal Go struct.
Then I made it satisfy the error interface:
```Go

func (d divideError) Error() string {
	return fmt.Sprintf("can not divide %v by zero", d.dividend)
}
```
The error interface is:
```Go

type error interface {
	Error() string
}
```
So any type with an `Error()` string method can be used as an error.
When I do:

`return 0, divideError{dividend: dividend}`

Go stores a value of type `divideError` inside an interface value of type `error`.
Under the hood, an interface value holds:

- a concrete type (divideError)
- a value of that type (my struct)
This lets me later do things like:
```Go
if e, ok := err.(divideError); ok {
    // you can inspect e.dividend
}
```
### Approach 2: `errors.New("no dividing by 0")`
`errors.New` returns a value of type error.
Under the hood in the standard library, it looks roughly like this:
```Go
func divide(x, y float64) (float64, error) {
	if y == 0 {
		var err error = errors.New("no dividing by 0")
		return 0, err
	}
	return x / y, nil
}
```

So `errors.New` is just returning a simple struct that holds a string.
No extra fields, no custom data, no type-specific behavior—just a message.

So: `errors.New` = lightweight, **message-only** error.

## Panic/Recover
Panic and recover shouldn't be used instead of errors, because:
1. Crash control flow: panic jumps out of normal execution, making code harder to reason about and debug.
2. Hide real error handling: Using recover instead of returning error values breaks Go’s idiomatic style and makes APIs unclear.
3. Are easy to misuse: It’s tempting to treat panic/recover like exceptions, which leads to fragile, hard‑to‑maintain code.

- If you want your program to cleanly exit in an unrecoverable way, use `log.Fatal()`.

# Loops
- The basic loop in Go is written in standard C-like syntax, without the parantheses:
```Go
for INITIAL; CONDITION; AFTER{
}
```
## While in GO is a for-loop that only has a CONDITION:
```GO
for CONDITION {
  // do some stuff while CONDITION is true
}
```
- If you omit the condition, you will get an infinite loop.

- Whenever we want to change the control flow of a loop we can use the `continue` and `break` keywords.
## continue
- The `continue` keyword stops the current iteration of a loop and continues to the next iteration. `continue` is a powerful way to use the guard clause pattern within loops.
## break
- The `break` keyword stops the current iteration of a loop and exits the loop.
- Go provides syntactic sugar **Range** to iterate easily over elements of a slice:
```Go
for INDEX, ELEMENT := range SLICE {
}
```
- The element is a copy of the value at that index.
# Slices

* Go has two related sequence types:

  * **Array**: fixed-size container that physically stores its elements.
  * **Slice**: dynamic, runtime-sized view over an underlying array.

* Arrays matter because slices are built on top of arrays.

* Arrays are **fixed-size** collections of the **same type**.

* The array size is part of the type, not just metadata.

* Array type syntax is **`[N]T`**, and `N` must be a **compile-time constant**.

* Consequences:

  * `[3]int` and `[4]int` are different types.
  * You cannot use runtime values like `len(messages)` inside `[N]T`.

* Array examples:

  ```Go
  var myInts [10]int
  primes := [6]int{2, 3, 5, 7, 11, 13}
  ```

* Fixed-size constraint: `[10]int` cannot grow to 11 elements.

* Slice type syntax is **`[]T`** with empty brackets, meaning length is not part of the type.

* A slice is a flexible view with runtime-controlled length.

* The zero value of a slice is **`nil`**.

* Slices are designed for sizes known **at runtime**:

  * `length := len(messages)` is runtime information.
  * You can create a slice with that length.
  * You cannot create an array with that length.

* Slicing an array creates a view into a region of the array:

  ```Go
  primes := [6]int{2, 3, 5, 7, 11, 13}
  mySlice := primes[1:4]
  ```

* Slicing syntax: `arrayName[lowIndex:highIndex]`

  * `lowIndex` is inclusive
  * `highIndex` is exclusive

* Internals that explain most slice behavior:

  * A slice is a small header containing:

    * a pointer to the first element in the underlying array
    * the length
    * the capacity
  * Passing a slice to a function copies the header, but the pointer still targets the same underlying array, so element updates are shared.

* Two boundaries you must keep clear:

  * Mutating elements vs changing the slice header:

    * `s[i] = ...` mutates the underlying array and is visible to other slices sharing it.
    * `s = s[:2]` only changes the local slice header, not the caller’s slice header.
  * `append` can break sharing:

    * If capacity is sufficient, `append` grows within the same underlying array.
    * If capacity is insufficient, Go allocates a new underlying array and copies elements, so the returned slice may point somewhere else.

* Creating slices:

  * `make([]T, len, cap)` creates a slice with allocated backing storage.
  * Capacity is usually omitted and defaults to length.
  * Elements are initialized to the zero value.

  ```Go
  costs := make([]float64, length)
  ```

  * A slice literal creates a slice with initial values:

  ```Go
  mySlice := []string{"I", "love", "go"}
  fmt.Println(len(mySlice))
  fmt.Println(cap(mySlice))
  mySlice[0] = "you"
  ```

* Empty vs nil slices:

  * `var s []T` is a **nil slice**.
  * `[]T{}` is an **empty, non-nil slice** (len 0, but initialized).
  * For a nil slice, `len(s)` and `cap(s)` both return 0.

* One-line rule (fast recall):

  * Brackets with a number: **array**, number must be **compile-time**: `[N]T`
  * Empty brackets: **slice**, size can be **runtime**: `[]T`

* Variadic functions and slices:

  * A variadic parameter `(...T)` is received inside the function as a `[]T`.
  * `fmt.Println()` and `fmt.Sprintf()` are variadic.
  * The spread operator `...` expands a slice into variadic arguments at the call site.

* `append` fundamentals:

  * `append` grows a slice dynamically and returns the updated slice, so you typically reassign:

    * `s = append(s, x)`
  * If the underlying array is too small, `append` allocates a new one and the returned slice points to it.

* Learning from the `filterMessages` mistake:

  * If you plan to **append**, your slice should start with **length 0**.
  * `make([]T, len)` creates a slice that already contains `len` zero values, so appending adds after them.
  * Use:

    * `make([]T, 0, cap)` when building via `append`
    * `make([]T, len)` when you will assign by index (`s[i] = ...`)
  * Correct pattern:

  ```go
  func filterMessages(messages []Message, filterType string) []Message {
      filtered := make([]Message, 0, len(messages))
      for _, msg := range messages {
          if msg.Type() == filterType {
              filtered = append(filtered, msg)
          }
      }
      return filtered
  }
  ```
