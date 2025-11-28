# 🧹 Clean Code 實戰指南  
> **撰寫乾淨、可維護、可測試的程式碼 — For C#, TypeScript, Python Developers**

#### Clean Code 的本質

  「任何傻瓜都能寫出電腦能理解的程式碼。優秀的程式設計師寫出人類能理解的程式碼。」(Martin Fowler)

  Clean Code 不只是「能運行的程式碼」，更是**專業開發者對品質的承諾與責任**。它代表：
  - 可讀性高：其他開發者（包括6個月後的自己）能快速理解
  - 可維護性強：修改功能時不會牽一髮動全身
  - 可測試：容易撰寫單元測試
  - 表達清晰的意圖：程式碼本身就是最好的文件
  - 專業態度：展現對團隊、對產品、對未來維護者的尊重與責任感

---

## 📖 目錄
1. 命名規則 (Naming)
2. 函式設計 (Functions)
3. 條件判斷 (Conditionals)
4. 類別與物件 (Classes & Objects)
5. 註解原則 (Comments)
6. 重構與避免重複 (Refactoring)
7. 錯誤處理 (Error Handling)
8. 一致性與風格工具 (Consistency)
9. 進階：抽象層次與依賴反轉 (Abstraction & Dependency)
10. 進階：模組化與測試性 (Modularization & Testability)
11. 進階：不可變性與型別安全 (Immutability & Type Safety)
12. 進階：魔術數字/字串處理 (Magic Numbers & Strings)
13. 進階：「高內聚、低耦合」的黃金法則：S.O.L.I.D. 架構思維
14. 附錄：Clean Code Checklist

---

## 一、命名規則 (Naming)

### ✅ 原則
- 名稱要清楚表達意圖（self-explanatory）
- 不使用模糊或縮寫變數（例如 tmp, x, obj）
- 函式名稱應使用動詞開頭

### 🧩 範例

#### C#
```csharp
var currentDate = DateTime.Now;
void CalculateTotalPrice(decimal basePrice, decimal taxRate) { }
```

#### TypeScript
```typescript
let userIds = [1, 2, 3];
function processUserLogin(user: User) {}
```

#### Python
```python
user_count = 10
def calculate_discount(price, rate): return price * (1 - rate)
```

---

## 二、函式設計 (Functions)

### ✅ 核心原則

#### 1. 簡短！
> 關於函式的首要準則，就是要簡短。第二項準則，就是要比第一項的簡短函式還要更簡短。

#### 2. 只做一件事（Single Responsibility）
- 如果能用文字描述函式功能時，包含了「不同層次」的抽象概念步驟，就表示做了不只一件事
- 所有行為應該在「同個層級」

#### 3. 區塊和縮排
- If、else、while等語句應該只有一行
- 函式不應該包含過多巢狀結構  
- 縮排程度應該只包含1-2層

#### 4. 避免輸出型參數
- 不使用會修改傳入參數的函式
- 優先使用物件導向方式（如 `this`、`self`）而非傳遞 reference
- 參數數量 ≤ 3

#### 5. 函式名稱反映目的，而非細節

### 🧩 範例

#### ❌ 不好的寫法
```typescript
// 做了太多事情
function processUserData(user: User, gameBoard: Cell[]) {
  // 驗證使用者
  if (!user.email.includes('@')) throw new Error('Invalid email');
  
  // 儲存到資料庫
  database.save(user);
  
  // 修改遊戲板（輸出型參數）
  gameBoard.push(new Cell());
  
  // 發送通知
  emailService.notify(user);
}
```

#### ✅ 好的寫法
```typescript
// 每個函式只做一件事
function validateUser(user: User): void {
  if (!user.email.includes('@')) {
    throw new Error('Invalid email');
  }
}

function saveUser(user: User): void {
  database.save(user);
}

function notifyUser(user: User): void {
  emailService.notify(user);
}

function processUser(user: User): void {
  validateUser(user);
  saveUser(user);
  notifyUser(user);
}

// 避免輸出型參數，使用物件導向
class GameBoard {
  private cells: Cell[] = [];
  
  addMineCell(): void {
    this.cells.push(new Cell(CellType.Mine));
  }
  
  getFlaggedCells(): Cell[] {
    return this.cells.filter(cell => cell.isFlagged);
  }
}
```

#### C#
```csharp
// 單一職責
public void ProcessUser(User user) {
    ValidateUser(user);
    SaveUser(user);
    NotifyUser(user);
}

private void ValidateUser(User user) {
    if (!user.Email.Contains("@")) 
        throw new ArgumentException("Invalid email");
}

private void SaveUser(User user) {
    _repository.Save(user);
}

private void NotifyUser(User user) {
    _emailService.Send(user);
}
```

#### Python
```python
def process_user(user):
    """處理使用者 - 只做一件抽象層級的事"""
    validate_user(user)
    save_user(user)
    notify_user(user)

def validate_user(user):
    """驗證使用者資料"""
    if '@' not in user.email:
        raise ValueError('Invalid email')

def save_user(user):
    """儲存使用者到資料庫"""
    database.save(user)

def notify_user(user):
    """發送通知給使用者"""
    email_service.send(user)

# 避免輸出型參數的例子
class GameBoard:
    def __init__(self):
        self.cells = []
    
    def add_mine_cell(self):
        """使用 self 而非傳遞 reference"""
        self.cells.append(Cell(cell_type='mine'))
    
    def get_flagged_cells(self):
        """取得被插旗的格子 - 單一職責"""
        return [cell for cell in self.cells if cell.is_flagged]
```

### 💡 判斷函式是否只做一件事的方法

用文字描述函式功能：
- ✅ **好例子**：「取得被插旗的地雷格子」- 清楚的單一目的
- ❌ **壞例子**：「建立資料庫連線，取得被插旗的地雷格子，並寫入日誌」- 包含不同抽象層次的步驟

---

## 三、條件判斷 (Conditionals)

