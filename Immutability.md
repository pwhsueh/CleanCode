## 十四、不可變性與型別安全 (Immutability & Type Safety)

> 「可變狀態是萬惡之源。」— 函數式編程社群的共識

### ❓ 為什麼不可變性與型別安全如此重要？

在現代軟體開發中，**不可變性 (Immutability)** 和**型別安全 (Type Safety)** 是提升程式碼品質的兩大支柱。它們能顯著降低 Bug、提升可維護性，並讓程式碼更容易推理。

---

### 一、不可變性 (Immutability)

#### 📖 什麼是不可變性？

**不可變 (Immutable)** 意味著一個物件或資料結構一旦被建立，就不能再被修改。任何「修改」操作都會產生一個新的物件，而不是改變原有的物件。

**可變 vs 不可變**：

```typescript
// ❌ 可變 (Mutable)：直接修改原物件
const user = { name: "Alice", age: 25 };
user.age = 26;  // 原物件被改變了
console.log(user);  // { name: "Alice", age: 26 }

// ✅ 不可變 (Immutable)：產生新物件
const user = { name: "Alice", age: 25 };
const updatedUser = { ...user, age: 26 };  // 建立新物件
console.log(user);         // { name: "Alice", age: 25 } (原物件不變)
console.log(updatedUser);  // { name: "Alice", age: 26 } (新物件)
```

---

#### 💔 可變性帶來的問題

##### **1. 難以追蹤狀態變化**

當物件可以在任何地方被修改，你很難知道它在程式執行過程中經歷了什麼變化。

```typescript
// ❌ 可變物件導致的混亂
function processOrder(order: Order) {
  validateOrder(order);      // 會不會修改 order？
  calculateDiscount(order);  // 會不會修改 order？
  applyTax(order);           // 會不會修改 order？
  saveOrder(order);          // 現在的 order 是什麼狀態？
}

// 每個函式都可能偷偷修改 order，導致難以追蹤
function calculateDiscount(order: Order) {
  order.total = order.total * 0.9;  // 修改了原物件！
  order.discountApplied = true;     // 又修改了！
}
```

##### **2. 併發問題 (Race Conditions)**

在多執行緒或非同步環境中，可變物件可能被多個操作同時修改，導致資料不一致。

```python
# ❌ 可變物件在併發環境中的危險
class BankAccount:
    def __init__(self, balance):
        self.balance = balance
    
    def withdraw(self, amount):
        if self.balance >= amount:
            # 假設這裡有延遲...
            time.sleep(0.1)
            self.balance -= amount  # 另一個執行緒可能也在修改 balance
            return True
        return False

# 兩個執行緒同時提款，可能導致餘額變成負數！
```

##### **3. 意外的副作用**

當你把物件傳給一個函式，你無法確定函式是否會修改它。

```csharp
// ❌ 意外修改了傳入的物件
public void ProcessUsers(List<User> users)
{
    // 呼叫者可能沒預期到 users 會被修改
    users.RemoveAll(u => u.IsInactive);
    users.Sort((a, b) => a.Name.CompareTo(b.Name));
}

// 呼叫端
var myUsers = GetUsers();
ProcessUsers(myUsers);  // myUsers 被修改了！原本的資料不見了
```

##### **4. 難以測試與除錯**

可變物件的狀態可能在任何時候被任何程式碼修改，導致測試結果不穩定。

---

#### ✅ 不可變性的優點

##### **1. 易於推理 (Easier to Reason About)**

不可變物件的狀態永遠不會改變，你可以安心地在程式各處傳遞它，不用擔心被修改。

```typescript
// ✅ 不可變物件：清晰易懂
function processOrder(order: Order): Order {
  const validated = validateOrder(order);      // 回傳新物件
  const discounted = applyDiscount(validated); // 回傳新物件
  const taxed = applyTax(discounted);          // 回傳新物件
  return taxed;  // 原始的 order 完全沒變
}
```

##### **2. 執行緒安全 (Thread-Safe)**

