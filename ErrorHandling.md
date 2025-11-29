## 九、錯誤處理 (Error Handling)

### 🎯 Clean Code 核心心法

錯誤處理不是事後補救，而是程式設計的核心部分。好的錯誤處理能讓程式碼更健壯、更易維護，壞的錯誤處理會讓程式碼變得混亂且難以追蹤問題。

**錯誤處理的本質**：
- **錯誤處理是業務邏輯的一部分**，不是可有可無的附屬品
- **讓錯誤資訊有意義**，幫助開發者快速定位問題
- **在適當的層級處理錯誤**，不要到處 try-catch
- **使用例外而非錯誤碼**，讓主要邏輯更清晰

---

### ✅ 原則 1：使用例外 (Exceptions) 而非錯誤碼

**為什麼？** 錯誤碼會讓呼叫端必須立即處理，導致業務邏輯與錯誤處理混在一起。

#### ❌ 使用錯誤碼（不好）

**TypeScript**
```typescript
// 錯誤碼讓程式碼變得難讀
function saveUser(user: User): number {
  if (!user.email) return -1;  // 無效的 email
  if (!user.name) return -2;   // 無效的 name
  if (database.isDown()) return -3;  // 資料庫錯誤
  
  // 真正的業務邏輯被淹沒在錯誤檢查中
  database.save(user);
  return 0;  // 成功
}

// 呼叫端必須檢查每個錯誤碼
const result = saveUser(newUser);
if (result === -1) {
  console.error("Email is invalid");
} else if (result === -2) {
  console.error("Name is invalid");
} else if (result === -3) {
  console.error("Database error");
} else {
  console.log("Success");
}
```

**C#**
```csharp
// ❌ 使用錯誤碼
public int ProcessPayment(Payment payment)
{
    if (payment.Amount <= 0) return -1;
    if (string.IsNullOrEmpty(payment.CardNumber)) return -2;
    if (PaymentGateway.IsUnavailable()) return -3;
    
    PaymentGateway.Charge(payment);
    return 0;
}
```

**Python**
```python
# ❌ 使用錯誤碼
def process_order(order):
    if not order.items:
        return -1  # 空訂單
    if order.total < 0:
        return -2  # 無效金額
    if not inventory.check_stock(order):
        return -3  # 庫存不足
    
    database.save(order)
    return 0  # 成功
```

#### ✅ 使用例外（好）

**TypeScript**
```typescript
// 業務邏輯清晰，錯誤處理分離
function saveUser(user: User): void {
  validateUser(user);  // 驗證邏輯獨立
  database.save(user);  // 主要邏輯清晰
}

function validateUser(user: User): void {
  if (!user.email) {
    throw new ValidationError("User email is required");
  }
  if (!user.name) {
    throw new ValidationError("User name is required");
  }
}

// 呼叫端可以專注於業務流程
try {
  saveUser(newUser);
  console.log("User saved successfully");
} catch (error) {
  if (error instanceof ValidationError) {
    console.error("Validation failed:", error.message);
  } else if (error instanceof DatabaseError) {
    console.error("Database error:", error.message);
  } else {
    throw error;  // 未預期的錯誤，往上拋
  }
}
```

**C#**
```csharp
// ✅ 使用例外
public void ProcessPayment(Payment payment)
{
    ValidatePayment(payment);
    PaymentGateway.Charge(payment);
}

private void ValidatePayment(Payment payment)
{
    if (payment.Amount <= 0)
        throw new ValidationException("Payment amount must be positive");
        
    if (string.IsNullOrEmpty(payment.CardNumber))
        throw new ValidationException("Card number is required");
}

// 使用
try
{
    ProcessPayment(payment);
    Console.WriteLine("Payment processed successfully");
}
catch (ValidationException ex)
{
    Console.Error.WriteLine($"Validation error: {ex.Message}");
}
catch (PaymentGatewayException ex)
{
    Console.Error.WriteLine($"Payment failed: {ex.Message}");
}
```

