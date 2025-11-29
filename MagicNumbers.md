## 六、魔術數字/字串處理 (Magic Numbers & Strings)

> 「任何未經命名的數字或字串都是對未來維護者的詛咒。」

### ❓ 什麼是魔術數字/字串？

**魔術數字 (Magic Number)** 是指直接出現在程式碼中的數值常量，沒有任何說明其用途或意義。
**魔術字串 (Magic String)** 則是指直接硬編碼在程式碼中的字串常量，通常代表狀態碼、配置鍵值或業務規則。

這些「魔術值」之所以「魔術」，是因為它們的意義只有當初寫下它的人知道，其他人（包括未來的自己）都需要「猜測」或「查資料」才能理解。

---

### 💔 為什麼魔術數字/字串是問題？

#### 1. **降低可讀性：需要猜測意圖**

當你看到一個數字或字串時，必須停下來思考：「這個值代表什麼？為什麼是這個數字？」

#### ❌ 問題範例

**TypeScript**
```typescript
function calculatePrice(quantity: number) {
  let price = quantity * 299;  // 299 是什麼？單價？

  if (quantity > 10) {  // 10 是什麼門檻？
    price = price * 0.9;  // 0.9 是什麼折扣？90%？10% off？
  }

  if (price > 5000) {  // 5000 又是什麼門檻？
    return price * 0.95;  // 0.95 又是什麼折扣？
  }

  return price;
}

// 訂單狀態檢查
if (order.status === 3) {  // 3 代表什麼？已完成？已取消？處理中？
  processOrder(order);
}

// 使用者權限檢查
if (user.role === "ADM") {  // 為什麼是 "ADM" 而不是 "ADMIN"？
  grantAccess();
}
```

**C#**
```csharp
public void ProcessPayment(decimal amount)
{
    if (amount > 10000)  // 為什麼是 10000？什麼門檻？
    {
        VerifyIdentity();  // 需要額外驗證
    }

    decimal fee = amount * 0.03;  // 0.03 是什麼費用率？手續費？稅金？

    if (DateTime.Now.Hour < 9 || DateTime.Now.Hour > 17)  // 為什麼是 9 和 17？
    {
        fee += 50;  // 50 是什麼？加班費？
    }
}
```

**Python**
```python
def send_notification(user):
    if user.age < 18:  # 18 還算清楚（成年年齡）
        return

    # 但這些就很難懂了
    retry_count = 0
    while retry_count < 3:  # 為什麼是 3 次？
        try:
            api.send(user.email, timeout=5000)  # 5000 是毫秒還是秒？
            break
        except:
            time.sleep(2)  # 為什麼等 2 秒？
            retry_count += 1
```

**問題總結**：
- `299` 是產品單價，但沒有說明
- `0.9` 代表 10% 折扣，但看起來像 90%，容易混淆
- `3` 代表某種訂單狀態，但完全不知道是什麼
- `"ADM"` 是縮寫，但為什麼不是 `"ADMIN"`？

---

#### 2. **難以維護：修改時容易遺漏**

當業務規則改變時，你需要在整個專案中搜尋所有相關的魔術數字，並逐一修改。這不僅耗時，還容易遺漏。

**範例：運費計算分散在多處**

```python
# ❌ 魔術數字散落各處
def calculate_shipping(weight):
    if weight < 5:  # 輕量門檻
        return 60
    elif weight < 10:  # 中量門檻
        return 120
    else:
        return 180

def estimate_delivery_time(weight):
    if weight < 5:  # 又出現 5！如果改規則要改兩處
        return 1
    elif weight < 10:  # 又出現 10！
        return 2
    else:
        return 3

def should_use_express(weight):
    return weight < 5  # 又又出現 5！！！
```

**問題**：如果輕量門檻從 5 公斤改成 3 公斤，你需要修改至少 3 處，極易遺漏。

---

#### 3. **容易出錯：數值打錯或誤用**

當相同的數值在多處使用時，打字錯誤或誤用類似數值的風險大增。

```csharp
// ❌ 多處使用相同數值，但其中一處打錯
decimal taxRate = 0.05;  // 稅率 5%

// ... 100 行後
decimal finalPrice = price * 1.05;  // ✅ 正確：price + 5% tax

// ... 200 行後
decimal estimate = basePrice * 1.5;  // ❌ 錯誤！應該是 1.05，卻打成 1.5
```

---

#### 4. **無法表達業務意圖**

數字和字串本身沒有語義，無法傳達背後的業務邏輯。

```typescript
// ❌ 3 代表什麼狀態？已完成？已取消？處理中？
if (order.status === 3) {
  refundOrder(order);  // 看到這裡才知道 3 可能是「已取消」
}

// ❌ 這些字串代表什麼？
if (user.accountType === "PRM") {  // PRM 是 Premium？
  applyDiscount();
}

if (notification.priority === "H") {  // H 是 High？
  sendImmediately();
}
```

