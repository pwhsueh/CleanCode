## 五、DRY 原則：避免重複 (Don't Repeat Yourself)

> 「每一項知識在系統中都必須有單一、明確、權威的表達。」— Andy Hunt & Dave Thomas, The Pragmatic Programmer

### ❓ 什麼是 DRY？

**DRY (Don't Repeat Yourself)** 是軟體開發中最重要的原則之一。它的核心概念是：**相同的知識或邏輯不應該在系統中重複出現多次。**

當你發現自己在複製貼上程式碼時，這通常是一個警訊，表示你違反了 DRY 原則。

### 💔 為什麼重複是個問題？

1.  **維護困難**：當相同邏輯散落在多處，修改時必須同步更新所有地方，容易遺漏造成不一致。
2.  **增加 Bug 風險**：修復一個 Bug 時，可能忘記修復其他重複的地方。
3.  **降低可讀性**：重複的程式碼會讓檔案變得冗長，核心邏輯被稀釋。
4.  **違反單一真相來源原則**：沒有一個明確的「權威版本」，每個副本都可能略有不同。

### 🧩 識別重複的類型

#### 1. 直接重複 (Literal Duplication)

最明顯的重複形式：相同或幾乎相同的程式碼出現在多個地方。

-   ❌ **不好的寫法**
    ```typescript
    function calculateEmployeeSalary(employee: Employee) {
      let salary = employee.baseSalary;
      salary = salary + (salary * 0.1); // 10% bonus
      salary = salary - (salary * 0.05); // 5% tax
      return salary;
    }

    function calculateContractorPay(contractor: Contractor) {
      let pay = contractor.hourlyRate * contractor.hoursWorked;
      pay = pay + (pay * 0.1); // 10% bonus (重複)
      pay = pay - (pay * 0.05); // 5% tax (重複)
      return pay;
    }
    ```

-   ✅ **好的寫法 (提煉共同邏輯)**
    ```typescript
    // 提煉出共同的計算邏輯
    function applyBonusAndTax(amount: number): number {
      const BONUS_RATE = 0.1;
      const TAX_RATE = 0.05;
      return amount * (1 + BONUS_RATE) * (1 - TAX_RATE);
    }

    function calculateEmployeeSalary(employee: Employee): number {
      return applyBonusAndTax(employee.baseSalary);
    }

    function calculateContractorPay(contractor: Contractor): number {
      const grossPay = contractor.hourlyRate * contractor.hoursWorked;
      return applyBonusAndTax(grossPay);
    }
    ```

#### 2. 結構性重複 (Structural Duplication)

邏輯結構相似，但細節不同。這種重複更隱蔽，但同樣有害。

-   ❌ **不好的寫法**
    ```csharp
    public void SendEmailNotification(User user, string message) {
        if (!user.EmailVerified) return;
        var emailService = new EmailService();
        emailService.Connect();
        emailService.Send(user.Email, message);
        emailService.Disconnect();
    }

    public void SendSmsNotification(User user, string message) {
        if (!user.PhoneVerified) return;
        var smsService = new SmsService();
        smsService.Connect();
        smsService.Send(user.Phone, message);
        smsService.Disconnect();
    }

    public void SendPushNotification(User user, string message) {
        if (!user.PushEnabled) return;
        var pushService = new PushService();
        pushService.Connect();
        pushService.Send(user.DeviceId, message);
        pushService.Disconnect();
    }
    ```

-   ✅ **好的寫法 (使用策略模式或介面抽象)**
    ```csharp
    // 定義通用介面
    public interface INotificationService {
        void Connect();
        void Send(string recipient, string message);
        void Disconnect();
    }

    // 統一的發送邏輯
    public void SendNotification(
        INotificationService service, 
        string recipient, 
        string message) 
    {
        service.Connect();
        service.Send(recipient, message);
        service.Disconnect();
    }

    // 使用時
    if (user.EmailVerified) {
        SendNotification(new EmailService(), user.Email, message);
    }
    if (user.PhoneVerified) {
        SendNotification(new SmsService(), user.Phone, message);
    }
    ```

#### 3. 條件邏輯重複 (Conditional Duplication)

相同的條件判斷邏輯散落在多處。

-   ❌ **不好的寫法**
    ```python
    def can_view_admin_panel(user):
        return user.role == 'Admin' or user.role == 'SuperAdmin'

    def can_delete_users(user):
        return user.role == 'Admin' or user.role == 'SuperAdmin'

    def can_modify_settings(user):
        return user.role == 'Admin' or user.role == 'SuperAdmin'
    ```

-   ✅ **好的寫法 (集中定義權限邏輯)**
    ```python
    def is_admin(user):
        """單一真相來源：定義什麼是管理員"""
        return user.role in ['Admin', 'SuperAdmin']

    def can_view_admin_panel(user):
        return is_admin(user)

    def can_delete_users(user):
        return is_admin(user)

    def can_modify_settings(user):
        return is_admin(user)
    ```

### 🎯 如何實踐 DRY？

#### 1. 提煉函式 (Extract Function)

將重複的程式碼提煉成一個獨立的、有清楚命名的函式。

#### 2. 使用常數或配置 (Constants & Configuration)

避免魔術數字和字串的重複。

```typescript
// ❌ 不好
if (user.age >= 18) { ... }
if (user.age >= 18) { ... } // 重複

// ✅ 好
const ADULT_AGE = 18;
if (user.age >= ADULT_AGE) { ... }
if (user.age >= ADULT_AGE) { ... }
```

#### 3. 使用繼承或組合 (Inheritance or Composition)

當多個類別有相同行為時，考慮提煉到父類別或共享的工具類別。

#### 4. 利用高階函式與泛型 (Higher-Order Functions & Generics)

```typescript
// ❌ 重複的過濾邏輯
const activeUsers = users.filter(u => u.isActive);
const activeOrders = orders.filter(o => o.isActive);

// ✅ 提煉共同邏輯
const filterActive = <T extends { isActive: boolean }>(items: T[]) => 
  items.filter(item => item.isActive);

const activeUsers = filterActive(users);
const activeOrders = filterActive(orders);
```

### ⚠️ 注意：不要過度 DRY

**並非所有相似的程式碼都應該合併。** 有時候，兩段程式碼雖然現在看起來相似，但它們背後的**業務意圖或變化方向**可能完全不同。

如果強行合併這種「巧合的相似性」，未來當其中一個需求改變時，你就必須拆分它們，反而增加複雜度。

**經驗法則：「三次重複原則 (Rule of Three)」**
-   第一次寫：直接實現
-   第二次看到類似邏輯：注意到但先不重構
-   第三次重複出現：這時應該認真考慮提煉共同邏輯

### 💡 DRY 的本質

DRY 不只是避免「程式碼重複」，更深層的意義是避免「知識重複」。每一個業務規則、每一個演算法、每一個資料結構的定義，都應該在系統中只有一個權威版本。

當你需要修改某個邏輯時，應該只需要改一個地方，而不是在整個系統中搜尋所有可能的副本。

---

[← 回目錄](./README.md) | [下一章：魔術數字 →](./MagicNumbers.md)