**Python**
```python
# ✅ 使用例外
def process_order(order):
    validate_order(order)
    check_inventory(order)
    database.save(order)

def validate_order(order):
    if not order.items:
        raise ValidationError("Order must contain at least one item")
    if order.total < 0:
        raise ValidationError("Order total cannot be negative")

# 使用
try:
    process_order(new_order)
    print("Order processed successfully")
except ValidationError as e:
    print(f"Validation error: {e}")
except InventoryError as e:
    print(f"Inventory error: {e}")
```

---

### ✅ 原則 2：提供有意義的錯誤訊息與上下文

**為什麼？** 好的錯誤訊息能讓開發者快速定位問題，節省除錯時間。

#### ❌ 不好的錯誤訊息

```typescript
// 太籠統，無法得知原因
throw new Error("Invalid input");

// 沒有上下文資訊
throw new Error("Failed");

// 技術細節過多，對使用者無意義
throw new Error("NullPointerException at line 42");
```

#### ✅ 好的錯誤訊息

**TypeScript**
```typescript
// 明確指出問題與原因
throw new ValidationError(
  "User email is invalid",
  { email: user.email, userId: user.id }
);

// 包含有用的上下文資訊
throw new DatabaseError(
  "Failed to save user to database",
  { 
    operation: "INSERT",
    table: "users",
    userId: user.id,
    cause: originalError
  }
);

// 建議解決方案
throw new ConfigurationError(
  "API key is missing. Please set OPENAI_API_KEY in your environment variables."
);
```

**C#**
```csharp
// 包含完整上下文的自訂例外
public class OrderProcessingException : Exception
{
    public string OrderId { get; }
    public string CustomerId { get; }
    
    public OrderProcessingException(
        string message, 
        string orderId, 
        string customerId,
        Exception innerException = null
    ) : base(message, innerException)
    {
        OrderId = orderId;
        CustomerId = customerId;
    }
}

// 使用
throw new OrderProcessingException(
    "Failed to process order: insufficient inventory",
    order.Id,
    order.CustomerId
);
```

**Python**
```python
# 包含上下文的自訂例外
class PaymentError(Exception):
    def __init__(self, message, payment_id=None, amount=None, cause=None):
        super().__init__(message)
        self.payment_id = payment_id
        self.amount = amount
        self.cause = cause
    
    def __str__(self):
        context = f" (Payment ID: {self.payment_id}, Amount: {self.amount})"
        return f"{super().__str__()}{context}"

# 使用
raise PaymentError(
    "Payment gateway timeout",
    payment_id=payment.id,
    amount=payment.amount,
    cause=original_exception
)
```

---

### ✅ 原則 3：定義明確的例外層級結構

**為什麼？** 讓呼叫端能夠根據不同類型的錯誤做出適當的反應。

#### ✅ 建立例外階層

**TypeScript**
```typescript
// 基礎例外類別
class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500,
    public context?: any
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

// 業務邏輯例外
class BusinessError extends AppError {
  constructor(message: string, context?: any) {
    super(message, "BUSINESS_ERROR", 400, context);
  }
}

// 具體的業務例外
class ValidationError extends BusinessError {
  constructor(message: string, context?: any) {
    super(message, context);
    this.code = "VALIDATION_ERROR";
  }
}

class AuthorizationError extends BusinessError {
  constructor(message: string, context?: any) {
    super(message, context);
    this.code = "AUTHORIZATION_ERROR";
    this.statusCode = 403;
  }
}

class NotFoundError extends BusinessError {
  constructor(resource: string, id: string) {
    super(`${resource} not found`, { resource, id });
    this.code = "NOT_FOUND";
    this.statusCode = 404;
  }
}

// 系統級例外
class SystemError extends AppError {
  constructor(message: string, context?: any) {
    super(message, "SYSTEM_ERROR", 500, context);
  }
}

class DatabaseError extends SystemError {
  constructor(message: string, context?: any) {
    super(message, context);
    this.code = "DATABASE_ERROR";
  }
}

class ExternalServiceError extends SystemError {
  constructor(service: string, message: string) {
    super(message, { service });
    this.code = "EXTERNAL_SERVICE_ERROR";
  }
}
```