---

### ✅ 解決方案

---

#### **方案 1：使用具名常數 (Named Constants)**

將魔術數字/字串提煉成有意義的常數，並在程式碼頂部或專屬的常數檔案中定義。

**命名規則**：
- 使用全大寫 + 底線分隔（多數語言的慣例）
- 名稱應清楚表達業務意圖，而非只是重述數值

**TypeScript**
```typescript
// ✅ 定義具名常數
const UNIT_PRICE = 299;
const BULK_ORDER_THRESHOLD = 10;
const BULK_DISCOUNT_RATE = 0.9;  // 90% = 10% off
const HIGH_VALUE_THRESHOLD = 5000;
const HIGH_VALUE_DISCOUNT_RATE = 0.95;  // 95% = 5% off

function calculatePrice(quantity: number): number {
  let price = quantity * UNIT_PRICE;

  // 批量訂單折扣
  if (quantity > BULK_ORDER_THRESHOLD) {
    price = price * BULK_DISCOUNT_RATE;
  }

  // 高價值訂單額外折扣
  if (price > HIGH_VALUE_THRESHOLD) {
    price = price * HIGH_VALUE_DISCOUNT_RATE;
  }

  return price;
}
```

**優點**：
- 程式碼自我說明，不需要註解
- 修改業務規則時只需改一處
- 避免打字錯誤

**C#**
```csharp
public class PaymentProcessor
{
    // 定義常數
    private const decimal HIGH_AMOUNT_THRESHOLD = 10000m;
    private const decimal STANDARD_FEE_RATE = 0.03m;
    private const decimal AFTER_HOURS_FEE = 50m;
    private const int BUSINESS_HOUR_START = 9;
    private const int BUSINESS_HOUR_END = 17;

    public void ProcessPayment(decimal amount)
    {
        // 高額交易需要額外驗證
        if (amount > HIGH_AMOUNT_THRESHOLD)
        {
            VerifyIdentity();
        }

        // 計算標準手續費
        decimal fee = amount * STANDARD_FEE_RATE;

        // 非營業時間加收額外費用
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

**Python**
```python
# 常數定義（Python 慣例使用全大寫）
ADULT_AGE = 18
MAX_RETRY_ATTEMPTS = 3
API_TIMEOUT_MS = 5000
RETRY_DELAY_SECONDS = 2

def send_notification(user):
    # 未成年使用者不發送通知
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

#### **方案 2：使用列舉 (Enums)**

當處理**有限的、預定義的選項**（如狀態、類型、角色）時，列舉是最佳選擇。

**為什麼使用列舉？**
- **型別安全**：編譯器會檢查你是否使用了正確的值
- **自動補全**：IDE 可以提供所有可能的選項
- **防止拼寫錯誤**：字串容易拼錯，列舉不會
- **語義清晰**：`OrderStatus.Cancelled` 比 `3` 或 `"CANCELLED"` 更易讀

**TypeScript**
```typescript
// ❌ 使用魔術數字
if (order.status === 0) { /* 待處理 */ }
if (order.status === 1) { /* 處理中 */ }
if (order.status === 2) { /* 已完成 */ }
if (order.status === 3) { /* 已取消 */ }

// ✅ 使用數字列舉
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

// ✅ 使用字串列舉（更易除錯，序列化友善）
enum UserRole {
  Admin = "ADMIN",
  Manager = "MANAGER",
  User = "USER",
  Guest = "GUEST"
}

if (user.role === UserRole.Admin) {
  grantFullAccess();
}

// ✅ 處理支付方式
enum PaymentMethod {
  CreditCard = "CREDIT_CARD",
  DebitCard = "DEBIT_CARD",
  PayPal = "PAYPAL",
  BankTransfer = "BANK_TRANSFER"
}

function processPayment(method: PaymentMethod, amount: number) {
  switch (method) {
    case PaymentMethod.CreditCard:
      return processCreditCard(amount);
    case PaymentMethod.PayPal:
      return processPayPal(amount);
    // TypeScript 會警告你如果遺漏了某個 case
  }
}
```

**C#**
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

public enum NotificationPriority
{
    Low,
    Medium,
    High,
    Urgent
}

// 使用
public void HandleOrder(Order order)
{
    if (order.Status == OrderStatus.Cancelled)
    {
        RefundOrder(order);
    }
    else if (order.Status == OrderStatus.Completed)
    {
        SendReceipt(order);
    }
}

public void CheckPermission(User user)
{
    if (user.Role == UserRole.Admin)
    {
        GrantFullAccess();
    }
}

