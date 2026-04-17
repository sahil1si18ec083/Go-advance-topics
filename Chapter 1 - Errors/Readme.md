# 🧠 Understanding Custom Errors in Go (Using a Real Bank Account Example)

If you’re coming from Java or JavaScript, Go’s error handling can feel unusual at first.

No exceptions. No try-catch.

But once you understand it, you realize it’s **simple, powerful, and very explicit**.

In this article, we’ll learn custom errors in Go using a **real example** — a Bank Account system.

---

## 🏦 The Example We’ll Use

We have a `BankAccount` struct:

```go
type BankAccount struct {
	ID         string
	Owner      string
	Balance    float64
	MinBalance float64
}
```

And operations like:

* Create account
* Deposit
* Withdraw
* Transfer

Now let’s see how errors fit into this.

---

## 🔍 What is an error in Go?

```go
type error interface {
    Error() string
}
```

👉 Any type that implements `Error() string` is an error.

---

## 🛠️ Step 1: Creating Custom Errors

From our bank system:

```go
type NegativeAmountError struct{}

func (e *NegativeAmountError) Error() string {
    return "amount cannot be negative"
}
```

```go
type InsufficientFundsError struct{}

func (e *InsufficientFundsError) Error() string {
    return "insufficient funds"
}
```

```go
type ExceedsLimitError struct{}

func (e *ExceedsLimitError) Error() string {
    return "transaction exceeds limit"
}
```

---

## 🚀 Step 2: Using Them in Real Code

### Deposit

```go
func (a *BankAccount) Deposit(amount float64) error {
	if amount < 0 {
		return &NegativeAmountError{}
	}

	if amount > MaxTransactionAmount {
		return &ExceedsLimitError{}
	}

	a.Balance += amount
	return nil
}
```

---

### Withdraw

```go
func (a *BankAccount) Withdraw(amount float64) error {
	if amount < 0 {
		return &NegativeAmountError{}
	}

	if amount > MaxTransactionAmount {
		return &ExceedsLimitError{}
	}

	if a.Balance-amount < a.MinBalance {
		return &InsufficientFundsError{}
	}

	a.Balance -= amount
	return nil
}
```

---

### Account Creation

```go
func NewBankAccount(id, owner string, initialBalance, minBalance float64) (*BankAccount, error) {
	if id == "" || owner == "" {
		return nil, &AccountError{}
	}

	if initialBalance < 0 || minBalance < 0 {
		return nil, &NegativeAmountError{}
	}

	if initialBalance < minBalance {
		return nil, &InsufficientFundsError{}
	}

	return &BankAccount{
		ID:         id,
		Owner:      owner,
		Balance:    initialBalance,
		MinBalance: minBalance,
	}, nil
}
```

---

## 🔑 The Most Important Insight

When you return:

```go
return &InsufficientFundsError{}
```

👉 You are NOT returning a string
👉 You are returning a **type**

---

## 🧠 Type vs Message

| Concept                          | Purpose         |
| -------------------------------- | --------------- |
| Type (`*InsufficientFundsError`) | Used in logic   |
| Message (`"insufficient funds"`) | Used for humans |

---

## ❌ Common Mistake

```go
return errors.New("insufficient funds")
```

Why this is bad:

* Creates a generic error
* Loses type information
* Breaks type-based handling

---

## ✅ Correct Approach

```go
return &InsufficientFundsError{}
```

---

## ⚙️ Where is `Error()` actually used?

You never call:

```go
err.Error()
```

But when you do:

```go
fmt.Println(err)
```

👉 Go internally calls:

```go
err.Error()
```

---

## 🧪 Type Checking (Important)

```go
var insufficientErr *InsufficientFundsError

if errors.As(err, &insufficientErr) {
    fmt.Println("Handle insufficient funds case")
}
```

👉 This works only if you returned the **actual type**

---

## ❌ Unnecessary Pattern

```go
type NegativeAmountError struct {
    val error
}
```

👉 This is not needed.

---

## ✅ Idiomatic Version

```go
type NegativeAmountError struct{}
```

👉 Keep error types minimal unless you need extra data.

---

## 📦 When to Add Fields

Only when you need context:

```go
type InsufficientFundsError struct {
    Balance  float64
    Required float64
}

func (e *InsufficientFundsError) Error() string {
    return fmt.Sprintf("balance %.2f < required %.2f", e.Balance, e.Required)
}
```

---

## 🧠 Final Mental Model

Don’t think:

> “I am returning an error message”

Think:

> “I am returning a type that describes what went wrong”

---

## 🔥 One-Line Takeaway

👉 **In Go, errors are types — not just messages**

---

## ✅ Summary

* Create a struct for each error type
* Implement `Error() string`
* Return `&MyError{}`
* Avoid `errors.New()` for custom errors
* Add fields only when needed
* Use types for logic, messages for display

---

## 🚀 What to Learn Next

* `errors.Is`
* `errors.As`
* Error wrapping (`fmt.Errorf("%w")`)

These are heavily used in real-world backend systems and interviews.

---

If you’ve understood this, you’ve unlocked one of the most important concepts in Go 🚀
