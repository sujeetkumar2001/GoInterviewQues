# GoInterviewQues

# Why Go?

Go is a statically typed, compiled language designed for simplicity, speed, and efficiency.

### 1. Simplicity and Readability
Go’s syntax is easy to understand, which enables quick onboarding for new developers. Its minimalistic design reduces complexity, allowing developers to focus on writing clean and maintainable code without excessive language features.

### 2. High Performance
Go is faster than many languages due to built-in tools such as garbage collection and Goroutines, which support efficient concurrency and memory management. These features make Go suitable for high-performance applications, even under heavy workloads.

### 3. Cross-Platform Deployment
Go’s static binaries run seamlessly across multiple operating systems, making deployment straightforward, especially in containerized environments. This cross-platform compatibility simplifies scaling and automating infrastructure.

### 4. Industry Trust
Go is trusted by major companies like Google, Uber, and Docker for building reliable, large-scale applications. Its proven scalability and dependability make it a top choice for modern, production-grade software.


# Arrays 
to store the data and it has fixed size, and is passed by value ( you assign or pass an array to another variable or function, a copy is created)



# Slices 
A slice is a dynamically-sized, flexible view into the elements of an array.it just describes a section of an underlying array.
Slices are references to arrays, so when you pass a slice to a function, any changes made to the slice’s elements affect the original data.

Length is the actual number of elements the slice currently holds, while capacity is the maximum number of elements the slice can grow to without allocating a new underlying array.
If the slice’s capacity is exceeded, Go creates a new, larger array, copies the elements over, and then returns a new slice pointing to this new array.

Slicing a slice does not reduce the capacity; the new slice still refers to the same underlying array.
Capacity only shrinks when you create a new slice explicitly with a smaller capacity.

```
package main

import (
	"fmt"
	"runtime"
)

func main() {
	// Initial slice with length and capacity of 3
	s := make([]int, 3, 3)
	s[0], s[1], s[2] = 1, 2, 3

	fmt.Printf("Before append - Slice: %v, Length: %d, Capacity: %d\n", s, len(s), cap(s))

	// Keep a reference to the original slice before appending
	original := s

	// Append to exceed capacity, creating a new array
	s = append(s, 4)
	fmt.Printf("After append - Slice: %v, Length: %d, Capacity: %d\n", s, len(s), cap(s))

	// Trigger garbage collection
	runtime.GC()

	// Check memory stats
	var memStats runtime.MemStats
	runtime.ReadMemStats(&memStats)
	fmt.Printf("Alloc = %v MiB\n", memStats.Alloc/1024/1024)

	// Access the original slice
	fmt.Printf("Original slice still accessible: %v\n", original)
}

```

# Function Closure 
In Go, a closure is a function value that references variables from outside its own scope.
These referenced variables are "captured" by the function, allowing the closure to access and modify them even after the outer function has returned.

Function closure allow functions to "remember" variables from their surrounding environment even after the outer function has returned.

How Closures Work
A closure is created when:

An inner function references variables from its enclosing (outer) function.
The enclosing function exits, but the inner function retains access to the outer function's scope.(variables (its reference))

```
func counter(start int) func() int {
    count := start
    return func() int {
        count++ // Captures `count`
        return count
    }
}

func main() {
    increment := counter(10)
    fmt.Println(increment()) // 11
    fmt.Println(increment()) // 12
}
```

```
func greet(name string) func() {
    return func() {
        fmt.Println("Hello,", name) // Captures `name`
    }
}

func main() {
    sayHello := greet("Alice")
    sayHello() // Prints: Hello, Alice
}

```

Common pitfall for closure 

Capturing Loop Variables: Variables declared in a for loop can cause unintended behavior because closures capture their references, not their values.
```
func main() {
    var funcs []func()
    for i := 0; i < 3; i++ {
        funcs = append(funcs, func() { fmt.Println(i) })
    }
    for _, fn := range funcs {
        fn() // Prints: 3, 3, 3
    }
}

```
fix, create local copy, so when closure is created , a local copy reference  is created  and stored by closure.
```
for i := 0; i < 3; i++ {
    v := i
    funcs = append(funcs, func() { fmt.Println(v) })
}

```