不可變物件天生就是執行緒安全的，因為沒有人能修改它。

```python
# ✅ 不可變物件在併發環境中安全
from dataclasses import dataclass

@dataclass(frozen=True)
class BankAccount:
    balance: float
    
    def withdraw(self, amount: float) -> 'BankAccount':
        if self.balance >= amount:
            # 回傳新物件，不修改原物件
            return BankAccount(balance=self.balance - amount)
        raise ValueError("Insufficient funds")

# 多個執行緒操作時，每個都會得到新的物件，不會互相干擾
```

##### **3. 時間旅行與歷史追蹤**

不可變資料結構讓你可以輕鬆保存歷史狀態（如 Undo/Redo 功能）。

```typescript
// ✅ 使用不可變性實現歷史記錄
class DocumentEditor {
  private history: Document[] = [];
  private currentIndex = -1;

  applyChange(doc: Document, change: Change): Document {
    const newDoc = { ...doc, content: change.newContent };
    this.history.push(newDoc);
    this.currentIndex++;
    return newDoc;
  }

  undo(): Document | null {
    if (this.currentIndex > 0) {
      this.currentIndex--;
      return this.history[this.currentIndex];
    }
    return null;
  }

  redo(): Document | null {
    if (this.currentIndex < this.history.length - 1) {
      this.currentIndex++;
      return this.history[this.currentIndex];
    }
    return null;
  }
}
```

##### **4. 記憶化與快取 (Memoization)**

不可變物件的結果可以安全地快取，因為它永遠不會變。

```typescript
// ✅ 不可變物件讓快取變得簡單
const cache = new Map<string, ExpensiveResult>();

function expensiveOperation(data: ImmutableData): ExpensiveResult {
  const key = JSON.stringify(data);
  
  if (cache.has(key)) {
    return cache.get(key)!;  // 安全地回傳快取結果
  }
  
  const result = doExpensiveCalculation(data);
  cache.set(key, result);
  return result;
}
```

---

#### 🛠️ 各語言的不可變性實作

##### **TypeScript / JavaScript**

**1. 使用展開運算子 (Spread Operator)**

```typescript
// ✅ 物件的不可變更新
const user = { name: "Alice", age: 25, email: "alice@example.com" };
const updatedUser = { ...user, age: 26 };  // 只改 age

// ✅ 陣列的不可變操作
const numbers = [1, 2, 3];
const newNumbers = [...numbers, 4];  // 添加元素
const filtered = numbers.filter(n => n > 1);  // 過濾
const mapped = numbers.map(n => n * 2);  // 轉換
```

**2. 使用 Object.freeze()**

```typescript
// ✅ 凍結物件，防止修改
const config = Object.freeze({
  apiUrl: "https://api.example.com",
  timeout: 5000
});

config.timeout = 3000;  // 在嚴格模式下會拋錯，否則靜默失敗
console.log(config.timeout);  // 5000 (沒有被修改)
```

**3. 使用 Immer 函式庫（推薦）**

Immer 讓你用「可變」的語法寫出「不可變」的邏輯。

```typescript
import produce from 'immer';

// ✅ 看起來像可變，實際上是不可變
const baseState = {
  users: [
    { id: 1, name: "Alice" },
    { id: 2, name: "Bob" }
  ],
  settings: { theme: "dark" }
};

const nextState = produce(baseState, draft => {
  // draft 是一個代理物件，可以直接修改
  draft.users.push({ id: 3, name: "Charlie" });
  draft.settings.theme = "light";
});

console.log(baseState.users.length);  // 2 (原物件不變)
console.log(nextState.users.length);  // 3 (新物件)
```

**4. TypeScript 的 readonly**