**C#**
```csharp
// 基礎應用程式例外
public abstract class AppException : Exception
{
    public string ErrorCode { get; }
    public int StatusCode { get; }
    
    protected AppException(
        string message, 
        string errorCode,
        int statusCode = 500,
        Exception innerException = null
    ) : base(message, innerException)
    {
        ErrorCode = errorCode;
        StatusCode = statusCode;
    }
}

// 業務例外
public class ValidationException : AppException
{
    public ValidationException(string message)
        : base(message, "VALIDATION_ERROR", 400) { }
}

public class NotFoundException : AppException
{
    public NotFoundException(string resource, string id)
        : base($"{resource} with id '{id}' not found", "NOT_FOUND", 404) { }
}

public class UnauthorizedException : AppException
{
    public UnauthorizedException(string message = "Unauthorized access")
        : base(message, "UNAUTHORIZED", 401) { }
}

// 系統例外
public class DatabaseException : AppException
{
    public DatabaseException(string message, Exception innerException = null)
        : base(message, "DATABASE_ERROR", 500, innerException) { }
}

public class ExternalServiceException : AppException
{
    public string ServiceName { get; }
    
    public ExternalServiceException(string serviceName, string message)
        : base(message, "EXTERNAL_SERVICE_ERROR", 503)
    {
        ServiceName = serviceName;
    }
}
```

**Python**
```python
# 基礎應用程式例外
class AppError(Exception):
    def __init__(self, message, error_code="APP_ERROR", status_code=500, context=None):
        super().__init__(message)
        self.error_code = error_code
        self.status_code = status_code
        self.context = context or {}

# 業務例外
class BusinessError(AppError):
    def __init__(self, message, context=None):
        super().__init__(message, "BUSINESS_ERROR", 400, context)

class ValidationError(BusinessError):
    def __init__(self, message, context=None):
        super().__init__(message, context)
        self.error_code = "VALIDATION_ERROR"

class AuthorizationError(BusinessError):
    def __init__(self, message="Unauthorized access", context=None):
        super().__init__(message, context)
        self.error_code = "AUTHORIZATION_ERROR"
        self.status_code = 403

class NotFoundError(BusinessError):
    def __init__(self, resource, resource_id):
        super().__init__(
            f"{resource} with id '{resource_id}' not found",
            {"resource": resource, "id": resource_id}
        )
        self.error_code = "NOT_FOUND"
        self.status_code = 404

# 系統例外
class SystemError(AppError):
    def __init__(self, message, context=None):
        super().__init__(message, "SYSTEM_ERROR", 500, context)

class DatabaseError(SystemError):
    def __init__(self, message, context=None):
        super().__init__(message, context)
        self.error_code = "DATABASE_ERROR"

class ExternalServiceError(SystemError):
    def __init__(self, service_name, message):
        super().__init__(message, {"service": service_name})
        self.error_code = "EXTERNAL_SERVICE_ERROR"
        self.status_code = 503
```

---

### ✅ 原則 4：在正確的層級處理錯誤

**為什麼？** 過早或過晚處理錯誤都會造成問題。

#### 🎯 錯誤處理的層級原則

1. **低層級（資料存取層）**：轉換技術例外為業務例外
2. **中層級（業務邏輯層）**：專注於業務邏輯，讓例外向上傳播
3. **高層級（API/UI層）**：統一處理例外，轉換為適當的回應

#### ✅ 範例：多層級錯誤處理