Explain the problem the closure solves.

Present the Problem
Imagine we have a bank account system, and we need to track the balance securely.
The balance should only be modified through controlled operations like deposit or withdrawal, 
and direct access to the balance should not be allowed.

```
func bankAccount(initialBalance int) func(int) int {
    balance := initialBalance // Private variable
    return func(amount int) int {
        balance += amount // Only accessible here
        return balance
    }
}

func main() {
    account := bankAccount(100) // Initialize with 100
    fmt.Println(account(50))  // Deposit 50: 150
    fmt.Println(account(-30)) // Withdraw 30: 120
}

```


#  new vs make 
new allocates memory for a single value and returns a pointer to it, while make initializes and allocates memory for slices, maps, and channels, returning the value itself (not a pointer).
Go's new and make allocate memory  on the heap.

Variables: Stack for locals, heap for dynamics.
Pointers: The pointer variable is stack-allocated, but what it points to depends on the allocation method.

Go that illustrates a local pointer variable allocated on the stack, while the memory it points to is allocated on the heap:

```

func example() {
    ptr := new(int) // Allocate an integer on the heap.
    *ptr = 42       // Assign a value to the memory on the heap.

    fmt.Printf("Pointer address: %p\n", ptr) // Pointer stored on the stack.
    fmt.Printf("Value: %d\n", *ptr)         // Memory it points to is on the heap.
}

func main() {
    example()
}
```

Note: go does not allow dereferencing  of a constant , complile time error.

# pointer 
pointer is a variable that stores the memory address of another variable. Pointers are especially useful when you want to share large data structures without copying them, or when you want to modify the original data within a function or another part of your program.

# Error Handling
Go encourages checking and handling errors immediately after they occur, making the code flow more explicit.
Custom errors can be created with the errors.New or fmt.Errorf functions to add context to error messages.
developers can create types that implement the error interface.
```
type MyError struct {
    Message string
    Code    int
}

func (e *MyError) Error() string {
    return fmt.Sprintf("Error %d: %s", e.Code, e.Message)
}
```

# GO avoids exception
By errors handling explicitly go ensure error is handled right away when error comes,
Go’s error handling model removes the need to understand exception hierarchies, try/catch blocks, or unwinding the stack for errors.
Exception handling requires stack unwinding and often incurs performance overhead.
Go’s approach is generally more performant, as there’s no need for complex exception handling mechanisms.

# Interfaces 
interfaces define a set of method signatures.Interfaces in Go are abstract types .
An interface is satisfied implicitly in Go. If a type implements all the methods declared in an interface,
the type is said to "satisfy" the interface, even without explicitly declaring that it does so.

In Go, functions can also use interfaces to achieve dynamic behavior, allowing them to operate on multiple types as long as the types satisfy a specific interface. 
Here’s how you can use interfaces in functions:

Note : Implicit Implementation: You don’t need to explicitly specify that a type implements an interface.
As long as it provides the required methods, it satisfies the interface.

```
package main

import (
	"fmt"
)

type Shape interface {
	Perimeter(side float64) float64
}

type Square struct {
	side float64
}

func (s Square) Perimeter(side float64) float64 {
	return 4 * side
}

type Circle struct {
	radius float64
}

func (s Circle) Perimeter(radius float64) float64 {
	return 2 * 3.14 * radius
}

func main() {
	s := Square{side: 4.12}
	c := Circle{radius: 1.2}

	var shape Shape

	shape = s
	fmt.Printf("perimeter square:%.2f\n", shape.Perimeter(s.side))

	shape = c
	fmt.Printf("perimeter circle:%.2f\n", shape.Perimeter(c.radius))
}

```


# PolyMorphism
in general, Polymorphism as the ability of a message to be displayed in more than one form.
polymorphism allows you to define one interface and have multiple implementations.
it allows us to create more flexible code and reusable code.Understanding when and how to use it can make your code more maintainable.