```typescript
// ✅ 使用 readonly 強制不可變
interface User {
  readonly id: string;
  readonly name: string;
  email: string;  // 可變
}

const user: User = { id: "1", name: "Alice", email: "alice@example.com" };
// user.id = "2";  // ❌ 編譯錯誤：無法修改 readonly 屬性
user.email = "new@example.com";  // ✅ 可以修改

// ✅ 深層 readonly
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

interface NestedData {
  user: {
    name: string;
    settings: {
      theme: string;
    };
  };
}

const data: DeepReadonly<NestedData> = {
  user: {
    name: "Alice",
    settings: { theme: "dark" }
  }
};

// data.user.settings.theme = "light";  // ❌ 編譯錯誤
```

**5. 使用 as const**

```typescript
// ✅ 字面值的完全不可變
const config = {
  apiUrl: "https://api.example.com",
  retries: 3,
  endpoints: ["users", "orders"]
} as const;

// config.retries = 5;  // ❌ 編譯錯誤
// config.endpoints.push("products");  // ❌ 編譯錯誤

// 型別會變成：
// {
//   readonly apiUrl: "https://api.example.com";
//   readonly retries: 3;
//   readonly endpoints: readonly ["users", "orders"];
// }
```

---

##### **C#**

**1. 使用 record 類型（C# 9+）**

```csharp
// ✅ record 預設就是不可變的
public record User(string Name, string Email);

var user = new User("Alice", "alice@example.com");
// user.Name = "Bob";  // ❌ 編譯錯誤：init-only 屬性

// ✅ 使用 with 建立修改過的副本
var updatedUser = user with { Name = "Bob" };
Console.WriteLine(user.Name);         // Alice (原物件不變)
Console.WriteLine(updatedUser.Name);  // Bob (新物件)
```

**2. 使用 readonly 欄位與屬性**

```csharp
// ✅ 不可變類別
public class ImmutableUser
{
    public string Name { get; }
    public string Email { get; }

    public ImmutableUser(string name, string email)
    {
        Name = name;
        Email = email;
    }

    // 提供方法建立修改後的副本
    public ImmutableUser WithName(string newName)
    {
        return new ImmutableUser(newName, Email);
    }
}
```

**3. 使用 ImmutableCollections**

```csharp
using System.Collections.Immutable;

// ✅ 不可變集合
ImmutableList<string> names = ImmutableList.Create("Alice", "Bob");
ImmutableList<string> newNames = names.Add("Charlie");  // 回傳新集合

Console.WriteLine(names.Count);     // 2 (原集合不變)
Console.WriteLine(newNames.Count);  // 3 (新集合)

// ✅ 不可變字典
ImmutableDictionary<string, int> ages = ImmutableDictionary<string, int>.Empty
    .Add("Alice", 25)
    .Add("Bob", 30);

var updatedAges = ages.SetItem("Alice", 26);  // 回傳新字典
```

**4. init-only 屬性（C# 9+）**

```csharp
// ✅ 允許在初始化時設定，之後不可變
public class Configuration
{
    public string ApiUrl { get; init; }
    public int Timeout { get; init; }
}

var config = new Configuration 
{ 
    ApiUrl = "https://api.example.com",
    Timeout = 5000 
};

// config.Timeout = 3000;  // ❌ 編譯錯誤
```

