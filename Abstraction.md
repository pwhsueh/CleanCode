## 十二、抽象層次與依賴反轉 (Abstraction & Dependency)

> 「高層模組不應該依賴於低層模組，兩者都應該依賴於抽象。」— Robert C. Martin

### ❓ 為什麼抽象層次與依賴反轉如此重要？

在軟體設計中，**抽象層次 (Abstraction Levels)** 和**依賴反轉 (Dependency Inversion)** 是構建可維護、可測試、可擴展系統的關鍵。它們讓程式碼更容易理解、修改和測試。

---

### 一、抽象層次 (Abstraction Levels)

#### 📖 什麼是抽象層次？

**抽象層次**是指程式碼中不同的概念層級。高層次的程式碼描述「做什麼」(What)，低層次的程式碼描述「怎麼做」(How)。

**層次分類**：
- **高層抽象**：業務邏輯、用例 (Use Cases)
- **中層抽象**：領域模型、服務層
- **低層抽象**：資料存取、技術細節

---

#### 💔 混亂的抽象層次

當一個函式或類別混合了不同層次的抽象，程式碼會變得難以理解和維護。

**❌ 不好的範例：混合抽象層次**

```typescript
// ❌ 這個函式混合了高層邏輯與低層細節
async function processOrder(orderId: string) {
  // 高層：業務邏輯
  console.log(`Processing order ${orderId}`);
  
  // 低層：SQL 查詢細節
  const connection = await mysql.createConnection({
    host: 'localhost',
    user: 'root',
    password: 'password',
    database: 'shop'
  });
  const [rows] = await connection.execute(
    'SELECT * FROM orders WHERE id = ?',
    [orderId]
  );
  const order = rows[0];
  
  // 中層：業務規則
  if (order.total > 1000) {
    order.requiresApproval = true;
  }
  
  // 低層：HTTP 請求細節
  const response = await fetch('https://payment-gateway.com/charge', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer sk_test_12345'
    },
    body: JSON.stringify({
      amount: order.total * 100,
      currency: 'usd',
      source: order.cardToken
    })
  });
  
  // 低層：JSON 解析
  const result = await response.json();
  
  // 高層：業務邏輯
  if (result.status === 'success') {
    console.log('Order processed successfully');
  }
}
```

**問題**：
- 難以理解整體流程（被技術細節淹沒）
- 難以測試（需要模擬資料庫和支付 API）
- 難以重用（邏輯與實作細節緊密耦合）
- 難以修改（改資料庫或支付方式需要改這個函式）

---

#### ✅ 清晰的抽象層次

**原則：一個函式應該只在一個抽象層次上操作**

**✅ 好的範例：分離抽象層次**

```typescript
// ✅ 高層：清晰的業務流程
async function processOrder(orderId: string): Promise<void> {
  const order = await orderRepository.findById(orderId);
  
  if (requiresApproval(order)) {
    await approvalService.requestApproval(order);
  }
  
  const payment = await paymentService.charge(order);
  
  if (payment.isSuccessful()) {
    await orderRepository.markAsProcessed(order);
    await notificationService.sendConfirmation(order);
  }
}

// ✅ 中層：業務規則
function requiresApproval(order: Order): boolean {
  return order.total > 1000;
}

// ✅ 低層：資料存取細節（封裝在 Repository）
class OrderRepository {
  async findById(id: string): Promise<Order> {
    const connection = await this.getConnection();
    const [rows] = await connection.execute(
      'SELECT * FROM orders WHERE id = ?',
      [id]
    );
    return this.mapToOrder(rows[0]);
  }
  
  private async getConnection() {
    return mysql.createConnection(this.config);
  }
  
  private mapToOrder(row: any): Order {
    return new Order(row.id, row.total, row.status);
  }
}

// ✅ 低層：支付細節（封裝在 Service）
class PaymentService {
  async charge(order: Order): Promise<Payment> {
    const response = await this.gateway.charge({
      amount: order.total,
      source: order.cardToken
    });
    return new Payment(response.id, response.status);
  }
  
  private gateway = new StripeGateway(this.config);
}
```

**優點**：
- ✅ 高層函式讀起來像一篇文章
- ✅ 每個函式只專注於一個層次
- ✅ 容易測試（可以模擬 Repository 和 Service）
- ✅ 容易修改（改資料庫只需改 Repository）

---

#### 🎯 抽象層次的實踐原則

##### **1. 降階原則 (Stepdown Rule)**

程式碼應該像一篇文章，從高層到低層逐步展開。

