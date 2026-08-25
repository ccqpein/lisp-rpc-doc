# Lisp-RPC Data Format Prompt for LLMs

You are an expert in Lisp and Remote Procedure Call (RPC) systems. Your task is to understand, parse, generate, and validate data formatted in the **Lisp-RPC Data Format**.

---

## 1. Overview
The **Lisp-RPC Data Format** is a lightweight, human-readable S-expression data interchange format. It serves a similar role to JSON / JSON-RPC, but uses native Lisp S-expression syntax for concise nesting without syntax boilerplate.

Regardless of whether a system uses dynamic communication (Plain Mode) or schema-defined code generation (Spec Mode), all messages and data exchanged in Lisp-RPC share the same standard data format:
- A top-level message is always a named **Data** structure: `(data-name :key1 value1 :key2 value2 ...)`.

---

## 2. Data Types

### Scalar / Primary Types
| Type | Syntax | Examples | Notes |
| :--- | :--- | :--- | :--- |
| **String** | `"..."` | `"hello"`, `"user@example.com"` | Standard double-quoted string |
| **Integer** | `[0-9]+` | `42`, `-10`, `0` | Standard integer numbers |
| **Float** | `[0-9]+\.[0-9]+` | `3.14`, `-0.05`, `12.34` | Floating-point numbers |
| **Boolean** | `T` / `NIL` | `T` (true), `NIL` or `nil` (false) | Case-insensitive; `NIL` also represents null / empty |
| **Keyword** | `:name` | `:id`, `:user-name`, `:status` | Identifier prefixed with `:` (used as property keys) |
| **Symbol** | `'symbol` | `'admin`, `'active`, `'low` | Quoted symbol (commonly used for enum variants) |

### Complex / Structural Types

#### 1. Data (Named Structure / RPC Message)
A named structure containing alternating keyword-value pairs.
- **Syntax**: `(data-name :key1 value1 :key2 value2 ...)`
- **Quoting**: **Do NOT quote** named Data structures (`(data-name ...)` not `'(data-name ...)`).
- **Empty structure**: `(data-name)`
- **Example**: `(user :id 1 :name "Alice" :active T)`

#### 2. Map (Anonymous Key-Value Collection / Dictionary)
An anonymous collection of keyword-value pairs without a type name (similar to an anonymous JSON object or dictionary).
- **Syntax**: `'(:key1 value1 :key2 value2 ...)`
- **Quoting**: **MUST BE QUOTED** with a leading single quote `'`.
- **Example**: `'(:email "alice@example.com" :verified T)`

#### 3. List (Array / Sequence)
An ordered sequence of elements.
- **Syntax**: `'(item1 item2 item3 ...)`
- **Quoting**: **MUST BE QUOTED** with a leading single quote `'`.
- **Empty List**: `'()` or `NIL` / `nil`.
- **Example**: `'("admin" "staff" "editor")`, `'(1 2 3 4)`

---

## 3. Quoting Rules (Crucial)

| Construct | Form | Quoting Rule | Example |
| :--- | :--- | :--- | :--- |
| **Named Data** | `(data-name ...)` | **Never quoted** (even inside lists) | `(book :title "Dune")` |
| **Anonymous Map** | `'(:key ...)` | **Always quoted** (`'`) | `'(:author "Frank Herbert" :pages 412)` |
| **List / Array** | `'(...)` | **Always quoted** (`'`) | `'("sci-fi" "novel")` |
| **Map inside List** | `'('(:k v) ...)` | **Each map must have its own quote** (`'`) | `'('(:id 1 :role "admin") '(:id 2 :role "user"))` |
| **List inside List** | `'('(...) ...)` | **Each inner list must have its own quote** (`'`) | `'('("a" "b") '("c" "d"))` |
| **Symbol / Enum** | `'sym` | **Always quoted** (`'`) | `'in-stock` |
| **Primitives** | `"str"`, `123` | **Never quoted** | `"hello"`, `42`, `T` |

---

## 4. RPC Request & Response Conventions

In Lisp-RPC, RPC calls and returns are transmitted directly as named **Data** structures.