**5. 結構紀錄 (struct record，C# 10+)**

```csharp
// ✅ 不可變的值型別（效能更好）
public readonly record struct Point(int X, int Y);

var p1 = new Point(10, 20);
var p2 = p1 with { X = 15 };  // 建立新副本
```

---

##### **Python**

**1. 使用 dataclass 的 frozen 參數**

```python
from dataclasses import dataclass

# ✅ 不可變資料類別
@dataclass(frozen=True)
class User:
    name: str
    email: str

user = User(name="Alice", email="alice@example.com")
# user.name = "Bob"  # ❌ 執行時錯誤：FrozenInstanceError

# ✅ 建立修改後的副本
from dataclasses import replace

updated_user = replace(user, name="Bob")
print(user.name)         # Alice (原物件不變)
print(updated_user.name) # Bob (新物件)
```

**2. 使用 NamedTuple**

```python
from typing import NamedTuple

# ✅ 不可變的具名元組
class Point(NamedTuple):
    x: int
    y: int

point = Point(10, 20)
# point.x = 15  # ❌ 屬性錯誤：無法設定屬性

# ✅ 使用 _replace 建立新物件
new_point = point._replace(x=15)
```

**3. 使用 tuple 和 frozenset**

```python
# ✅ 不可變的序列
immutable_list = (1, 2, 3)
# immutable_list[0] = 5  # ❌ 型別錯誤

# ✅ 不可變的集合
immutable_set = frozenset([1, 2, 3])
# immutable_set.add(4)  # ❌ 屬性錯誤：frozenset 沒有 add 方法
```

**4. 使用 attrs 函式庫（推薦）**

```python
from attrs import define, frozen

# ✅ 使用 attrs 建立不可變類別
@frozen
class User:
    name: str
    email: str
    age: int = 0

user = User(name="Alice", email="alice@example.com", age=25)
# user.age = 26  # ❌ 執行時錯誤

# ✅ 使用 evolve 建立修改後的副本
from attrs import evolve

updated_user = evolve(user, age=26)
```

**5. 使用 Pydantic (數據驗證 + 不可變)**

```python
from pydantic import BaseModel, Field

# ✅ 不可變且帶驗證的模型
class User(BaseModel):
    name: str
    email: str
    age: int = Field(ge=0, le=150)

    class Config:
        frozen = True  # 啟用不可變性

user = User(name="Alice", email="alice@example.com", age=25)
# user.age = 26  # ❌ 驗證錯誤

# ✅ 使用 copy 建立修改後的副本
updated_user = user.copy(update={"age": 26})
```

---

### 二、型別安全 (Type Safety)

#### 📖 什麼是型別安全？

**型別安全**是指在編譯時期（或執行時期）檢查變數、函式參數、回傳值的型別是否正確，防止型別錯誤導致的 Bug。

**動態型別 vs 靜態型別**：

```python
# Python (動態型別)：執行時才發現錯誤
def add(a, b):
    return a + b

result = add(5, "hello")  # 執行時錯誤：unsupported operand type(s)
```

```typescript
// TypeScript (靜態型別)：編譯時就發現錯誤
function add(a: number, b: number): number {
    return a + b;
}

const result = add(5, "hello");  // ❌ 編譯錯誤：Argument of type 'string' is not assignable to parameter of type 'number'
```

---

#### 💔 缺乏型別安全的問題

##### **1. 執行時錯誤**

```javascript
// ❌ JavaScript：沒有型別檢查
function calculateDiscount(price, discountRate) {
    return price * discountRate;
}

// 使用者傳錯參數順序
const result = calculateDiscount(0.1, 100);  // 10，而不是預期的 90
```

##### **2. 難以重構**

```python
# ❌ 重新命名屬性時，容易遺漏
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

def print_user_name(user):
    print(user.name)  # 如果 User.name 被改成 User.full_name，這裡不會報錯

# 執行時才發現錯誤
```

##### **3. API 使用錯誤**

```javascript
// ❌ 不知道函式期望什麼參數
fetchUser(userId, true, "admin", { cache: false });  
// 第2個參數是什麼？第3個參數是什麼？第4個參數有哪些選項？
```

---

#### ✅ 型別安全的優點

##### **1. 提早發現錯誤**

```typescript
// ✅ TypeScript 在編譯時就發現問題
interface User {
    id: string;
    name: string;
    email: string;
}

function deleteUser(userId: number) {  // 期望 number
    // ...
}

const user: User = { id: "123", name: "Alice", email: "alice@example.com" };
deleteUser(user.id);  // ❌ 編譯錯誤：string 不能賦值給 number
```

##### **2. 自動補全與文件**

```typescript
// ✅ IDE 可以提供自動補全
interface ApiResponse<T> {
    data: T;
    status: number;
    message: string;
}

function handleResponse(response: ApiResponse<User>) {
    response.  // IDE 會自動顯示 data, status, message
    response.data.  // IDE 會自動顯示 User 的屬性
}
```

##### **3. 安全的重構**

```csharp
// ✅ C# 重新命名時，編譯器會找出所有使用處
public class User
{
    public string FullName { get; set; }  // 從 Name 改成 FullName
}

void PrintUserName(User user)
{
    Console.WriteLine(user.Name);  // ❌ 編譯錯誤：User 沒有 Name 屬性
}
```

##### **4. 明確的契約**

```typescript
// ✅ 清楚的函式簽章
interface FetchOptions {
    cache?: boolean;
    timeout?: number;
    retries?: number;
}

function fetchUser(
    userId: string,
    options?: FetchOptions
): Promise<User> {
    // 實作
}

// 使用時一目了然
fetchUser("123", { cache: true, timeout: 5000 });
```

---

#### 🛠️ 各語言的型別安全實作

##### **TypeScript**

**1. 使用嚴格模式**

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,              // 啟用所有嚴格檢查
    "noImplicitAny": true,       // 禁止隱式 any
    "strictNullChecks": true,    // 嚴格的 null 檢查
    "strictFunctionTypes": true, // 嚴格的函式型別檢查
    "noUnusedLocals": true,      // 禁止未使用的變數
    "noUnusedParameters": true   // 禁止未使用的參數
  }
}
```

**2. 使用聯合型別與類型守衛**

```typescript
// ✅ 聯合型別
type Result<T> = 
  | { success: true; data: T }
  | { success: false; error: string };

