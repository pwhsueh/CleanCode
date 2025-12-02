## 十三、模組化與測試性 (Modularization & Testability)

> 「如果你的程式碼難以測試，那是因為它設計不良。」— Michael Feathers

### ❓ 為什麼模組化與測試性如此重要？

**模組化 (Modularization)** 和**測試性 (Testability)** 是軟體品質的兩大基石。好的模組化設計不僅讓程式碼易於理解和維護，更是高測試性的前提。而高測試性則確保程式碼的正確性和穩定性。

---

### 一、模組化 (Modularization)

#### 📖 什麼是模組化？

**模組化**是指將大型系統分解成多個獨立、可重用、職責單一的模組（或稱為元件、套件）。每個模組都有明確的邊界和介面。

**模組化的核心原則**：
- **高內聚 (High Cohesion)**：模組內部的元素緊密相關
- **低耦合 (Low Coupling)**：模組之間的依賴關係最小化
- **單一職責 (Single Responsibility)**：每個模組只負責一件事
- **明確介面 (Clear Interface)**：模組之間透過定義良好的介面溝通

---

#### 💔 缺乏模組化的問題

**❌ 單體式程式碼 (Monolithic Code)**

```typescript
// ❌ 所有程式碼混在一起
class Application {
  async handleUserRegistration(email: string, password: string) {
    // 1. 驗證邏輯
    if (!email.includes('@')) {
      throw new Error('Invalid email');
    }
    if (password.length < 8) {
      throw new Error('Password too short');
    }
    
    // 2. 密碼處理
    const salt = crypto.randomBytes(16).toString('hex');
    const hash = crypto.pbkdf2Sync(password, salt, 1000, 64, 'sha512').toString('hex');
    
    // 3. 資料庫操作
    const connection = await mysql.createConnection({
      host: 'localhost',
      user: 'root',
      password: 'password',
      database: 'myapp'
    });
    await connection.execute(
      'INSERT INTO users (email, password_hash, salt) VALUES (?, ?, ?)',
      [email, hash, salt]
    );
    
    // 4. 發送 Email
    const transporter = nodemailer.createTransport({
      service: 'gmail',
      auth: {
        user: 'noreply@example.com',
        pass: 'app-password'
      }
    });
    await transporter.sendMail({
      from: 'noreply@example.com',
      to: email,
      subject: 'Welcome!',
      text: 'Thanks for registering'
    });
    
    // 5. 記錄日誌
    fs.appendFileSync('app.log', `User registered: ${email}\n`);
  }
}
```

**問題**：
- ❌ 難以理解（所有邏輯混在一起）
- ❌ 難以測試（需要模擬資料庫、Email、檔案系統）
- ❌ 難以重用（驗證邏輯、密碼處理無法在其他地方使用）
- ❌ 難以修改（改 Email 發送方式要動這個大函式）
- ❌ 違反單一職責原則

---

#### ✅ 良好的模組化設計

**原則：按照職責和層次分離模組**

```
src/
 ├── domain/              # 領域層（業務邏輯）
 │   ├── entities/        # 實體
 │   │   └── User.ts
 │   ├── services/        # 領域服務
 │   │   └── UserService.ts
 │   └── repositories/    # 資料存取介面
 │       └── IUserRepository.ts
 │
 ├── application/         # 應用層（用例/場景）
 │   └── useCases/
 │       └── RegisterUserUseCase.ts
 │
 ├── infrastructure/      # 基礎設施層（技術實作）
 │   ├── database/
 │   │   └── MySQLUserRepository.ts
 │   ├── email/
 │   │   └── NodemailerEmailService.ts
 │   └── logging/
 │       └── FileLogger.ts
 │
 ├── presentation/        # 表現層（API/UI）
 │   └── controllers/
 │       └── UserController.ts
 │
 └── tests/              # 測試
     ├── unit/
     ├── integration/
     └── e2e/
```

**重構後的模組化程式碼**：

**1. 領域層：User 實體**
```typescript
// domain/entities/User.ts
export class User {
  constructor(
    public readonly id: string,
    public readonly email: string,
    private readonly passwordHash: string
  ) {}

  static create(email: string, passwordHash: string): User {
    return new User(
      crypto.randomUUID(),
      email,
      passwordHash
    );
  }

  verifyPassword(password: string, hasher: IPasswordHasher): boolean {
    return hasher.verify(password, this.passwordHash);
  }
}
```

