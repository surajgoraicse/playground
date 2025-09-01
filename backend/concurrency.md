
## problem 1 : ticket booking system

You are designing a **ticket booking system** for a concert.
- There are **100 tickets** available.
- Multiple users are trying to book tickets **at the same time**.
- Each booking request is processed in a **separate goroutine** (to simulate real-world concurrent requests).
- If multiple users try to book a ticket at the same time, the system must ensure:
    1. No ticket is **double-booked**.
    2. The total number of tickets never goes below zero.


### solution : 

- mutex lock should also enclose the case where ticketCount = 0
```go
if ticketCount == 0 {
	fmt.Println("No more tickets are available")
	return
}
```


```go
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

var ticketCount = 100
var wg sync.WaitGroup
var mu sync.Mutex

func bookTicket(userID int) {
	defer wg.Done()

	want := rand.Intn(5) + 1  // number of tickets each user wants

	mu.Lock()   // avoid race condition
	defer mu.Unlock()

	if ticketCount == 0 {
		fmt.Printf("User %d: No more tickets are available\n", userID)
		return
	}

	if want <= ticketCount {
		ticketCount -= want
		fmt.Printf("User %d booked %d ticket(s). Tickets left: %d\n",
			userID, want, ticketCount)
	} else {
		fmt.Printf("User %d wanted %d ticket(s), but only %d left. Booking failed.\n",
			userID, want, ticketCount)
	}
}

func main() {
	for i := 1; i <= 50; i++ {
		wg.Add(1)
		go bookTicket(i)
	}

	wg.Wait()
	fmt.Println("\nFinal tickets available:", ticketCount)
}
```



## problem 2 : banking system to simulate deposite and withdraw

You are building a **banking system** where multiple users can **deposit** and **withdraw** money from a shared account at the same time.
- The account starts with **₹10,000**.
- Each user action (deposit/withdraw) is processed in a **separate goroutine** (simulating concurrent transactions).
- You must ensure that:
    1. Balance never goes below zero.
    2. Deposits and withdrawals don’t conflict with each other (i.e., prevent race conditions).
👉 You need to use **goroutines** to simulate users making transactions and a **mutex** to protect the shared `balance`.



### solution : 

- define deposite and withdraw function seperately.
	- use rand.Intn(2) for equal probability for calling methods.
- handle the situation of withdraw when balance is 0 and less than withdraw amount. 

```go
package main

import (
	"fmt"
	"math/rand"
	"sync"
)

var mu sync.Mutex
var wg sync.WaitGroup
var balance = 10000
var users = 40

func deposit(user int, amount int) {
	defer wg.Done()

	mu.Lock()
	defer mu.Unlock()

	balance += amount
	fmt.Printf("User %d deposited %d. Balance: %d\n", user, amount, balance)
}

func withdraw(user int, amount int) {
	defer wg.Done()

	mu.Lock()
	defer mu.Unlock()

	if -amount > balance { // prevent overdraft
		fmt.Printf("User %d tried to withdraw %d, but insufficient balance! Balance: %d\n",
			user, -amount, balance)
		return
	}

	balance += amount
	fmt.Printf("User %d withdrew %d. Balance: %d\n", user, -amount, balance)
}

func main() {
	for i := 1; i <= users; i++ {
		wg.Add(1)

		if rand.Intn(2) == 0 { // deposit
			amount := rand.Intn(10000) + 1 // 1–10000
			go deposit(i, amount)
		} else { // withdraw
			amount := -(rand.Intn(4000) + 1) // -1 to -4000
			go withdraw(i, amount)
		}
	}

	wg.Wait()
	fmt.Println("\nFinal balance:", balance)
}
```



## 3. 