In Go, polymorphism is achieved through interfaces.
Run-Time Polymorphism (Interfaces):
Go relies on interfaces to achieve polymorphism at runtime.

Compile-time polymorphism: no suppport in go, early bunding of types at compile time.where the compiler 
determines which method to call based on the method’s signature and the object’s type at compile-time

Benefits
Code Reusability: Write generic code that works with multiple types.
Scalability: Add new types without changing existing code.
Decoupling: Functions and methods depend on interfaces, not concrete types, improving maintainability.

Using Generics (Since Go 1.18) Go’s generics allow you to write functions that operate on multiple types, reducing the need for overloading.

```
package main

import "fmt"

func Add[T int | float64](a, b T) T {
    return a + b
}

func main() {
    fmt.Println(Add(1, 2))          // 3 (int)
    fmt.Println(Add(1.1, 2.2))      // 3.3 (float64)
}

```

variadic functions, interfaces, and generics provide robust alternatives.

# OOPs in go
Encapsulation:
It is defined as the wrapping up of data under a single unit. It is the mechanism that binds together code and the data it manipulates. 
Another way to think about encapsulation is, that it is a protective shield that prevents the data from being accessed by the code outside this shield.

Encapsulation is achieved using structs and methods. By defining export and unexported (private) field in struct.

```
package main

import "fmt"

type Person struct {
    name string // unexported (private)
    Age  int    // exported (public)
}

func (p *Person) GetName() string {
    return p.name
}

func (p *Person) SetName(name string) {
    p.name = name
}

func main() {
    p := Person{Age: 30}
    p.SetName("Alice")
    fmt.Println("Name:", p.GetName()) // Access through methods
    fmt.Println("Age:", p.Age)       // Direct access to exported field
}

```

Inheritance :
Inheritance allows one type to acquire properties and behaviors from another type.
Go doesn’t have class-based inheritance, but you can achieve similar behavior using embedding.
```
package main

import "fmt"

type Animal struct {
    Name string
}

func (a Animal) Speak() {
    fmt.Println(a.Name, "makes a sound.")
}

type Dog struct {
    Animal // Embedding
    Breed  string
}

func main() {
    d := Dog{
        Animal: Animal{Name: "Buddy"},
        Breed:  "Golden Retriever",
    }
    d.Speak() // Inherited behavior
    fmt.Println("Breed:", d.Breed)
}

```

Abstraction :
Abstraction hides implementation details. In Go, interfaces are the primary mechanism for achieving abstraction.

Composition :
Composition is the concept of building complex types by combining simpler types. 
```
package main

import "fmt"

type Engine struct {
    Horsepower int
}

type Car struct {
    Make   string
    Model  string
    Engine // Embedded struct
}

func main() {
    c := Car{
        Make:   "Toyota",
        Model:  "Camry",
        Engine: Engine{Horsepower: 200},
    }
    fmt.Println("Car:", c.Make, c.Model, "with", c.Engine.Horsepower, "HP")
}

```

No Classes: Go uses structs and methods instead of classes.
No Inheritance: Go encourages composition through embedding rather than inheritance.
Interfaces over Abstract Classes: Interfaces are more flexible because they are satisfied implicitly.
No Overloading or Method Overriding: Functions and methods must have unique names, and there’s no concept of overriding methods.

# Defer keyword
Go is used to schedule a function call to be executed after the surrounding function completes,
regardless of whether it completes normally or through a return or a panic.

Execution Timing: The deferred function runs just before the enclosing function exits.
LIFO Execution Order: If multiple defers are defined, they are executed in reverse order of their appearance.
Captured Arguments: Arguments to the deferred function are evaluated when the defer statement is encountered, not when the function is executed.

Common used:
Resource cleanup 
With mutex unlocking the resource after the function work is done. 
Releasing the database connection.