**2. 領域層：Repository 介面**
```typescript
// domain/repositories/IUserRepository.ts
export interface IUserRepository {
  save(user: User): Promise<void>;
  findByEmail(email: string): Promise<User | null>;
  existsByEmail(email: string): Promise<boolean>;
}
```

**3. 領域層：驗證服務**
```typescript
// domain/services/UserValidator.ts
export class UserValidator {
  validateEmail(email: string): void {
    if (!email.includes('@')) {
      throw new ValidationError('Invalid email format');
    }
    
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      throw new ValidationError('Invalid email format');
    }
  }

  validatePassword(password: string): void {
    if (password.length < 8) {
      throw new ValidationError('Password must be at least 8 characters');
    }
    
    if (!/[A-Z]/.test(password)) {
      throw new ValidationError('Password must contain uppercase letter');
    }
    
    if (!/[0-9]/.test(password)) {
      throw new ValidationError('Password must contain number');
    }
  }
}
```

**4. 領域層：密碼處理介面**
```typescript
// domain/services/IPasswordHasher.ts
export interface IPasswordHasher {
  hash(password: string): Promise<string>;
  verify(password: string, hash: string): boolean;
}
```

**5. 應用層：註冊用例**
```typescript
// application/useCases/RegisterUserUseCase.ts
export class RegisterUserUseCase {
  constructor(
    private userRepository: IUserRepository,
    private passwordHasher: IPasswordHasher,
    private emailService: IEmailService,
    private logger: ILogger,
    private validator: UserValidator
  ) {}

  async execute(email: string, password: string): Promise<User> {
    // 1. 驗證
    this.validator.validateEmail(email);
    this.validator.validatePassword(password);

    // 2. 檢查重複
    const exists = await this.userRepository.existsByEmail(email);
    if (exists) {
      throw new DomainError('Email already registered');
    }

    // 3. 建立用戶
    const passwordHash = await this.passwordHasher.hash(password);
    const user = User.create(email, passwordHash);

    // 4. 儲存
    await this.userRepository.save(user);

    // 5. 發送歡迎信
    await this.emailService.sendWelcomeEmail(email);

    // 6. 記錄
    this.logger.info(`User registered: ${email}`);

    return user;
  }
}
```

**6. 基礎設施層：MySQL Repository 實作**
```typescript
// infrastructure/database/MySQLUserRepository.ts
export class MySQLUserRepository implements IUserRepository {
  constructor(private connection: mysql.Connection) {}

  async save(user: User): Promise<void> {
    await this.connection.execute(
      'INSERT INTO users (id, email, password_hash) VALUES (?, ?, ?)',
      [user.id, user.email, user.passwordHash]
    );
  }

  async findByEmail(email: string): Promise<User | null> {
    const [rows] = await this.connection.execute(
      'SELECT * FROM users WHERE email = ?',
      [email]
    );
    return rows[0] ? this.mapToUser(rows[0]) : null;
  }

  async existsByEmail(email: string): Promise<boolean> {
    const [rows] = await this.connection.execute(
      'SELECT COUNT(*) as count FROM users WHERE email = ?',
      [email]
    );
    return rows[0].count > 0;
  }

  private mapToUser(row: any): User {
    return new User(row.id, row.email, row.password_hash);
  }
}
```

**7. 基礎設施層：Email 服務實作**
```typescript
// infrastructure/email/NodemailerEmailService.ts
export class NodemailerEmailService implements IEmailService {
  private transporter: nodemailer.Transporter;

  constructor(config: EmailConfig) {
    this.transporter = nodemailer.createTransport(config);
  }

  async sendWelcomeEmail(email: string): Promise<void> {
    await this.transporter.sendMail({
      from: 'noreply@example.com',
      to: email,
      subject: 'Welcome!',
      text: 'Thanks for registering'
    });
  }
}
```

**8. 表現層：Controller**
```typescript
// presentation/controllers/UserController.ts
export class UserController {
  constructor(private registerUseCase: RegisterUserUseCase) {}

  async register(req: Request, res: Response) {
    try {
      const { email, password } = req.body;
      const user = await this.registerUseCase.execute(email, password);
      res.status(201).json({ id: user.id, email: user.email });
    } catch (error) {
      if (error instanceof ValidationError) {
        res.status(400).json({ error: error.message });
      } else if (error instanceof DomainError) {
        res.status(409).json({ error: error.message });
      } else {
        res.status(500).json({ error: 'Internal server error' });
      }
    }
  }
}
```