function processResult<T>(result: Result<T>): void {
  if (result.success) {
    console.log(result.data);   // TypeScript 知道這裡有 data
  } else {
    console.error(result.error); // TypeScript 知道這裡有 error
  }
}
```

**3. 使用泛型**

```typescript
// ✅ 型別安全的泛型函式
function firstElement<T>(arr: T[]): T | undefined {
    return arr[0];
}

const numbers = [1, 2, 3];
const first = firstElement(numbers);  // 型別是 number | undefined

const strings = ["a", "b", "c"];
const firstStr = firstElement(strings);  // 型別是 string | undefined
```

**4. 使用型別斷言的最佳實踐**

```typescript
// ❌ 過度使用 as（危險）
const user = getUser() as User;  // 繞過型別檢查

// ✅ 使用型別守衛
function isUser(obj: any): obj is User {
    return obj && typeof obj.name === 'string' && typeof obj.email === 'string';
}

const data = getUser();
if (isUser(data)) {
    console.log(data.name);  // 型別安全
}
```

---

##### **C#**

**1. 啟用 Nullable Reference Types（C# 8+）**

```csharp
// ✅ csproj 啟用
<PropertyGroup>
  <Nullable>enable</Nullable>
</PropertyGroup>

// ✅ 明確標記可空與不可空
public class UserService
{
    // 不可空：必須有值
    public string GetUserName(User user)
    {
        return user.Name;  // Name 必須有值
    }

    // 可空：可能為 null
    public User? FindUserById(string id)
    {
        return database.Find(id);  // 可能回傳 null
    }

    public void ProcessUser(User? user)
    {
        if (user is null)
        {
            throw new ArgumentNullException(nameof(user));
        }

        // 這裡 user 不會是 null
        Console.WriteLine(user.Name);
    }
}
```

**2. 使用泛型約束**

```csharp
// ✅ 泛型約束確保型別安全
public class Repository<T> where T : class, IEntity
{
    public void Save(T entity)
    {
        // 編譯器確保 T 一定有 IEntity 的成員
        Console.WriteLine($"Saving entity with ID: {entity.Id}");
    }
}
```

**3. 使用 Pattern Matching**

```csharp
// ✅ 型別安全的模式匹配
public decimal CalculateDiscount(IUser user) => user switch
{
    PremiumUser premium => premium.MembershipLevel * 0.1m,
    RegularUser => 0.05m,
    GuestUser => 0m,
    _ => throw new ArgumentException("Unknown user type")
};
```

---

##### **Python**

**1. 使用型別提示 (Type Hints)**

```python
from typing import List, Optional, Union, Dict