### ✅ 原則
- 使用早期回傳 (early return)
- 避免巢狀結構

#### TypeScript
```typescript
function isEligible(user?: User) {
  if (!user) return false;
  return user.age >= 18;
}
```

#### Python
```python
def is_eligible(user):
    if not user:
        return False
    return user.age >= 18
```

#### C#
```csharp
bool IsEligible(User? user)
{
    if (user == null) return false;
    return user.Age >= 18;
}
```

---

## 四、類別與物件 (Classes & Objects)

### ✅ 原則
- 單一職責原則 (SRP)
- 用封裝保護內部狀態

#### TypeScript
```typescript
class UserValidator {
  static isValidEmail(email: string) {
    return email.includes("@");
  }
}
```

#### C#
```csharp
class UserValidator {
    public bool IsValidEmail(string email) => email.Contains("@");
}
```

#### Python
```python
class UserValidator:
    @staticmethod
    def is_valid_email(email: str) -> bool:
        return '@' in email
```

---

## 五、註解 (Comments)

### ✅ 原則
- 說明「為什麼」而非「做什麼」

```csharp
// Skip already processed record
index += 1;
```

---

## 六、重構與避免重複 (Refactoring)

```typescript
const dashboardMap = {
  admin: showAdminDashboard,
  manager: showManagerDashboard,
  guest: showGuestDashboard,
};
dashboardMap[user.role]?.();
```

---

## 七、錯誤處理 (Error Handling)

#### C#
```csharp
class ValidationException : Exception {
    public ValidationException(string message) : base(message) { }
}
```

#### TypeScript
```typescript
class ValidationError extends Error {
  constructor(message: string, public context?: any) {
    super(message);
    this.name = "ValidationError";
  }
}
```

#### Python
```python
class ValidationError(Exception):
    def __init__(self, message, context=None):
        super().__init__(message)
        self.context = context
```

---

## 八、一致性與風格工具 (Consistency)

| 工具              | 功能               | 適用語言   |
| ----------------- | ------------------ | ---------- |
| Prettier          | 排版格式化         | TypeScript |
| ESLint            | 語法檢查           | TypeScript |
| EditorConfig      | 統一縮排與換行     | 全語言     |
| StyleCop / Roslyn | 程式風格檢查       | C#         |
| Black / flake8    | 自動排版、靜態檢查 | Python     |

---

## 九、進階：抽象層次與依賴反轉 (Abstraction & Dependency)

#### TypeScript
```typescript
interface EmailSender {
  send(email: string, message: string): void;
}

class SmtpSender implements EmailSender {
  send(email: string, message: string) {
    console.log(`SMTP sending to ${email}`);
  }
}

class UserService {
  constructor(private sender: EmailSender) {}
  register(email: string) {
    this.sender.send(email, "Welcome!");
  }
}
```

#### C#
```csharp
interface IEmailSender {
    void Send(string email, string message);
}

class SmtpSender : IEmailSender {
    public void Send(string email, string message) =>
        Console.WriteLine($"SMTP sending to {email}");
}

class UserService {
    private readonly IEmailSender _sender;
    public UserService(IEmailSender sender) => _sender = sender;
    public void Register(string email) => _sender.Send(email, "Welcome!");
}
```

#### Python
```python
from abc import ABC, abstractmethod

class EmailSender(ABC):
    @abstractmethod
    def send(self, email, message): pass

class SmtpSender(EmailSender):
    def send(self, email, message):
        print(f"SMTP sending to {email}")

class UserService:
    def __init__(self, sender: EmailSender):
        self.sender = sender

    def register(self, email):
        self.sender.send(email, "Welcome!")
```

---

## 十、進階：模組化與測試性 (Modularization & Testability)

```
src/
 ├── domain/      # 業務邏輯
 ├── infra/       # 外部資源 (DB, API)
 ├── app/         # 控制流程
 ├── tests/       # 單元測試
```

---

## 十二、進階：不可變性與型別安全 (Immutability & Type Safety)

#### TypeScript
```typescript
function updateUser(user: User) {
  return { ...user, name: "NewName" }; // immutable
}
```

#### C#
```csharp
record User(string Name, string Email); // immutable by default
```

#### Python
```python
from dataclasses import dataclass

@dataclass(frozen=True)
class User:
    name: str
    email: str
```

---
## 十三、進階：魔術數字/字串處理 (Magic Numbers & Strings)
  ❌ 問題範例
#### TypeScript
```typescript
  function calculatePrice(quantity: number) {
    let price = quantity * 299;

    if (quantity > 10) {
      price = price * 0.9;  // 這個 0.9 是什麼意思？
    }

    if (price > 5000) {
      return price * 0.95;  // 這個 5000 和 0.95 又是什麼？
    }

    return price;
  }

  // 訂單狀態檢查
  if (order.status === 3) {  // 3 代表什麼？
    processOrder(order);
  }

  // 使用者權限檢查
  if (user.role === "ADM") {  // 為什麼是 "ADM" 而不是 "ADMIN"？
    grantAccess();
  }
```

#### C#
```csharp
  public void ProcessPayment(decimal amount)
  {
      if (amount > 10000)  // 為什麼是 10000？
      {
          // 需要額外驗證
          VerifyIdentity();
      }

      decimal fee = amount * 0.03;  // 0.03 是什麼費用？

      if (DateTime.Now.Hour < 9 || DateTime.Now.Hour > 17)  // 為什麼是 9 和 17？
      {
          fee += 50;  // 50 是什麼？
      }
  }
```

