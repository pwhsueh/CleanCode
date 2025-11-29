## 十五、「高內聚、低耦合」的黃金法則：S.O.L.I.D. 架構思維

> **核心目標：寫出「高內聚、低耦合」的程式。**

判斷軟體架構好壞的**黃金準則**。

---

### 🎯 什麼是「高內聚、低耦合」？

在深入 S.O.L.I.D. 原則之前，我們必須先理解軟體設計中最重要的兩個概念：**內聚 (Cohesion)** 和 **耦合 (Coupling)**。這兩個概念是評估程式碼品質的核心指標。

---

#### 📦 內聚 (Cohesion)：模組內部的團結程度

**內聚**是指一個模組（類別、函式、套件）內部的元素（方法、屬性、功能）彼此相關的緊密程度。

**高內聚**意味著：
-   模組內的所有元素都緊密圍繞著**同一個核心職責**或**目標**。
-   每個元素的存在都是為了支持這個核心功能。
-   模組就像一個專業團隊，每個成員都在為同一個目標協作。

**低內聚**意味著：
-   模組內的元素各做各的，彼此關聯性低。
-   這個模組就像一個雜貨店，什麼都賣，但沒有明確的定位。

**生活比喻：**
-   **高內聚**：一家專賣咖啡的咖啡廳，裡面有咖啡師、烘豆師、收銀員，大家都為了「提供優質咖啡體驗」這個目標服務。
-   **低內聚**：一家店同時賣咖啡、修手機、剪頭髮。雖然可能很方便，但職責混亂，當你想要「改善咖啡品質」時，不知道該找誰。

**程式碼範例：**

❌ **低內聚 (雜貨店類別)**
```typescript
// 這個類別什麼都做，職責混亂
class UserManager {
  // 使用者資料管理
  createUser(name: string, email: string) { ... }
  deleteUser(userId: string) { ... }
  
  // 密碼處理（應該獨立出去）
  hashPassword(password: string) { ... }
  validatePassword(password: string) { ... }
  
  // 郵件發送（完全不相關的職責）
  sendWelcomeEmail(email: string) { ... }
  sendPasswordResetEmail(email: string) { ... }
  
  // 日誌記錄（也不相關）
  logActivity(message: string) { ... }
}
```

✅ **高內聚 (職責清晰的專業類別)**
```typescript
// 每個類別只專注於一個核心職責

// 專注於使用者資料的 CRUD
class UserRepository {
  create(user: User): void { ... }
  findById(id: string): User { ... }
  update(user: User): void { ... }
  delete(id: string): void { ... }
}

// 專注於密碼安全
class PasswordService {
  hash(password: string): string { ... }
  validate(password: string, hash: string): boolean { ... }
  generateResetToken(): string { ... }
}

// 專注於郵件發送
class EmailService {
  sendWelcomeEmail(email: string): void { ... }
  sendPasswordResetEmail(email: string, token: string): void { ... }
}

// 專注於日誌記錄
class Logger {
  info(message: string): void { ... }
  error(message: string, error: Error): void { ... }
}
```

---

#### 🔗 耦合 (Coupling)：模組之間的依賴程度

**耦合**是指不同模組（類別、函式、套件）之間相互依賴的緊密程度。

**低耦合**意味著：
-   模組之間的依賴關係少且明確。
-   修改一個模組時，不太會影響到其他模組。
-   模組可以獨立開發、測試、部署。

**高耦合**意味著：
-   模組之間緊密糾纏，牽一髮動全身。
-   修改 A 模組可能會連帶影響 B、C、D 模組。
-   很難單獨測試或重用某個模組。

**生活比喻：**
-   **低耦合**：樂高積木。每個積木都可以獨立存在，也可以與其他積木組合，但拆掉一個積木不會影響其他積木的結構。
-   **高耦合**：混凝土建築。所有牆壁、樑柱都澆灌在一起，想改變其中一部分（如加一扇窗）就可能影響整個結構的穩定性。

**程式碼範例：**

❌ **高耦合 (牽一髮動全身)**
```csharp
// OrderService 直接依賴具體的實作類別
public class OrderService
{
    // 直接實例化，緊密耦合
    private MySqlDatabase _database = new MySqlDatabase();
    private SmtpEmailService _emailService = new SmtpEmailService();
    private StripePaymentGateway _paymentGateway = new StripePaymentGateway();

    public void PlaceOrder(Order order)
    {
        // 如果要換資料庫（如改用 PostgreSQL），必須修改這個類別
        _database.Save(order);
        
        // 如果要換支付方式（如改用 PayPal），必須修改這個類別
        _paymentGateway.Charge(order.TotalAmount);
        
        // 如果要換郵件服務（如改用 SendGrid），必須修改這個類別
        _emailService.Send(order.CustomerEmail, "Order Confirmed");
    }
}
```