**優點**：
- ✅ 職責清晰（每個模組只做一件事）
- ✅ 易於理解（可以獨立閱讀每個模組）
- ✅ 易於測試（可以單獨測試每個模組）
- ✅ 易於重用（驗證邏輯可在其他地方使用）
- ✅ 易於修改（改 Email 只需替換 EmailService）
- ✅ 符合 SOLID 原則

---

### 二、測試性 (Testability)

#### 📖 什麼是測試性？

**測試性**是指程式碼容易被測試的程度。高測試性的程式碼具有以下特徵：
- 容易建立測試實例
- 容易模擬依賴
- 容易驗證結果
- 測試執行快速
- 測試獨立且可重複

---

#### 💔 難以測試的程式碼

**❌ 範例：緊密耦合的程式碼**

```typescript
// ❌ 難以測試
class OrderService {
  processOrder(orderId: string) {
    // 1. 直接建立依賴（無法替換）
    const db = new MySQLDatabase();
    const emailService = new SmtpEmailService();
    const paymentGateway = new StripeGateway();
    
    // 2. 使用全域狀態
    const config = global.appConfig;
    
    // 3. 依賴具體時間
    const now = new Date();
    if (now.getHours() < 9 || now.getHours() > 17) {
      throw new Error('Outside business hours');
    }
    
    // 4. 直接呼叫外部 API
    const order = db.query(`SELECT * FROM orders WHERE id = ${orderId}`);
    const result = paymentGateway.charge(order.amount);
    
    // 5. 使用靜態方法
    Logger.log('Order processed');
    
    // 6. 混合業務邏輯與技術細節
    if (result.success) {
      emailService.send(order.email, 'Order confirmed');
    }
  }
}
```

**問題**：
- ❌ 無法控制依賴（必須真的連接資料庫、Email、支付 API）
- ❌ 無法控制時間（無法測試非營業時間的邏輯）
- ❌ 測試緩慢（需要真實的外部服務）
- ❌ 測試不穩定（依賴網路、資料庫狀態）
- ❌ 難以隔離測試（牽一髮動全身）

---

#### ✅ 易於測試的程式碼

**原則：依賴注入 + 介面抽象 + 純函式**

**✅ 範例：可測試的程式碼**

```typescript
// ✅ 易於測試
export class OrderService {
  constructor(
    private orderRepository: IOrderRepository,
    private emailService: IEmailService,
    private paymentGateway: IPaymentGateway,
    private timeProvider: ITimeProvider,
    private logger: ILogger,
    private config: AppConfig
  ) {}

  async processOrder(orderId: string): Promise<void> {
    // 1. 檢查營業時間（可以注入假的 TimeProvider）
    if (!this.isBusinessHours()) {
      throw new BusinessHoursError('Outside business hours');
    }

    // 2. 載入訂單（可以注入假的 Repository）
    const order = await this.orderRepository.findById(orderId);
    if (!order) {
      throw new OrderNotFoundError(orderId);
    }

    // 3. 處理付款（可以注入假的 PaymentGateway）
    const result = await this.paymentGateway.charge(order.amount);

    // 4. 發送確認信（可以注入假的 EmailService）
    if (result.isSuccessful) {
      await this.emailService.send(
        order.customerEmail,
        'Order confirmed'
      );
      
      // 5. 記錄日誌（可以注入假的 Logger）
      this.logger.info(`Order ${orderId} processed`);
    }
  }

  // 6. 純函式（易於測試）
  private isBusinessHours(): boolean {
    const now = this.timeProvider.now();
    const hour = now.getHours();
    return hour >= this.config.businessHoursStart &&
           hour <= this.config.businessHoursEnd;
  }
}
```

**單元測試範例**：