**TypeScript**
```typescript
// === 資料存取層 (Data Access Layer) ===
class UserRepository {
  async findById(id: string): Promise<User> {
    try {
      const result = await database.query("SELECT * FROM users WHERE id = ?", [id]);
      
      if (!result) {
        throw new NotFoundError("User", id);
      }
      
      return result;
    } catch (error) {
      // 轉換低階技術例外為業務例外
      if (error instanceof NotFoundError) {
        throw error;  // 業務例外直接向上拋
      }
      
      // 資料庫錯誤包裝為 DatabaseError
      throw new DatabaseError(
        "Failed to fetch user from database",
        { userId: id, originalError: error.message }
      );
    }
  }
  
  async save(user: User): Promise<void> {
    try {
      await database.query("INSERT INTO users ...", [user]);
    } catch (error) {
      // 處理特定的資料庫錯誤
      if (error.code === "23505") {  // Unique constraint violation
        throw new ValidationError(
          "User with this email already exists",
          { email: user.email }
        );
      }
      
      throw new DatabaseError("Failed to save user", { error: error.message });
    }
  }
}

// === 業務邏輯層 (Business Logic Layer) ===
class UserService {
  constructor(private userRepo: UserRepository) {}
  
  async createUser(userData: CreateUserDto): Promise<User> {
    // 業務驗證
    this.validateUserData(userData);
    
    // 讓資料層的例外自然向上傳播
    const user = new User(userData);
    await this.userRepo.save(user);
    
    return user;
  }
  
  private validateUserData(data: CreateUserDto): void {
    if (!data.email || !data.email.includes("@")) {
      throw new ValidationError("Invalid email format", { email: data.email });
    }
    
    if (data.age < 18) {
      throw new ValidationError("User must be at least 18 years old", { age: data.age });
    }
  }
}

// === API 層 (Presentation Layer) ===
// 統一的錯誤處理中介軟體
app.use((error: Error, req: Request, res: Response, next: NextFunction) => {
  // 記錄錯誤
  logger.error({
    error: error.message,
    stack: error.stack,
    path: req.path,
    method: req.method
  });
  
  // 根據例外類型回傳適當的 HTTP 回應
  if (error instanceof AppError) {
    res.status(error.statusCode).json({
      error: {
        code: error.code,
        message: error.message,
        ...(process.env.NODE_ENV === "development" && { context: error.context })
      }
    });
  } else {
    // 未預期的錯誤，不要暴露內部細節
    res.status(500).json({
      error: {
        code: "INTERNAL_SERVER_ERROR",
        message: "An unexpected error occurred"
      }
    });
  }
});

// Controller
app.post("/api/users", async (req, res, next) => {
  try {
    const user = await userService.createUser(req.body);
    res.status(201).json(user);
  } catch (error) {
    next(error);  // 交給錯誤處理中介軟體
  }
});
```

**C#**
```csharp
// === 資料存取層 ===
public class UserRepository
{
    public async Task<User> FindByIdAsync(string id)
    {
        try
        {
            var user = await _dbContext.Users.FindAsync(id);
            
            if (user == null)
            {
                throw new NotFoundException("User", id);
            }
            
            return user;
        }
        catch (NotFoundException)
        {
            throw;  // 業務例外直接向上拋
        }
        catch (Exception ex)
        {
            throw new DatabaseException($"Failed to fetch user with id {id}", ex);
        }
    }
}

// === 業務邏輯層 ===
public class UserService
{
    private readonly UserRepository _userRepo;
    
    public async Task<User> CreateUserAsync(CreateUserDto userData)
    {
        ValidateUserData(userData);
        
        var user = new User(userData);
        await _userRepo.SaveAsync(user);
        
        return user;
    }
    
    private void ValidateUserData(CreateUserDto data)
    {
        if (string.IsNullOrEmpty(data.Email) || !data.Email.Contains("@"))
        {
            throw new ValidationException("Invalid email format");
        }
    }
}

// === API 層 ===
// 全域錯誤處理中介軟體
public class ErrorHandlingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ErrorHandlingMiddleware> _logger;
    
    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (AppException ex)
        {
            _logger.LogError(ex, "Application error occurred");
            
            context.Response.StatusCode = ex.StatusCode;
            await context.Response.WriteAsJsonAsync(new
            {
                error = new
                {
                    code = ex.ErrorCode,
                    message = ex.Message
                }
            });
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unexpected error occurred");
            
            context.Response.StatusCode = 500;
            await context.Response.WriteAsJsonAsync(new
            {
                error = new
                {
                    code = "INTERNAL_SERVER_ERROR",
                    message = "An unexpected error occurred"
                }
            });
        }
    }
}
```

