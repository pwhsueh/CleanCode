## 八、類別與物件 (Classes & Objects)

> 「類別應該簡短！Classes should be short!」— Robert C. Martin

物件導向設計不只是把資料和函式包在一起，更是關於**職責分配、封裝、抽象與組合**。好的類別設計讓系統更容易理解、測試與擴充。

---

### ✅ 核心原則

#### 1. 單一職責原則 (Single Responsibility Principle, SRP)

**定義**：一個類別應該只有一個改變的理由。換句話說，**一個類別應該只負責一件事**。

**為什麼？**
- 職責越多，類別越難理解和維護
- 當一個職責需要修改時，可能會意外影響其他職責
- 違反 SRP 的類別通常會變得龐大且難以測試

#### ❌ 違反 SRP 的範例

**TypeScript**
```typescript
// ❌ 這個類別做了太多事：驗證、儲存、發送通知
class User {
  constructor(
    public email: string,
    public name: string
  ) {}

  // 職責 1: 驗證邏輯
  isValidEmail(): boolean {
    return this.email.includes("@");
  }

  // 職責 2: 資料庫操作
  save(): void {
    database.execute(`INSERT INTO users VALUES ('${this.email}', '${this.name}')`);
  }

  // 職責 3: 通知邏輯
  sendWelcomeEmail(): void {
    emailService.send(this.email, "Welcome!");
  }
}
```

**問題**：
- 如果資料庫邏輯改變（從 SQL 改成 NoSQL），需要修改 `User` 類別
- 如果通知方式改變（從 Email 改成 SMS），需要修改 `User` 類別
- 測試 `User` 時，需要模擬資料庫和 Email 服務，增加測試複雜度

#### ✅ 遵守 SRP 的範例

**TypeScript**
```typescript
// ✅ User 類別只負責表示使用者資料與業務規則
class User {
  constructor(
    public readonly email: string,
    public readonly name: string
  ) {
    this.validateEmail(email);
  }

  private validateEmail(email: string): void {
    if (!email.includes("@")) {
      throw new Error("Invalid email format");
    }
  }
}

// ✅ UserRepository 負責資料持久化
class UserRepository {
  save(user: User): void {
    database.execute(
      `INSERT INTO users VALUES ('${user.email}', '${user.name}')`
    );
  }

  findByEmail(email: string): User | null {
    // 查詢邏輯
  }
}

// ✅ NotificationService 負責發送通知
class NotificationService {
  sendWelcomeEmail(user: User): void {
    emailService.send(user.email, `Welcome, ${user.name}!`);
  }
}

// 使用
const user = new User("john@example.com", "John");
const repository = new UserRepository();
const notificationService = new NotificationService();

repository.save(user);
notificationService.sendWelcomeEmail(user);
```

**C#**
```csharp
// ✅ 每個類別只有一個職責
public class User
{
    public string Email { get; }
    public string Name { get; }

    public User(string email, string name)
    {
        if (!email.Contains("@"))
            throw new ArgumentException("Invalid email");
        
        Email = email;
        Name = name;
    }
}

public class UserRepository
{
    private readonly IDatabase _database;

    public UserRepository(IDatabase database)
    {
        _database = database;
    }

    public void Save(User user)
    {
        _database.Execute($"INSERT INTO users VALUES ('{user.Email}', '{user.Name}')");
    }
}

public class NotificationService
{
    private readonly IEmailService _emailService;

    public NotificationService(IEmailService emailService)
    {
        _emailService = emailService;
    }

    public void SendWelcomeEmail(User user)
    {
        _emailService.Send(user.Email, $"Welcome, {user.Name}!");
    }
}
```

**Python**
```python
# ✅ 職責清晰分離
class User:
    def __init__(self, email: str, name: str):
        if '@' not in email:
            raise ValueError("Invalid email format")
        self.email = email
        self.name = name

class UserRepository:
    def __init__(self, database):
        self.database = database
    
    def save(self, user: User):
        self.database.execute(
            f"INSERT INTO users VALUES ('{user.email}', '{user.name}')"
        )
    
    def find_by_email(self, email: str) -> User | None:
        # 查詢邏輯
        pass

class NotificationService:
    def __init__(self, email_service):
        self.email_service = email_service
    
    def send_welcome_email(self, user: User):
        self.email_service.send(user.email, f"Welcome, {user.name}!")
```

---

#### 2. 封裝 (Encapsulation)：保護內部狀態