// 使用 [Flags] 處理可組合的選項
[Flags]
public enum FilePermissions
{
    None = 0,
    Read = 1,
    Write = 2,
    Execute = 4,
    Delete = 8
}

// 可以組合使用
var permissions = FilePermissions.Read | FilePermissions.Write;
if (permissions.HasFlag(FilePermissions.Write))
{
    // 允許寫入
}
```

**Python**
```python
from enum import Enum, IntEnum, Flag, auto

# ✅ 數字列舉
class OrderStatus(IntEnum):
    PENDING = 0
    PROCESSING = 1
    COMPLETED = 2
    CANCELLED = 3

# ✅ 字串列舉
class UserRole(Enum):
    ADMIN = "ADMIN"
    MANAGER = "MANAGER"
    USER = "USER"
    GUEST = "GUEST"

# ✅ 自動編號
class Priority(Enum):
    LOW = auto()
    MEDIUM = auto()
    HIGH = auto()
    URGENT = auto()

# 使用
def handle_order(order):
    if order.status == OrderStatus.CANCELLED:
        refund_order(order)
    elif order.status == OrderStatus.COMPLETED:
        send_receipt(order)

def check_permission(user):
    if user.role == UserRole.ADMIN:
        grant_full_access()

# ✅ 使用 Flag 處理可組合的選項
class Permission(Flag):
    NONE = 0
    READ = auto()
    WRITE = auto()
    EXECUTE = auto()
    DELETE = auto()

# 可以組合使用
permissions = Permission.READ | Permission.WRITE
if Permission.WRITE in permissions:
    # 允許寫入
    pass
```

---

#### **方案 3：使用配置物件/類別**

當有**一組相關的常數**需要統一管理時，可以將它們組織成配置物件或類別。

**優點**：
- 邏輯分組，易於找到和修改
- 避免全域命名空間污染
- 方便集中管理和版本控制

**TypeScript**
```typescript
// ✅ 組織相關常數成配置物件
const PricingConfig = {
  unitPrice: 299,
  bulkOrder: {
    threshold: 10,
    discountRate: 0.9  // 10% off
  },
  highValue: {
    threshold: 5000,
    discountRate: 0.95  // 5% off
  },
  tax: {
    rate: 0.05,
    freeThreshold: 1000  // 超過此金額免稅
  }
} as const;  // as const 確保不可變