####  Python
```python
  def send_notification(user):
      if user.age < 18:  # 18 還算清楚
          return

      # 但這些就很難懂了
      retry_count = 0
      while retry_count < 3:  # 為什麼是 3？
          try:
              api.send(user.email, timeout=5000)  # 5000 是毫秒還是秒？
              break
          except:
              time.sleep(2)  # 為什麼等 2 秒？
              retry_count += 1

  ---
  🤔 為什麼魔術數字/字串是問題？

  1. 降低可讀性

  // 這段程式碼在做什麼？需要猜測
  if (user.score > 80 && user.loginDays > 30 && user.purchaseCount > 5) {
    user.level = 2;
  }

  2. 難以維護

  # 如果商業規則改變，需要找出所有地方修改
  def calculate_shipping(weight):
      if weight < 5:
          return 60
      elif weight < 10:
          return 120
      else:
          return 180

  def estimate_delivery_time(weight):
      if weight < 5:  # 又出現 5，如果改規則要改兩處
          return 1
      # ...

  3. 容易出錯

  // 多處使用相同數值，但其中一處打錯
  decimal taxRate = 0.05;
  // ... 100 行後
  decimal finalPrice = price * 1.05;  // 正確
  // ... 200 行後
  decimal estimate = basePrice * 1.5;  // 錯誤！應該是 1.05

  4. 無法表達業務意圖

  // 3 代表什麼狀態？已完成？已取消？處理中？
  if (order.status === 3) {
    refundOrder(order);
  }
```
  ---
  ✅ 解決方案

  方案 1：使用具名常數 (Named Constants)

#### TypeScript
```typescript
  // 定義常數
  const UNIT_PRICE = 299;
  const BULK_ORDER_THRESHOLD = 10;
  const BULK_DISCOUNT_RATE = 0.9;
  const HIGH_VALUE_THRESHOLD = 5000;
  const HIGH_VALUE_DISCOUNT_RATE = 0.95;

  function calculatePrice(quantity: number) {
    let price = quantity * UNIT_PRICE;

    if (quantity > BULK_ORDER_THRESHOLD) {
      price = price * BULK_DISCOUNT_RATE;
    }

    if (price > HIGH_VALUE_THRESHOLD) {
      return price * HIGH_VALUE_DISCOUNT_RATE;
    }

    return price;
  }
```
####  C#
```csharp
  public class PaymentProcessor
  {
      private const decimal HIGH_AMOUNT_THRESHOLD = 10000m;
      private const decimal STANDARD_FEE_RATE = 0.03m;
      private const decimal AFTER_HOURS_FEE = 50m;
      private const int BUSINESS_HOUR_START = 9;
      private const int BUSINESS_HOUR_END = 17;

      public void ProcessPayment(decimal amount)
      {
          if (amount > HIGH_AMOUNT_THRESHOLD)
          {
              VerifyIdentity();
          }

          decimal fee = amount * STANDARD_FEE_RATE;

          if (IsAfterHours())
          {
              fee += AFTER_HOURS_FEE;
          }
      }

      private bool IsAfterHours()
      {
          int currentHour = DateTime.Now.Hour;
          return currentHour < BUSINESS_HOUR_START ||
                 currentHour > BUSINESS_HOUR_END;
      }
  }
```
####  Python
```python
  # 常數定義（Python 慣例使用大寫）
  ADULT_AGE = 18
  MAX_RETRY_ATTEMPTS = 3
  API_TIMEOUT_MS = 5000
  RETRY_DELAY_SECONDS = 2

  def send_notification(user):
      if user.age < ADULT_AGE:
          return

      retry_count = 0
      while retry_count < MAX_RETRY_ATTEMPTS:
          try:
              api.send(user.email, timeout=API_TIMEOUT_MS)
              break
          except Exception:
              time.sleep(RETRY_DELAY_SECONDS)
              retry_count += 1
```
  ---
  方案 2：使用列舉 (Enums)

  處理狀態、類型、選項等固定值時，使用列舉更好。

#### TypeScript
```typescript
  // ❌ 使用魔術數字
  if (order.status === 0) { /* 待處理 */ }
  if (order.status === 1) { /* 處理中 */ }
  if (order.status === 2) { /* 已完成 */ }
  if (order.status === 3) { /* 已取消 */ }

  // ✅ 使用列舉
  enum OrderStatus {
    Pending = 0,
    Processing = 1,
    Completed = 2,
    Cancelled = 3
  }

  if (order.status === OrderStatus.Pending) {
    startProcessing(order);
  }

  if (order.status === OrderStatus.Cancelled) {
    refundOrder(order);
  }

  // 或使用字串列舉（更易讀）
  enum UserRole {
    Admin = "ADMIN",
    Manager = "MANAGER",
    User = "USER",
    Guest = "GUEST"
  }

  if (user.role === UserRole.Admin) {
    grantFullAccess();
  }
```
#### C#
```csharp
  // ✅ 使用列舉
  public enum OrderStatus
  {
      Pending = 0,
      Processing = 1,
      Completed = 2,
      Cancelled = 3
  }

  public enum UserRole
  {
      Admin,
      Manager,
      User,
      Guest
  }

  public void HandleOrder(Order order)
  {
      if (order.Status == OrderStatus.Cancelled)
      {
          RefundOrder(order);
      }
  }

  public void CheckPermission(User user)
  {
      if (user.Role == UserRole.Admin)
      {
          GrantFullAccess();
      }
  }
```
#### Python
```python
  from enum import Enum, IntEnum

  # 數字列舉
  class OrderStatus(IntEnum):
      PENDING = 0
      PROCESSING = 1
      COMPLETED = 2
      CANCELLED = 3

  # 字串列舉
  class UserRole(Enum):
      ADMIN = "ADMIN"
      MANAGER = "MANAGER"
      USER = "USER"
      GUEST = "GUEST"

  def handle_order(order):
      if order.status == OrderStatus.CANCELLED:
          refund_order(order)
      elif order.status == OrderStatus.COMPLETED:
          send_receipt(order)

  def check_permission(user):
      if user.role == UserRole.ADMIN:
          grant_full_access()
```
---
  方案 3：使用配置物件/類別

  當相關常數很多時，可以組織成配置物件。

 #### TypeScript