**定義**：隱藏類別的內部實作細節，只透過公開的介面與外界互動。

**為什麼？**
- 防止外部程式碼任意修改物件的內部狀態，導致資料不一致
- 讓類別的實作可以自由改變，而不影響使用者
- 提供驗證和業務規則的統一入口

#### ❌ 缺乏封裝的範例

**TypeScript**
```typescript
// ❌ 所有欄位都是 public，任何人都可以隨意修改
class BankAccount {
  public balance: number = 0;
  public isLocked: boolean = false;
}

// 外部可以直接修改，繞過所有驗證
const account = new BankAccount();
account.balance = -1000; // 餘額變成負數！
account.isLocked = false; // 即使帳戶應該被鎖定
```

#### ✅ 良好封裝的範例

**TypeScript**
```typescript
// ✅ 透過私有欄位和公開方法來控制狀態變更
class BankAccount {
  private balance: number = 0;
  private isLocked: boolean = false;

  // 只提供必要的公開方法
  deposit(amount: number): void {
    if (this.isLocked) {
      throw new Error("Account is locked");
    }
    if (amount <= 0) {
      throw new Error("Deposit amount must be positive");
    }
    this.balance += amount;
  }

  withdraw(amount: number): void {
    if (this.isLocked) {
      throw new Error("Account is locked");
    }
    if (amount > this.balance) {
      throw new Error("Insufficient funds");
    }
    this.balance -= amount;
  }

  // 只允許讀取餘額，不允許直接修改
  getBalance(): number {
    return this.balance;
  }

  lock(): void {
    this.isLocked = true;
  }
}

// 使用
const account = new BankAccount();
account.deposit(1000); // ✅ 透過方法操作
// account.balance = -1000; // ❌ 編譯錯誤：balance 是私有的
```

**C#**
```csharp
// ✅ 使用 private 欄位和 public 方法
public class BankAccount
{
    private decimal _balance = 0;
    private bool _isLocked = false;

    public void Deposit(decimal amount)
    {
        if (_isLocked)
            throw new InvalidOperationException("Account is locked");
        
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive");
        
        _balance += amount;
    }

    public void Withdraw(decimal amount)
    {
        if (_isLocked)
            throw new InvalidOperationException("Account is locked");
        
        if (amount > _balance)
            throw new InvalidOperationException("Insufficient funds");
        
        _balance -= amount;
    }

    public decimal GetBalance() => _balance;

    public void Lock() => _isLocked = true;
}
```

**Python**
```python
# ✅ 使用 _ 前綴表示私有屬性（慣例）
class BankAccount:
    def __init__(self):
        self._balance = 0
        self._is_locked = False
    
    def deposit(self, amount: float):
        if self._is_locked:
            raise Exception("Account is locked")
        if amount <= 0:
            raise ValueError("Amount must be positive")
        self._balance += amount
    
    def withdraw(self, amount: float):
        if self._is_locked:
            raise Exception("Account is locked")
        if amount > self._balance:
            raise Exception("Insufficient funds")
        self._balance -= amount
    
    @property
    def balance(self) -> float:
        """只讀屬性"""
        return self._balance
    
    def lock(self):
        self._is_locked = True
```

---

#### 3. 小型類別優於大型類別 (Small Classes)

**經驗法則**：
- 一個類別應該在 **100-200 行**以內
- 如果一個類別超過 300 行，很可能違反了 SRP
- 類別的成員變數應該保持在**合理範圍內**
  - 如果成員變數過多(例如超過 10-15 個),可能表示類別承擔了太多職責
  - 但對於資料模型類別(DTO/Entity)來說,擁有較多屬性是正常的
  - 關鍵是要問:「這些變數是否都服務於同一個職責?」

**判斷標準**：如果你需要「向下捲動才能看完一個類別」,它可能太大了。

#### ❌ 龐大的類別

```typescript
// ❌ 超過 500 行的「神級類別」(God Class)
class OrderManager {
  validateOrder() { /* 50 lines */ }
  calculateTax() { /* 30 lines */ }
  applyDiscount() { /* 40 lines */ }
  checkInventory() { /* 60 lines */ }
  updateInventory() { /* 50 lines */ }
  processPayment() { /* 70 lines */ }
  sendConfirmationEmail() { /* 40 lines */ }
  generateInvoice() { /* 60 lines */ }
  updateCustomerPoints() { /* 30 lines */ }
  logTransaction() { /* 40 lines */ }
  // ... 還有更多方法
}
```

