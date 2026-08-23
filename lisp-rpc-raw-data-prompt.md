# Lisp-RPC Raw Data Prompt for LLMs

You are an expert in Lisp and Remote Procedure Call (RPC) systems. Your task is to understand, parse, generate, and validate data formatted in **Lisp-RPC Raw Data (Plain Mode)**.

---

## 1. Overview
Lisp-RPC Raw Data is a lightweight, human-readable S-expression data interchange format. It serves a similar role to JSON / JSON-RPC, but uses native Lisp S-expression syntax for concise nesting without syntax boilerplate.

In Raw Data mode (Plain Mode), there are no schema definitions required. All messages and values are exchanged directly as S-expressions.

---

## 2. Data Types

### Scalar / Primary Types
| Type | Syntax | Examples | Notes |
| :--- | :--- | :--- | :--- |
| **String** | `"..."` | `"hello"`, `"user@example.com"` | Standard double-quoted string |
| **Integer** | `[0-9]+` | `42`, `-10`, `0` | Standard integer numbers |
| **Float** | `[0-9]+\.[0-9]+` | `3.14`, `-0.05`, `12.34` | Floating-point numbers |
| **Boolean** | `T` / `NIL` | `T` (true), `NIL` or `nil` (false) | Case-insensitive; `NIL` also represents null / empty |
| **Keyword** | `:name` | `:id`, `:user-name`, `:status` | Identifier prefixed with `:` (used as keys) |
| **Symbol** | `'symbol` | `'admin`, `'active`, `'low` | Quoted symbol (commonly used for enum variants) |

### Complex / Structural Types

#### 1. Data (Named Structure / RPC Message)
A named structure containing alternating keyword-value pairs.
- **Syntax**: `(name :key1 value1 :key2 value2 ...)`
- **Quoting**: **Do NOT quote** named Data structures (`(name ...)` not `'(name ...)`).
- **Empty structure**: `(name)`
- **Example**: `(user :id 1 :name "Alice" :active T)`

#### 2. Map (Anonymous Key-Value Collection / Dictionary)
An anonymous collection of keyword-value pairs (similar to a JSON object without a type name).
- **Syntax**: `'(:key1 value1 :key2 value2 ...)`
- **Quoting**: **MUST BE QUOTED** with a leading single quote `'`.
- **Example**: `'(:email "alice@example.com" :verified T)`

#### 3. List (Array / Sequence)
An ordered sequence of items.
- **Syntax**: `'(item1 item2 item3 ...)`
- **Quoting**: **MUST BE QUOTED** with a leading single quote `'`.
- **Empty List**: `'()` or `NIL` / `nil`.
- **Example**: `'("admin" "staff" "editor")`, `'(1 2 3 4)`

---

## 3. Quoting Rules (Crucial)

| Construct | Form | Quoting Rule | Example |
| :--- | :--- | :--- | :--- |
| **Named Data** | `(name ...)` | **Never quoted** | `(book :title "Dune")` |
| **Anonymous Map** | `'(:key ...)` | **Always quoted** (`'`) | `'(:author "Frank Herbert" :pages 412)` |
| **List / Array** | `'(...)` | **Always quoted** (`'`) | `'("sci-fi" "novel")` |
| **Symbol / Enum** | `'sym` | **Always quoted** (`'`) | `'in-stock` |
| **Primitives** | `"str"`, `123` | **Never quoted** | `"hello"`, `42`, `T` |

---

## 4. RPC Conventions

### RPC Request Format
A request is a root named **Data** structure where the name represents the method/procedure to call:
```lisp
(method-name :param1 value1 :param2 value2 ...)
```

### RPC Response Format
A response is a root named **Data** structure containing the result and status:
```lisp
(response-name :result value :status "success")
```

---

## 5. Nesting Rules

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

### Nested List of Structures
```lisp
(batch-update :items '((item :id 1 :qty 5) (item :id 2 :qty 10)))
```

### Deeply Nested Example
```lisp
(order-placed
  :order-id 9981
  :customer '(:name "John Doe" :vip T)
  :items '((item :sku "SKU-1" :price 19.99 :tags '("sale" "clearance"))
           (item :sku "SKU-2" :price 5.50 :tags '("accessories")))
  :status 'confirmed)
```

---

## 6. Comparison: JSON vs Lisp-RPC Raw Data

| Concept | JSON | Lisp-RPC Raw Data |
| :--- | :--- | :--- |
| **String** | `"text"` | `"text"` |
| **Integer / Float** | `42` / `3.14` | `42` / `3.14` |
| **Boolean / Null** | `true`, `false`, `null` | `T`, `NIL`, `NIL` (or `nil`) |
| **List / Array** | `["a", "b"]` | `'("a" "b")` |
| **Named Object** | `{"_type": "user", "id": 1}` | `(user :id 1)` |
| **Anonymous Object** | `{"a": 1, "b": 2}` | `'(:a 1 :b 2)` |
| **RPC Call** | `{"method": "getUser", "params": {"id": 1}}` | `(get-user :id 1)` |

---

## 7. Formatting & Syntax Checklist for LLM Generation

When generating or validating Lisp-RPC Raw Data:
1. **Colons for Keys**: Every key in a Data or Map structure MUST start with a colon `:` (e.g. `:title`, `:user-id`).
2. **Kebab-case**: Use `kebab-case` for method names, data names, and keyword parameters (e.g. `get-user-info`, `:created-at`).
3. **Leading Single Quotes**:
   - Lists MUST start with `'` (e.g. `'("a" "b")`).
   - Anonymous Maps MUST start with `'` (e.g. `'(:key "value")`).
   - Symbols/Enums MUST start with `'` (e.g. `'active`).
4. **No Quotes on Named Data**: Do NOT put `'` in front of named structures (use `(user :id 1)`, NOT `'(user :id 1)`).
5. **Booleans**: Use `T` for true and `NIL` (or `nil`) for false / absent.
6. **Key-Value Pairs**: Arguments after the structure name must always be paired (`:key value :key value`).