**Python**
```python
# === 資料存取層 ===
class UserRepository:
    async def find_by_id(self, user_id: str) -> User:
        try:
            result = await database.query("SELECT * FROM users WHERE id = ?", [user_id])
            
            if not result:
                raise NotFoundError("User", user_id)
            
            return User.from_dict(result)
        except NotFoundError:
            raise  # 業務例外直接向上拋
        except Exception as e:
            raise DatabaseError(
                f"Failed to fetch user with id {user_id}",
                {"user_id": user_id, "cause": str(e)}
            )

# === 業務邏輯層 ===
class UserService:
    def __init__(self, user_repo: UserRepository):
        self.user_repo = user_repo
    
    async def create_user(self, user_data: dict) -> User:
        self._validate_user_data(user_data)
        
        user = User(**user_data)
        await self.user_repo.save(user)
        
        return user
    
    def _validate_user_data(self, data: dict):
        if not data.get("email") or "@" not in data["email"]:
            raise ValidationError(
                "Invalid email format",
                {"email": data.get("email")}
            )

# === API 層 (Flask 範例) ===
from flask import Flask, jsonify, request

app = Flask(__name__)

# 全域錯誤處理
@app.errorhandler(AppError)
def handle_app_error(error):
    response = {
        "error": {
            "code": error.error_code,
            "message": str(error)
        }
    }
    
    if app.debug:
        response["error"]["context"] = error.context
    
    return jsonify(response), error.status_code

@app.errorhandler(Exception)
def handle_unexpected_error(error):
    app.logger.error(f"Unexpected error: {error}", exc_info=True)
    
    return jsonify({
        "error": {
            "code": "INTERNAL_SERVER_ERROR",
            "message": "An unexpected error occurred"
        }
    }), 500

@app.route("/api/users", methods=["POST"])
async def create_user():
    user_data = request.get_json()
    user = await user_service.create_user(user_data)
    return jsonify(user.to_dict()), 201
```

---

### ✅ 原則 5：不要返回或傳遞 Null

**為什麼？** `null` 是十億美元的錯誤（Tony Hoare），會導致 `NullPointerException`。

#### ❌ 返回 Null（不好）

```typescript
function findUser(id: string): User | null {
  const user = database.query(...);
  return user ?? null;  // 強迫呼叫端處理 null
}

// 呼叫端必須檢查 null，容易忘記
const user = findUser("123");
console.log(user.name);  // 💥 可能會出錯
```

#### ✅ 使用例外或 Optional Pattern（好）

**TypeScript**
```typescript
// 方案 1：找不到就拋出例外
function findUser(id: string): User {
  const user = database.query(...);
  
  if (!user) {
    throw new NotFoundError("User", id);
  }
  
  return user;
}

// 方案 2：使用 Optional/Maybe pattern
type Optional<T> = T | undefined;

function findUserOptional(id: string): Optional<User> {
  return database.query(...);
}

// 使用 Optional chaining
const userName = findUserOptional("123")?.name ?? "Unknown";

// 方案 3：返回空物件（Null Object Pattern）
function findUserOrDefault(id: string): User {
  const user = database.query(...);
  return user ?? User.createGuest();  // 返回預設的訪客使用者
}
```

**C#**
```csharp
// 使用 Nullable Reference Types (C# 8.0+)
#nullable enable

public User FindUser(string id)
{
    var user = database.Query(...);
    
    if (user == null)
    {
        throw new NotFoundException("User", id);
    }
    
    return user;  // 保證不是 null
}

// 或使用 Option/Maybe pattern (如 LanguageExt 套件)
public Option<User> FindUserOptional(string id)
{
    var user = database.Query(...);
    return user != null ? Some(user) : None;
}

// 使用
var result = FindUserOptional("123")
    .Match(
        Some: user => $"Found: {user.Name}",
        None: () => "User not found"
    );
```

**Python**
```python
# 方案 1：找不到就拋出例外
def find_user(user_id: str) -> User:
    user = database.query(...)
    
    if user is None:
        raise NotFoundError("User", user_id)
    
    return user

# 方案 2：使用 Optional (Python 3.10+)
from typing import Optional

def find_user_optional(user_id: str) -> Optional[User]:
    return database.query(...)

# 使用
user = find_user_optional("123")
if user:
    print(user.name)
else:
    print("User not found")

# 或使用 walrus operator
if user := find_user_optional("123"):
    print(user.name)
```