```typescript
// ✅ 從高到低的閱讀流程
async function handleUserRegistration(data: RegistrationData) {
  // 第一層：主要流程
  const user = await createUser(data);
  await sendWelcomeEmail(user);
  await logRegistration(user);
}

async function createUser(data: RegistrationData): Promise<User> {
  // 第二層：用戶創建步驟
  validateRegistrationData(data);
  const hashedPassword = await hashPassword(data.password);
  const user = new User(data.email, hashedPassword);
  await userRepository.save(user);
  return user;
}

function validateRegistrationData(data: RegistrationData): void {
  // 第三層：具體驗證邏輯
  if (!isValidEmail(data.email)) {
    throw new ValidationError('Invalid email');
  }
  if (!isStrongPassword(data.password)) {
    throw new ValidationError('Weak password');
  }
}
```

##### **2. 單一抽象層次原則 (Single Level of Abstraction Principle)**

一個函式內的所有語句應該在同一個抽象層次。

```csharp
// ❌ 混合抽象層次
public void ProcessPayment(Order order)
{
    // 高層
    if (order.Total > 1000)
    {
        // 低層：SQL 細節
        var connection = new SqlConnection(connectionString);
        connection.Open();
        var command = new SqlCommand("INSERT INTO approvals...", connection);
        command.ExecuteNonQuery();
    }
    
    // 中層：業務邏輯
    var discount = CalculateDiscount(order);
    order.Total -= discount;
}

// ✅ 單一抽象層次
public void ProcessPayment(Order order)
{
    // 所有語句都在同一個抽象層次
    if (RequiresApproval(order))
    {
        RequestApproval(order);
    }
    
    ApplyDiscount(order);
    ChargePayment(order);
}

private bool RequiresApproval(Order order)
{
    return order.Total > ApprovalThreshold;
}

private void RequestApproval(Order order)
{
    approvalRepository.Create(new Approval(order.Id));
}
```

##### **3. 依賴方向：高層依賴抽象，不依賴細節**

```python
# ❌ 高層直接依賴低層細節
class OrderService:
    def process_order(self, order_id):
        # 直接依賴 MySQL 實作
        connection = mysql.connector.connect(
            host="localhost",
            user="root",
            password="password"
        )
        cursor = connection.cursor()
        cursor.execute("SELECT * FROM orders WHERE id = %s", (order_id,))
        # ...

# ✅ 高層依賴抽象介面
class OrderService:
    def __init__(self, repository: OrderRepository):
        self.repository = repository
    
    def process_order(self, order_id: str) -> None:
        order = self.repository.find_by_id(order_id)
        # 業務邏輯...
        self.repository.save(order)

# 低層實作抽象介面
class MySQLOrderRepository(OrderRepository):
    def find_by_id(self, order_id: str) -> Order:
        # MySQL 具體實作
        pass
```

---

### 二、依賴反轉原則 (Dependency Inversion Principle)

#### 📖 什麼是依賴反轉？

**依賴反轉原則 (DIP)** 是 SOLID 原則中的 "D"，它指出：

1. **高層模組不應該依賴低層模組，兩者都應該依賴於抽象**
2. **抽象不應該依賴於細節，細節應該依賴於抽象**

這個原則「反轉」了傳統的依賴方向：不是高層依賴低層，而是兩者都依賴於抽象介面。

---

#### 💔 傳統的依賴方向（有問題）

```typescript
// ❌ 高層直接依賴低層具體實作
class UserService {
  private emailSender: SmtpEmailSender;  // 直接依賴具體類別
  private database: MySQLDatabase;       // 直接依賴具體類別
  
  constructor() {
    this.emailSender = new SmtpEmailSender();
    this.database = new MySQLDatabase();
  }
  
  register(email: string, password: string): void {
    const user = new User(email, password);
    this.database.insert(user);           // 寫死 MySQL
    this.emailSender.send(email, "Welcome!");  // 寫死 SMTP
  }
}
```

**問題**：
- ❌ 無法替換實作（例如改用 SendGrid 發信、改用 PostgreSQL）
- ❌ 難以測試（必須真的連接資料庫和 Email 服務）
- ❌ 高耦合（修改低層會影響高層）

---

#### ✅ 反轉的依賴方向（正確）