panic: When a panic occurs, the normal flow of the program stops, and Go starts unwinding the stack, calling any deferred functions in the proces
recover: recover can be used inside a deferred function to stop the panic and regain control of the program.

Use case of recover:
Preventing program crashes and maintaining stability.
Handling panics within goroutines


When a function call is deferred using the defer keyword, the arguments to the deferred function are evaluated immediately.when the defer statement is executed.
Note :Immediate Argument Evaluation
```
func example() {
    x := 10
    defer fmt.Println("Deferred value of x:", x) // x is captured here (value is 10)
    x = 20
    fmt.Println("Current value of x:", x)       // x is updated to 20
}

example()

```

If the deferred function accesses variables directly, it will capture their address and reflect changes at execution time.
```
func example() {
    x := 10
    defer func() {
        fmt.Println("Deferred value of x:", x) // x is accessed directly
    }()
    x = 20
    fmt.Println("Current value of x:", x)
}

example()

```

Pass by Value:

If you want the deferred function to use the value as it was at the time of the defer statement, explicitly pass a copy.

Pass by Reference:

When a pointer or reference-type argument is passed to a deferred function, any changes to the underlying value or structure before the deferred function executes will be visible.
This is because defer captures the reference, not a copy of the value.

# How does garbage collection work in Go?
Go automatically manage memory by reclaiming unused memory.
Go uses a garbage collector to free up memory that is not used by the program. 

Mark-and-Sweep Algorithm:

The garbage collector uses a mark-and-sweep approach:
Mark Phase: Identifies all "reachable" objects starting from the roots (e.g., global variables, local stack references).
Sweep Phase: Reclaims memory for objects that were not marked, as they are no longer in use.

Steps
mark-and-sweep algorithm:

Mark Phase: It starts by identifying all objects that are still in use (reachable) from roots such as global variables, stack references, and heap pointers.
Sweep Phase: It reclaims memory occupied by objects that were not marked as reachable.
runtime.GC

# Goroutine vs Threads 
A goroutine is a lightweight thread of execution .managed by Go runtime(Goroutine Scheduling)
Blocking in a Goroutine doesn’t block the whole program
fast
require low memory

A thread is a lightweight process that shares the same memory space and resources as its parent process.managed by OS
Blocking in a thread can block the entire process
slow
require low memory

# Channel 
Channels in Go are a mechanism that allow goroutines to communicate and synchronize their execution
Blocking Nature:
Sending to a channel (ch <- value) blocks the sender until another goroutine reads from the channel.
Receiving from a channel (value := <-ch) blocks the receiver until another goroutine writes to the channel.

Benefits of Using Channels:
safe communication; Channels prevent race conditions when sharing data between goroutines.
Once the main function completes, the application terminates, killing all child goroutines.

```
package main

import (
	"fmt"
	"sync"
	"time"
)

func worker(i int, ch chan int, wg *sync.WaitGroup) {
	defer wg.Done()

	time.Sleep(2 * time.Second)

	fmt.Printf("work completed:%v", <-ch)
}

func main() {
	w := 5
	pool := 10

	ch := make(chan int, pool)

	wg := &sync.WaitGroup{}

	for i := 1; i <= w; i++ {
		wg.Add(1)
		ch <- i
		go worker(i, ch, wg)
	}

	//time.Sleep(5 * time.Second)
	wg.Wait()
}

```

Simple worker pool 
```
package main

import (
	"fmt"
	"sync"
	"time"
)

func worker(i int, ch chan int, result chan string, wg *sync.WaitGroup) {
	defer wg.Done()

	// Simulate work
	time.Sleep(2 * time.Second)

	// Process the work and send result to resultCh
	result <- fmt.Sprintf("Worker %d completed with job %d", i, <-ch)
}

func main() {
	w := 3
	pool := 10

	// Create channels
	emailCh := make(chan int, pool)
	resultCh := make(chan string, pool)

	var wg sync.WaitGroup

	// Start workers
	for i := 1; i <= w; i++ {
		wg.Add(1)
		go worker(i, emailCh, resultCh, &wg)
	}

	// Send jobs to the workers
	for i := 1; i <= pool; i++ {
		emailCh <- i
	}

	// Close the emailCh channel to signal no more jobs will be sent
	close(emailCh)

	// Wait for all workers to finish
	wg.Wait()

	// Close the resultCh channel as no more results will be sent
	close(resultCh)

	// Process results
	for result := range resultCh {
		fmt.Println(result)
	}
}
```

