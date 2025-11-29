## 四、註解原則 (Comments)

> 「不要為糟糕的程式碼寫註解——重寫它。」— Brian Kernighan & P.J. Plauger

註解是一把雙面刃。好的註解能夠傳達重要的背景資訊或設計決策，但糟糕的註解會變成技術債務，誤導開發者，甚至比沒有註解更糟。

### ✅ 核心原則

#### 1. 程式碼應該自我說明 (Self-Documenting Code)

**最好的註解就是不需要註解。** 如果你的程式碼需要註解才能讓人理解它在做什麼，那應該優先考慮重構程式碼，而不是加註解。

-   ❌ **不好的寫法 (用註解解釋糟糕的命名)**
    ```typescript
    // Check if user is an adult
    if (u.a >= 18) {
      // ...
    }
    ```

-   ✅ **好的寫法 (程式碼本身就很清楚)**
    ```typescript
    // 不需要註解，程式碼已經說明了一切
    if (user.age >= 18) {
      // ...
    }
    
    // 或更好的做法：提煉成一個函式
    if (isAdult(user)) {
      // ...
    }
    ```

#### 2. 說明「為什麼」而非「做什麼」(Explain Why, Not What)

程式碼本身已經清楚說明了「做什麼」(What) 和「怎麼做」(How)。註解應該用來解釋：
-   **為什麼要這麼做？** (Why)
-   **為什麼選擇這個方法而不是其他方法？**
-   **這段程式碼背後的業務邏輯或限制是什麼？**

-   ❌ **不好的寫法 (描述顯而易見的事)**
    ```csharp
    // Increment index by 1
    index += 1;
    
    // Loop through all users
    foreach (var user in users) {
        // ...
    }
    ```

-   ✅ **好的寫法 (解釋為什麼)**
    ```csharp
    // Skip the header row in CSV file
    index += 1;
    
    // We must process users in registration order due to referral bonus logic
    foreach (var user in users.OrderBy(u => u.RegisteredAt)) {
        // ...
    }
    ```

-   ✅ **好的寫法 (解釋業務邏輯或限制)**
    ```python
    # The API rate limit is 100 requests/minute, so we add a delay
    time.sleep(0.6)
    
    # Use MD5 here because the legacy system only accepts MD5 hashes
    # TODO: Migrate to SHA256 when the legacy system is upgraded
    password_hash = hashlib.md5(password.encode()).hexdigest()
    ```

#### 3. 避免這些類型的註解

**a) 多餘的註解 (Redundant Comments)**

這些註解只是重複程式碼已經表達的內容，毫無價值。

```typescript
// ❌ 多餘的註解
// Get the user name
const userName = user.getName();

// Set the status to active
user.setStatus('active');

// Return true
return true;
```

**b) 日誌型註解 (Change Log Comments)**

這類註解試圖記錄程式碼的修改歷史，但版本控制系統（如 Git）已經做得更好了。

```csharp
// ❌ 不要這樣做
// 2023-01-15: John added validation
// 2023-03-20: Mary fixed the bug in line 42
// 2023-06-10: Bob refactored this method
public void ProcessOrder(Order order) {
    // ...
}
```

**c) 被註解掉的程式碼 (Commented-Out Code)**

這是最糟糕的註解類型之一。被註解掉的程式碼會累積成技術債務，沒人知道它是否還有用，也沒人敢刪除它。

```python
# ❌ 不要保留被註解掉的程式碼
def calculate_price(item):
    # old_price = item.price * 1.1
    # discount = get_discount(item)
    # return old_price - discount
    
    return item.price * (1 - item.discount_rate)
```

**解決方案：直接刪除！版本控制系統會保存歷史記錄，需要時可以找回。**

**d) 位置標記 (Position Markers)**

過度使用分隔線或標記會讓程式碼顯得雜亂。

```typescript
// ❌ 避免過度使用
////////////////////////////////////////////
// Public Methods
////////////////////////////////////////////

function getUserById(id: string) { ... }

////////////////////////////////////////////
// Private Methods
////////////////////////////////////////////

function validateId(id: string) { ... }
```

### ✨ 什麼時候應該寫註解？

雖然我們強調「程式碼應該自我說明」，但在以下情況，註解是有價值的：

#### 1. 解釋複雜的業務邏輯或演算法

```typescript
// ✅ 好的註解
// Calculate shipping cost using tiered pricing:
// - Orders under $50: $10 flat rate
// - Orders $50-$100: $5 flat rate
// - Orders over $100: Free shipping
// - Exception: Oversized items always add $15
function calculateShippingCost(order: Order): number {
  // ...
}
```

#### 2. 警告其他開發者注意潛在問題

```csharp
// ✅ 好的註解
// WARNING: This method is NOT thread-safe. 
// Ensure it's only called from single-threaded context.
public void UpdateCache(string key, object value) {
    // ...
}
```

#### 3. TODO 和 FIXME 註解

這類註解應該包含清楚的說明和追蹤資訊（如 Issue 編號）。

```python
# ✅ 好的註解
# TODO(Issue #123): Implement retry logic for failed API calls
# FIXME: This workaround causes memory leak under high load (see Issue #456)
def fetch_data():
    # ...
```

#### 4. 公開 API 的文件註解 (Documentation Comments)

如果你在撰寫供他人使用的函式庫或 API，應該使用標準的文件註解格式（如 JSDoc、XML Documentation Comments、Docstrings）。

**TypeScript (JSDoc)**
```typescript
/**
 * Calculates the total price including tax.
 * 
 * @param items - Array of items in the cart
 * @param taxRate - Tax rate as a decimal (e.g., 0.05 for 5%)
 * @returns The total price with tax applied
 * @throws {ValidationError} If items array is empty or taxRate is negative
 */
function calculateTotal(items: Item[], taxRate: number): number {
  // ...
}
```

**C# (XML Documentation)**
```csharp
/// <summary>
/// Validates user credentials and returns an authentication token.
/// </summary>
/// <param name="username">The user's username</param>
/// <param name="password">The user's password</param>
/// <returns>An authentication token if successful, null otherwise</returns>
/// <exception cref="ArgumentNullException">Thrown when username or password is null</exception>
public string? Authenticate(string username, string password) {
    // ...
}
```

**Python (Docstrings)**
```python
def calculate_discount(price: float, customer_tier: str) -> float:
    """
    Calculate the discount amount based on customer tier.
    
    Args:
        price: The original price of the item
        customer_tier: Customer tier ('bronze', 'silver', 'gold')
    
    Returns:
        The discount amount to be subtracted from the price
    
    Raises:
        ValueError: If customer_tier is not recognized
        
    Example:
        >>> calculate_discount(100.0, 'gold')
        20.0
    """
    # ...
```

### 💡 註解的維護

**重要原則：如果你修改了程式碼，必須同時更新相關的註解。** 

過時的註解比沒有註解更危險，因為它會誤導讀者。如果你發現註解已經與程式碼不符，請立即更新或刪除它。

---

[← 回目錄](./00-README.md) | [下一章：DRY原則 →](./05-DRY.md)