```typescript
// tests/unit/OrderService.test.ts
describe('OrderService', () => {
  let orderService: OrderService;
  let mockRepository: MockOrderRepository;
  let mockEmailService: MockEmailService;
  let mockPaymentGateway: MockPaymentGateway;
  let mockTimeProvider: MockTimeProvider;
  let mockLogger: MockLogger;

  beforeEach(() => {
    // 建立 Mock 物件
    mockRepository = new MockOrderRepository();
    mockEmailService = new MockEmailService();
    mockPaymentGateway = new MockPaymentGateway();
    mockTimeProvider = new MockTimeProvider();
    mockLogger = new MockLogger();

    // 注入 Mock 依賴
    orderService = new OrderService(
      mockRepository,
      mockEmailService,
      mockPaymentGateway,
      mockTimeProvider,
      mockLogger,
      { businessHoursStart: 9, businessHoursEnd: 17 }
    );
  });

  describe('processOrder', () => {
    it('should process order successfully during business hours', async () => {
      // Arrange（準備）
      const orderId = '123';
      const order = new Order(orderId, 100, 'customer@example.com');
      
      mockTimeProvider.setTime(new Date('2024-01-01 10:00:00')); // 營業時間
      mockRepository.addOrder(order);
      mockPaymentGateway.setSuccessResponse();

      // Act（執行）
      await orderService.processOrder(orderId);

      // Assert（驗證）
      expect(mockPaymentGateway.chargeWasCalled).toBe(true);
      expect(mockPaymentGateway.chargedAmount).toBe(100);
      expect(mockEmailService.sentEmails).toHaveLength(1);
      expect(mockEmailService.sentEmails[0].to).toBe('customer@example.com');
      expect(mockLogger.logs).toContain('Order 123 processed');
    });

    it('should throw error outside business hours', async () => {
      // Arrange
      mockTimeProvider.setTime(new Date('2024-01-01 20:00:00')); // 非營業時間

      // Act & Assert
      await expect(orderService.processOrder('123'))
        .rejects
        .toThrow(BusinessHoursError);
      
      // 驗證沒有呼叫付款或發信
      expect(mockPaymentGateway.chargeWasCalled).toBe(false);
      expect(mockEmailService.sentEmails).toHaveLength(0);
    });

    it('should throw error when order not found', async () => {
      // Arrange
      mockTimeProvider.setTime(new Date('2024-01-01 10:00:00'));
      mockRepository.clear(); // 沒有訂單

      // Act & Assert
      await expect(orderService.processOrder('999'))
        .rejects
        .toThrow(OrderNotFoundError);
    });

    it('should not send email when payment fails', async () => {
      // Arrange
      const order = new Order('123', 100, 'customer@example.com');
      mockTimeProvider.setTime(new Date('2024-01-01 10:00:00'));
      mockRepository.addOrder(order);
      mockPaymentGateway.setFailureResponse();

      // Act
      await orderService.processOrder('123');

      // Assert
      expect(mockPaymentGateway.chargeWasCalled).toBe(true);
      expect(mockEmailService.sentEmails).toHaveLength(0); // 不應發送 Email
    });
  });
});
```

---

### 三、提升測試性的技巧

#### **1. 依賴注入 (Dependency Injection)**

```typescript
// ❌ 難以測試：內部建立依賴
class UserService {
  register(email: string) {
    const db = new Database();  // 寫死
    db.save({ email });
  }
}

// ✅ 易於測試：注入依賴
class UserService {
  constructor(private db: IDatabase) {}
  
  register(email: string) {
    this.db.save({ email });
  }
}

// 測試時注入 Mock
const mockDb = new MockDatabase();
const service = new UserService(mockDb);
```

---

#### **2. 介面抽象 (Interface Abstraction)**

```csharp
// ❌ 依賴具體類別
public class OrderProcessor
{
    private MySQLDatabase db = new MySQLDatabase();
    
    public void Process(Order order)
    {
        db.Save(order);
    }
}

// ✅ 依賴抽象介面
public class OrderProcessor
{
    private readonly IDatabase _db;
    
    public OrderProcessor(IDatabase db)
    {
        _db = db;
    }
    
    public void Process(Order order)
    {
        _db.Save(order);
    }
}

// 測試時使用 Mock
var mockDb = new Mock<IDatabase>();
var processor = new OrderProcessor(mockDb.Object);
```

---

#### **3. 避免靜態方法和全域狀態**

```python
# ❌ 使用全域變數
current_user = None

def process_order(order_id):
    if current_user is None:
        raise Exception("Not logged in")
    # ...

# ✅ 明確傳遞依賴
def process_order(order_id: str, user: User):
    if user is None:
        raise Exception("Not logged in")
    # ...

# 測試時可以傳入假的 User
test_user = User("test@example.com")
process_order("123", test_user)
```

---

#### **4. 純函式優先**

```typescript
// ❌ 依賴外部狀態
class PriceCalculator {
  private taxRate = 0.1;  // 可變狀態
  
  calculate(price: number): number {
    return price * (1 + this.taxRate);
  }
}

// ✅ 純函式（無副作用）
function calculatePrice(price: number, taxRate: number): number {
  return price * (1 + taxRate);
}

// 測試非常簡單
expect(calculatePrice(100, 0.1)).toBe(110);
expect(calculatePrice(200, 0.05)).toBe(210);
```

---

#### **5. 時間抽象**