odd even

```
package main

import (
	"fmt"
	"sync"
)

func main() {
	nums := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}

	oddCh := make(chan int, len(nums)/2)
	evenCh := make(chan int, len(nums)/2)

	var wg sync.WaitGroup

	for i := 0; i < len(nums); i++ {
		if nums[i]%2 == 0 {
			evenCh <- nums[i]
		} else {
			oddCh <- nums[i]
		}
	}

	close(evenCh)
	close(oddCh)

	wg.Add(1)

	go func() {
		defer wg.Done()
		fmt.Print("odd:")
		for i := range oddCh {
			fmt.Printf(" %d", i)
		}
	}()

	//wg.Wait() // guarentee the order

	wg.Add(1)

	go func() {
		defer wg.Done()
		fmt.Print("even:")
		for i := range evenCh {
			fmt.Printf(" %d", i)
		}
	}()

	wg.Wait() // does not gaurantee the odd or even in order

}

```
counting number from 1 to 10 

```
package main

import (
	"fmt"
)

func main() {
	nums := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}

	signal := make(chan bool, 1)
	evenCh := make(chan int)

	done := make(chan bool)

	signal <- true

	go func() {
		for i := 1; i <= len(nums); i += 2 {
			<-signal
			fmt.Printf(" %d", i)

			evenCh <- i + 1
		}

		close(evenCh)

	}()

	go func() {

		for val := range evenCh {
			fmt.Printf(" %d", val)
			signal <- true
		}

		done <- true

	}()

	<-done
	close(done)
	close(signal)
}

```

```
package main

import (
	"fmt"
)

func main() {
	nums := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}

	signal := make(chan bool)
	evenCh := make(chan int)

	done := make(chan bool)

	go func() {
		for i := 1; i <= len(nums); i += 2 {
			<-signal
			fmt.Printf(" %d", i)

			evenCh <- i + 1
		}

		close(evenCh)

	}()

	signal <- true

	go func() {

		for val := range evenCh {
			fmt.Printf(" %d", val)
			if val == 10 {
				break
			}

			signal <- true
		}

		done <- true

	}()

	<-done
	close(done)
	close(signal)
}

```

odd and even 
```
package main

import (
	"fmt"
)

func main() {
	odd := make(chan bool)
	even := make(chan bool)
	done := make(chan bool)

	go func() {
		for i := 0; i <= 5; i += 2 {
			<-odd
			fmt.Println(i)
			even <- true
		}
	}()

	go func() {
		for i := 1; i <= 5; i += 2 {
			<-even

			fmt.Println(i)
			if i == 5 {
				done <- true
				close(even)
				close(odd)
				break
			}

			odd <- true
		}
	}()

	odd <- true

	<-done
	close(done)
}

```

```
package main

import (
	"fmt"
)

func main() {
	nums := []int{2, 4, 5, 6, 78, 3, 5, 5, 2, 2}

	evenCh := make(chan int)
	oddCh := make(chan int)

	done := make(chan bool)

	go func() {
		for val := range evenCh {
			fmt.Printf(" %d", val)
		}

		done <- true
	}()

	go func() {
		for val := range oddCh {
			fmt.Printf(" %d", val)
		}

		done <- true
	}()

	for i := 0; i < len(nums); i++ {
		if nums[i]%2 == 0 {
			evenCh <- nums[i]
		}
	}

	close(evenCh)

	for i := 0; i < len(nums); i++ {
		if nums[i]%2 != 0 {
			oddCh <- nums[i]
		}
	}

	close(oddCh)

	<-done // closing
	<-done // closing
}

```


