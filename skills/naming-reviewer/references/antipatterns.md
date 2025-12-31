# Common Naming Antipatterns

This document catalogs common naming mistakes and how to fix them.

## 1. Generic Verbs

### The Problem
Using vague verbs that don't convey specific meaning.

**Antipattern:**
```python
def process(data):
    ...

def handle(request):
    ...

def manage(users):
    ...

def do_thing():
    ...
```

**Fix:**
```python
def parseJsonData(data):
    ...

def authenticateRequest(request):
    ...

def deactivateExpiredUsers(users):
    ...

def calculateMonthlyRevenue():
    ...
```

**Why:** Generic verbs force readers to look at implementation to understand purpose.

---

## 2. Redundant Context

### The Problem
Repeating information already clear from the enclosing context.

**Antipattern:**
```python
class User:
    def saveUser(self):
        ...

    def getUserName(self):
        return self.userName

class Order:
    order_id: int
    order_total: float
    order_items: list
```

**Fix:**
```python
class User:
    def save(self):
        ...

    def getName(self):
        return self.name

class Order:
    id: int
    total: float
    items: list
```

**Why:** The class name already provides context. Repeating it is noise.

---

## 3. Hungarian Notation and Type Encoding

### The Problem
Encoding type information in variable names.

**Antipattern:**
```python
strUserName = "John"
intAge = 25
lstUsers = []
dictConfig = {}
bIsActive = True
```

**Fix:**
```python
userName = "John"
age = 25
users = []
config = {}
isActive = True
```

**Why:** Modern IDEs show type information. Encoding it in names is redundant and brittle.

---

## 4. Abbreviation Soup

### The Problem
Using non-standard abbreviations that hurt readability.

**Antipattern:**
```python
usrMgr = UserManager()
msgSvc = MessageService()
btnClk = handleButtonClick()
numItms = 5
```

**Fix:**
```python
userManager = UserManager()
messageService = MessageService()
handleButtonClick = handleButtonClick()
itemCount = 5
```

**Why:** Abbreviations save a few characters but cost significant readability.

---

## 5. Single-Letter Variables in Large Scopes

### The Problem
Using single letters for long-lived or important variables.

**Antipattern:**
```python
def processOrder(o):
    # 50 lines of code...
    if o.total > 100:
        applyDiscount(o)

    for i in o.items:
        # 20 lines of code...
        validateItem(i)
```

**Fix:**
```python
def processOrder(order):
    # 50 lines of code...
    if order.total > 100:
        applyDiscount(order)

    for item in order.items:
        # 20 lines of code...
        validateItem(item)
```

**Why:** Single letters are fine for tiny scopes (2-3 lines), but become cryptic in larger contexts.

---

## 6. Negative Booleans

### The Problem
Using negative phrasing for boolean variables.

**Antipattern:**
```python
isNotValid = True
notActive = False
disabled = True

if not isNotValid:  # Double negative!
    ...

if not disabled:
    ...
```

**Fix:**
```python
isValid = False
isActive = True
isEnabled = False

if isValid:
    ...

if isEnabled:
    ...
```

**Why:** Negative booleans create confusing double negatives when used with `if not`.

---

## 7. Overly Generic Names

### The Problem
Names so generic they convey no meaning.

**Antipattern:**
```python
data = getData()
info = getInfo()
temp = calculate()
result = process()
manager = Manager()
helper = Helper()
```

**Fix:**
```python
userProfiles = getUserProfiles()
customerInfo = getCustomerContactInfo()
monthlyTotal = calculateMonthlyTotal()
validationResult = validateUserInput()
paymentProcessor = PaymentProcessor()
emailFormatter = EmailFormatter()
```

**Why:** Generic names provide zero information about purpose or content.

---

## 8. Magic Numbers and Strings

### The Problem
Using literal values without explanation.

**Antipattern:**
```python
if user.age >= 18:
    ...

if status == "ACTIVE":
    ...

for i in range(5):  # Why 5?
    retry()
```

**Fix:**
```python
LEGAL_AGE = 18
STATUS_ACTIVE = "ACTIVE"
MAX_RETRY_ATTEMPTS = 5

if user.age >= LEGAL_AGE:
    ...

if status == STATUS_ACTIVE:
    ...

for attempt in range(MAX_RETRY_ATTEMPTS):
    retry()
```

**Why:** Named constants self-document the meaning and make values easy to change.