#### ✅ 拆分成多個小類別

```typescript
// ✅ 每個類別專注於一個領域
class OrderValidator {
  validate(order: Order): void { /* 驗證邏輯 */ }
}

class PricingCalculator {
  calculateTotal(order: Order): number { /* 計算邏輯 */ }
  applyDiscount(order: Order, discount: Discount): number { /* 折扣邏輯 */ }
}

class InventoryManager {
  checkAvailability(items: Item[]): boolean { /* 檢查庫存 */ }
  reserve(items: Item[]): void { /* 預留庫存 */ }
}

class PaymentProcessor {
  process(payment: Payment): PaymentResult { /* 支付邏輯 */ }
}

class OrderNotifier {
  sendConfirmation(order: Order): void { /* 發送通知 */ }
}

// 協調者類別 (Orchestrator)：組合這些小類別
class OrderService {
  constructor(
    private validator: OrderValidator,
    private calculator: PricingCalculator,
    private inventory: InventoryManager,
    private payment: PaymentProcessor,
    private notifier: OrderNotifier
  ) {}

  placeOrder(order: Order): void {
    this.validator.validate(order);
    const total = this.calculator.calculateTotal(order);
    this.inventory.reserve(order.items);
    this.payment.process({ amount: total, orderId: order.id });
    this.notifier.sendConfirmation(order);
  }
}
```

---

#### 4. 高內聚 (High Cohesion)

**定義**：類別中的方法和變數應該彼此相關，共同服務於類別的主要職責。

**判斷標準**：
- 類別的每個方法都應該操作至少一個成員變數
- 如果一個方法完全不使用類別的任何成員變數，它可能不屬於這個類別

#### ❌ 低內聚的類別

```typescript
// ❌ 方法之間沒有關聯
class Utilities {
  private userDatabase: Database;
  private logFile: FileStream;

  // 只使用 userDatabase
  getUserById(id: string): User {
    return this.userDatabase.query(`SELECT * FROM users WHERE id = ${id}`);
  }

  // 只使用 logFile
  writeLog(message: string): void {
    this.logFile.write(message);
  }

  // 完全不使用任何成員變數
  calculateTax(amount: number): number {
    return amount * 0.1;
  }
}
```

**問題**：這個類別的職責不清晰，方法之間沒有內聚性。

#### ✅ 高內聚的類別

```typescript
// ✅ 將職責分離成多個內聚的類別
class UserRepository {
  constructor(private database: Database) {}

  getUserById(id: string): User {
    return this.database.query(`SELECT * FROM users WHERE id = ${id}`);
  }

  saveUser(user: User): void {
    this.database.execute(`INSERT INTO users ...`);
  }
}

class Logger {
  constructor(private logFile: FileStream) {}

  writeLog(message: string): void {
    this.logFile.write(message);
  }

  writeError(error: Error): void {
    this.logFile.write(`ERROR: ${error.message}`);
  }
}

class TaxCalculator {
  private readonly TAX_RATE = 0.1;

  calculate(amount: number): number {
    return amount * this.TAX_RATE;
  }
}
```

---

#### 5. 依賴注入 (Dependency Injection)：讓類別更易測試

**定義**：類別不應該自己建立它所依賴的物件，而是從外部接收（通常透過建構子）。

**為什麼？**
- 讓類別與具體實作解耦，更容易替換依賴
- 測試時可以注入模擬 (Mock) 物件
- 符合「依賴反轉原則」(Dependency Inversion Principle)

#### ❌ 硬編碼依賴

```typescript
// ❌ UserService 直接建立依賴，難以測試
class UserService {
  private repository = new UserRepository(); // 寫死了具體實作
  private emailService = new EmailService(); // 寫死了具體實作

  createUser(email: string, name: string): void {
    const user = new User(email, name);
    this.repository.save(user);
    this.emailService.send(email, "Welcome!");
  }
}

// 測試時的困境：
// - 無法替換 UserRepository，測試會真的操作資料庫
// - 無法替換 EmailService，測試會真的發送 Email
```

#### ✅ 使用依賴注入