# ✅ 函式型別提示
def process_users(users: List[User], active_only: bool = False) -> List[User]:
    if active_only:
        return [u for u in users if u.is_active]
    return users

# ✅ Optional 表示可能為 None
def find_user(user_id: str) -> Optional[User]:
    user = database.query(user_id)
    return user if user else None

# ✅ Union 表示多種可能型別
def parse_id(value: Union[str, int]) -> str:
    return str(value)
```

**2. 使用 mypy 進行靜態型別檢查**

```bash
# 安裝 mypy
pip install mypy

# 執行型別檢查
mypy your_module.py
```

```python
# ✅ mypy 會檢查型別錯誤
def add_numbers(a: int, b: int) -> int:
    return a + b

result = add_numbers(5, "hello")  # mypy 會報錯
```

**3. 使用 Protocol (Structural Subtyping)**

```python
from typing import Protocol

# ✅ 定義協議（類似 TypeScript 的 interface）
class Drawable(Protocol):
    def draw(self) -> None: ...

class Circle:
    def draw(self) -> None:
        print("Drawing circle")

class Square:
    def draw(self) -> None:
        print("Drawing square")

def render(shape: Drawable) -> None:
    shape.draw()

# 只要有 draw 方法，就符合 Drawable 協議
render(Circle())  # ✅
render(Square())  # ✅
```

**4. 使用 TypedDict**

```python
from typing import TypedDict

# ✅ 為字典定義型別
class UserDict(TypedDict):
    name: str
    email: str
    age: int

def create_user(data: UserDict) -> User:
    return User(**data)

# mypy 會檢查 key 是否正確
user_data: UserDict = {
    "name": "Alice",
    "email": "alice@example.com",
    "age": 25
}
```

---

### 💡 最佳實踐總結

| 原則         | 不可變性                                        | 型別安全                               |
| ------------ | ----------------------------------------------- | -------------------------------------- |
| **優先使用** | 不可變資料結構                                  | 靜態型別檢查                           |
| **避免**     | 直接修改物件                                    | any / dynamic 型別                     |
| **工具**     | Immer / ImmutableCollections / frozen dataclass | TypeScript strict / C# Nullable / mypy |
| **測試**     | 易於測試（無副作用）                            | 編譯時發現錯誤                         |
| **併發**     | 天生執行緒安全                                  | 避免型別錯誤導致的競爭條件             |

---

### 🎯 實戰建議

#### **1. 漸進式採用**

```typescript
// 開始：允許部分可變
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000
};

// 改善：使用 readonly
interface Config {
  readonly apiUrl: string;
  readonly timeout: number;
}

// 最佳：使用 as const
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000
} as const;
```

#### **2. 在邊界處轉換**

```typescript
// ✅ 在系統邊界（API）接收可變資料，立即轉換成不可變
async function fetchUser(id: string): Promise<Readonly<User>> {
  const response = await api.get(`/users/${id}`);
  return Object.freeze(response.data);  // 轉換成不可變
}
```

#### **3. 使用 Linter 強制執行**

```json
// .eslintrc.js
{
  "rules": {
    "no-param-reassign": "error",     // 禁止修改參數
    "prefer-const": "error",          // 優先使用 const
    "@typescript-eslint/prefer-readonly": "warn"  // 建議使用 readonly
  }
}
```

---

### 🔑 核心要點

1. **不可變性讓程式碼更容易推理**：你可以安心傳遞物件，不用擔心被修改
2. **型別安全提早發現錯誤**：在編譯時而非執行時發現問題
3. **兩者相輔相成**：不可變 + 型別安全 = 更健壯的程式碼
4. **選擇合適的工具**：record (C#)、frozen dataclass (Python)、Immer (TypeScript)
5. **漸進式導入**：從關鍵模組開始，逐步擴展到整個專案

**記住**：「Make it work, make it right, make it fast」— Kent Beck。不可變性與型別安全是「make it right」的重要手段。

---
[← 回目錄](./README.md) | [下一章：SOLID →](./SOLID.md)