---

## 9. Inconsistent Vocabulary

### The Problem
Using different words for the same concept throughout the codebase.

**Antipattern:**
```python
getUser()
fetchCustomer()
retrieveOrder()

deleteUser()
removeCustomer()
destroyOrder()
```

**Fix:**
```python
# Choose one verb per concept
getUser()
getCustomer()
getOrder()

deleteUser()
deleteCustomer()
deleteOrder()
```

**Why:** Consistency reduces cognitive load and makes patterns obvious.

---

## 10. Unclear Collection Names

### The Problem
Not clearly indicating plural/collection nature.

**Antipattern:**
```python
user = [user1, user2, user3]  # Singular name for plural data
userList = [...]  # Type encoding
```

**Fix:**
```python
users = [user1, user2, user3]
activeUsers = [...]
```

**Why:** Plural names make it immediately clear that multiple items are involved.

---

## 11. Getters/Setters Misuse

### The Problem
Using get/set for operations that do more than simple access.

**Antipattern:**
```python
def getUser(id):
    # Makes API call, validates, transforms...
    user = api.fetch(id)
    validate(user)
    return transform(user)

def setConfig(config):
    # Validates, persists to DB, notifies...
    validate(config)
    db.save(config)
    notify_watchers(config)
```

**Fix:**
```python
def fetchAndValidateUser(id):
    user = api.fetch(id)
    validate(user)
    return transform(user)

def updateAndPersistConfig(config):
    validate(config)
    db.save(config)
    notify_watchers(config)
```

**Why:** `get`/`set` implies simple, cheap operations. Complex operations deserve descriptive names.

---

## 12. Vague Function Parameters

### The Problem
Parameter names that don't describe what's expected.

**Antipattern:**
```python
def sendEmail(to, from, data, flag):
    ...

def processPayment(amount, type, info):
    ...
```

**Fix:**
```python
def sendEmail(recipientEmail, senderEmail, messageBody, includeAttachment):
    ...

def processPayment(amountInCents, paymentMethod, billingAddress):
    ...
```

**Why:** Clear parameter names serve as inline documentation.

---

## 13. Overly Long Names

### The Problem
Names that are unnecessarily verbose.

**Antipattern:**
```python
def getTheListOfAllActiveUserAccountsFromTheDatabase():
    ...

thisIsTheVariableThatStoresTheCurrentUserSessionInformation = {}
```

**Fix:**
```python
def getActiveUsers():
    ...

currentUserSession = {}
```

**Why:** Be descriptive, but concise. Remove words that don't add information.

---

## 14. Class Name Suffixes Overuse

### The Problem
Adding unnecessary "Manager", "Handler", "Helper" suffixes.

**Antipattern:**
```python
class UserManager:
    def manageUser(): ...

class EmailHandler:
    def handleEmail(): ...

class StringHelper:
    def helpWithStrings(): ...
```

**Fix:**
```python
class UserService:  # Or UserRepository, UserValidator - be specific
    def createUser(): ...

class EmailSender:  # Describes what it does
    def send(): ...

class StringFormatter:  # Describes the role
    def format(): ...
```

**Why:** Suffixes like Manager/Handler/Helper are too generic. Be specific about the role.

---

## 15. Unclear Boolean Prefixes

### The Problem
Not using standard boolean prefixes.

**Antipattern:**
```python
valid = True  # Reads like a command
active = False  # Ambiguous
allowed = True  # Could be noun or adjective
```

**Fix:**
```python
isValid = True
isActive = False
isAllowed = True
# Or
hasPermission = True
canEdit = False
shouldRetry = True
```

**Why:** Prefixes make boolean nature clear and read naturally in conditionals.

---

## Detection Checklist

When reviewing code, watch for these red flags:

- ❌ Functions named `process`, `handle`, `manage`, `do`
- ❌ Variables with type prefixes: `str`, `int`, `lst`, `dict`
- ❌ Single-letter variables beyond tiny scopes
- ❌ Abbreviations: `usr`, `msg`, `btn`, `num`
- ❌ Class/attribute name repetition: `user.userName`
- ❌ Generic names: `data`, `info`, `temp`, `result`
- ❌ Magic numbers and strings
- ❌ Negative booleans: `notValid`, `disabled`
- ❌ Inconsistent vocabulary across similar operations
- ❌ Overly long names (>4 words)
- ❌ Missing plural indicators for collections
