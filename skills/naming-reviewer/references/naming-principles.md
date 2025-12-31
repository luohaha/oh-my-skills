# Master-Level Naming Principles

## Core Philosophy

**Names are the primary documentation of code.** A well-chosen name eliminates the need for comments and makes code self-explanatory. Names should reveal intent, not implementation.

## Universal Naming Principles

### 1. Reveal Intent, Not Implementation

**Good:**
```
getUsersWithActiveSubscriptions()
activeSubscribers
findCustomersByLocation()
```

**Bad:**
```
getData()
list1
queryDB()
```

**Principle:** The name should tell WHY something exists, not HOW it works.

### 2. Use Searchable, Pronounceable Names

**Good:**
```
MAX_RETRY_ATTEMPTS = 3
userRegistrationDate
elapsedTimeInDays
```

**Bad:**
```
mra = 3
usrregd
etd
```

**Principle:** Names should be easy to discuss in conversation and find with search tools.

### 3. Avoid Mental Mapping

**Good:**
```
for user in users:
for product in products:
```

**Bad:**
```
for i in users:
for x in products:
```

**Exception:** Single-letter variables are acceptable in very short scopes (< 5 lines) for standard conventions like `i` for index.

### 4. Choose Specific Over Generic

**Good:**
```
calculateMonthlyRevenue()
parseJsonConfiguration()
validateEmailFormat()
```

**Bad:**
```
process()
handle()
doStuff()
```

**Principle:** Generic verbs like "process", "handle", "manage" are red flags. Be specific about the action.

### 5. Avoid Encodings and Prefixes

**Good:**
```
Customer (class)
email (variable)
EmailValidator (class)
```

**Bad:**
```
CCustomer (Hungarian notation)
m_email (member prefix)
IEmailValidator (interface prefix in languages that don't require it)
```

**Exception:** Prefixes may be acceptable in languages with established conventions (e.g., `I` for interfaces in C#).

### 6. Context-Aware Naming

**Good:**
```
class User:
    def save():  # Not saveUser(), context is clear
    def activate():

order.totalPrice  # Not order.orderTotalPrice
```

**Bad:**
```
class User:
    def saveUser():  # Redundant

order.orderTotalPrice  # Redundant "order"
```

**Principle:** Don't repeat context that's already clear from the enclosing scope.

### 7. Consistency in Vocabulary

**Good:** Pick one word per concept and stick to it
- `get` for all retrieval operations
- `create` for all creation operations
- `update` for all modification operations

**Bad:** Using different words for the same concept
- `get`, `fetch`, `retrieve` mixed throughout
- `create`, `make`, `build` mixed throughout

## Language-Specific Conventions

### Python
- `snake_case` for functions and variables
- `PascalCase` for classes
- `UPPER_SNAKE_CASE` for constants
- `_leading_underscore` for internal/private members
- Avoid `__double_leading` except for name mangling

### JavaScript/TypeScript
- `camelCase` for functions and variables
- `PascalCase` for classes and React components
- `UPPER_SNAKE_CASE` for constants
- `_leadingUnderscore` or `#private` for private members

### Java
- `camelCase` for methods and variables
- `PascalCase` for classes and interfaces
- `UPPER_SNAKE_CASE` for constants
- Avoid prefixes like `I` for interfaces

### Go
- `MixedCaps` or `mixedCaps` (case determines visibility)
- Avoid underscores
- Short, concise names in small scopes
- Longer, descriptive names in larger scopes

## Scope-Length Trade-off

**Short scope → Short names acceptable:**
```python
for i in range(10):
    print(i)

with open(file) as f:
    data = f.read()
```

**Long scope → Descriptive names required:**
```python
class CustomerOrderProcessor:
    def processCustomerOrder(self, customerOrder):
        # Long-lived variable needs descriptive name
        totalOrderAmount = 0
```

## Boolean Naming

**Use affirmative names with prefixes:**
- `is`, `has`, `can`, `should`, `will`

**Good:**
```
isActive
hasPermission
canEdit
shouldRetry
willExpire
```

**Bad:**
```
notInactive  # Double negative
disabled  # Negative phrasing
flag  # Not descriptive
```

## Function/Method Naming

### Commands (perform actions)
Use verb phrases:
- `calculateTotal()`
- `sendEmail()`
- `validateInput()`

### Queries (return information)
Use noun phrases or questions:
- `getUser()`
- `isValid()`
- `hasPermission()`

### Avoid ambiguous verbs
**Bad:** `process()`, `handle()`, `manage()`, `do()`
**Good:** `parseJson()`, `validateEmail()`, `formatDate()`

## Class Naming

**Good:**
- Nouns: `Customer`, `Order`, `EmailService`
- Descriptive roles: `PaymentProcessor`, `UserAuthenticator`, `DataValidator`

**Bad:**
- Verbs: `ProcessPayment` (use `PaymentProcessor`)
- Generic: `Manager`, `Handler`, `Helper` (be more specific)
- Abbreviations: `CustMgr`, `UsrProc`

## Variable Naming

### Collections
Use plural nouns:
```
users = []
activeOrders = []
emailAddresses = set()
```

### Single items
Use singular nouns:
```
user = getUser()
order = findOrder()
```

### Numbers/Counts
Use clear prefixes:
```
totalUsers
userCount
maxRetries
averageScore
```

## Magic Numbers and Strings

**Never use magic values directly:**

**Bad:**
```python
if user.age > 18:
if status == "ACTIVE":
```

**Good:**
```python
LEGAL_AGE = 18
STATUS_ACTIVE = "ACTIVE"

if user.age > LEGAL_AGE:
if status == STATUS_ACTIVE:
```

## Abbreviations

**General rule: Avoid abbreviations unless they're universally understood**

**Acceptable:**
- `id` (identifier)
- `url` (uniform resource locator)
- `html` (hypertext markup language)
- `http` (hypertext transfer protocol)

**Avoid:**
- `usr` → use `user`
- `msg` → use `message`
- `btn` → use `button`
- `num` → use `number` or `count`

## Summary Checklist

Before finalizing any name, ask:

1. ✅ Does it reveal intent without needing a comment?
2. ✅ Can I pronounce it in conversation?
3. ✅ Can I search for it easily?
4. ✅ Is it specific rather than generic?
5. ✅ Does it follow the project's language conventions?
6. ✅ Is it consistent with similar concepts in the codebase?
7. ✅ Does it avoid unnecessary prefixes/encodings?
8. ✅ Is the length appropriate for its scope?