# How would you avoid a deadlock in a concurrent program?
1. Lock Ordering: 
2. Timeouts and Contexts: Use timeouts or context packages to prevent goroutines from waiting indefinitely.
```
ctx, cancel := context.WithTimeout(context.Background(), time.Second*5)
defer cancel()

select {
case <-someChannel:
    // Process data
case <-ctx.Done():
    fmt.Println("Operation timed out")
}

```

3.Proper channel management: Closing channels at the right time prevents blocking in goroutines that are waiting to read from closed channels

# Channels 
Unbuffered Channels:
Definition: An unbuffered channel has no capacity to hold any values. A value can only be sent on an unbuffered channel when there is a receiver ready to receive it.
Behavior:

When a goroutine sends data on an unbuffered channel, it will block until another goroutine is ready to receive the value.
Similarly, a goroutine trying to receive data from an unbuffered channel will block until another goroutine sends a value.

Buffered Channels:
Definition: A buffered channel has a capacity to hold a specified number of values. A value can be sent on a buffered channel without an immediate receiver, as long as the channel is not full.

Behavior:

When sending data on a buffered channel, the sender will only block if the buffer is full.
When receiving data from a buffered channel, the receiver will only block if the buffer is empty.
The channel can hold a specified number of values in memory before blocking either the sender or receiver.

example for buff
Example Use Case:

A producer-consumer problem where the producer is generating tasks at a different rate than the consumer is processing them. You can buffer a few tasks so that the producer can continue producing without waiting for the consumer to process every task immediately.


# Race conditions

A race condition occurs when two or more threads can access shared data and they try to change it at the same time

to avoid 
1. Using Channels for Communication
   Channels are another way to avoid race conditions. By using channels to pass data between goroutines, you ensure that only one goroutine can access or modify a piece of data at a time, as only one goroutine can send or receive data from a channel.
2. Using Mutexes (Synchronization)
   A Mutex (short for "Mutual Exclusion") is a synchronization primitive that ensures only one goroutine can access a critical section of code at a time.

# Select 
The select Statement in Go
The select statement in Go is a powerful concurrency primitive that allows a goroutine to wait on multiple channels simultaneously.
It provides a way to handle multiple channel operations (send and receive) without blocking on any single channel, making it especially useful in concurrent programming.

How select Works:
Multiple Channel Operations: The select statement is like a switch, but instead of evaluating variables, it works with channels. It blocks until one of the channels is ready for communication (either a value is sent or received).
Non-blocking: If a channel operation is ready (e.g., data is available for receiving, or space is available for sending), the corresponding case in the select block executes. If none of the channels are ready, the select statement will block until one becomes ready.
Random Selection: When multiple channels are ready, Go randomly selects one of the ready channels to proceed.

```
select {
case ch1 <- value:
    // Send value to ch1
case value := <-ch2:
    // Receive value from ch2
case <-time.After(time.Second):
    // Timeout after 1 second
default:
    // Executes if no case is ready (non-blocking)
}

```

use case: 
Timeouts and Deadlocks: The select statement can be used to implement timeouts or avoid deadlocks in concurrent programs. For example, if a goroutine waits for a channel but wants to give up after a certain amount of time, it can use select with a time.After case to handle the timeout scenario.

```
package main

import "fmt"
import "time"

func sendData(ch chan string, delay time.Duration, msg string) {
    time.Sleep(delay)
    ch <- msg
}

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)

    go sendData(ch1, 2*time.Second, "Message from ch1")
    go sendData(ch2, 1*time.Second, "Message from ch2")

    select {
    case msg1 := <-ch1:
        fmt.Println("Received from ch1:", msg1)
    case msg2 := <-ch2:
        fmt.Println("Received from ch2:", msg2)
    }
}

```

# reverse a string

```
func main() {
	str := "hello world"

	s := []rune(str)

	l := 0
	r := len(s) - 1

	for l < r {
		s[l], s[r] = s[r], s[l]
		l++
		r--
	}

	fmt.Print(string(s))
}
```