```typescript
// ✅ 定義抽象介面
interface EmailSender {
  send(email: string, message: string): Promise<void>;
}

interface UserRepository {
  save(user: User): Promise<void>;
  findByEmail(email: string): Promise<User | null>;
}

// ✅ 高層依賴抽象
class UserService {
  constructor(
    private emailSender: EmailSender,      // 依賴抽象
    private repository: UserRepository     // 依賴抽象
  ) {}
  
  async register(email: string, password: string): Promise<void> {
    const user = new User(email, password);
    await this.repository.save(user);
    await this.emailSender.send(email, "Welcome!");
  }
}

// ✅ 低層實作抽象
class SmtpEmailSender implements EmailSender {
  async send(email: string, message: string): Promise<void> {
    // SMTP 實作細節
  }
}

class SendGridEmailSender implements EmailSender {
  async send(email: string, message: string): Promise<void> {
    // SendGrid 實作細節
  }
}

class MySQLUserRepository implements UserRepository {
  async save(user: User): Promise<void> {
    // MySQL 實作細節
  }
  
  async findByEmail(email: string): Promise<User | null> {
    // MySQL 查詢實作
  }
}

// ✅ 組裝（在應用程式入口）
const emailSender = new SmtpEmailSender();
const repository = new MySQLUserRepository();
const userService = new UserService(emailSender, repository);

// ✅ 測試時可以輕鬆替換
const mockEmailSender = new MockEmailSender();
const mockRepository = new InMemoryUserRepository();
const userService = new UserService(mockEmailSender, mockRepository);
```

**優點**：
- ✅ 可以輕鬆替換實作（插拔式設計）
- ✅ 容易測試（注入 Mock 物件）
- ✅ 低耦合（高層不知道低層的實作細節）
- ✅ 符合開放封閉原則（新增實作不需修改高層程式碼）

---

#### 🛠️ 各語言的依賴反轉實作

##### **TypeScript**

**1. 使用 Interface（介面）**

```typescript
// ✅ 定義抽象介面
interface EmailSender {
  send(email: string, message: string): Promise<void>;
}

interface PaymentGateway {
  charge(amount: number, token: string): Promise<PaymentResult>;
}

// ✅ 高層依賴介面
class OrderService {
  constructor(
    private emailSender: EmailSender,
    private paymentGateway: PaymentGateway
  ) {}
  
  async processOrder(order: Order): Promise<void> {
    const result = await this.paymentGateway.charge(
      order.total,
      order.cardToken
    );
    
    if (result.isSuccessful) {
      await this.emailSender.send(
        order.customerEmail,
        'Order confirmed'
      );
    }
  }
}

// ✅ 多種實作
class SmtpSender implements EmailSender {
  async send(email: string, message: string): Promise<void> {
    console.log(`SMTP sending to ${email}: ${message}`);
  }
}

class SendGridSender implements EmailSender {
  async send(email: string, message: string): Promise<void> {
    console.log(`SendGrid sending to ${email}: ${message}`);
  }
}

class StripGateway implements PaymentGateway {
  async charge(amount: number, token: string): Promise<PaymentResult> {
    // Stripe API 呼叫
    return new PaymentResult(true, 'txn_123');
  }
}

class PayPalGateway implements PaymentGateway {
  async charge(amount: number, token: string): Promise<PaymentResult> {
    // PayPal API 呼叫
    return new PaymentResult(true, 'ppl_456');
  }
}
```

**2. 使用依賴注入容器（進階）**

```typescript
// 使用 InversifyJS
import { Container, injectable, inject } from 'inversify';

// 定義識別符號
const TYPES = {
  EmailSender: Symbol.for('EmailSender'),
  PaymentGateway: Symbol.for('PaymentGateway'),
  OrderService: Symbol.for('OrderService')
};

// 標記為可注入
@injectable()
class OrderService {
  constructor(
    @inject(TYPES.EmailSender) private emailSender: EmailSender,
    @inject(TYPES.PaymentGateway) private paymentGateway: PaymentGateway
  ) {}
}

@injectable()
class SmtpSender implements EmailSender {
  async send(email: string, message: string): Promise<void> {
    // 實作
  }
}

// 配置容器
const container = new Container();
container.bind<EmailSender>(TYPES.EmailSender).to(SmtpSender);
container.bind<PaymentGateway>(TYPES.PaymentGateway).to(StripGateway);
container.bind<OrderService>(TYPES.OrderService).to(OrderService);

// 取得實例（自動解析依賴）
const orderService = container.get<OrderService>(TYPES.OrderService);
```

---

##### **C#**

**1. 使用 Interface 和建構子注入**