**TypeScript**
```typescript
// ✅ 透過建構子注入依賴
class UserService {
  constructor(
    private repository: UserRepository,
    private emailService: EmailService
  ) {}

  createUser(email: string, name: string): void {
    const user = new User(email, name);
    this.repository.save(user);
    this.emailService.send(email, "Welcome!");
  }
}

// 生產環境使用真實實作
const userService = new UserService(
  new UserRepository(),
  new EmailService()
);

// 測試環境注入 Mock 物件
const mockRepository = { save: jest.fn() };
const mockEmailService = { send: jest.fn() };
const userService = new UserService(mockRepository, mockEmailService);
```

**C#**
```csharp
// ✅ 依賴注入 + 介面
public interface IUserRepository
{
    void Save(User user);
}

public interface IEmailService
{
    void Send(string email, string message);
}

public class UserService
{
    private readonly IUserRepository _repository;
    private readonly IEmailService _emailService;

    // 依賴透過建構子注入
    public UserService(IUserRepository repository, IEmailService emailService)
    {
        _repository = repository;
        _emailService = emailService;
    }

    public void CreateUser(string email, string name)
    {
        var user = new User(email, name);
        _repository.Save(user);
        _emailService.Send(email, "Welcome!");
    }
}

// 測試
[Test]
public void CreateUser_ShouldSaveAndSendEmail()
{
    var mockRepo = new Mock<IUserRepository>();
    var mockEmail = new Mock<IEmailService>();
    var service = new UserService(mockRepo.Object, mockEmail.Object);

    service.CreateUser("test@example.com", "Test User");

    mockRepo.Verify(r => r.Save(It.IsAny<User>()), Times.Once);
    mockEmail.Verify(e => e.Send("test@example.com", "Welcome!"), Times.Once);
}
```

**Python**
```python
# ✅ 依賴注入
class UserService:
    def __init__(self, repository, email_service):
        self.repository = repository
        self.email_service = email_service
    
    def create_user(self, email: str, name: str):
        user = User(email, name)
        self.repository.save(user)
        self.email_service.send(email, "Welcome!")

# 測試
def test_create_user():
    mock_repo = Mock()
    mock_email = Mock()
    service = UserService(mock_repo, mock_email)
    
    service.create_user("test@example.com", "Test User")
    
    mock_repo.save.assert_called_once()
    mock_email.send.assert_called_once_with("test@example.com", "Welcome!")
```

---

#### 6. 組合優於繼承 (Composition over Inheritance)

**為什麼？**
- 繼承會建立緊密耦合，子類別依賴父類別的實作細節
- 深層繼承階層難以理解和維護
- 組合更靈活，可以在執行時動態改變行為

#### ❌ 過度使用繼承

```typescript
// ❌ 繼承階層過深，難以擴充
class Animal {
  eat() { console.log("Eating"); }
}

class Bird extends Animal {
  fly() { console.log("Flying"); }
}

class Penguin extends Bird {
  // 問題：企鵝不會飛，但繼承了 fly()
  fly() {
    throw new Error("Penguins can't fly!");
  }
}
```

#### ✅ 使用組合

```typescript
// ✅ 使用組合來組裝行為
interface Eater {
  eat(): void;
}

interface Flyer {
  fly(): void;
}

interface Swimmer {
  swim(): void;
}

class Bird implements Eater, Flyer {
  eat() { console.log("Eating"); }
  fly() { console.log("Flying"); }
}

class Penguin implements Eater, Swimmer {
  eat() { console.log("Eating"); }
  swim() { console.log("Swimming"); }
  // 不需要繼承不需要的 fly() 方法
}

class Duck implements Eater, Flyer, Swimmer {
  eat() { console.log("Eating"); }
  fly() { console.log("Flying"); }
  swim() { console.log("Swimming"); }
}
```

---

### 💡 類別設計的經驗法則

1. **類別應該簡短**：如果超過 200 行，考慮拆分
2. **命名應該精確**：如果類別名稱包含 "Manager"、"Processor"、"Handler"，可能職責不清
3. **成員變數應該合理**：如果成員變數過多(例如超過 10-15 個),可能表示類別承擔了太多職責。但對於資料模型類別(DTO/Entity)擁有較多屬性是正常的
4. **公開方法應該少**：一個類別不應該暴露超過 10 個公開方法
5. **避免「資料類別」**：只有 getter/setter 沒有行為的類別通常是設計不良的徵兆
6. **避免「神級類別」(God Class)**：一個類別不應該知道或控制太多事情

---

[← 回目錄](./README.md) | [下一章：錯誤處理 →](./ErrorHandling.md)