# sync.Mutex vs sync.RMutex
sync.Mutex (Mutual Exclusion)
A Mutex (short for "Mutual Exclusion") is a simple locking mechanism used to ensure that only one goroutine can access the critical section of code at a time

sync.RWMutex (Read-Write Mutex)
A RWMutex (Read-Write Mutex) is a more advanced synchronization primitive that allows multiple read-only goroutines to access the critical section simultaneously while ensuring exclusive access for a write operation.
Read Lock (RLock): Multiple goroutines can hold the read lock simultaneously, as long as no goroutine has the write lock.
Write Lock (Lock): Only one goroutine can hold the write lock, and while it does, no other goroutine can hold either the read or write lock.

When to Use Each:
Use sync.Mutex when:

The resource is frequently modified (i.e., there are many writes).
You need exclusive access for every operation (both read and write).
Simplicity is a priority, as sync.Mutex is straightforward and works well for most basic use cases.
Use sync.RWMutex when:

The resource is read frequently, but writes are infrequent.
You want to allow multiple goroutines to read the resource at the same time while ensuring exclusive access for writes.
You need better performance when there are a lot of concurrent readers.

# Context 
context package provides a way to manage the lifecycle of requests and control the cancellation, deadlines, and timeouts for goroutines and API calls.
Context Object:

A context is an object that carries deadlines, cancellation signals, and other request-scoped values across API boundaries and goroutines.
Cancellation:
Timeouts and Deadlines:
Context Propagation:

context.Background(): Creates a context that is typically used at the root of a request chain, with no cancellation, timeout, or values associated.
context.WithCancel(parent): Creates a new context that can be canceled. When the cancel function is called, the context becomes done, and any goroutines that are listening for its cancellation will stop.
context.WithTimeout(parent, timeout): Creates a new context with a specified timeout. The context will be automatically canceled when the timeout expires
context.WithDeadline(parent, deadline): Creates a new context that will be canceled at a specific time (the deadline).
context.WithValue(parent, key, value): Allows attaching request-scoped values to the context.

# sync.WaitGroup vs sync.Once 
sync.Once
Purpose:
The sync.Once type is used to ensure that a certain function or block of code is executed only once, even if it is called from multiple goroutines. This is useful for scenarios where you want to initialize something exactly once, such as setting up shared resources, configurations, or doing expensive setup work
sync.Once ensures that a function is executed exactly once, even if it is called multiple times from different goroutines, making it ideal for tasks like initialization that should only happen once.

Example use cases: lazy initialization of shared resources, configuration setup, database connection setup.

# Select with time.After 
```
package main

import (
	"fmt"
	"net/http"
	"time"
)

func makeRequestWithTimeout(url string, timeout time.Duration) (string, error) {
	// Create a channel to signal the result of the HTTP request
	resultChan := make(chan string)

	// Start a goroutine to perform the HTTP request
	go func() {
		resp, err := http.Get(url)
		if err != nil {
			resultChan <- fmt.Sprintf("Error: %v", err)
			return
		}
		defer resp.Body.Close()

		// Read and return the response
		var response string
		if resp.StatusCode == http.StatusOK {
			response = "Request Successful"
		} else {
			response = fmt.Sprintf("HTTP Error: %d", resp.StatusCode)
		}
		resultChan <- response
	}()

	// Use select statement to wait for either the result or timeout
	select {
	case res := <-resultChan: // If the request is successful or fails
		return res, nil
	case <-time.After(timeout): // If the timeout duration passes
		return "", fmt.Errorf("request timed out after %s", timeout)
	}
}

func main() {
	url := "https://httpbin.org/delay/3" // A URL that delays the response for 3 seconds
	timeout := 2 * time.Second

	// Call the function to make the request with a timeout
	result, err := makeRequestWithTimeout(url, timeout)
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println("Result:", result)
	}
}

```

# GoRuntime 

# How does Go’s scheduler work for goroutines?