```csharp
// ✅ 定義抽象介面
public interface IEmailSender
{
    Task SendAsync(string email, string message);
}

public interface IPaymentGateway
{
    Task<PaymentResult> ChargeAsync(decimal amount, string token);
}

// ✅ 高層依賴介面
public class OrderService
{
    private readonly IEmailSender _emailSender;
    private readonly IPaymentGateway _paymentGateway;

    public OrderService(IEmailSender emailSender, IPaymentGateway paymentGateway)
    {
        _emailSender = emailSender;
        _paymentGateway = paymentGateway;
    }

    public async Task ProcessOrderAsync(Order order)
    {
        var result = await _paymentGateway.ChargeAsync(
            order.Total,
            order.CardToken
        );

        if (result.IsSuccessful)
        {
            await _emailSender.SendAsync(
                order.CustomerEmail,
                "Order confirmed"
            );
        }
    }
}

// ✅ 具體實作
public class SmtpSender : IEmailSender
{
    public async Task SendAsync(string email, string message)
    {
        Console.WriteLine($"SMTP sending to {email}: {message}");
        await Task.CompletedTask;
    }
}

public class StripeGateway : IPaymentGateway
{
    public async Task<PaymentResult> ChargeAsync(decimal amount, string token)
    {
        // Stripe API 呼叫
        return new PaymentResult(true, "txn_123");
    }
}
```

**2. 使用 .NET 的依賴注入容器**

```csharp
// Startup.cs 或 Program.cs
public void ConfigureServices(IServiceCollection services)
{
    // 註冊服務
    services.AddScoped<IEmailSender, SmtpSender>();
    services.AddScoped<IPaymentGateway, StripeGateway>();
    services.AddScoped<OrderService>();
    
    // 可以輕鬆切換實作
    // services.AddScoped<IEmailSender, SendGridSender>();
    // services.AddScoped<IPaymentGateway, PayPalGateway>();
}

// Controller 中使用
public class OrderController : ControllerBase
{
    private readonly OrderService _orderService;

    // 自動注入
    public OrderController(OrderService orderService)
    {
        _orderService = orderService;
    }

    [HttpPost]
    public async Task<IActionResult> CreateOrder(Order order)
    {
        await _orderService.ProcessOrderAsync(order);
        return Ok();
    }
}
```

---

##### **Python**

**1. 使用抽象基類 (ABC)**

```python
from abc import ABC, abstractmethod
from typing import Protocol

# ✅ 使用 ABC 定義抽象介面
class EmailSender(ABC):
    @abstractmethod
    async def send(self, email: str, message: str) -> None:
        pass

class PaymentGateway(ABC):
    @abstractmethod
    async def charge(self, amount: float, token: str) -> 'PaymentResult':
        pass

# ✅ 高層依賴抽象
class OrderService:
    def __init__(
        self,
        email_sender: EmailSender,
        payment_gateway: PaymentGateway
    ):
        self.email_sender = email_sender
        self.payment_gateway = payment_gateway
    
    async def process_order(self, order: Order) -> None:
        result = await self.payment_gateway.charge(
            order.total,
            order.card_token
        )
        
        if result.is_successful:
            await self.email_sender.send(
                order.customer_email,
                "Order confirmed"
            )

# ✅ 具體實作
class SmtpSender(EmailSender):
    async def send(self, email: str, message: str) -> None:
        print(f"SMTP sending to {email}: {message}")

class SendGridSender(EmailSender):
    async def send(self, email: str, message: str) -> None:
        print(f"SendGrid sending to {email}: {message}")

class StripeGateway(PaymentGateway):
    async def charge(self, amount: float, token: str) -> PaymentResult:
        # Stripe API 呼叫
        return PaymentResult(True, "txn_123")
```

**2. 使用 Protocol (結構性子型別，Python 3.8+)**

```python
from typing import Protocol

# ✅ 使用 Protocol（類似 TypeScript 的 interface）
class EmailSender(Protocol):
    async def send(self, email: str, message: str) -> None: ...

class PaymentGateway(Protocol):
    async def charge(self, amount: float, token: str) -> PaymentResult: ...

# ✅ 不需要顯式繼承，只要有相同方法即可
class SmtpSender:
    async def send(self, email: str, message: str) -> None:
        print(f"SMTP sending to {email}")

# SmtpSender 自動符合 EmailSender Protocol
```

**3. 使用依賴注入框架（dependency-injector）**

```python
from dependency_injector import containers, providers

# ✅ 定義容器
class Container(containers.DeclarativeContainer):
    config = providers.Configuration()
    
    email_sender = providers.Singleton(
        SmtpSender,
        host=config.smtp.host,
        port=config.smtp.port
    )
    
    payment_gateway = providers.Singleton(
        StripeGateway,
        api_key=config.stripe.api_key
    )
    
    order_service = providers.Factory(
        OrderService,
        email_sender=email_sender,
        payment_gateway=payment_gateway
    )

# ✅ 使用
container = Container()
container.config.from_yaml('config.yaml')

order_service = container.order_service()
await order_service.process_order(order)
```