```typescript
// ❌ 直接使用 Date
class Scheduler {
  isOverdue(deadline: Date): boolean {
    return new Date() > deadline;  // 無法控制「現在」
  }
}

// ✅ 注入時間提供者
interface ITimeProvider {
  now(): Date;
}

class Scheduler {
  constructor(private timeProvider: ITimeProvider) {}
  
  isOverdue(deadline: Date): boolean {
    return this.timeProvider.now() > deadline;
  }
}

// 測試時可以控制時間
class MockTimeProvider implements ITimeProvider {
  private currentTime: Date;
  
  setTime(time: Date) {
    this.currentTime = time;
  }
  
  now(): Date {
    return this.currentTime;
  }
}

const mockTime = new MockTimeProvider();
mockTime.setTime(new Date('2024-01-01'));
const scheduler = new Scheduler(mockTime);
```

---

#### **6. 避免複雜的建構子**

```csharp
// ❌ 建構子做太多事
public class UserService
{
    public UserService()
    {
        // 在建構子中建立連線（難以測試）
        var connection = new SqlConnection(connectionString);
        connection.Open();
        // 初始化其他資源...
    }
}

// ✅ 建構子只接收依賴
public class UserService
{
    private readonly IUserRepository _repository;
    
    public UserService(IUserRepository repository)
    {
        _repository = repository;
    }
}
```

---

### 四、測試金字塔 (Test Pyramid)

```
        /\
       /  \  E2E Tests (少量)
      /----\
     /      \ Integration Tests (中量)
    /--------\
   /          \ Unit Tests (大量)
  /____________\
```

#### **1. 單元測試 (Unit Tests) - 70%**
- 測試單一函式或類別
- 隔離所有依賴（使用 Mock）
- 快速、穩定、大量

```typescript
// 單元測試範例
describe('UserValidator', () => {
  it('should reject invalid email', () => {
    const validator = new UserValidator();
    expect(() => validator.validateEmail('invalid'))
      .toThrow('Invalid email');
  });
});
```

#### **2. 整合測試 (Integration Tests) - 20%**
- 測試多個模組的協作
- 可能使用真實資料庫（測試環境）
- 中速、較穩定、適量

```typescript
// 整合測試範例
describe('RegisterUserUseCase', () => {
  it('should save user to database', async () => {
    const db = await createTestDatabase();
    const useCase = new RegisterUserUseCase(
      new MySQLUserRepository(db),
      new BcryptHasher(),
      new MockEmailService(),
      new MockLogger()
    );
    
    await useCase.execute('test@example.com', 'password123');
    
    const user = await db.query('SELECT * FROM users WHERE email = ?', 
                                 ['test@example.com']);
    expect(user).toBeDefined();
  });
});
```

#### **3. 端對端測試 (E2E Tests) - 10%**
- 測試完整的用戶流程
- 使用真實環境
- 慢速、較脆弱、少量

```typescript
// E2E 測試範例（使用 Playwright）
test('user registration flow', async ({ page }) => {
  await page.goto('http://localhost:3000/register');
  await page.fill('input[name="email"]', 'test@example.com');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button[type="submit"]');
  
  await expect(page).toHaveURL('/dashboard');
  await expect(page.locator('text=Welcome')).toBeVisible();
});
```

---

### 💡 最佳實踐總結

| 原則         | 模組化                            | 測試性                     |
| ------------ | --------------------------------- | -------------------------- |
| **核心目標** | 高內聚、低耦合                    | 易於驗證正確性             |
| **關鍵技巧** | 分層架構、依賴反轉                | 依賴注入、介面抽象         |
| **避免**     | 單體程式碼、緊密耦合              | 全域狀態、靜態方法         |
| **結構**     | Domain/Application/Infrastructure | Unit/Integration/E2E       |
| **好處**     | 易理解、易維護、易擴展            | 快速反饋、高信心、回歸防護 |

---

### 🔑 核心要點

1. **模組化是測試性的基礎**：好的模組化設計讓測試變得簡單
2. **依賴注入是關鍵**：透過注入依賴，讓程式碼可測試
3. **介面抽象讓替換容易**：可以輕鬆注入 Mock 物件
4. **純函式最易測試**：無副作用、無外部依賴
5. **遵循測試金字塔**：大量單元測試、適量整合測試、少量E2E測試
6. **設計即文件**：好的模組結構本身就是最好的文件

**記住**：「先寫測試，再寫程式碼 (TDD)」不只是開發方法，更是設計工具。如果你發現程式碼難以測試，那通常意味著設計需要改進。測試會倒逼你寫出更好的設計！

---

[← 回目錄](./README.md) | [下一章：不可變性 →](./Immutability.md)