**問題：**
-   `OrderService` 與 `MySqlDatabase`, `SmtpEmailService`, `StripePaymentGateway` 緊密耦合。
-   任何一個外部服務的改變，都需要修改 `OrderService`。
-   無法在測試時用假的實作（Mock）替換這些服務。

✅ **低耦合 (依賴抽象，可替換)**
```csharp
// 定義抽象介面
public interface IDatabase {
    void Save(Order order);
}

public interface IEmailService {
    void Send(string to, string message);
}

public interface IPaymentGateway {
    void Charge(decimal amount);
}

// OrderService 只依賴抽象介面，不依賴具體實作
public class OrderService
{
    private readonly IDatabase _database;
    private readonly IEmailService _emailService;
    private readonly IPaymentGateway _paymentGateway;

    // 透過建構子注入依賴（Dependency Injection）
    public OrderService(
        IDatabase database, 
        IEmailService emailService, 
        IPaymentGateway paymentGateway)
    {
        _database = database;
        _emailService = emailService;
        _paymentGateway = paymentGateway;
    }

    public void PlaceOrder(Order order)
    {
        // 呼叫抽象介面，不關心具體實作
        _database.Save(order);
        _paymentGateway.Charge(order.TotalAmount);
        _emailService.Send(order.CustomerEmail, "Order Confirmed");
    }
}

// 使用時（在主程式或 IoC 容器中）
var orderService = new OrderService(
    new MySqlDatabase(),        // 可替換成 PostgreSqlDatabase
    new SmtpEmailService(),     // 可替換成 SendGridEmailService
    new StripePaymentGateway()  // 可替換成 PayPalPaymentGateway
);
```

**優點：**
-   `OrderService` 不再依賴具體的實作，只依賴抽象介面。
-   要更換資料庫或支付方式，只需要提供新的實作類別，無需修改 `OrderService`。
-   測試時可以輕易替換成假的實作（Mock）。

---

#### ⚖️ 高內聚、低耦合的關係

**高內聚**和**低耦合**是相輔相成的：
-   **高內聚**確保每個模組專注於自己的職責，不做不該做的事。
-   **低耦合**確保模組之間只透過明確、最小化的介面互動，減少相互影響。

**理想狀態：**
```
[高內聚模組 A] <-- 明確介面 --> [高內聚模組 B]
     ↓                               ↓
  職責清晰                         職責清晰
  容易測試                         容易測試
  容易重用                         容易重用
```

**最糟狀態：**
```
[低內聚模組 X] <-- 複雜糾纏 --> [低內聚模組 Y]
     ↓                               ↓
  職責混亂                         職責混亂
  難以測試                         難以測試
  難以修改                         難以修改
```

---

#### 💡 如何實現「高內聚、低耦合」？

1.  **單一職責原則 (SRP)**：確保每個類別只有一個改變的理由 → **提高內聚**
2.  **依賴反轉原則 (DIP)**：依賴抽象而非具體實作 → **降低耦合**
3.  **介面隔離原則 (ISP)**：不要強迫客戶依賴不需要的介面 → **降低耦合**
4.  **開放封閉原則 (OCP)**：透過擴充而非修改來添加功能 → **降低耦合**
5.  **里氏替換原則 (LSP)**：確保子類別可以替換父類別 → **降低耦合**

接下來，我們將深入探討這些原則。

---

### 🧭 S.O.L.I.D. 原則簡介

這五個物件導向設計原則能讓程式更容易維護與擴充，就像生活中良好的系統設計一樣有秩序。

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

// 1. 定義一個抽象的策略介面
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

// 1. 定義一個抽象的形狀介面
public interface IShape
{
    double GetArea();
}

// 2. 矩形實作這個介面
public class Rectangle : IShape
{
    public double Width { get; set; }
    public double Height { get; set; }

    public double GetArea() => Width * Height;
}

// 3. 正方形也實作這個介面
public class Square : IShape
{
    public double Side { get; set; }

    public double GetArea() => Side * Side;
}

// --- 使用時 ---
public void ProcessShape(IShape shape)
{
    // 不需要關心是什麼形狀，直接計算面積
    double area = shape.GetArea();
    Console.WriteLine($"Shape area: {area}");
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

# 1. 定義介面：可工作的
class IWorkable(ABC):
    @abstractmethod
    def work(self):
        pass

# 2. 定義介面：可被餵食的
class IFeedable(ABC):
    @abstractmethod
    def eat(self):
        pass

# 3. 定義介面：會開會的
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
**問題：** 如果現在要新增「LINE Pay」支付，`OrderService` 就必須被修改，甚至可能需要加入 `if/else` 來判斷支付類型，這會讓程式碼越來越複雜。

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

**🎯 總結：** 低耦合讓你的程式碼模組像可替換的零件，你可以輕鬆升級、替換或移除它們，而不會對整個系統造成衝擊。

---

[← 回目錄](./00-README.md) | [下一章：Checklist →](./16-Checklist.md)
