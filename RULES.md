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