```typescript
  // ✅ 組織相關常數
  const PricingConfig = {
    unitPrice: 299,
    bulkOrder: {
      threshold: 10,
      discountRate: 0.9
    },
    highValue: {
      threshold: 5000,
      discountRate: 0.95
    }
  } as const;

  const ShippingConfig = {
    weight: {
      light: 5,
      medium: 10
    },
    fee: {
      light: 60,
      medium: 120,
      heavy: 180
    },
    deliveryDays: {
      light: 1,
      medium: 2,
      heavy: 3
    }
  } as const;

  function calculateShipping(weight: number) {
    if (weight < ShippingConfig.weight.light) {
      return ShippingConfig.fee.light;
    }
    if (weight < ShippingConfig.weight.medium) {
      return ShippingConfig.fee.medium;
    }
    return ShippingConfig.fee.heavy;
  }
```
####  C#
```csharp
  public static class AppConfig
  {
      public static class Payment
      {
          public const decimal HighAmountThreshold = 10000m;
          public const decimal StandardFeeRate = 0.03m;
          public const decimal AfterHoursFee = 50m;
      }

      public static class BusinessHours
      {
          public const int Start = 9;
          public const int End = 17;
      }

      public static class Retry
      {
          public const int MaxAttempts = 3;
          public const int DelaySeconds = 2;
      }
  }

  // 使用
  if (amount > AppConfig.Payment.HighAmountThreshold)
  {
      VerifyIdentity();
  }
````
####  Python
```python
  # 使用類別組織配置
  class PaymentConfig:
      HIGH_AMOUNT_THRESHOLD = 10000
      STANDARD_FEE_RATE = 0.03
      AFTER_HOURS_FEE = 50

  class BusinessHoursConfig:
      START = 9
      END = 17

  class RetryConfig:
      MAX_ATTEMPTS = 3
      DELAY_SECONDS = 2
      TIMEOUT_MS = 5000

  # 或使用字典
  SHIPPING_CONFIG = {
      'weight_thresholds': {'light': 5, 'medium': 10},
      'fees': {'light': 60, 'medium': 120, 'heavy': 180},
      'delivery_days': {'light': 1, 'medium': 2, 'heavy': 3}
  }

  def calculate_shipping(weight):
      thresholds = SHIPPING_CONFIG['weight_thresholds']
      fees = SHIPPING_CONFIG['fees']

      if weight < thresholds['light']:
          return fees['light']
      elif weight < thresholds['medium']:
          return fees['medium']
      else:
          return fees['heavy']
```
  ---
  🎯 實戰範例：重構前後對比

  範例 1：使用者等級系統

  ❌ 重構前
```typescript
  function updateUserLevel(user: User) {
    if (user.score > 80 && user.loginDays > 30 && user.purchaseCount > 5) {
      user.level = 2;
      user.discount = 0.15;
    } else if (user.score > 50 && user.loginDays > 15 && user.purchaseCount > 2) {
      user.level = 1;
      user.discount = 0.1;
    } else {
      user.level = 0;
      user.discount = 0;
    }
  }
```
  ✅ 重構後
```typescript
  enum UserLevel {
    Regular = 0,
    Silver = 1,
    Gold = 2
  }

  const LEVEL_REQUIREMENTS = {
    [UserLevel.Gold]: {
      minScore: 80,
      minLoginDays: 30,
      minPurchaseCount: 5,
      discount: 0.15
    },
    [UserLevel.Silver]: {
      minScore: 50,
      minLoginDays: 15,
      minPurchaseCount: 2,
      discount: 0.1
    },
    [UserLevel.Regular]: {
      discount: 0
    }
  };

  function meetsRequirement(user: User, level: UserLevel): boolean {
    const req = LEVEL_REQUIREMENTS[level];
    return user.score >= req.minScore &&
           user.loginDays >= req.minLoginDays &&
           user.purchaseCount >= req.minPurchaseCount;
  }

  function updateUserLevel(user: User) {
    if (meetsRequirement(user, UserLevel.Gold)) {
      user.level = UserLevel.Gold;
      user.discount = LEVEL_REQUIREMENTS[UserLevel.Gold].discount;
    } else if (meetsRequirement(user, UserLevel.Silver)) {
      user.level = UserLevel.Silver;
      user.discount = LEVEL_REQUIREMENTS[UserLevel.Silver].discount;
    } else {
      user.level = UserLevel.Regular;
      user.discount = LEVEL_REQUIREMENTS[UserLevel.Regular].discount;
    }
  }
```
  ---
  📋 何時可以接受「魔術數字」？

  例外情況

  1. 數學常數或通用慣例
```csharp
  // ✅ 可接受：這些是數學常數
  const area = radius * radius * Math.PI;
  const circumference = 2 * Math.PI * radius;

  // ✅ 可接受：100% 是通用慣例
  const percentage = (value / total) * 100;

  // ✅ 可接受：0 和 1 在某些情境下很明確
  array.length === 0  // 陣列為空
  items[0]            // 第一個元素

  2. 上下文已經非常清楚

  # ✅ 可接受：在這個上下文中，7 很明確就是「一週的天數」
  def get_week_start(date):
      days_since_monday = date.weekday()
      return date - timedelta(days=days_since_monday)

  # 但如果會重複使用，還是建議定義常數
  DAYS_PER_WEEK = 7

  3. 測試程式碼中的範例資料

  // ✅ 測試中可接受：這些只是範例數值
  [Test]
  public void Should_Calculate_Discount_For_VIP()
  {
      var user = new User { Score = 100, Level = 2 };
      var result = calculator.Calculate(user);
      Assert.AreEqual(150, result);
  }