const ShippingConfig = {
  weight: {
    lightMax: 5,
    mediumMax: 10
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

// 使用
function calculateShipping(weight: number): number {
  if (weight <= ShippingConfig.weight.lightMax) {
    return ShippingConfig.fee.light;
  }
  if (weight <= ShippingConfig.weight.mediumMax) {
    return ShippingConfig.fee.medium;
  }
  return ShippingConfig.fee.heavy;
}

function estimateDelivery(weight: number): number {
  if (weight <= ShippingConfig.weight.lightMax) {
    return ShippingConfig.deliveryDays.light;
  }
  if (weight <= ShippingConfig.weight.mediumMax) {
    return ShippingConfig.deliveryDays.medium;
  }
  return ShippingConfig.deliveryDays.heavy;
}
```

**C#**
```csharp
// ✅ 使用靜態類別組織配置
public static class AppConfig
{
    public static class Payment
    {
        public const decimal HighAmountThreshold = 10000m;
        public const decimal StandardFeeRate = 0.03m;
        public const decimal AfterHoursFee = 50m;
        public const decimal MinimumPayment = 1m;
    }

    public static class BusinessHours
    {
        public const int Start = 9;
        public const int End = 17;
        public static readonly TimeSpan LunchBreakStart = TimeSpan.FromHours(12);
        public static readonly TimeSpan LunchBreakEnd = TimeSpan.FromHours(13);
    }

    public static class Retry
    {
        public const int MaxAttempts = 3;
        public const int DelaySeconds = 2;
        public const int TimeoutMs = 5000;
    }

    public static class UserLevel
    {
        public const int RegularMinScore = 0;
        public const int SilverMinScore = 50;
        public const int GoldMinScore = 80;
        
        public const decimal RegularDiscount = 0m;
        public const decimal SilverDiscount = 0.1m;
        public const decimal GoldDiscount = 0.15m;
    }
}

// 使用
if (amount > AppConfig.Payment.HighAmountThreshold)
{
    VerifyIdentity();
}

if (user.Score >= AppConfig.UserLevel.GoldMinScore)
{
    discount = AppConfig.UserLevel.GoldDiscount;
}
```

**Python**
```python
# ✅ 使用類別組織配置
class PaymentConfig:
    """支付相關配置"""
    HIGH_AMOUNT_THRESHOLD = 10000
    STANDARD_FEE_RATE = 0.03
    AFTER_HOURS_FEE = 50
    MINIMUM_PAYMENT = 1

class BusinessHoursConfig:
    """營業時間配置"""
    START = 9
    END = 17
    LUNCH_BREAK_START = 12
    LUNCH_BREAK_END = 13

class RetryConfig:
    """重試機制配置"""
    MAX_ATTEMPTS = 3
    DELAY_SECONDS = 2
    TIMEOUT_MS = 5000
    BACKOFF_MULTIPLIER = 2  # 指數退避倍數

class ShippingConfig:
    """運費配置"""
    WEIGHT_THRESHOLDS = {
        'light': 5,
        'medium': 10
    }
    FEES = {
        'light': 60,
        'medium': 120,
        'heavy': 180
    }
    DELIVERY_DAYS = {
        'light': 1,
        'medium': 2,
        'heavy': 3
    }

# 或使用字典（適合從檔案載入配置）
SHIPPING_CONFIG = {
    'weight_thresholds': {'light': 5, 'medium': 10},
    'fees': {'light': 60, 'medium': 120, 'heavy': 180},
    'delivery_days': {'light': 1, 'medium': 2, 'heavy': 3}
}

# 使用
def calculate_fee(amount):
    if amount > PaymentConfig.HIGH_AMOUNT_THRESHOLD:
        verify_identity()
    return amount * PaymentConfig.STANDARD_FEE_RATE

def calculate_shipping(weight):
    thresholds = ShippingConfig.WEIGHT_THRESHOLDS
    fees = ShippingConfig.FEES
    
    if weight <= thresholds['light']:
        return fees['light']
    elif weight <= thresholds['medium']:
        return fees['medium']
    else:
        return fees['heavy']
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
```
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

**改善效果**：
- ✅ 業務規則集中管理，修改時只需改一處
- ✅ 程式碼自我說明，不需要註解
- ✅ 容易測試：可以輕鬆調整 `LEVEL_REQUIREMENTS` 進行測試
- ✅ 容易擴充：新增 Platinum 等級只需添加一個配置項

---

#### **方案 4：使用環境變數或配置檔案（外部配置）**

對於可能會**根據環境改變**的值（開發、測試、生產），應該從外部配置載入，而非硬編碼。

**TypeScript (Node.js)**
```typescript
// ✅ 從環境變數讀取配置
const AppConfig = {
  database: {
    host: process.env.DB_HOST || 'localhost',
    port: parseInt(process.env.DB_PORT || '5432'),
    maxConnections: parseInt(process.env.DB_MAX_CONNECTIONS || '10')
  },
  api: {
    timeout: parseInt(process.env.API_TIMEOUT_MS || '5000'),
    maxRetries: parseInt(process.env.API_MAX_RETRIES || '3'),
    baseUrl: process.env.API_BASE_URL || 'https://api.example.com'
  },
  features: {
    enableCache: process.env.ENABLE_CACHE === 'true',
    debugMode: process.env.DEBUG === 'true'
  }
};

// 或從 JSON 配置檔讀取
import config from './config.json';

// config.json
{
  "database": {
    "maxConnections": 10,
    "timeout": 30000
  },
  "payment": {
    "feeRate": 0.03,
    "minimumAmount": 1
  }
}
```

**C# (.NET)**
```csharp
// ✅ 使用 appsettings.json
// appsettings.json
{
  "Payment": {
    "HighAmountThreshold": 10000,
    "StandardFeeRate": 0.03,
    "AfterHoursFee": 50
  },
  "BusinessHours": {
    "Start": 9,
    "End": 17
  }
}

// 使用強型別 Options Pattern
public class PaymentOptions
{
    public decimal HighAmountThreshold { get; set; }
    public decimal StandardFeeRate { get; set; }
    public decimal AfterHoursFee { get; set; }
}

public class PaymentService
{
    private readonly PaymentOptions _options;

    public PaymentService(IOptions<PaymentOptions> options)
    {
        _options = options.Value;
    }

    public void ProcessPayment(decimal amount)
    {
        if (amount > _options.HighAmountThreshold)
        {
            VerifyIdentity();
        }
    }
}
```

**Python**
```python
# ✅ 從環境變數讀取
import os

class Config:
    """應用程式配置"""
    DB_HOST = os.getenv('DB_HOST', 'localhost')
    DB_PORT = int(os.getenv('DB_PORT', '5432'))
    API_TIMEOUT_MS = int(os.getenv('API_TIMEOUT_MS', '5000'))
    DEBUG_MODE = os.getenv('DEBUG', 'false').lower() == 'true'

# 或從 YAML 檔案讀取
import yaml

with open('config.yaml', 'r') as f:
    config = yaml.safe_load(f)
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

[← 回目錄](./00-README.md) | [下一章：語言風格 →](./10-IdiomaticCode.md)
