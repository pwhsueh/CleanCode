## 三、條件判斷 (Conditionals)

條件判斷是程式中最常見的邏輯分支，但如果寫得不好，很容易變成複雜、難以閱讀的巢狀結構 (nested structure)。當 `if` 判斷不斷向右延伸，程式碼在視覺上會形成一個類似箭頭或金字塔的形狀，這種結構也常被戲稱為「波動拳」，大幅降低可讀性。

Clean Code 的核心原則是**讓程式碼路徑盡可能保持線性、扁平**，使讀者能像閱讀一般文章一樣順暢地理解。

### ✅ 核心原則：使用衛語句 (Guard Clauses) 提早回傳

**衛語句**是一種程式設計模式，核心思想是在函式開頭先處理所有的**邊界條件 (edge cases)** 或**無效情況**，並立即回傳或拋出錯誤。

這樣做的好處是：
1.  **減少巢狀**：避免了 `if-else` 的層層包裹，讓主邏輯（Happy Path）不再需要縮排。
2.  **提高可讀性**：讀者可以先在函式開頭看到所有例外情況，然後專注於後面的核心業務邏輯。
3.  **職責清晰**：將「檢查」與「執行」分離。

#### 範例：透過「衛語句」和「提煉函式」來扁平化巢狀結構

-   ❌ **不好的寫法 (深度巢狀)**
    ```typescript
    // 縮排層級過多，難以閱讀
    function generateReport(user: User, orders: Order[]) {
      // 第 1 層
      if (user) {
        // 第 2 層
        if (user.isActive) {
          // 第 3 層
          if (user.role === 'Admin' || user.role === 'Manager') {
            let reportLines: string[] = [];
            // 第 4 層
            for (const order of orders) {
              // 第 5 層
              if (order.amount > 1000) {
                reportLines.push(`High value order: ${order.id}`);
              }
            }
            return reportLines;
          } else {
            return ["Error: Insufficient permissions"];
          }
        } else {
          return ["Error: User is not active"];
        }
      } else {
        return ["Error: User not found"];
      }
    }
    ```

-   ✅ **好的寫法 (扁平化)**
    ```typescript
    // 1. 使用衛語句 (Guard Clauses) 提早回傳，處理錯誤或邊界情況
    function generateReport(user: User, orders: Order[]) {
      if (!user) return ["Error: User not found"];
      if (!user.isActive) return ["Error: User is not active"];
      if (!isPrivilegedUser(user)) return ["Error: Insufficient permissions"];
    
      // 核心邏輯變得清晰，不再被 if-else 包圍
      return generateHighValueOrderLines(orders);
    }
    
    // 2. 將權限判斷和報表生成邏輯提煉成獨立函式
    function isPrivilegedUser(user: User): boolean {
      return user.role === 'Admin' || user.role === 'Manager';
    }
    
    function generateHighValueOrderLines(orders: Order[]): string[] {
      const HIGH_VALUE_THRESHOLD = 1000;
      return orders
        .filter(order => order.amount > HIGH_VALUE_THRESHOLD)
        .map(order => `High value order: ${order.id}`);
    }
    ```

### 🧩 更多範例

#### C#
```csharp
// ✅ 使用衛語句
bool IsEligibleForDiscount(User? user)
{
    // 先處理所有無效情況
    if (user == null) return false;
    if (!user.IsActive) return false;
    if (user.IsGoldMember) return true;
    
    // 核心邏輯
    return user.PurchaseHistory.Sum(p => p.Amount) > 1000;
}

// ❌ 不好的寫法 (巢狀)
bool IsEligibleForDiscount_Bad(User? user)
{
    if (user != null)
    {
        if (user.IsActive)
        {
            if (user.IsGoldMember)
            {
                return true;
            }
            else
            {
                return user.PurchaseHistory.Sum(p => p.Amount) > 1000;
            }
        }
    }
    return false;
}
```

#### Python
```python
# ✅ 使用衛語句
def is_eligible(user):
    if not user:
        return False
    if not user.is_active:
        return False
    
    return user.age >= 18

# ❌ 不好的寫法 (巢狀)
def is_eligible_bad(user):
    if user:
        if user.is_active:
            if user.age >= 18:
                return True
            else:
                return False
        else:
            return False
    else:
        return False
```

---

[← 回目錄](./README.md) | [下一章：註解原則 →](./Comments.md)