```
---
  ✅ 最佳實踐總結

  | 情境                       | 建議做法              | 範例                               |
  | -------------------------- | --------------------- | ---------------------------------- |
  | 業務規則相關數字           | 使用具名常數          | const MIN_AGE = 18                 |
  | 狀態/類型/選項             | 使用列舉              | enum OrderStatus                   |
  | 配置參數                   | 集中管理到配置檔/類別 | AppConfig.Payment.FeeRate          |
  | 字串常數（狀態碼、訊息等） | 使用常數或列舉        | const ERROR_INVALID_EMAIL = "E001" |
  | 重複出現的值               | 一定要定義常數        | 避免多處寫 0.05                    |
  | 計算公式中的係數           | 命名說明用途          | const TAX_RATE = 0.05              |

  ---

## 十四、進階：「高內聚、低耦合」的黃金法則：S.O.L.I.D. 架構思維
判斷軟體架構好壞的**黃金準則**。
> **核心目標：寫出**「高內聚、低耦合」的程式。
### 🧭 S.O.L.I.D. 原則簡介
==================

這五個物件導向設計原則能讓程式更容易維護與擴充，就像生活中良好的系統設計一樣有秩序。\
S.O.L.I.D. 代表：

1.  **S -- Single Responsibility Principle**：單一職責原則
2.  **O -- Open/Closed Principle**：開放封閉原則
3.  **L -- Liskov Substitution Principle**：里氏替換原則
4.  **I -- Interface Segregation Principle**：介面隔離原則
5.  **D -- Dependency Inversion Principle**：依賴反轉原則

* * * * *

#### 1️⃣ 單一職責原則 (Single Responsibility)
----------------------------------

**概念：**
一個類別或模組應該只有**一個**改變它的理由。換句話說，它應該只專注於**一項**核心職責。

**生活比喻：**
一把瑞士刀雖然方便，但如果又要當螺絲起子、又要開罐頭、又要剪指甲，一旦其中一個功能壞了，整把刀都可能受影響。專業的工具（螺絲起子、開罐器）各自獨立，更可靠、也更容易修理。

**🎯 核心目標：**
- **降低耦合**：一個類別只做一件事，減少與其他類別的牽連。
- **提高內聚**：類別內部的方法和屬性都緊密圍繞著同一個核心功能。
- **易於理解與維護**：當你需要修改「報表生成」邏輯時，你很清楚要去 `ReportGenerator` 類別，而不是在一個混雜了資料庫存取、檔案儲存的巨大類別中尋找。

**❌ 壞例子：一個 `UserService` 什麼都做**
這個類別混合了使用者資料處理、密碼驗證、和日誌記錄三種完全不同的職責。

```csharp
// C#
public class UserService
{
    // 職責1: 使用者資料處理
    public void RegisterUser(string username, string password)
    {
        if (!IsValid(username, password))
        {
            // 職責3: 記錄錯誤日誌
            File.WriteAllText("error.log", "Invalid user data");
            return;
        }
        // ... 儲存到資料庫 ...
    }

    // 職責2: 驗證邏輯
    private bool IsValid(string username, string password)
    {
        return !string.IsNullOrEmpty(username) && password.Length > 8;
    }
}
```

**✅ 改進：職責分離**
將三個職責拆分成獨立的類別，每個類別只專注一件事。

```csharp
// C#

// 職責1: 專注於驗證
public class UserValidator
{
    public bool Validate(string username, string password)
    {
        return !string.IsNullOrEmpty(username) && password.Length > 8;
    }
}

// 職責2: 專注於資料庫操作
public class UserRepository
{
    public void Save(string username, string password)
    {
        // ... 儲存到資料庫的邏輯 ...
    }
}

// 職責3: 專注於日誌記錄
public class Logger
{
    public void LogError(string message)
    {
        File.WriteAllText("error.log", message);
    }
}

// 高層次的協調者 (Orchestrator)
public class UserService
{
    private readonly UserValidator _validator;
    private readonly UserRepository _repository;
    private readonly Logger _logger;

    public UserService(UserValidator validator, UserRepository repository, Logger logger)
    {
        _validator = validator;
        _repository = repository;
        _logger = logger;
    }

    public void RegisterUser(string username, string password)
    {
        if (!_validator.Validate(username, password))
        {
            _logger.LogError("Invalid user data provided for username: " + username);
            return;
        }
        _repository.Save(username, password);
    }
}
```

**🤔 如何判斷是否違反 SRP？**
問自己：「這個類別會因為哪些不同的原因而需要修改？」
- 如果「驗證規則」改變了，需要改 `UserService`。
- 如果「資料庫儲存方式」改變了（例如從 SQL Server 換到 MongoDB），需要改 `UserService`。
- 如果「日誌記錄方式」改變了（例如從寫檔案改成送到雲端），需要改 `UserService`。

只要答案超過一個，就可能違反了單一職責原則。

* * * * *

#### 2️⃣ 開放封閉原則 (Open/Closed)
------------------------

**概念：**
軟體實體（類別、模組、函式）應該對**擴充**開放，但對**修改**封閉。

**生活比喻：**
你的手機可以透過安裝 App (擴充) 來增加新功能（如外送、遊戲），但你不需要為了裝 App 而拆開手機、修改內部電路 (修改)。

**🎯 核心目標：**
- **增加新功能時，不動舊程式碼**：這能確保舊有、穩定的功能不會因為新需求而被改壞。
- **降低測試成本**：只需測試新加入的功能，不必重新測試整個系統。
- **提高系統彈性**：讓系統像樂高一樣，可以輕易地組合、添加新零件。

**❌ 壞例子：根據不同支付方式計算手續費**
每次新增一種支付方式（例如 Line Pay、街口支付），你都必須回來**修改** `PaymentProcessor` 這個類別，在 `if/else` 中加入新的邏輯。

```typescript
// TypeScript
class PaymentProcessor {
    public calculateFee(amount: number, type: 'CreditCard' | 'PayPal'): number {
        if (type === 'CreditCard') {
            return amount * 0.02; // 信用卡手續費 2%
        } else if (type === 'PayPal') {
            return amount * 0.03 + 10; // PayPal 手續費 3% + 固定費用 10 元
        }
        // 如果要新增 Line Pay，就要改這裡！
        return 0;
    }
}
```

**✅ 改進：使用策略模式 (Strategy Pattern)**
定義一個共同的 `IPaymentStrategy` 介面，讓各種支付方式各自實現該介面。`PaymentProcessor` 只依賴這個抽象介面，而不需要知道具體的支付細節。

```typescript
// TypeScript