---

### 🎯 實戰指南

#### **1. 何時使用抽象？**

**需要抽象的情境**：
- ✅ 可能有多種實作（如支付方式、通知方式）
- ✅ 需要測試時替換（如資料庫、外部 API）
- ✅ 未來可能變更（如從 MySQL 換到 PostgreSQL）
- ✅ 第三方服務（如 Email、SMS、雲端儲存）

**不需要過度抽象的情境**：
- ❌ 只會有一種實作且不太可能改變
- ❌ 簡單的值物件 (Value Objects)
- ❌ 工具函式（如字串處理、日期格式化）

```typescript
// ❌ 過度抽象：為簡單工具建立介面
interface StringFormatter {
  format(str: string): string;
}

class UpperCaseFormatter implements StringFormatter {
  format(str: string): string {
    return str.toUpperCase();
  }
}

// ✅ 簡單函式即可
function formatUpperCase(str: string): string {
  return str.toUpperCase();
}
```

---

#### **2. 介面隔離原則 (Interface Segregation)**

不要讓客戶端依賴它不需要的方法。

```csharp
// ❌ 胖介面：強迫實作不需要的方法
public interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
}

public class Robot : IWorker
{
    public void Work() { /* 機器人工作 */ }
    public void Eat() { throw new NotImplementedException(); }  // 機器人不吃飯！
    public void Sleep() { throw new NotImplementedException(); } // 機器人不睡覺！
}

// ✅ 拆分成小介面
public interface IWorkable
{
    void Work();
}

public interface IEatable
{
    void Eat();
}

public interface ISleepable
{
    void Sleep();
}

public class Robot : IWorkable
{
    public void Work() { /* 機器人工作 */ }
}

public class Human : IWorkable, IEatable, ISleepable
{
    public void Work() { /* 人類工作 */ }
    public void Eat() { /* 人類吃飯 */ }
    public void Sleep() { /* 人類睡覺 */ }
}
```

---

#### **3. 依賴注入的三種方式**

##### **方式 1：建構子注入（推薦）**

```typescript
// ✅ 建構子注入：依賴必須提供，不可變
class OrderService {
  constructor(
    private readonly emailSender: EmailSender,
    private readonly repository: OrderRepository
  ) {}
}
```

##### **方式 2：屬性注入（較少使用）**

```csharp
// 屬性注入：依賴是可選的
public class OrderService
{
    public IEmailSender EmailSender { get; set; }
    
    public void ProcessOrder(Order order)
    {
        EmailSender?.SendAsync(order.Email, "Confirmed");
    }
}
```

##### **方式 3：方法注入（特殊情況）**

```python
# 方法注入：每次呼叫時提供依賴
class OrderService:
    def process_order(self, order: Order, email_sender: EmailSender):
        # 使用傳入的 email_sender
        email_sender.send(order.email, "Confirmed")
```

---

### 💡 最佳實踐總結

| 原則                   | 說明                         | 範例                                  |
| ---------------------- | ---------------------------- | ------------------------------------- |
| **單一抽象層次**       | 一個函式內的語句應在同一層次 | 不要混合 SQL 和業務邏輯               |
| **降階原則**           | 程式碼從高層到低層逐步展開   | 主函式 → 子函式 → 細節函式            |
| **依賴抽象不依賴細節** | 高層模組依賴介面而非具體類別 | 依賴 `IEmailSender` 而非 `SmtpSender` |
| **介面隔離**           | 保持介面小而專注             | 拆分大介面成多個小介面                |
| **建構子注入**         | 透過建構子提供依賴           | 讓依賴明確且不可變                    |
| **適度抽象**           | 不要過度設計                 | 只在需要時才建立抽象                  |

---

### 🔑 核心要點

1. **抽象層次讓程式碼更易讀**：高層描述「做什麼」，低層描述「怎麼做」
2. **依賴反轉讓程式碼更靈活**：可以輕鬆替換實作、容易測試
3. **介面是契約**：定義「能做什麼」而非「怎麼做」
4. **建構子注入是首選**：依賴明確、不可變、易於測試
5. **避免過度抽象**：只在真正需要時才建立抽象層
6. **測試是最好的驗證**：如果難以測試，通常是設計有問題

**記住**：「好的架構讓重要的決定可以延遲。」— Uncle Bob。透過抽象和依賴反轉，你可以延遲選擇具體的技術實作，讓系統更容易適應變化。

---

[← 回目錄](./README.md) | [下一章：模組化 →](./Modularization.md)
