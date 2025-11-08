### 00. How to start go
- Installing Go: Go installation and GOPATH setup.
- Understanding the Workspace: Explanation of go.mod and GOPATH.
- Basic Commands:
```bash
go mod init project_name   # initial a project
go run file_name.go        # run file
go run project_name        # run entire project
go get -u "repo link"      # import module
go mod tidy                # Update import mod after placing the module repo in the require section of go.mod
go build                   # build exe

```      
## Table of Contents
1. [Data Types and common variable rules](#01-Data-Types-and-common-variable-rules)
2. [Statement](#02-Statement)
3. [Loop](#03-Loop)
4. [Function](#04-Function)
5. [call back function](#05-call-back-function)
6. [Common Struct and method ](#06-Common-Struct-and-method )
7. [Common Built-in methods](#07-Common-Built-in-methods)
8. [String manipulation](#08-String-manipulation)
9. [List, Slice, Map, Struct also project base usecase](#09-List-Slice-Map-Struct-also-project-base-usecase)
10. [OOP Concept with Interface and struct](#10-OOP-Concept-with-Interface-and-struct)
11. [Threading or goroutine, Basic Request, Data Scrapping etc](#11-Threading-or-goroutine-Basic-Request-Data-Scrapping-etc)
12. [Error Handling](#12-Error-Handling)
13. [File Handling and IO](#13-File-Handling-and-IO)
14. [Others](#14-Others)


### 01. Data Types and common variable rules
[Return Table of Contents](#Table-of-Contents)
- **Variable Declaration**: 
  - Use `var` for explicit declarations:  
    ```go
    var x int = 10
    ```
  - Use short declaration (`:=`) for concise syntax:  
    ```go
    y := 20
    ```
  - Constants are declared with `const`:  
    ```go
    const pi = 3.14
    ```
- **Zero Values**: 
  - Uninitialized variables are assigned a zero value based on their type:
    - `int`: `0`
    - `float64`: `0.0`
    - `string`: `""`
    - `bool`: `false`

- **Type Aliases**: 
  - Create custom types or aliases using `type`:
    ```go
    type Celsius float64
    var temperature Celsius = 25.5
    ```
  - Type aliases help improve code readability and enforce type safety.

- **Array**: 
	```go
	var numbers [3]int // numbers will be [0, 0, 0]
	names := [...]string{"Alice", "Bob", "Charlie"} // Length will be 3
 	_ = append(names[:], "samrat")
    links := []string{} // links is a dynamic slice, starts at length 0
    // The result of 'append' must be assigned back to the slice variable
    links = append(links, "http://example.com")
	```
- **Map**:
	- Simple map
	```go
	func main() {
	    // 1. Define and initialize a simple map
	    populations := make(map[string]int) // key string and value int
	    // 2. Add elements
	    populations["New York"] = 8398748
	    // 3. Access an element
	    fmt.Printf("Population of Los Angeles: %d\n", populations["Los Angeles"])
	    // 4. Update an element
	    populations["Chicago"] = 2705995
	    // 5. Delete an element
	    delete(populations, "New York")
	    // 6. Iterate over the map
	    fmt.Println("\nRemaining Cities:")
	    for city, pop := range populations {
	        fmt.Printf("%s: %d\n", city, pop)
	    }
	}
	```
	- Map with struct
	```go
	// Define the User struct
	type User struct {
	    Email string
	    Active bool
	}
	
	func main() {
	    // Define a map where the key is an integer User ID and the value is a User struct
	    users := make(map[int]User)
	    // Add users to the map
	    users[101] = User{Email: "alice@example.com", Active: true}
	    users[102] = User{Email: "bob@example.com", Active: false}
	    // Access the struct and its fields
	    fmt.Printf("User 101 Email: %s\n", users[101].Email)
	    // To modify a struct *value* within a map, you must copy it out, modify it, and put it back in:
	    user102 := users[102]
	    user102.Active = true
	    users[102] = user102
	    fmt.Printf("User 102 Active Status: %v\n", users[102].Active)
	}
	```
	- map with interface
	```go
	func main() {
	    // Define a map where keys are strings, and values can be anything (interface{})
	    userData := make(map[string]interface{})
        // or userData := make(map[string]any)
	
	    // Add different types of values
	    userData["username"] = "gopher1984"
	    userData["userID"] = 55
	    userData["balance"] = 123.45
	    userData["is_admin"] = false
	    fmt.Println(userData)
	    // Accessing values requires a type assertion to use them safely:
	    if username, ok := userData["username"].(string); ok {
	        fmt.Printf("Username is a string: %s\n", username)
	    }
	    if balance, ok := userData["balance"].(float64); ok {
	        fmt.Printf("Balance is a float: %.2f\n", balance)
	    }
	}
	```
- **Type Struct**:
	- Simple uses
	```go
	// Represents a 'Book' entity with specific attributes.
	type Book struct {
	    Title    string
	    Author   string
	    Pages    int
	    Isbn     string
	}
	func main() {
	    // Create an instance of the Book struct
	    book1 := Book{
	        Title:  "The Go Programming Language",
	        Author: "Brian Kernighan",
	        Pages:  384,
	        Isbn:   "978-0134190440",
	    }
	    fmt.Printf("Book Title: %s by %s\n", book1.Title, book1.Author)
	}
	```
	- Uses in method
	```go
	type Rectangle struct {
	    Width, Height float64
	}
	// Method named 'Area' associated with the Rectangle struct
	func (r Rectangle) Area() float64 {
	    return r.Width * r.Height
	}
	// Method named 'Perimeter' associated with the Rectangle struct
	func (r Rectangle) Perimeter() float64 {
	    return 2 * (r.Width + r.Height)
	}
	func main() {
	    rect := Rectangle{Width: 10, Height: 5}
	    fmt.Printf("Rectangle Area: %.2f\n", rect.Area())
	    fmt.Printf("Rectangle Perimeter: %.2f\n", rect.Perimeter())
	}
	```
- **Type Inference**: 
  - The `:=` operator infers the type from the assigned value:
    ```go
    var z interface{}
    var z any
    z := "Hello" // z is inferred as string
    ```
  - Use `var` when you need to explicitly specify the type.
  - 
  
### 02. Statement
[Return Table of Contents](#Table-of-Contents)
- If-Else: Multi-condition handling.
```go
package main
import ("fmt")
func main(){
	var x int;
	fmt.Scan(&x);
	if x%2 == 0 {
		fmt.Println("Even")
	} else if x > 1{
		fmt.Println("Odd")
	} else{
		fmt.Println("Neither")
	}
}
// Define a variable in statement never work in global area so, for that case first declear empty variable in global space
```
- Defer
```go
package main
import "fmt"
func main() {
    defer fmt.Println("This will be printed last")
    fmt.Println("This will be printed first")
}
// This will be printed first
// This will be printed last
```
### 03. Loop
[Return Table of Contents](#Table-of-Contents)
- For Loop with range
```go
fruits := []string{"apple", "banana", "orange"}
for i := range fruits{
	fmt.Println(fruits[i])
}
```
- Index, value
```go
var fruit = []string{"Apple", "Banana", "Watermelon"}
for index, value := range fruit{
	fruit := fmt.Sprintf("index is : %d name is : %s", index, value)
	fmt.Println(fruit)
	// fmt.Printf("Index is : %d value is :%s \n", index, value)
	// %s = string, %d = decimal/int,  %v = value, %t = type, %f =float, %+v = struct with field name
}
```
- Normal Loop in one statement
```go
fruits := []string{"apple", "banana", "orange"}
for i := 0; i<len(fruits); i++{
	fmt.Println(fruits[i])
}
```
- Like while Loop
```go
fruits := []string{"apple", "banana", "orange"}
i := 0
for i < len(fruits){
	fmt.Println(fruits[i])
	i++
}
```
- Infinity loop with break
```go
i := 1
for{
	fmt.Println(i)
	if i == 100{
		break
	}
	i += 1
}
```
- loop with goto
```go
	i := 1
	for{
		fmt.Println(i)
		if i == 100{
			goto finish
		}
		i += 1
	}
	finish:
		fmt.Println("Finished")
```

### 04. Function
[Return Table of Contents](#Table-of-Contents)
- Normal Function
```go
func add(x, y int) int {  // func name(arg1, arg2, type) return_type { area}  return type can be multiple (int, string)
	return x + y
}
func main() {
	result := add(1, -1)
	fmt.Println(result)
}
```
- Function with error handle
```go
package main
import (
	"errors"
	"fmt"
)
func add(x, y int) (int, error) {  // return errpr type with value
	if x < 0 || y < 0 {
		return 0, errors.New("x or y is less than 0")
	}else{
		return x + y, nil
	}
}
func main() {
	// result, _ := add(1, -1)   // if receive only value then  use _ instead or err or this kind
	// fmt.Println(result)

	result, err := add(1, 2)
	if err != nil {
		fmt.Println(err)	
	}else{
		fmt.Println(result)
	}
}
```
- Pass unkwon multiple arguments argument
```go
func calculate(a ...int) int {
	sum := 0
	for i := range a {
		sum += a[i]
	}
	return sum
}
func main() {
	lsit := []int{1, 2, 3, 4, 5}
	result := calculate(lsit...)
	// result = calculate(1, 2, 3, 4, 5)
	fmt.Println(result)
}
```
- Pass array as argument
```go
func calculate(a []int) int {
	sum := 0
	for i := range a {
		sum += a[i]
	}
	return sum
}

func main() {
	lsit := []int{1, 2, 3, 4, 5}
	result := calculate(lsit)
	fmt.Println(result)
}
```


### 05. call back function
[Return Table of Contents](#Table-of-Contents)
- Passing another Functions & Anonymous function
```go
package main

func calculate(x, y int, callback func(int)int) int{
	result := x + y
	return callback(result)
}
func add(x int) int{
	return x * x
}

func main() {
	x, y := 1, 2
	result1 := calculate(x, y, add)  // call another function as a parameter
	result2 := calculate(x, y, func(x int) int{  // call anonymous function, rules is same amount and type args invoked callback
		return x * x * x
	})
	println(result1)
	println(result2)
}
```
### 06. Common Struct and method 
[Return Table of Contents](#Table-of-Contents)
- Simple Struct
```go
package main
import "fmt"
type Student struct {
	id   int
	name string
	GPA  float32
}
func main() {
	// s1 := Student{1, "John", 3.93}
	s1 := Student{id : 1, name : "John", GPA: 3.93}  // best way dosen't give error less args input
	fmt.Println(s1.id)
	s1.id = 2 // change the value of id
	fmt.Println(s1.name)
	fmt.Println(s1.GPA)
	fmt.Println(s1.id)

}
```
- Methods
```go
package main
import "fmt"
type Student struct {
	id   int
	name string
	number  int
	GPA float32
}

func (s *Student) calculateGPA(){   // if dosen't pass as pointer then main struct never update it will return just a copy
	if s.number > 80 {
		s.GPA = 4.0
	}else if s.number > 70 {
		s.GPA = 3.0
	}else {
		s.GPA = 2.0
	}
}

func main() {
	s1 := Student{id : 1, name :"John", number: 75}
	s1.calculateGPA()
	fmt.Println(s1.GPA)
}
```
- Handle slice/array with struct 
```go
func main() {
	// Create a slice of students
	students := []Student{
		{id: 1, name: "Samrat Biswas", number: 60},
		{id: 2, name: "John Doe", number: 85},
		{id: 3, name: "Jane Smith", number: 75},
		{id: 4, name: "Alice Johnson", number: 65},
	}

	// Calculate GPA for each student
	for i := range students {
		students[i].calculateGpa()
	}

	// Print the details of all students
	for _, student := range students {
		fmt.Printf("ID: %d, Name: %s, Number: %d, GPA: %.1f\n",
			student.id, student.name, student.number, student.GPA)
	}
}
```
### 07. Common Built-in methods
[Return Table of Contents](#Table-of-Contents)
- fmt
```go
fmt.Print("Hello, ")
fmt.Print("World!")
// Output: Hello, World!

fmt.Println("Hello, World!")
fmt.Println("Welcome to Go!")
// Hello, World!
// Welcome to Go!

name := "Alice"
age := 30
fmt.Printf("Name: %s, Age: %d\n", name, age)
// Output: Name: Alice, Age: 30

name := "Bob"
age := 25
s := fmt.Sprintf("Name: %s, Age: %d", name, age)
fmt.Println(s)
// Output: Name: Bob, Age: 25

s := fmt.Sprintln("Hello, World!")
fmt.Print(s)
// Output: Hello, World!

err := fmt.Errorf("invalid value: %d", 42)
fmt.Println(err)
// Output: invalid value: 42

var name string
var age int
fmt.Print("Enter your name and age: ")
fmt.Scan(&name, &age)
fmt.Printf("Name: %s, Age: %d\n", name, age)
// Input: Alice 30
// Output: Name: Alice, Age: 30

var name string
var age int
fmt.Print("Enter your name and age: ")
fmt.Scanf("%s %d", &name, &age)
fmt.Printf("Name: %s, Age: %d\n", name, age)
// Input: Alice 30
// Output: Name: Alice, Age: 30

var name string
var age int
fmt.Print("Enter your name and age: ")
fmt.Scanln(&name, &age)
fmt.Printf("Name: %s, Age: %d\n", name, age)
// Input: Alice 30
// Output: Name: Alice, Age: 30

// fmt.Printf("Index is : %d value is :%s \n", index, value)
// %s = string, %d = decimal/int,  %v = value, %t = type, %f =float, %+v = struct with field name, %t = boolean, %p = pointer, %b = binrar
```
- reflect
```go

```
- Time (Now, Sleep, After, Tick)
```go

```
- len()
```go

```
- cap()
```go

```
- make()
```go

```
- new()
```go

```
- copy()
```go

```
- append()
```go

```
- delete()
```go

```
- close()
```go

```
- panic()
```go

```
- recover()
```go

```
- math
```go

```
- strings
```go

```
- sort
```go

```


### 08. String manipulation
[Return Table of Contents](#Table-of-Contents)
- String Operations:
- - Concatenation and splitting.
- - Trimming, prefix, and suffix checks.
- String Formatting (fmt.Sprintf)
- Regular Expressions
- String to other types (strconv)
```go
package main

import (
	"fmt"
	"regexp"
	"strconv"
	"strings"
)

// Helper to print section headers
func header(title string) {
	fmt.Printf("\n=== %s ===\n", title)
}

func main() {
	// 1. Concatenation and Splitting
	header("1. Concatenation & Splitting")
	
	// Concatenation
	s1 := "Hello"
	s2 := "World"
	concat1 := s1 + " " + s2 + "!"                 // Using +
	concat2 := fmt.Sprintf("%s %s!", s1, s2)       // Using fmt.Sprintf
	concat3 := strings.Join([]string{s1, s2, "Go"}, " ") // Using strings.Join

	fmt.Println("Concat +:", concat1)
	fmt.Println("fmt.Sprintf:", concat2)
	fmt.Println("strings.Join:", concat3)

	// Splitting
	sentence := "apple,banana,orange,grape"
	parts1 := strings.Split(sentence, ",")         // Split by comma
	parts2 := strings.SplitN(sentence, ",", 2)     // Split into at most 2 parts
	parts3 := strings.Fields("Go is awesome!")     // Split by whitespace

	fmt.Println("Split:", parts1)
	fmt.Println("SplitN (2):", parts2)
	fmt.Println("Fields:", parts3)

	// 2. Trimming, Prefix, Suffix
	header("2. Trimming, Prefix, Suffix")

	str := "   !!Hello World!!   "

	// Trimming
	trimmed := strings.TrimSpace(str)                    // Remove leading/trailing whitespace
	trimmed = strings.Trim(trimmed, "!")                 // Remove '!' from both ends
	trimmed = strings.TrimLeft(str, " !")                // Left trim
	trimmed = strings.TrimRight(str, " !")               // Right trim
	trimmed = strings.TrimPrefix("###GoLang", "###")     // Remove prefix if exists
	trimmed = strings.TrimSuffix("GoLang@@@", "@@@")     // Remove suffix if exists

	fmt.Println("Original:  '" + str + "'")
	fmt.Println("TrimSpace: '" + strings.TrimSpace(str) + "'")
	fmt.Println("Trim '!':  '" + strings.Trim(str, "!") + "'")
	fmt.Println("TrimPrefix: '" + strings.TrimPrefix("###GoLang", "###") + "'")
	fmt.Println("TrimSuffix: '" + strings.TrimSuffix("GoLang@@@", "@@@") + "'")

	// Prefix / Suffix checks
	fmt.Println("HasPrefix 'Go'?:", strings.HasPrefix("GoLang", "Go"))
	fmt.Println("HasSuffix 'Lang'?:", strings.HasSuffix("GoLang", "Lang"))

	// 3. String Formatting with fmt.Sprintf
	header("3. String Formatting (fmt.Sprintf)")

	name := "Alice"
	age := 25
	price := 99.99
	active := true

	// Different formatting verbs
	formatted1 := fmt.Sprintf("Name: %s, Age: %d", name, age)
	formatted2 := fmt.Sprintf("Price: $%.2f", price)
	formatted3 := fmt.Sprintf("User: %s is %t (age %d)", name, active, age)
	formatted4 := fmt.Sprintf("Hex: %x, Binary: %b", 255, 10)

	fmt.Println(formatted1)
	fmt.Println(formatted2)
	fmt.Println(formatted3)
	fmt.Println(formatted4)

	// Padding and alignment
	fmt.Printf("Left  : '%-10s'|\n", name)   // Left-align, width 10
	fmt.Printf("Right : '%10s'|\n", name)    // Right-align
	fmt.Printf("ZeroPad: '%010d'|\n", age)   // Zero-padded number

	// 4. Regular Expressions
	header("4. Regular Expressions")

	email := "user@example.com"
	phone := "+1-123-456-7890"
	text := "Contact: john.doe@email.co or call +1 (555) 123-4567"

	// Email pattern
	emailRegex := regexp.MustCompile(`[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}`)
	emails := emailRegex.FindAllString(text, -1)
	fmt.Println("Emails found:", emails)

	// Phone pattern (simple)
	phoneRegex := regexp.MustCompile(`\+?\d[\d -]{8,}\d`)
	phones := phoneRegex.FindAllString(text, -1)
	fmt.Println("Phones found:", phones)

	// Replace
	cleaned := emailRegex.ReplaceAllString(text, "[EMAIL]")
	cleaned = phoneRegex.ReplaceAllString(cleaned, "[PHONE]")
	fmt.Println("Redacted:", cleaned)

	// Validate
	isValidEmail := emailRegex.MatchString(email)
	isValidPhone := regexp.MustCompile(`^\+?\d{1,3}[- ]?\d{3}[- ]?\d{3}[- ]?\d{4}$`).MatchString(phone)
	fmt.Println("Valid email?", isValidEmail)
	fmt.Println("Valid phone?", isValidPhone)

	// 5. String to Other Types (strconv)
	header("5. String to Other Types (strconv)")

	// String to int
	intStr := "42"
	num, err := strconv.Atoi(intStr)
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Printf("%s -> int: %d (type: %T)\n", intStr, num, num)
	}

	// String to int64
	num64, err := strconv.ParseInt("123456789012345", 10, 64)
	fmt.Printf("ParseInt: %d\n", num64)

	// String to float
	floatStr := "3.14159"
	f, err := strconv.ParseFloat(floatStr, 64)
	fmt.Printf("%s -> float64: %.5f\n", floatStr, f)

	// String to bool
	boolStr := "true"
	b, err := strconv.ParseBool(boolStr)
	fmt.Printf("%s -> bool: %t\n", boolStr, b)

	// Other types to string
	backToStr1 := strconv.Itoa(100)
	backToStr2 := strconv.FormatFloat(2.71828, 'f', 4, 64)
	backToStr3 := strconv.FormatBool(false)

	fmt.Println("int -> string:", backToStr1)
	fmt.Println("float -> string:", backToStr2)
	fmt.Println("bool -> string:", backToStr3)

	// Bonus: Quote and Unquote
	header("Bonus: Quote & Unquote")
	quoted := strconv.Quote("She said: \"Hello!\"")
	fmt.Println("Quoted:", quoted)
	unquoted, _ := strconv.Unquote(quoted)
	fmt.Println("Unquoted:", unquoted)
}
```
### 09. List, Slice, Map, Struct also project base usecase 
[Return Table of Contents](#Table-of-Contents)
- List - with all manipulation's methods (loop, append, delete, pop etc )
- Slice - with all manipulation's methods (loop, append, delete, pop etc )
- Map - with all manipulation (loop, insert key value, remove etc)
```go
func main() {
	students := make(map[string] string)
	students["name"] = "John"
	students["age"] = "20"
	students["GPA"] = "3.14"
	students["isStudent"] = "true"

	for key, value := range students {
		println(key, value)
	}
}
```
- Struct - with all manipulation (nested structs, loop, append, remove etc)
Project Use Cases (which collection/sequence data type where should use)

### 10. OOP Concept with Interface and struct
[Return Table of Contents](#Table-of-Contents)
- constructor:
```go
package main
import "fmt"
type Person struct {
	Name string
	Age  int
}

// Constructor function
func NewPerson(name string, age int) *Person {
	return &Person{Name: name, Age: age}
}

func main() {
	// Create a new instance using the constructor
	person := NewPerson("Alice", 30)
	fmt.Println(person)
}
```
- Method
```go
func (p *Person) Greet() {
	fmt.Printf("Hello, my name is %s and I am %d years old.\n", p.Name, p.Age)
}
func main() {
	person := NewPerson("Alice", 30)
	// Call the method
	person.Greet()
}
```
- Call Method in Another Method
```go
// Another method that calls Greet
func (p *Person) Introduce() {
	fmt.Println("Let me introduce myself:")
	p.Greet()
}

func main() {
	person := NewPerson("Alice", 30)
	// Call the method
	person.Introduce()
}
```
- Call Method in Another Method with Argument Pass
```go
// Method on Person struct with an argument
func (p *Person) GreetWithMessage(message string) {
	fmt.Printf("%s! My name is %s and I am %d years old.\n", message, p.Name, p.Age)
}
// Another method that calls GreetWithMessage
func (p *Person) IntroduceWithMessage(message string) {
	fmt.Println("Let me introduce myself:")
	p.GreetWithMessage(message)
}

func main() {
	person := NewPerson("Alice", 30)
	// Call the method
	person.IntroduceWithMessage("Greetings")
}
```
- Interfaces for Polymorphism
```go
package main
import "fmt"

// Define an interface
type Greeter interface {
	Greet()
}
// Define a struct
type Person struct {
	Name string
	Age  int
}
// Implement the Greet method from the Greeter interface
func (p *Person) Greet() {
	fmt.Printf("Hello, my name is %s and I am %d years old.\n", p.Name, p.Age)
}
// Another struct that implements the Greeter interface
type Robot struct {
	ID string
}
// Implement the Greet method from the Greeter interface
func (r *Robot) Greet() {
	fmt.Printf("Greetings, I am robot %s.\n", r.ID)
}
// Function that accepts a Greeter interface
func Introduce(g Greeter) {
	g.Greet()
}
func main() {
	// Create instances
	person := &Person{Name: "Alice", Age: 30}
	robot := &Robot{ID: "XJ-9"}
	// Use the Introduce function with different types
	Introduce(person)
	Introduce(robot)
}
```
- Composition over Inheritance:
```go
type Engine struct {
	Power int
}
type Car struct {
	Engine
	Brand string
}
func main() {
	car := Car{
		Engine: Engine{Power: 150},
		Brand:  "Toyota",
	}
	fmt.Printf("Car brand: %s, Engine power: %d\n", car.Brand, car.Power)
}
```
- Inheritance
```go
type Animal struct {
	Name string
}

type Dog struct {
	Animal
	Breed string
}

func (a Animal) Speak() {
	fmt.Println("Animal speaks")
}

func (d Dog) Speak() {
	fmt.Printf("%s barks\n", d.Name)
}

func main() {
	dog := Dog{
		Animal: Animal{Name: "Buddy"},
		Breed:  "Golden Retriever",
	}
	dog.Speak()
}
```
- Pointer vs. value receivers.
```go
type Counter struct {
	Value int
}
// Method with a pointer receiver (modifies the struct)
func (c *Counter) Increment() {
	c.Value++
}
// Method with a value receiver (does not modify the struct)
func (c Counter) Display() {
	fmt.Printf("Counter Value: %d\n", c.Value)
}
func main() {
	counter := Counter{Value: 0}
	counter.Increment()
	counter.Display()
}
```
- Design Patterns:
- - Factory, Singleton, and Adapter in Go.

### 11. Threading or goroutine, Basic Request, Data Scrapping etc
[Return Table of Contents](#Table-of-Contents)
- Threading:
- - Using goroutines for lightweight concurrency.
- - Synchronization with sync.WaitGroup and sync.Mutex.
- Requests:
- - Basic HTTP requests with net/http.
- - Parsing responses (io.Reader).
- Data Scraping:
- - Using colly for web scraping.
- - HTML parsing with goquery.

### 12. Error Handling
[Return Table of Contents](#Table-of-Contents)
- Simple error handeling
```go
package main
import (
	"fmt"
	"strconv"
)
func convert_number(n string) (int64, error){        // int8, 16, 32, 64
	result, err := strconv.ParseInt(n, 10, 64)   // 8, 16, 32, 64
    if err != nil{
		return 0, err
	}else{
		return result, nil      // return int8(result), int16(result), int32(result),  int32(result)/result, 
	}
}

func main(){
	// x,_ := convert_number("10a")
	// fmt.Println(x)
	x, err := convert_number("10a")
	if err != nil{
		fmt.Println(err)
	}else{
		fmt.Println(x)
	}
}
```
- Custom Dynamic Error Handle - to integer convert
```go
package main

import (
	"fmt"
	"strconv"
)
func to_integer(n interface{}) (int32, error){

	switch v := n.(type){
	case int:
		return int32(v), nil
	
	case float32:
		return int32(v), nil
	case string:
		result, err := strconv.ParseInt(v, 10, 32)
		if err != nil{
			return 0, err
		}else{
			return int32(result), nil
		}
	default:
		return 0, fmt.Errorf("Unsupported assigned")

	}
}

func main(){
	// x,_ := to_integer("10a")
	// fmt.Println(x)
	x, err := to_integer("10a")
	if err != nil{
		fmt.Println(err)
	}else{
		fmt.Println(x)
	}
}

```
- Dynamic error handle - to string convert
```go
package main
import (
	"fmt"
	"strconv"
)
func to_string(number interface{}) (string, error) {
	switch value := number.(type) {
	case int8, int16, int32, int64, int:
		// return fmt.Sprintf("%d", value), nil
		return strconv.FormatInt(int64(value.(int)), 10), nil
	case float32, float64:
		return strconv.FormatFloat(value.(float64), 'f', 2, 64), nil
	case string:
		return value, nil
	default:
		return "", fmt.Errorf("unsupported type: %t", value)
	}
}
func main() {
	x, err := to_string(42)
	if err != nil {
		fmt.Println(err)
	}else{
		fmt.Println(x)
	}
}
```
- Dynamic error handle - to Float convert
```go
package main
import (
	"fmt"
	"strconv"
)
func to_float(number interface{}) (float64, error) {
	switch value := number.(type) {
	case int8, int16, int32, int64, int:
		return float64(value.(int)), nil
	case string:
		result, err := strconv.ParseFloat(value, 64)
		if err != nil{
			return 0, err
		}else{
			return result, nil
		}
	case float32, float64:
		return value.(float64), nil
	default:
		return 0, fmt.Errorf("unsupported value %t", value)
	}
}

func main() {

	x, err := to_float(102)
	if err != nil{
		fmt.Println(err)
	}else{
		fmt.Printf("%.1f", x)
	}

}
```

### 13. File Handling and IO
[Return Table of Contents](#Table-of-Contents)
- File Operations:
- - Reading and writing files (os and io/ioutil).
- Buffering:
- - Using bufio for efficient IO.
- File Manipulation:
- - Checking existence, renaming, and deleting files.
  
## 14. Others
[Return Table of Contents](#Table-of-Contents)
- Time
```go
package main
import ("time"
	"fmt")
func main(){

	fmt.Println(time.Now().Local().Format("01-02-2006 Monday 15:04:05"))
	// Printf is a function that can format string and return value and error status
	currentTime, err := fmt.Printf(time.Now().Format("01-02-2006 Monday 15:04:05"))
	if err != nil{
		fmt.Println(currentTime, err)
	}
	// 
	formatTime := fmt.Sprintf(time.Now().Format("01-02-2006 Monday 15:04:05"))
	fmt.Println(formatTime)
}
```