// 1. 定義一個抽象策略介面
interface IPaymentStrategy {
    calculateFee(amount: number): number;
}

// 2. 為每種支付方式建立具體的策略類別
class CreditCardStrategy implements IPaymentStrategy {
    calculateFee(amount: number): number {
        return amount * 0.02;
    }
}

class PayPalStrategy implements IPaymentStrategy {
    calculateFee(amount: number): number {
        return amount * 0.03 + 10;
    }
}

// ✨ 新增 Line Pay 功能，完全不用改舊程式碼！
class LinePayStrategy implements IPaymentStrategy {
    calculateFee(amount: number): number {
        return amount * 0.015; // Line Pay 手續費 1.5%
    }
}

// 3. 處理器依賴抽象介面，而不是具體實現
class PaymentProcessor {
    private strategy: IPaymentStrategy;

    // 透過建構子傳入具體的策略
    constructor(strategy: IPaymentStrategy) {
        this.strategy = strategy;
    }

    public calculateFee(amount: number): number {
        // Processor 不關心現在是哪種支付方式，只管呼叫介面的方法
        return this.strategy.calculateFee(amount);
    }
}

// --- 使用方式 ---
const creditCardProcessor = new PaymentProcessor(new CreditCardStrategy());
console.log(creditCardProcessor.calculateFee(1000)); // 輸出 20

const linePayProcessor = new PaymentProcessor(new LinePayStrategy());
console.log(linePayProcessor.calculateFee(1000)); // 輸出 15
```

**🤔 如何判斷是否違反 OCP？**
問自己：「當一個新的業務需求（例如新的折扣類型、新的匯出格式、新的支付方式）出現時，我是不是必須回去修改現有的、已經上線的程式碼？」

如果答案是「是」，那麼你的設計可能就缺乏彈性，違反了開放封閉原則。

* * * * *

#### 3️⃣ 里氏替換原則 (Liskov Substitution)
--------------------------------

**概念：**
任何基底類別（Base Class）可以出現的地方，其子類別（Subclass）都應該能夠替換它，而不會產生錯誤或改變程式的行為。簡單來說，**子類別必須完全遵從父類別的行為契約**。

**生活比喻：**
你去租車，訂了一台「轎車」（父類別）。取車時，店家給你一台「Toyota Altis」（子類別），你完全可以接受，因為它符合轎車的所有基本功能（四個輪子、有方向盤、能前進後退）。但如果店家給你一台「遙控汽車」，雖然它也是車，卻不能載人，破壞了你對「轎C#」的預期，這就違反了里氏替換原則。

**🎯 核心目標：**
- **保證繼承的正確性**：確保子類別沒有「意外地」改變父類別的行為。
- **避免在程式中使用 `if (obj is SubType)` 這種型別檢查**：如果你需要檢查物件的具體子類型才能決定下一步行為，那通常是 LSP 被違反的警訊。
- **讓多型 (Polymorphism) 可靠運作**：你可以安心地將一群父類別物件放進一個列表，並對它們進行統一操作，不用擔心某個子類別會「暴走」。

**❌ 壞例子：正方形繼承矩形**
數學上，正方形是矩形的一種。但在程式碼中，這種繼承關係通常會出問題。

```csharp
// C#
public class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }

    public int GetArea() => Width * Height;
}

// 正方形繼承矩形
public class Square : Rectangle
{
    private int _side;

    public override int Width
    {
        get => _side;
        set { _side = value; base.Width = value; base.Height = value; }
    }

    public override int Height
    {
        get => _side;
        set { _side = value; base.Width = value; base.Height = value; }
    }
}

// --- 使用時出現問題 ---
public class AreaCalculator
{
    public void Test(Rectangle r)
    {
        r.Width = 5;
        r.Height = 4;
        // 我們的預期：面積應該是 5 * 4 = 20
        // 但如果傳入的是 Square，面積會變成 4 * 4 = 16！
        Console.WriteLine($"Expected 20, got {r.GetArea()}");
    }
}
```
在這個例子中，`Square` 改變了 `Rectangle` 的行為：設定 `Width` 會意外地影響 `Height`。這破壞了使用者的預期，因此 `Square` 不能安全地替換 `Rectangle`。

**✅ 改進：打破不合適的繼承關係**
讓它們都實現一個共同的 `IShape` 介面，或者乾脆讓它們成為獨立的類別。

```csharp
// C#
public interface IShape
{
    int GetArea();
}

public class Rectangle : IShape
{
    public int Width { get; }
    public int Height { get; }

    public Rectangle(int width, int height)
    {
        Width = width;
        Height = height;
    }

    public int GetArea() => Width * Height;
}

public class Square : IShape
{
    public int Side { get; }

    public Square(int side)
    {
        Side = side;
    }