---

### ✅ 原則 6：不要吞掉例外 (Don't Swallow Exceptions)

**為什麼？** 靜默失敗會讓問題難以追蹤。

#### ❌ 吞掉例外（非常糟糕）

```typescript
try {
  await dangerousOperation();
} catch (error) {
  // 什麼都不做，錯誤消失了 💥
}

try {
  await saveData();
} catch (error) {
  console.log("oops");  // 太籠統，沒有幫助
}
```

#### ✅ 正確處理例外

```typescript
try {
  await saveData();
} catch (error) {
  // 記錄完整錯誤資訊
  logger.error("Failed to save data", {
    error: error.message,
    stack: error.stack,
    context: { userId, timestamp }
  });
  
  // 重新拋出或包裝例外
  throw new DatabaseError("Data persistence failed", { cause: error });
}

// 如果真的需要忽略某些例外，要明確註明原因
try {
  analytics.track(event);
} catch (error) {
  // 分析追蹤失敗不影響主流程，但要記錄
  logger.warn("Analytics tracking failed (non-critical)", { error });
  // 不重新拋出，因為這不是關鍵功能
}
```

---

### 📋 Clean Code 錯誤處理檢查清單

- [ ] **使用例外而非錯誤碼**，讓業務邏輯更清晰
- [ ] **提供有意義的錯誤訊息**，包含足夠的上下文資訊
- [ ] **建立清晰的例外階層**，區分業務例外與系統例外
- [ ] **在正確的層級處理錯誤**，不要過早或過晚處理
- [ ] **避免返回 null**，使用例外或 Optional pattern
- [ ] **不要吞掉例外**，至少要記錄日誌
- [ ] **讓例外類別包含足夠的上下文**，方便除錯
- [ ] **在 API 層統一處理例外**，轉換為適當的 HTTP 回應
- [ ] **區分可恢復與不可恢復的錯誤**
- [ ] **測試錯誤處理路徑**，確保例外處理正確運作

---

### 🎓 進階心法

1. **Fail Fast 原則**：盡早發現錯誤，不要讓錯誤狀態在系統中傳播
   ```typescript
   function processOrder(order: Order): void {
     // 在開頭就驗證所有前置條件
     if (!order) throw new ValidationError("Order is required");
     if (!order.items.length) throw new ValidationError("Order must have items");
     if (order.total <= 0) throw new ValidationError("Invalid order total");
     
     // 確保所有前置條件都滿足後，才執行主要邏輯
     fulfillOrder(order);
   }
   ```

2. **錯誤恢復策略**：對於暫時性錯誤，實作重試機制
   ```typescript
   async function callExternalAPI(url: string, maxRetries = 3): Promise<Response> {
     for (let attempt = 1; attempt <= maxRetries; attempt++) {
       try {
         return await fetch(url);
       } catch (error) {
         if (attempt === maxRetries) {
           throw new ExternalServiceError("API", "Max retries exceeded");
         }
         
         // 指數退避
         await sleep(Math.pow(2, attempt) * 1000);
       }
     }
   }
   ```

3. **使用 Result 物件模式**（適合不想拋出例外的情境）
   ```typescript
   type Result<T, E = Error> = 
     | { ok: true; value: T }
     | { ok: false; error: E };
   
   function parseUserInput(input: string): Result<User, ValidationError> {
     if (!input.includes("@")) {
       return { 
         ok: false, 
         error: new ValidationError("Invalid email") 
       };
     }
     
     return { 
       ok: true, 
       value: new User(input) 
     };
   }
   
   // 使用
   const result = parseUserInput(userInput);
   if (result.ok) {
     console.log("User:", result.value);
   } else {
     console.error("Error:", result.error.message);
   }
   ```

---

[← 回目錄](./00-README.md) | [下一章：魔術數字 →](./09-MagicNumbers.md)