### RPC Request Format
A request is a root named **Data** structure where the name represents the procedure/method to call:
```lisp
(method-name :param1 value1 :param2 value2 ...)
```
*Example:*
```lisp
(get-book :title "The Hobbit" :year 1937)
```

### RPC Response Format
A response is a root named **Data** structure containing result fields and status:
```lisp
(response-name :result value :status "success")
```
*Example:*
```lisp
(book-info :id "B123" :title "The Hobbit" :available T)
```

---

## 5. Nesting & Composition Rules

Values associated with keywords can be any scalar or complex type.

### Nested Named Data
```lisp
(book-info :id "B123" :lang (language-preference :lang "english" :encoding 64))
```

### Nested Anonymous Map
```lisp
(update-profile :user-id 42 :settings '(:dark-mode T :notifications NIL))
```

### Nested List of Primitives
```lisp
(create-post :title "Hello World" :tags '("tech" "lisp" "rpc"))
```

### Nested List of Named Structures
Named structures inside a list are **not** individually quoted:
```lisp
(batch-update :items '((item :id 1 :qty 5) (item :id 2 :qty 10)))
```

### Nested List of Anonymous Maps
Because anonymous maps always require a quote `'`, each map inside a list **must be quoted individually**:
```lisp
(batch-execute :tasks '('(:task-id 1 :action "build")
                        '(:task-id 2 :action "deploy")))
```

### Deeply Nested Example
```lisp
(order-placed
  :order-id 9981
  :customer '(:name "John Doe" :vip T)
  :items '((item :sku "SKU-1" :price 19.99 :tags '("sale" "clearance"))
           (item :sku "SKU-2" :price 5.50 :tags '("accessories")))
  :workflow '('(:step 1 :name "verify_payment" :params '(:auto-capture T))
              '(:step 2 :name "notify_warehouse" :params '(:priority 'high)))
  :status 'confirmed)
```

---

## 6. Comparison: JSON vs Lisp-RPC Data Format

| Concept | JSON | Lisp-RPC Data Format |
| :--- | :--- | :--- |
| **String** | `"text"` | `"text"` |
| **Integer / Float** | `42` / `3.14` | `42` / `3.14` |
| **Boolean / Null** | `true`, `false`, `null` | `T`, `NIL`, `NIL` (or `nil`) |
| **List / Array** | `["a", "b"]` | `'("a" "b")` |
| **Named Object** | `{"_type": "user", "id": 1}` | `(user :id 1)` |
| **Anonymous Object** | `{"a": 1, "b": 2}` | `'(:a 1 :b 2)` |
| **List of Named Objects** | `[{"_type": "x", "a": 1}]` | `'((x :a 1))` |
| **List of Anonymous Objects** | `[{"a": 1}, {"a": 2}]` | `'('(:a 1) '(:a 2))` |
| **RPC Call** | `{"method": "getUser", "params": {"id": 1}}` | `(get-user :id 1)` |

---

## 7. Formatting & Syntax Checklist for LLM Generation

When generating or validating Lisp-RPC Data:
1. **Colons for Keys**: Every key in a Data or Map structure MUST start with a colon `:` (e.g. `:title`, `:user-id`).
2. **Kebab-case**: Use `kebab-case` for method names, data names, and keyword parameters (e.g. `get-user-info`, `:created-at`).
3. **Leading Single Quotes**:
   - Lists MUST start with `'` (e.g. `'("a" "b")`).
   - Anonymous Maps MUST start with `'` (e.g. `'(:key "value")`).
   - **Maps inside a List** MUST each have their own quote `'` (e.g. `'('(:a 1) '(:b 2))`).
   - Symbols/Enums MUST start with `'` (e.g. `'active`).
4. **No Quotes on Named Data**: Do NOT put `'` in front of named structures, even when inside lists (use `(user :id 1)` or `'((user :id 1))`).
5. **Booleans**: Use `T` for true and `NIL` (or `nil`) for false / absent.
6. **Key-Value Pairs**: Arguments after the structure name must always be paired (`:key value :key value`).