    public int GetArea() => Side * Side;
}
```

**🤔 如何判斷是否違反 LSP？**
- **子類別是否拋出了父類別沒有的例外？** 例如，父類別的 `save` 方法不會拋錯，但子類別在某些情況下會拋出 `NotSupportedException`。
- **子類別的行為是否比父類別更「嚴格」？** 例如，父類別接受任何數字，但子類別只接受正數。
- **子類別的輸出是否不符合父類別的定義？** 例如，父類別的方法回傳一個列表，子類別卻回傳 `null`。

如果你的程式碼需要像這樣檢查子類別的型別，那幾乎可以肯定是違反了 LSP：
`if (shape is Square) { ... } else if (shape is Rectangle) { ... }`

* * * * *

#### 4️⃣ 介面隔離原則 (Interface Segregation)
----------------------------------

**概念：**
用戶端（使用介面的類別）不應該被強迫依賴它們用不到的方法。換句話說，**介面應該小而專一，而不是龐大而臃腫**。

**生活比喻：**
你去餐廳點餐，服務生不會給你一本包含所有菜色（中餐、西餐、日式料理）的超厚菜單，而是根據你的需求提供對應的菜單（例如午間套餐、單點菜單）。你只需要關心你想點的，而不用理會其他你不需要的資訊。

**🎯 核心目標：**
- **降低耦合**：用戶端只依賴它需要的最小介面，減少不必要的牽連。
- **提高內聚**：介面本身只包含一組高度相關的方法。
- **避免「胖介面」(Fat Interface)**：一個介面包含太多方法，會導致實作它的類別被迫實作一些對它來說毫無意義的方法。

**❌ 壞例子：一個萬能的 `IWorker` 介面**
這個介面強迫所有「工人」都要會工作、吃飯、和開會。但對於一個 `RobotWorker` 來說，`eat()` 和 `attendMeeting()` 是沒有意義的。

```python
# Python
from abc import ABC, abstractmethod

class IWorker(ABC):
    @abstractmethod
    def work(self):
        pass

    @abstractmethod
    def eat(self):
        pass

    @abstractmethod
    def attend_meeting(self):
        pass

# 人類工人，沒問題
class HumanWorker(IWorker):
    def work(self):
        print("Human is working.")
    def eat(self):
        print("Human is eating.")
    def attend_meeting(self):
        print("Human is attending a meeting.")

# 機器人被迫實作無意義的方法
class RobotWorker(IWorker):
    def work(self):
        print("Robot is working.")
    def eat(self):
        # 這個方法對機器人沒有意義，只能放著或拋出例外
        pass
    def attend_meeting(self):
        # 機器人也不用開會
        pass
```

**✅ 改進：拆分成更小的、專一的介面**
將 `IWorker` 拆分成 `IWorkable`, `IFeedable`, `IMeetingParticipant`。類別可以根據自身能力，選擇性地實作這些小介面。

```python
# Python
from abc import ABC, abstractmethod

# 介面1: 會工作的
class IWorkable(ABC):
    @abstractmethod
    def work(self):
        pass

# 介面2: 會吃飯的
class IFeedable(ABC):
    @abstractmethod
    def eat(self):
        pass

# 介面3: 會開會的
class IMeetingParticipant(ABC):
    @abstractmethod
    def attend_meeting(self):
        pass

# 人類可以做全部三件事
class HumanWorker(IWorkable, IFeedable, IMeetingParticipant):
    def work(self):
        print("Human is working.")
    def eat(self):
        print("Human is eating.")
    def attend_meeting(self):
        print("Human is attending a meeting.")

# 機器人只會工作
class RobotWorker(IWorkable):
    def work(self):
        print("Robot is working.")

# --- 使用時 ---
def manage_work(worker: IWorkable):
    worker.work()

def arrange_lunch(eater: IFeedable):
    eater.eat()

human = HumanWorker()
robot = RobotWorker()

manage_work(human)       # OK
manage_work(robot)       # OK
arrange_lunch(human)     # OK
# arrange_lunch(robot)   # 這行會直接報錯，因為 RobotWorker 沒有實作 IFeedable，符合預期！
```

**🤔 如何判斷是否違反 ISP？**
- **一個類別在實作某個介面時，是否有一個或多個方法是空的，或只是簡單地拋出 `NotImplementedException`？**
- **你的介面名稱是否聽起來很模糊或太通用，例如 `IProcess` 或 `IManager`？** 這通常意味著它包含了太多不相關的職責。
- **你是否經常需要將一個介面轉型成另一個介面，才能呼叫到你需要的方法？**

* * * * *

#### 5️⃣ 依賴反轉原則 (Dependency Inversion)
---------------------------------

**概念：**
1.  **高層模組不應該依賴於低層模組。兩者都應該依賴於抽象。**
2.  **抽象不應該依賴於細節。細節應該依賴於抽象。**

簡單來說，就是**程式碼應該依賴「介面」或「抽象類別」，而不是依賴「具體的實作類別」**。

**生活比喻：**
你開車時，操作的是「方向盤」、「油門」、「煞車」這些**抽象介面**。你不需要關心引擎是如何點火的、煞車皮是哪個牌子的（低層細節）。只要給你一台符合這些標準介面的車，你就能開。這讓你（高層模組）和具體的汽車零件（低層模組）解除了耦合。

**🎯 核心目標：**
- **解耦合 (Decoupling)**：高層業務邏輯和低層實作細節分離，讓兩者可以獨立變化。你可以輕易地替換資料庫、日誌工具或第三方服務，而不需要修改核心業務邏輯。
- **提高可測試性**：當依賴的是介面時，你可以在測試中輕易地傳入一個「假的」(Mock) 物件，來模擬各種情境，而不需要一個真實的資料庫或網路連線。
- **增加系統彈性與可維護性**。

**❌ 壞例子：`NotificationService` 直接依賴 `EmailSender`**
`NotificationService` (高層模組) 直接 `new` 了一個 `EmailSender` (低層模組) 來使用。這導致它們緊緊地綁在一起。

```typescript
// TypeScript

// 一個具體的 Email 發送服務
class EmailSender {
    send(email: string, message: string) {
        console.log(`Sending email to ${email}: ${message}`);
    }
}

// 訊息通知服務直接依賴這個具體的 Email 發送服務
class NotificationService {
    private emailSender: EmailSender;

    constructor() {
        // ❌ 問題：在這裡直接建立了 EmailSender 的實例
        this.emailSender = new EmailSender();
    }

