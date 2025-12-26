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