# Complete Go (Golang) Guide — From Zero to Platform Engineering

> You have ZERO Go experience. This guide teaches Go from scratch,
> focused on what you need for the REA Platform Engineer coding round.
> Go through this top-to-bottom. Each section builds on the previous.

---

## TABLE OF CONTENTS

1. [Why Go for Platform Engineering](#1-why-go)
2. [Installation & Setup](#2-installation)
3. [Your First Program](#3-first-program)
4. [Variables, Types & Constants](#4-variables)
5. [Control Flow: if, for, switch](#5-control-flow)
6. [Functions](#6-functions)
7. [Strings & String Manipulation](#7-strings)
8. [Arrays, Slices & Maps](#8-collections)
9. [Structs & Methods](#9-structs)
10. [Pointers](#10-pointers)
11. [Interfaces](#11-interfaces)
12. [Error Handling](#12-errors)
13. [Goroutines & Channels (Concurrency)](#13-concurrency)
14. [Packages & Modules](#14-packages)
15. [Working with JSON](#15-json)
16. [Working with YAML](#16-yaml)
17. [File I/O](#17-file-io)
18. [HTTP Servers & Clients](#18-http)
19. [CLI Tools with os/flag](#19-cli)
20. [Testing in Go](#20-testing)
21. [Go for Kubernetes & Platform Tooling](#21-k8s-tooling)
22. [Common Interview Patterns](#22-interview-patterns)
23. [Quick Reference Card](#23-reference)

---

## 1. WHY GO FOR PLATFORM ENGINEERING <a name="1-why-go"></a>

- **Static binaries**: `go build` → single binary, no dependencies, easy to deploy
- **Built-in concurrency**: goroutines + channels (lightweight threads)
- **Fast compilation**: Compile in seconds
- **Kubernetes is written in Go**: kubectl, Helm, Terraform, Docker — all Go
- **Strong standard library**: HTTP server, JSON, crypto, testing — all built-in
- **Cross-compilation**: Build for Linux from Mac/Windows: `GOOS=linux go build`

**At REA**: Platform tools, CLI utilities, Kubernetes operators, internal APIs — likely all in Go.

---

## 2. INSTALLATION & SETUP <a name="2-installation"></a>

### Install Go

```bash
# Windows (use installer):
# Download from https://go.dev/dl/ → go1.22.x.windows-amd64.msi
# Or with winget:
winget install GoLang.Go

# Linux:
wget https://go.dev/dl/go1.22.4.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.22.4.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc

# Verify:
go version
# go version go1.22.4 linux/amd64
```

### Create Your First Project

```bash
mkdir ~/go-learn && cd ~/go-learn
go mod init go-learn
# This creates go.mod — Go's version of package.json / requirements.txt
```

### VS Code Setup
- Install "Go" extension by Go Team at Google
- It will prompt to install `gopls` (Go language server) — say Yes
- Auto-formatting, auto-imports, error highlighting all work automatically

---

## 3. YOUR FIRST PROGRAM <a name="3-first-program"></a>

```go
// main.go
package main    // Every Go program starts with a package declaration
                // "main" is special — it's the entry point

import "fmt"    // "fmt" = format — for printing

func main() {  // main() function — program starts here
    fmt.Println("Hello, REA Platform Team!")
}
```

```bash
# Run directly (compile + run):
go run main.go

# Or build a binary:
go build -o myapp main.go
./myapp
```

### Key Differences from Python

| Python | Go |
|---|---|
| Interpreted | Compiled (to native binary) |
| Dynamic typing | Static typing (every variable has a fixed type) |
| Indentation matters | Braces `{}` for blocks |
| `def` for functions | `func` for functions |
| No semicolons | No semicolons (auto-inserted) |
| `print()` | `fmt.Println()` |
| Exceptions (try/except) | Error return values |
| Classes | Structs + methods |
| `import module` | `import "package"` |

---

## 4. VARIABLES, TYPES & CONSTANTS <a name="4-variables"></a>

### Declaring Variables

```go
package main

import "fmt"

func main() {
    // Method 1: var keyword (explicit type)
    var name string = "Vaibhav"
    var age int = 26
    var isEngineer bool = true

    // Method 2: Type inference (Go figures out the type)
    var city = "Gurugram"    // Go knows this is string

    // Method 3: Short declaration (:=) — MOST COMMON
    // Only works inside functions
    company := "REA Group"   // Go infers type as string
    experience := 3          // Go infers type as int
    salary := 25.5           // Go infers type as float64

    fmt.Println(name, age, isEngineer, city, company, experience, salary)

    // Zero values (Go initializes variables to zero values, never garbage)
    var x int       // 0
    var s string    // "" (empty string)
    var b bool      // false
    var f float64   // 0.0
    fmt.Println(x, s, b, f)
}
```

### Basic Types

```go
// Integers
var i int       = 42        // Platform-dependent size (32 or 64 bit)
var i8 int8     = 127       // -128 to 127
var i16 int16   = 32767
var i32 int32   = 2147483647
var i64 int64   = 9223372036854775807

// Unsigned integers (no negatives)
var u uint      = 42
var u8 uint8    = 255       // Same as byte
var u32 uint32  = 4294967295

// Floating point
var f32 float32 = 3.14
var f64 float64 = 3.14159265358979  // Default for decimals

// String
var s string = "Hello"
// Strings are IMMUTABLE in Go (like Python)
// Raw strings (no escaping): `C:\path\to\file`

// Boolean
var b bool = true    // true or false

// Byte (alias for uint8) — for raw data
var by byte = 'A'    // Single character
```

### Constants

```go
const Pi = 3.14159
const AppName = "PropertySearch"
const MaxRetries = 3

// Multiple constants
const (
    StatusOK    = 200
    StatusNotFound = 404
    StatusError = 500
)

// iota — auto-incrementing constants (like enum)
const (
    Sunday    = iota  // 0
    Monday            // 1
    Tuesday           // 2
    Wednesday         // 3
    Thursday          // 4
    Friday            // 5
    Saturday          // 6
)
```

### Type Conversion (Go has NO implicit conversion)

```go
// In Python: x = 42; y = float(x)  → works
// In Go: you MUST explicitly convert

var i int = 42
var f float64 = float64(i)    // int → float64
var s string = fmt.Sprintf("%d", i)  // int → string
var i2 int = int(f)           // float64 → int (truncates decimal)

// String ↔ Number conversions
import "strconv"
s := strconv.Itoa(42)              // int → string: "42"
n, err := strconv.Atoi("42")       // string → int: 42, nil
f, err := strconv.ParseFloat("3.14", 64)  // string → float64
```

---

## 5. CONTROL FLOW <a name="5-control-flow"></a>

### if / else

```go
// Basic
age := 25
if age >= 18 {
    fmt.Println("Adult")
} else {
    fmt.Println("Minor")
}

// if with initialization (very common in Go)
if err := doSomething(); err != nil {
    fmt.Println("Error:", err)
    return
}

// Multiple conditions
if score >= 90 {
    fmt.Println("A")
} else if score >= 80 {
    fmt.Println("B")
} else {
    fmt.Println("C")
}
```

### for (Go has ONLY `for`, no while/do-while)

```go
// Classic for loop (like C)
for i := 0; i < 10; i++ {
    fmt.Println(i)
}

// While-style (condition only)
count := 0
for count < 10 {
    fmt.Println(count)
    count++
}

// Infinite loop
for {
    fmt.Println("Running forever")
    break    // Use break to exit
}

// Range over slice (like Python's for x in list)
fruits := []string{"apple", "banana", "cherry"}
for index, value := range fruits {
    fmt.Printf("Index %d: %s\n", index, value)
}
// Skip index:
for _, fruit := range fruits {
    fmt.Println(fruit)
}

// Range over map
ages := map[string]int{"Alice": 30, "Bob": 25}
for name, age := range ages {
    fmt.Printf("%s is %d\n", name, age)
}

// Range over string (gives runes/characters)
for i, ch := range "Hello" {
    fmt.Printf("Position %d: %c\n", i, ch)
}
```

### switch

```go
// Basic switch (no need for break — Go auto-breaks)
day := "Monday"
switch day {
case "Monday":
    fmt.Println("Start of work week")
case "Friday":
    fmt.Println("TGIF!")
case "Saturday", "Sunday":   // Multiple values
    fmt.Println("Weekend!")
default:
    fmt.Println("Midweek")
}

// Switch with conditions (like if-else chain)
score := 85
switch {
case score >= 90:
    fmt.Println("A")
case score >= 80:
    fmt.Println("B")
default:
    fmt.Println("C")
}

// Type switch (check interface type)
var x interface{} = "hello"
switch v := x.(type) {
case string:
    fmt.Println("String:", v)
case int:
    fmt.Println("Int:", v)
default:
    fmt.Println("Unknown type")
}
```

---

## 6. FUNCTIONS <a name="6-functions"></a>

```go
// Basic function
func greet(name string) string {
    return "Hello, " + name
}

// Multiple parameters of same type
func add(a, b int) int {
    return a + b
}

// Multiple return values (VERY COMMON in Go)
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("cannot divide by zero")
    }
    return a / b, nil
}

// Usage:
result, err := divide(10, 3)
if err != nil {
    fmt.Println("Error:", err)
    return
}
fmt.Println("Result:", result)

// Named return values
func getUser() (name string, age int) {
    name = "Vaibhav"
    age = 26
    return    // Returns name and age automatically
}

// Variadic function (like Python's *args)
func sum(numbers ...int) int {
    total := 0
    for _, n := range numbers {
        total += n
    }
    return total
}
// Usage: sum(1, 2, 3, 4, 5) → 15

// Functions as values (like Python lambdas)
multiply := func(a, b int) int {
    return a * b
}
fmt.Println(multiply(3, 4))  // 12

// Closures
func counter() func() int {
    count := 0
    return func() int {
        count++
        return count
    }
}
c := counter()
fmt.Println(c())  // 1
fmt.Println(c())  // 2
fmt.Println(c())  // 3

// defer — runs when function exits (like Python's finally)
func readFile(path string) {
    file, err := os.Open(path)
    if err != nil {
        return
    }
    defer file.Close()    // This runs when readFile() returns
    // ... use file ...
    // file.Close() happens automatically at the end
}
```

---

## 7. STRINGS <a name="7-strings"></a>

```go
import (
    "fmt"
    "strings"
    "strconv"
)

// String basics
s := "Hello, World"
fmt.Println(len(s))           // 12 (byte length)
fmt.Println(s[0])             // 72 (byte value of 'H')
fmt.Println(string(s[0]))    // "H"

// Concatenation
greeting := "Hello" + " " + "World"

// String formatting (like Python f-strings)
name := "Vaibhav"
age := 26
msg := fmt.Sprintf("Name: %s, Age: %d", name, age)
// %s = string, %d = integer, %f = float, %v = any, %T = type
// %+v = struct with field names, %#v = Go syntax

// strings package (like Python string methods)
strings.Contains("Hello World", "World")      // true
strings.HasPrefix("Hello", "He")              // true
strings.HasSuffix("Hello", "lo")              // true
strings.ToUpper("hello")                      // "HELLO"
strings.ToLower("HELLO")                      // "hello"
strings.TrimSpace("  hello  ")                // "hello"
strings.Trim("##hello##", "#")                // "hello"
strings.Replace("foo bar foo", "foo", "baz", -1)  // "baz bar baz"
strings.Split("a,b,c", ",")                   // ["a", "b", "c"]
strings.Join([]string{"a", "b", "c"}, "-")    // "a-b-c"
strings.Count("hello", "l")                   // 2
strings.Index("hello", "ll")                  // 2 (-1 if not found)
strings.Repeat("ha", 3)                       // "hahaha"

// Multi-line strings (raw strings)
query := `
    SELECT * FROM properties
    WHERE city = 'Melbourne'
    ORDER BY price DESC
`

// String builder (efficient concatenation — like Python's io.StringIO)
var builder strings.Builder
for i := 0; i < 100; i++ {
    builder.WriteString(fmt.Sprintf("Line %d\n", i))
}
result := builder.String()

// String ↔ Number
strconv.Itoa(42)              // int → string: "42"
strconv.Atoi("42")            // string → int: 42, error
strconv.FormatFloat(3.14, 'f', 2, 64)  // float → string: "3.14"
strconv.ParseFloat("3.14", 64)  // string → float: 3.14, error
strconv.FormatBool(true)      // bool → string: "true"
strconv.ParseBool("true")    // string → bool: true, error
```

---

## 8. ARRAYS, SLICES & MAPS <a name="8-collections"></a>

### Arrays (Fixed size — rarely used directly)

```go
// Arrays have FIXED size (set at compile time)
var arr [5]int = [5]int{1, 2, 3, 4, 5}
arr2 := [3]string{"a", "b", "c"}
fmt.Println(arr[0])    // 1
fmt.Println(len(arr))  // 5
```

### Slices (Dynamic arrays — THIS IS WHAT YOU USE)

```go
// Slices are like Python lists — dynamic size
// This is what you'll use 99% of the time

// Create a slice
nums := []int{1, 2, 3, 4, 5}     // Literal
names := []string{"Alice", "Bob"} // Literal
empty := []int{}                   // Empty slice
var nilSlice []int                 // nil slice (different from empty!)

// make() — create with initial size/capacity
data := make([]int, 5)        // Length=5, all zeros: [0,0,0,0,0]
data2 := make([]int, 0, 10)   // Length=0, capacity=10

// Append (like Python's list.append)
nums = append(nums, 6)           // [1,2,3,4,5,6]
nums = append(nums, 7, 8, 9)     // Append multiple
nums = append(nums, []int{10, 11}...)  // Append another slice (... = spread)

// Slicing (like Python slicing)
sub := nums[1:4]    // Elements at index 1,2,3 (not 4)
first3 := nums[:3]  // First 3 elements
last3 := nums[len(nums)-3:]  // Last 3 elements

// Length and capacity
fmt.Println(len(nums))  // Number of elements
fmt.Println(cap(nums))  // Capacity (allocated)

// Iterate
for i, v := range nums {
    fmt.Printf("Index %d: Value %d\n", i, v)
}

// Check if slice is empty
if len(nums) == 0 {
    fmt.Println("Empty!")
}

// Delete element at index i
i := 2
nums = append(nums[:i], nums[i+1:]...)

// Sort
import "sort"
sort.Ints(nums)                // Sort in-place
sort.Strings(names)            // Sort strings
sort.Slice(nums, func(i, j int) bool {
    return nums[i] > nums[j]   // Custom: descending
})

// Contains (no built-in — write a helper)
func contains(slice []string, item string) bool {
    for _, v := range slice {
        if v == item {
            return true
        }
    }
    return false
}
// Go 1.21+: use slices.Contains()
import "slices"
slices.Contains(names, "Alice")  // true
```

### Maps (like Python dict)

```go
// Create a map
ages := map[string]int{
    "Alice": 30,
    "Bob":   25,
    "Carol": 35,
}

// Empty map
m := make(map[string]int)    // Empty map (ready to use)
var m2 map[string]int        // nil map (will panic if you write to it!)

// Access
age := ages["Alice"]          // 30
// If key doesn't exist → returns zero value (0 for int, "" for string)

// Check if key exists (VERY IMPORTANT pattern)
age, exists := ages["Dave"]
if !exists {
    fmt.Println("Dave not found")
}
// Shorthand:
if age, ok := ages["Dave"]; ok {
    fmt.Println("Dave's age:", age)
} else {
    fmt.Println("Not found")
}

// Add / Update
ages["Dave"] = 28      // Add new key
ages["Alice"] = 31     // Update existing

// Delete
delete(ages, "Bob")

// Iterate
for name, age := range ages {
    fmt.Printf("%s: %d\n", name, age)
}
// Note: Map iteration order is RANDOM (unlike Python 3.7+ dicts)

// Length
fmt.Println(len(ages))

// Nested maps
users := map[string]map[string]string{
    "user1": {
        "name":  "Vaibhav",
        "email": "vaibhav@example.com",
    },
}

// Map of slices
teams := map[string][]string{
    "platform": {"Alice", "Bob"},
    "backend":  {"Carol", "Dave"},
}
teams["platform"] = append(teams["platform"], "Eve")
```

---

## 9. STRUCTS & METHODS <a name="9-structs"></a>

Go doesn't have classes. It has **structs** (data) + **methods** (functions on structs).

```go
// Define a struct (like a Python class with __init__)
type Service struct {
    Name      string
    Namespace string
    Port      int
    Replicas  int
    Healthy   bool
}

// Create instances
svc := Service{
    Name:      "property-api",
    Namespace: "production",
    Port:      8080,
    Replicas:  3,
    Healthy:   true,
}

// Access fields
fmt.Println(svc.Name)       // "property-api"
svc.Replicas = 5            // Modify

// Create with positional values (not recommended — fragile)
svc2 := Service{"search", "prod", 9090, 2, true}

// Zero-value struct
var svc3 Service             // All fields are zero values
svc3.Name = "auth-service"

// Pointer to struct
svcPtr := &Service{Name: "cache"}
fmt.Println(svcPtr.Name)    // Go auto-dereferences — no need for (*svcPtr).Name

// Methods on structs
func (s Service) Info() string {
    return fmt.Sprintf("%s/%s (port %d, replicas %d)", s.Namespace, s.Name, s.Port, s.Replicas)
}
fmt.Println(svc.Info())  // "production/property-api (port 8080, replicas 3)"

// Pointer receiver (can modify the struct)
func (s *Service) Scale(replicas int) {
    s.Replicas = replicas   // This actually modifies the original struct
}
svc.Scale(10)
fmt.Println(svc.Replicas)   // 10

// Value receiver vs Pointer receiver:
// func (s Service) Method()  → gets a COPY, can't modify original
// func (s *Service) Method() → gets a POINTER, can modify original
// Rule of thumb: Use pointer receivers for methods that modify the struct
//                Use value receivers for read-only methods

// Struct embedding (like inheritance but better — composition)
type Container struct {
    Image   string
    CPU     string
    Memory  string
}

type Pod struct {
    Name       string
    Namespace  string
    Container          // Embedded! Pod "inherits" Container fields
}

pod := Pod{
    Name:      "web-pod",
    Namespace: "default",
    Container: Container{
        Image:  "nginx:1.25",
        CPU:    "100m",
        Memory: "128Mi",
    },
}
fmt.Println(pod.Image)     // "nginx:1.25" — accessed directly!
fmt.Println(pod.Container.Image)  // Also works

// Struct tags (metadata — used by JSON, YAML, DB libraries)
type Property struct {
    ID          int    `json:"id" yaml:"id"`
    Title       string `json:"title" yaml:"title"`
    Price       int    `json:"price" yaml:"price"`
    IsAvailable bool   `json:"is_available" yaml:"is_available"`
}
```

---

## 10. POINTERS <a name="10-pointers"></a>

Pointers are memory addresses. Go has pointers but NO pointer arithmetic (safe).

```go
// & = "address of"
// * = "value at address" (dereference)

x := 42
p := &x          // p is a pointer to x (type: *int)
fmt.Println(p)   // 0xc000018090 (memory address)
fmt.Println(*p)  // 42 (value at that address)

*p = 100         // Modify through pointer
fmt.Println(x)   // 100 (x changed!)

// Why pointers matter:
// In Go, everything is passed BY VALUE (copies)
// Without pointers, functions get copies and can't modify originals

func doubleWrong(x int) {
    x = x * 2    // Modifies the COPY, original unchanged
}

func doubleRight(x *int) {
    *x = *x * 2  // Modifies the ORIGINAL through pointer
}

val := 10
doubleWrong(val)
fmt.Println(val)     // Still 10

doubleRight(&val)
fmt.Println(val)     // Now 20

// nil pointer (like Python's None)
var p2 *int          // nil pointer
if p2 == nil {
    fmt.Println("Pointer is nil")
}
// *p2 would PANIC (nil pointer dereference) — always check!

// new() — allocates memory and returns a pointer
p3 := new(int)       // *int pointing to a new int (value = 0)
*p3 = 42
```

**When to use pointers:**
- When you need a function to modify its argument
- When passing large structs (avoid copying)
- When a value might be absent (nil pointer vs zero value)

---

## 11. INTERFACES <a name="11-interfaces"></a>

Interfaces define behavior. If a type has the right methods, it implements the interface **automatically** (no `implements` keyword needed — this is called "duck typing").

```go
// Define an interface
type HealthChecker interface {
    CheckHealth() (bool, error)
}

// Any struct with a CheckHealth() method satisfies this interface

type HTTPService struct {
    URL string
}

func (h HTTPService) CheckHealth() (bool, error) {
    resp, err := http.Get(h.URL + "/healthz")
    if err != nil {
        return false, err
    }
    defer resp.Body.Close()
    return resp.StatusCode == 200, nil
}

type TCPService struct {
    Host string
    Port int
}

func (t TCPService) CheckHealth() (bool, error) {
    conn, err := net.DialTimeout("tcp", fmt.Sprintf("%s:%d", t.Host, t.Port), 5*time.Second)
    if err != nil {
        return false, err
    }
    conn.Close()
    return true, nil
}

// Both implement HealthChecker — can be used interchangeably
func checkAll(services []HealthChecker) {
    for _, svc := range services {
        healthy, err := svc.CheckHealth()
        fmt.Printf("Healthy: %v, Error: %v\n", healthy, err)
    }
}

services := []HealthChecker{
    HTTPService{URL: "http://api.rea.com"},
    TCPService{Host: "db.rea.com", Port: 5432},
}
checkAll(services)

// Common standard library interfaces:

// fmt.Stringer — like Python's __str__
type Stringer interface {
    String() string
}
// If your struct has String(), fmt.Println uses it automatically
func (s Service) String() string {
    return fmt.Sprintf("%s (%d replicas)", s.Name, s.Replicas)
}

// error interface — Go's error type IS an interface
type error interface {
    Error() string
}

// io.Reader / io.Writer — the most important Go interfaces
type Reader interface {
    Read(p []byte) (n int, err error)
}
type Writer interface {
    Write(p []byte) (n int, err error)
}
// Files, HTTP bodies, buffers, network connections — all implement these

// Empty interface (interface{} or 'any') — accepts ANY type
func printAnything(v interface{}) {
    fmt.Println(v)
}
// Go 1.18+: use 'any' instead of 'interface{}'
func printAnything2(v any) {
    fmt.Println(v)
}

// Type assertion — get concrete type from interface
var i interface{} = "hello"
s, ok := i.(string)
if ok {
    fmt.Println("String:", s)
}
```

---

## 12. ERROR HANDLING <a name="12-errors"></a>

Go doesn't have exceptions (no try/catch). Errors are VALUES returned from functions.

```go
import (
    "errors"
    "fmt"
    "os"
)

// Functions return error as the LAST return value
func readConfig(path string) ([]byte, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, fmt.Errorf("failed to read config %s: %w", path, err)
        // %w wraps the original error (allows errors.Is/As later)
    }
    return data, nil
}

// Calling code MUST check the error
data, err := readConfig("/etc/app/config.yaml")
if err != nil {
    fmt.Println("Error:", err)
    os.Exit(1)
}
// If err is nil, data is safe to use

// THE GOLDEN RULE OF GO ERROR HANDLING:
// if err != nil {
//     return ..., err
// }
// You'll write this HUNDREDS of times. It's Go's style.

// Creating errors
err1 := errors.New("something went wrong")
err2 := fmt.Errorf("connection to %s failed: %w", "database", err1)

// Custom error types
type NotFoundError struct {
    Resource string
    ID       string
}

func (e *NotFoundError) Error() string {
    return fmt.Sprintf("%s with ID %s not found", e.Resource, e.ID)
}

// Return custom error
func findService(id string) (*Service, error) {
    // ... search ...
    return nil, &NotFoundError{Resource: "Service", ID: id}
}

// Check error type
_, err := findService("abc123")
var notFound *NotFoundError
if errors.As(err, &notFound) {
    fmt.Println("Not found:", notFound.Resource, notFound.ID)
}

// Check specific error
if errors.Is(err, os.ErrNotExist) {
    fmt.Println("File does not exist")
}

// panic & recover (like exceptions — only for truly unrecoverable errors)
// DON'T use panic for normal error handling
func mustParseConfig(path string) Config {
    data, err := os.ReadFile(path)
    if err != nil {
        panic("critical: config file missing: " + err.Error())
    }
    // ...
}

// recover — catch a panic (like try/except)
func safeExecute(f func()) {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered from panic:", r)
        }
    }()
    f()
}
```

---

## 13. GOROUTINES & CHANNELS (CONCURRENCY) <a name="13-concurrency"></a>

This is Go's superpower and a likely interview topic.

### Goroutines (lightweight threads)

```go
import (
    "fmt"
    "sync"
    "time"
)

// A goroutine is a function running concurrently
// Start one with the 'go' keyword

func checkHealth(url string) {
    fmt.Printf("Checking %s...\n", url)
    time.Sleep(2 * time.Second)  // Simulate network call
    fmt.Printf("%s is healthy\n", url)
}

func main() {
    // Sequential (slow — 6 seconds)
    checkHealth("http://api1.com")
    checkHealth("http://api2.com")
    checkHealth("http://api3.com")

    // Concurrent (fast — 2 seconds!)
    go checkHealth("http://api1.com")
    go checkHealth("http://api2.com")
    go checkHealth("http://api3.com")

    time.Sleep(3 * time.Second)  // Wait for goroutines (BAD way — see WaitGroup below)
}
```

### WaitGroup (wait for goroutines to finish)

```go
func main() {
    var wg sync.WaitGroup
    urls := []string{
        "http://api1.rea.com/healthz",
        "http://api2.rea.com/healthz",
        "http://api3.rea.com/healthz",
    }

    for _, url := range urls {
        wg.Add(1)                    // Increment counter
        go func(u string) {         // Start goroutine
            defer wg.Done()          // Decrement counter when done
            checkHealth(u)
        }(url)                       // Pass url as argument (avoid closure trap)
    }

    wg.Wait()                        // Block until all goroutines finish
    fmt.Println("All health checks complete!")
}
```

### Channels (communicate between goroutines)

```go
// Channels are pipes for sending data between goroutines

// Create a channel
ch := make(chan string)         // Unbuffered channel of strings
bch := make(chan int, 10)       // Buffered channel (capacity 10)

// Send and receive
go func() {
    ch <- "Hello"    // Send to channel (blocks until someone receives)
}()
msg := <-ch          // Receive from channel (blocks until someone sends)
fmt.Println(msg)     // "Hello"

// PRACTICAL EXAMPLE: Concurrent health checker with results
type HealthResult struct {
    URL     string
    Healthy bool
    Error   error
}

func checkHealthAsync(url string, results chan<- HealthResult) {
    resp, err := http.Get(url)
    if err != nil {
        results <- HealthResult{URL: url, Healthy: false, Error: err}
        return
    }
    defer resp.Body.Close()
    results <- HealthResult{URL: url, Healthy: resp.StatusCode == 200, Error: nil}
}

func main() {
    urls := []string{
        "http://api.rea.com/healthz",
        "http://search.rea.com/healthz",
        "http://auth.rea.com/healthz",
    }

    results := make(chan HealthResult, len(urls))

    for _, url := range urls {
        go checkHealthAsync(url, results)
    }

    // Collect results
    for i := 0; i < len(urls); i++ {
        r := <-results
        if r.Healthy {
            fmt.Printf("✓ %s is healthy\n", r.URL)
        } else {
            fmt.Printf("✗ %s is DOWN: %v\n", r.URL, r.Error)
        }
    }
}

// select — wait on multiple channels (like a switch for channels)
func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)

    go func() {
        time.Sleep(1 * time.Second)
        ch1 <- "result from service 1"
    }()
    go func() {
        time.Sleep(2 * time.Second)
        ch2 <- "result from service 2"
    }()

    // Wait for whichever finishes first
    select {
    case msg := <-ch1:
        fmt.Println("Got:", msg)
    case msg := <-ch2:
        fmt.Println("Got:", msg)
    case <-time.After(3 * time.Second):
        fmt.Println("Timeout!")
    }
}

// context — cancel goroutines (essential for platform tools)
import "context"

func fetchWithTimeout(url string) (string, error) {
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    req, _ := http.NewRequestWithContext(ctx, "GET", url, nil)
    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return "", err  // Returns error if timeout exceeded
    }
    defer resp.Body.Close()

    body, _ := io.ReadAll(resp.Body)
    return string(body), nil
}
```

### Mutex (protect shared data)

```go
import "sync"

type SafeCounter struct {
    mu    sync.Mutex
    count map[string]int
}

func (c *SafeCounter) Inc(key string) {
    c.mu.Lock()           // Lock before writing
    defer c.mu.Unlock()   // Unlock when done
    c.count[key]++
}

func (c *SafeCounter) Get(key string) int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count[key]
}
```

---

## 14. PACKAGES & MODULES <a name="14-packages"></a>

```bash
# Project structure:
myproject/
├── go.mod           # Module definition (like package.json)
├── go.sum           # Lock file (like package-lock.json)
├── main.go          # Entry point
├── internal/        # Private packages (can't be imported outside)
│   └── health/
│       └── checker.go
├── pkg/             # Public packages (can be imported by others)
│   └── config/
│       └── config.go
└── cmd/             # Multiple entry points
    ├── server/
    │   └── main.go
    └── cli/
        └── main.go
```

```go
// go.mod
module github.com/vaibhavkumar779/platform-tool

go 1.22

require (
    gopkg.in/yaml.v3 v3.0.1
    k8s.io/client-go v0.29.0
)
```

```bash
# Module commands
go mod init github.com/vaibhavkumar779/platform-tool  # Initialize module
go mod tidy                        # Add missing / remove unused dependencies
go get gopkg.in/yaml.v3            # Add a dependency
go get -u ./...                    # Update all dependencies
go mod download                    # Download dependencies
go mod vendor                      # Copy dependencies to vendor/
```

```go
// internal/health/checker.go
package health    // Package name (not file name!)

import "net/http"

// Exported function (starts with UPPERCASE)
func CheckURL(url string) (bool, error) {
    resp, err := http.Get(url)
    if err != nil {
        return false, err
    }
    defer resp.Body.Close()
    return resp.StatusCode == 200, nil
}

// unexported function (starts with lowercase — private to package)
func formatResult(url string, ok bool) string {
    // ...
}
```

```go
// main.go — import your own package
package main

import (
    "fmt"
    "github.com/vaibhavkumar779/platform-tool/internal/health"
)

func main() {
    ok, err := health.CheckURL("http://api.rea.com/healthz")
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    fmt.Println("Healthy:", ok)
}
```

**Go visibility rules:**
- `Uppercase` name → Exported (public) — accessible from other packages
- `lowercase` name → unexported (private) — only accessible within the same package
- This applies to functions, types, struct fields, constants, variables

---

## 15. WORKING WITH JSON <a name="15-json"></a>

```go
import (
    "encoding/json"
    "fmt"
    "os"
)

// Struct with JSON tags
type Property struct {
    ID       int      `json:"id"`
    Title    string   `json:"title"`
    Price    int      `json:"price"`
    City     string   `json:"city"`
    Features []string `json:"features"`
    Active   bool     `json:"active,omitempty"`  // omitempty: skip if false/zero
}

// =================== Marshal (Go struct → JSON) ===================
prop := Property{
    ID:       1,
    Title:    "Modern Apartment",
    Price:    750000,
    City:     "Melbourne",
    Features: []string{"pool", "garage", "garden"},
    Active:   true,
}

// Compact JSON
jsonBytes, err := json.Marshal(prop)
fmt.Println(string(jsonBytes))
// {"id":1,"title":"Modern Apartment","price":750000,"city":"Melbourne","features":["pool","garage","garden"],"active":true}

// Pretty JSON
jsonBytes, err := json.MarshalIndent(prop, "", "  ")
fmt.Println(string(jsonBytes))

// =================== Unmarshal (JSON → Go struct) ===================
jsonStr := `{"id": 2, "title": "Beach House", "price": 1200000, "city": "Sydney"}`

var prop2 Property
err := json.Unmarshal([]byte(jsonStr), &prop2)
if err != nil {
    fmt.Println("Error:", err)
    return
}
fmt.Println(prop2.Title)   // "Beach House"
fmt.Println(prop2.Price)   // 1200000

// =================== JSON from file ===================
file, _ := os.Open("properties.json")
defer file.Close()

var properties []Property
decoder := json.NewDecoder(file)
err := decoder.Decode(&properties)

// =================== JSON to file ===================
file, _ := os.Create("output.json")
defer file.Close()

encoder := json.NewEncoder(file)
encoder.SetIndent("", "  ")
encoder.Encode(properties)

// =================== Dynamic JSON (unknown structure) ===================
jsonStr := `{"name": "test", "count": 42, "tags": ["a", "b"]}`

var data map[string]interface{}
json.Unmarshal([]byte(jsonStr), &data)

name := data["name"].(string)   // Type assertion
count := data["count"].(float64) // JSON numbers are float64!
tags := data["tags"].([]interface{})
```

---

## 16. WORKING WITH YAML <a name="16-yaml"></a>

```bash
# Install YAML package
go get gopkg.in/yaml.v3
```

```go
import (
    "fmt"
    "os"
    "gopkg.in/yaml.v3"
)

type DeploymentConfig struct {
    Name     string            `yaml:"name"`
    Image    string            `yaml:"image"`
    Replicas int               `yaml:"replicas"`
    Ports    []int             `yaml:"ports"`
    Env      map[string]string `yaml:"env"`
    Resources struct {
        CPU    string `yaml:"cpu"`
        Memory string `yaml:"memory"`
    } `yaml:"resources"`
}

// Read YAML file
func loadConfig(path string) (*DeploymentConfig, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, err
    }

    var config DeploymentConfig
    if err := yaml.Unmarshal(data, &config); err != nil {
        return nil, err
    }
    return &config, nil
}

// Write YAML
config := DeploymentConfig{
    Name:     "property-api",
    Image:    "rea/property-api:v2.3.1",
    Replicas: 3,
    Ports:    []int{8080, 9090},
    Env: map[string]string{
        "NODE_ENV": "production",
        "LOG_LEVEL": "info",
    },
}

yamlBytes, _ := yaml.Marshal(config)
fmt.Println(string(yamlBytes))
// Output:
// name: property-api
// image: rea/property-api:v2.3.1
// replicas: 3
// ports:
//     - 8080
//     - 9090
// env:
//     LOG_LEVEL: info
//     NODE_ENV: production

// Write to file
os.WriteFile("config.yaml", yamlBytes, 0644)
```

---

## 17. FILE I/O <a name="17-file-io"></a>

```go
import (
    "bufio"
    "fmt"
    "os"
    "strings"
)

// =================== Read entire file ===================
data, err := os.ReadFile("/etc/nginx/nginx.conf")
if err != nil {
    fmt.Println("Error:", err)
    return
}
fmt.Println(string(data))

// =================== Write entire file ===================
content := []byte("Hello, World!\n")
err := os.WriteFile("output.txt", content, 0644)

// =================== Read line by line ===================
file, err := os.Open("access.log")
if err != nil {
    fmt.Println("Error:", err)
    return
}
defer file.Close()

scanner := bufio.NewScanner(file)
lineNum := 0
for scanner.Scan() {
    lineNum++
    line := scanner.Text()
    if strings.Contains(line, "ERROR") {
        fmt.Printf("Line %d: %s\n", lineNum, line)
    }
}

// =================== Append to file ===================
file, err := os.OpenFile("app.log", os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
if err != nil {
    return
}
defer file.Close()
file.WriteString(fmt.Sprintf("[%s] Server started\n", time.Now().Format(time.RFC3339)))

// =================== Check if file exists ===================
if _, err := os.Stat("/etc/nginx/nginx.conf"); os.IsNotExist(err) {
    fmt.Println("File does not exist")
}

// =================== List directory ===================
entries, _ := os.ReadDir("/var/www/html")
for _, entry := range entries {
    if entry.IsDir() {
        fmt.Printf("[DIR]  %s\n", entry.Name())
    } else {
        info, _ := entry.Info()
        fmt.Printf("[FILE] %s (%d bytes)\n", entry.Name(), info.Size())
    }
}

// =================== Walk directory tree ===================
import "path/filepath"
filepath.Walk("/var/www", func(path string, info os.FileInfo, err error) error {
    if err != nil {
        return err
    }
    if strings.HasSuffix(path, ".go") {
        fmt.Println(path)
    }
    return nil
})

// =================== Create directory ===================
os.MkdirAll("/var/www/app/logs", 0755)   // Creates parents too
```

---

## 18. HTTP SERVERS & CLIENTS <a name="18-http"></a>

### HTTP Server (essential for platform tools)

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "time"
)

// Response types
type HealthResponse struct {
    Status    string `json:"status"`
    Timestamp string `json:"timestamp"`
    Version   string `json:"version"`
}

type Service struct {
    Name    string `json:"name"`
    Status  string `json:"status"`
    Latency string `json:"latency"`
}

// Handler functions
func healthHandler(w http.ResponseWriter, r *http.Request) {
    resp := HealthResponse{
        Status:    "healthy",
        Timestamp: time.Now().Format(time.RFC3339),
        Version:   "1.0.0",
    }
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(resp)
}

func servicesHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodGet {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }

    services := []Service{
        {Name: "property-api", Status: "healthy", Latency: "23ms"},
        {Name: "search-service", Status: "healthy", Latency: "45ms"},
        {Name: "auth-service", Status: "degraded", Latency: "120ms"},
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(services)
}

func main() {
    http.HandleFunc("/healthz", healthHandler)
    http.HandleFunc("/api/services", servicesHandler)

    fmt.Println("Server starting on :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### HTTP Client (calling APIs)

```go
import (
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "time"
)

// Simple GET
func getServices(baseURL string) ([]Service, error) {
    // Create client with timeout
    client := &http.Client{Timeout: 10 * time.Second}

    resp, err := client.Get(baseURL + "/api/services")
    if err != nil {
        return nil, fmt.Errorf("request failed: %w", err)
    }
    defer resp.Body.Close()

    if resp.StatusCode != http.StatusOK {
        body, _ := io.ReadAll(resp.Body)
        return nil, fmt.Errorf("unexpected status %d: %s", resp.StatusCode, string(body))
    }

    var services []Service
    if err := json.NewDecoder(resp.Body).Decode(&services); err != nil {
        return nil, fmt.Errorf("failed to decode response: %w", err)
    }
    return services, nil
}

// POST with JSON body
func createService(baseURL string, svc Service) error {
    body, _ := json.Marshal(svc)

    resp, err := http.Post(baseURL+"/api/services", "application/json", bytes.NewReader(body))
    if err != nil {
        return err
    }
    defer resp.Body.Close()

    if resp.StatusCode != http.StatusCreated {
        return fmt.Errorf("unexpected status: %d", resp.StatusCode)
    }
    return nil
}

// Custom request with headers
func getWithAuth(url, token string) (*http.Response, error) {
    req, err := http.NewRequest("GET", url, nil)
    if err != nil {
        return nil, err
    }
    req.Header.Set("Authorization", "Bearer "+token)
    req.Header.Set("Accept", "application/json")

    client := &http.Client{Timeout: 10 * time.Second}
    return client.Do(req)
}
```

---

## 19. CLI TOOLS <a name="19-cli"></a>

### Using os.Args & flag package

```go
package main

import (
    "flag"
    "fmt"
    "os"
)

func main() {
    // Raw args
    // os.Args[0] = program name
    // os.Args[1:] = arguments

    // flag package (built-in argument parser)
    namespace := flag.String("namespace", "default", "Kubernetes namespace")
    verbose := flag.Bool("verbose", false, "Enable verbose output")
    timeout := flag.Int("timeout", 30, "Timeout in seconds")

    flag.Parse()

    fmt.Println("Namespace:", *namespace)
    fmt.Println("Verbose:", *verbose)
    fmt.Println("Timeout:", *timeout)
    fmt.Println("Remaining args:", flag.Args())

    // Usage: ./tool -namespace=production -verbose -timeout=60 extra1 extra2
}
```

### Complete CLI Tool Example: Service Status Checker

```go
package main

import (
    "encoding/json"
    "flag"
    "fmt"
    "net/http"
    "os"
    "sync"
    "time"
)

type ServiceConfig struct {
    Services []struct {
        Name string `json:"name"`
        URL  string `json:"url"`
    } `json:"services"`
}

type Result struct {
    Name    string
    URL     string
    Status  string
    Latency time.Duration
    Error   string
}

func checkService(name, url string, results chan<- Result, wg *sync.WaitGroup) {
    defer wg.Done()

    start := time.Now()
    client := &http.Client{Timeout: 5 * time.Second}
    resp, err := client.Get(url)
    latency := time.Since(start)

    if err != nil {
        results <- Result{Name: name, URL: url, Status: "DOWN", Latency: latency, Error: err.Error()}
        return
    }
    defer resp.Body.Close()

    status := "HEALTHY"
    if resp.StatusCode >= 400 {
        status = "UNHEALTHY"
    }
    results <- Result{Name: name, URL: url, Status: status, Latency: latency}
}

func main() {
    configFile := flag.String("config", "services.json", "Path to services config")
    flag.Parse()

    data, err := os.ReadFile(*configFile)
    if err != nil {
        fmt.Fprintf(os.Stderr, "Error reading config: %v\n", err)
        os.Exit(1)
    }

    var config ServiceConfig
    if err := json.Unmarshal(data, &config); err != nil {
        fmt.Fprintf(os.Stderr, "Error parsing config: %v\n", err)
        os.Exit(1)
    }

    results := make(chan Result, len(config.Services))
    var wg sync.WaitGroup

    for _, svc := range config.Services {
        wg.Add(1)
        go checkService(svc.Name, svc.URL, results, &wg)
    }

    // Close results channel when all goroutines complete
    go func() {
        wg.Wait()
        close(results)
    }()

    // Print results
    fmt.Printf("%-20s %-10s %-12s %s\n", "SERVICE", "STATUS", "LATENCY", "ERROR")
    fmt.Println(strings.Repeat("-", 70))

    hasFailures := false
    for r := range results {
        symbol := "✓"
        if r.Status != "HEALTHY" {
            symbol = "✗"
            hasFailures = true
        }
        fmt.Printf("%s %-20s %-10s %-12s %s\n", symbol, r.Name, r.Status, r.Latency.Round(time.Millisecond), r.Error)
    }

    if hasFailures {
        os.Exit(1)
    }
}
```

---

## 20. TESTING IN GO <a name="20-testing"></a>

```go
// health_test.go — test files end with _test.go
package health

import "testing"

// Test functions must start with Test and take *testing.T
func TestCheckURL_Healthy(t *testing.T) {
    // Arrange
    url := "https://httpbin.org/status/200"

    // Act
    healthy, err := CheckURL(url)

    // Assert
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
    if !healthy {
        t.Errorf("expected healthy=true, got false")
    }
}

func TestCheckURL_Unhealthy(t *testing.T) {
    healthy, err := CheckURL("https://httpbin.org/status/500")
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
    if healthy {
        t.Errorf("expected healthy=false, got true")
    }
}

// Table-driven tests (Go convention for testing multiple cases)
func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive", 2, 3, 5},
        {"negative", -1, -2, -3},
        {"zero", 0, 0, 0},
        {"mixed", -1, 5, 4},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Add(tt.a, tt.b)
            if result != tt.expected {
                t.Errorf("Add(%d, %d) = %d, want %d", tt.a, tt.b, result, tt.expected)
            }
        })
    }
}
```

```bash
# Run tests
go test ./...                     # All tests in project
go test -v ./...                  # Verbose (show each test)
go test -run TestAdd ./...        # Run specific test
go test -cover ./...              # Show coverage %
go test -coverprofile=cover.out ./...  # Generate coverage file
go tool cover -html=cover.out     # View coverage in browser
```

---

## 21. GO FOR KUBERNETES & PLATFORM TOOLING <a name="21-k8s-tooling"></a>

### Calling kubectl from Go

```go
import (
    "fmt"
    "os/exec"
    "strings"
)

func getPods(namespace string) ([]string, error) {
    cmd := exec.Command("kubectl", "get", "pods", "-n", namespace, "-o", "name")
    output, err := cmd.CombinedOutput()
    if err != nil {
        return nil, fmt.Errorf("kubectl failed: %s", string(output))
    }
    lines := strings.Split(strings.TrimSpace(string(output)), "\n")
    return lines, nil
}
```

### Using client-go (Kubernetes Go client)

```bash
go get k8s.io/client-go@latest
go get k8s.io/apimachinery@latest
```

```go
package main

import (
    "context"
    "fmt"
    "os"
    "path/filepath"

    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/client-go/kubernetes"
    "k8s.io/client-go/tools/clientcmd"
)

func main() {
    // Load kubeconfig
    kubeconfig := filepath.Join(os.Getenv("HOME"), ".kube", "config")
    config, err := clientcmd.BuildConfigFromFlags("", kubeconfig)
    if err != nil {
        fmt.Println("Error:", err)
        os.Exit(1)
    }

    // Create clientset
    clientset, err := kubernetes.NewForConfig(config)
    if err != nil {
        fmt.Println("Error:", err)
        os.Exit(1)
    }

    // List pods in all namespaces
    pods, err := clientset.CoreV1().Pods("").List(context.TODO(), metav1.ListOptions{})
    if err != nil {
        fmt.Println("Error:", err)
        os.Exit(1)
    }

    fmt.Printf("Found %d pods\n\n", len(pods.Items))
    for _, pod := range pods.Items {
        fmt.Printf("%-30s %-15s %-10s\n", pod.Name, pod.Namespace, pod.Status.Phase)
    }
}
```

### Build & Cross-Compile

```bash
# Build for current platform
go build -o platform-tool ./cmd/tool/

# Cross-compile for Linux (deploy to server)
GOOS=linux GOARCH=amd64 go build -o platform-tool-linux ./cmd/tool/

# Build with version info
go build -ldflags="-X main.version=1.2.3" -o platform-tool ./cmd/tool/

# Build a smaller binary
go build -ldflags="-s -w" -o platform-tool ./cmd/tool/
```

---

## 22. COMMON INTERVIEW PATTERNS <a name="22-interview-patterns"></a>

### Pattern 1: Process a list concurrently and collect results

```go
func processItems(items []string) []Result {
    results := make(chan Result, len(items))
    var wg sync.WaitGroup

    for _, item := range items {
        wg.Add(1)
        go func(i string) {
            defer wg.Done()
            // Process item
            results <- Result{Item: i, Status: "done"}
        }(item)
    }

    go func() {
        wg.Wait()
        close(results)
    }()

    var allResults []Result
    for r := range results {
        allResults = append(allResults, r)
    }
    return allResults
}
```

### Pattern 2: HTTP API with JSON

```go
func handleRequest(w http.ResponseWriter, r *http.Request) {
    // Read request body
    var req RequestBody
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, "invalid JSON", http.StatusBadRequest)
        return
    }

    // Process
    result, err := process(req)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    // Write response
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(result)
}
```

### Pattern 3: Read config file and use it

```go
type Config struct {
    Port     int      `yaml:"port"`
    LogLevel string   `yaml:"log_level"`
    Services []string `yaml:"services"`
}

func loadConfig(path string) (*Config, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, fmt.Errorf("reading config: %w", err)
    }
    var cfg Config
    if err := yaml.Unmarshal(data, &cfg); err != nil {
        return nil, fmt.Errorf("parsing config: %w", err)
    }
    return &cfg, nil
}
```

### Pattern 4: Retry with backoff

```go
func retry(attempts int, sleep time.Duration, f func() error) error {
    for i := 0; i < attempts; i++ {
        err := f()
        if err == nil {
            return nil
        }
        fmt.Printf("Attempt %d failed: %v. Retrying in %v...\n", i+1, err, sleep)
        time.Sleep(sleep)
        sleep *= 2  // Exponential backoff
    }
    return fmt.Errorf("failed after %d attempts", attempts)
}

// Usage:
err := retry(3, time.Second, func() error {
    return deployService("property-api")
})
```

---

## 23. QUICK REFERENCE CARD <a name="23-reference"></a>

### Go vs Python Cheat Sheet

| Task | Python | Go |
|---|---|---|
| Print | `print("hello")` | `fmt.Println("hello")` |
| Format string | `f"Name: {name}"` | `fmt.Sprintf("Name: %s", name)` |
| Variable | `x = 42` | `x := 42` |
| List/Slice | `nums = [1,2,3]` | `nums := []int{1,2,3}` |
| Append | `nums.append(4)` | `nums = append(nums, 4)` |
| Dict/Map | `d = {"a": 1}` | `d := map[string]int{"a": 1}` |
| Length | `len(x)` | `len(x)` |
| Function | `def foo(x):` | `func foo(x int) int {` |
| Multiple return | `return a, b` | `return a, b` |
| For loop | `for x in list:` | `for _, x := range list {` |
| While | `while True:` | `for {` |
| Error | `try/except` | `if err != nil {` |
| Class | `class Foo:` | `type Foo struct {` |
| Method | `def bar(self):` | `func (f Foo) bar() {` |
| Import | `import json` | `import "encoding/json"` |
| Read file | `open("f").read()` | `os.ReadFile("f")` |
| HTTP request | `requests.get(url)` | `http.Get(url)` |
| HTTP server | `flask.run()` | `http.ListenAndServe()` |
| JSON parse | `json.loads(s)` | `json.Unmarshal([]byte(s), &v)` |
| JSON dump | `json.dumps(obj)` | `json.Marshal(obj)` |
| Thread | `threading.Thread` | `go func()` |
| Sleep | `time.sleep(1)` | `time.Sleep(time.Second)` |

### Common Commands

```bash
go run main.go              # Compile and run
go build -o app             # Build binary
go test ./...               # Run all tests
go test -v -run TestName    # Run specific test
go mod init module-name     # Initialize module
go mod tidy                 # Clean dependencies
go fmt ./...                # Format all code
go vet ./...                # Find bugs
go doc fmt.Println          # View docs
```

### Standard Library Packages You'll Use Most

| Package | Purpose |
|---|---|
| `fmt` | Formatted I/O (Println, Sprintf, Printf) |
| `os` | OS operations (files, env vars, exit) |
| `os/exec` | Run external commands |
| `io` | I/O interfaces (Reader, Writer) |
| `strings` | String manipulation |
| `strconv` | String ↔ number conversion |
| `encoding/json` | JSON marshal/unmarshal |
| `net/http` | HTTP client and server |
| `time` | Time operations, durations, timers |
| `sync` | WaitGroup, Mutex, Once |
| `context` | Cancellation, timeouts, deadlines |
| `flag` | Command-line argument parsing |
| `log` | Logging |
| `path/filepath` | File path manipulation |
| `bufio` | Buffered I/O (line-by-line reading) |
| `sort` | Sorting slices |
| `errors` | Error creation, wrapping, unwrapping |
| `regexp` | Regular expressions |
| `testing` | Unit testing framework |