    notifyUser(email: string, message: string) {
        // ... 其他邏輯 ...
        this.emailSender.send(email, message);
    }
}
```
**問題：**
- `NotificationService` 緊緊依賴於 `EmailSender`，如果要改用其他通知方式（如簡訊、推播），就必須修改 `NotificationService` 的程式碼。
- 測試 `NotificationService` 時，無法輕易地替換掉 `EmailSender`，導致測試時會真的發送 Email。

**✅ 改進：依賴注入 (Dependency Injection)**
定義一個抽象的 `IMessageSender` 介面，讓 `NotificationService` 依賴這個介面。具體的 `EmailSender` 則從外部「注入」進來。

```typescript
// TypeScript

// 1. 定義抽象的訊息發送介面
interface IMessageSender {
    send(message: string, recipient: string): void;
}

// 2. 具體的 Email 發送服務實作這個介面
class EmailSender implements IMessageSender {
    send(message: string, recipient: string) {
        console.log(`Sending email to ${recipient}: ${message}`);
    }
}

// 3. 訊息通知服務依賴這個抽象介面
class NotificationService {
    private sender: IMessageSender;

    // ✅ 透過建構子注入依賴
    constructor(sender: IMessageSender) {
        this.sender = sender;
    }

    notifyUser(recipient: string, message: string) {
        // ... 其他邏輯 ...
        this.sender.send(message, recipient);
    }
}

// --- 使用時 ---
const emailService = new NotificationService(new EmailSender());
emailService.notifyUser('test@example.com', 'Hello, world!');
```
這就是**依賴注入 (Dependency Injection, DI)**，它是實現依賴反轉最常見的方法。

**🎯 總結：** 高內聚讓程式碼像一個整理有序的圖書館，每個書架（模組）都有明確的主題，你可以快速找到你需要的書（功能）。

### 🔌 低耦合 (Low Coupling) - 樂高積木

-   **意思：** 一個模組對另一個模組的依賴、了解或關聯程度。**低耦合**意味著模組之間是獨立的，一個模組的變更不會輕易地影響到另一個模組。

-   **比喻：**

    -   **👍 低耦合 (好)：** USB 裝置。你的電腦（主系統）有一個 USB **介面 (Interface)**，你可以隨意插入鍵盤、滑鼠、隨身碟。更換任何一個裝置都不會影響電腦的運作，因為它們都遵守共同的標準。

    -   **👎 高耦合 (壞)：** 焊接在主機板上的 CPU。如果 CPU 壞了，你無法輕易地將它拔下來換掉，很可能需要連同主機板一起更換。它們彼此**緊密耦合**。

-   **程式範例：**

    -   **👎 高耦合 (壞)：`OrderService` 直接依賴 `CreditCardApi`**
        `OrderService` 直接 `new` 一個 `CreditCardApi` 來用，並且呼叫它專屬的方法 `chargeCard()`。

        ```csharp
        // C#

        // 一個具體的信用卡支付 API
        public class CreditCardApi
        {
            public void ChargeCard(decimal amount, string cardNumber)
            {
                Console.WriteLine($"Charging {amount} to card {cardNumber}.");
            }
        }

        // 訂單服務直接依賴這個具體 API
        public class OrderService
        {
            private CreditCardApi _paymentApi = new CreditCardApi();

            public void ProcessOrder(Order order)
            {
                // ... 處理訂單邏輯 ...

                // ❌ 直接呼叫具體實作的方法
                _paymentApi.ChargeCard(order.Amount, order.CreditCardNumber);
            }
        }
        ```
        **問題：** 如果現在要新增「LINE Pay」支付，`OrderService` 就必須被修改，甚至可能需要加入 `if/else` 來判斷支付類型，這會讓程式碼越來越複雜。

    -   **👍 低耦合 (好)：`OrderService` 依賴 `IPaymentGateway` 介面**
        我們定義一個通用的 `IPaymentGateway` 介面，讓 `OrderService` 只依賴這個抽象。

        ```csharp
        // C#

        // 1. 定義一個抽象的支付閘道介面
        public interface IPaymentGateway
        {
            bool Pay(decimal amount, PaymentDetails details);
        }

        // 2. 讓各種支付方式去實作這個介面
        public class CreditCardGateway : IPaymentGateway
        {
            public bool Pay(decimal amount, PaymentDetails details)
            {
                Console.WriteLine($"Using Credit Card to pay {amount}.");
                // ... 呼叫信用卡 API 的邏輯 ...
                return true;
            }
        }

        public class LinePayGateway : IPaymentGateway
        {
            public bool Pay(decimal amount, PaymentDetails details)
            {
                Console.WriteLine($"Using LINE Pay to pay {amount}.");
                // ... 呼叫 LINE Pay API 的邏輯 ...
                return true;
            }
        }

        // 3. OrderService 依賴抽象介面，並透過 DI 注入
        public class OrderService
        {
            private readonly IPaymentGateway _paymentGateway;

            public OrderService(IPaymentGateway paymentGateway)
            {
                _paymentGateway = paymentGateway;
            }

            public void ProcessOrder(Order order)
            {
                // ... 處理訂單邏輯 ...

                // ✅ 只跟介面溝通，不在乎背後是誰
                _paymentGateway.Pay(order.Amount, order.PaymentDetails);
            }
        }

        // --- 使用時 ---
        var creditCardService = new OrderService(new CreditCardGateway());
        creditCardService.ProcessOrder(myOrder);

        var linePayService = new OrderService(new LinePayGateway());
        linePayService.ProcessOrder(myOtherOrder);
        ```
        現在，`OrderService` 和具體的支付方式完全**解耦**了。未來就算要新增一百種支付方式，`OrderService` 的程式碼也完全不需要更動。

**🎯 總結：** 低耦合讓你的程式碼模組像可替換的零件，你可以輕鬆升級、替換或移除它們，而不會對整個系統造成衝擊。**高內聚、低耦合**是判斷軟體設計優劣的黃金準則。

---

## 十五、多線程安全性 (Thread Safety)
