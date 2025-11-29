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

### 基礎篇：建立良好習慣
1. 命名規則 (Naming)
2. 函式設計 (Functions)
3. 條件判斷 (Conditionals)
4. 註解原則 (Comments)
5. DRY 原則：避免重複 (Don't Repeat Yourself)

### 進階篇：深化設計思維
6. 理解與管理副作用 (Side Effects)
7. 類別與物件 (Classes & Objects)
8. 錯誤處理 (Error Handling)
9. 魔術數字/字串處理 (Magic Numbers & Strings)

### 架構篇：系統化思考
10. 撰寫符合語言風格的程式碼 (Idiomatic Code)
11. 一致性與風格工具 (Consistency)
12. 抽象層次與依賴反轉 (Abstraction & Dependency)
13. 模組化與測試性 (Modularization & Testability)
14. 不可變性與型別安全 (Immutability & Type Safety)
15. 「高內聚、低耦合」的黃金法則：S.O.L.I.D. 架構思維

### 附錄
16. Clean Code Checklist

---

## 一、命名規則 (Naming)

### ✅ 核心原則

1.  **名稱應清楚表達意圖 (Self-Explanatory)**
    -   變數名稱應該能回答所有關於它的大問題：它為何存在？它的作用是什麼？它如何被使用？
    -   好的命名讓程式碼讀起來像一篇流暢的文章，而不是需要解密的謎題。

2.  **避免縮寫與無意義的名稱 (Avoid Abbreviations & Disinformation)**
    -   **不要使用縮寫**：除非是廣為人知的慣例 (如 `db` 代表 `database`)，否則應使用全名。`userRegistrationService` 遠比 `usrRegSvc` 好。
    -   **避免單字母變數**：除了在迴圈中的 `i`, `j`, `k` 等計數器，應避免使用 `a`, `b`, `x` 等單字母變數。
    -   **不要提供錯誤線索**：例如，一個變數名為 `userList`，但它的型別卻不是 `List` 或 `Array`，這會誤導讀者。

3.  **變數、函式、類別的詞性 (Use Correct Parts of Speech)**
    -   **變數與類別應為名詞或名詞片語**：它們代表物件或概念。
        -   `customer`, `shoppingCart`, `userAccount`
    -   **函式應為動詞或動詞片語**：它們代表動作。
        -   `postPayment()`, `deletePage()`, `save()`
    -   **布林值應像一個判斷題**：讓讀者能直覺地回答「是」或「否」。

### 🧩 具體實踐與範例

**1. 布林值命名：使用 `is`, `has`, `can`, `should` 開頭**

布林值的命名應該讓 `if (variable)` 讀起來像一個流暢的英文句子。

| 語言           | 範例                                                                                                   |
| :------------- | :----------------------------------------------------------------------------------------------------- |
| **C#**         | `bool isVisible = true;`<br/>`bool hasChildren = false;`<br/>`if (canExecute) { ... }`                 |
| **TypeScript** | `let isVisible: boolean = true;`<br/>`let hasChildren: boolean = false;`<br/>`if (canExecute) { ... }` |
| **Python**     | `is_visible = True`<br/>`has_children = False`<br/>`if can_execute: ...`                               |

**2. 變數與類別命名：使用名詞**

變數代表資料，類別代表藍圖，都應使用名詞。

| 語言           | 範例                                                                                                        |
| :------------- | :---------------------------------------------------------------------------------------------------------- |
| **C#**         | `string customerName = "John";`<br/>`int userCount = 10;`<br/>`class OrderProcessor { ... }`                |
| **TypeScript** | `let customerName: string = "John";`<br/>`const userCount: number = 10;`<br/>`class OrderProcessor { ... }` |
| **Python**     | `customer_name = "John"`<br/>`user_count = 10`<br/>`class OrderProcessor: ...`                              |

**3. 函式命名：使用動詞**

函式代表一個動作，應以動詞開頭，清楚說明它「做什麼」。

| 語言           | 範例                                                                                                                |
| :------------- | :------------------------------------------------------------------------------------------------------------------ |
| **C#**         | `void CalculateTotalPrice(...)`<br/>`User GetUserById(int id)`<br/>`void SaveChanges()`                             |
| **TypeScript** | `function calculateTotalPrice(...)`<br/>`function getUserById(id: number): User`<br/>`function saveChanges(): void` |
| **Python**     | `def calculate_total_price(...)`<br/>`def get_user_by_id(id)`<br/>`def save_changes()`                              |

**4. 透過文字表達概念，而非實作細節**

好的命名應該描述「業務意圖」，而不是「資料結構」或「演算法」。這讓程式碼更有彈性，當底層實作改變時，不需要修改呼叫端的程式碼。

- ❌ **不好的命名 (透露實作細節)**
  - `userDict`: 透露了它是一個 Dictionary。
  - `customerArray`: 透露了它是一個 Array。
  - `encryptPasswordWithMD5()`: 將加密演算法寫死在名稱中。如果未來要更換成 `SHA256`，函式名稱就變得不準確，容易造成誤解。

- ✅ **好的命名 (描述業務概念)**
  - `usersById`: 描述了「用 ID 索引使用者」的意圖。
  - `activeCustomers`: 描述了「活躍客戶」的集合。
  - `encryptPassword()`: 只描述「加密密碼」這個核心概念。底層使用何種演算法，是實作細節，不應暴露給呼叫者。

---

## 二、函式設計 (Functions)

### ✅ 核心原則

#### 1. 簡短！
> 關於函式的首要準則，就是要簡短。第二項準則，就是要比第一項的簡短函式還要更簡短。

#### 2. 只做一件事（Single Responsibility）
- 如果能用文字描述函式功能時，包含了「不同層次」的抽象概念步驟，就表示做了不只一件事
- 所有行為應該在「同個層級」

#### 3. 區塊與縮排 (Blocks and Indenting)

-   **將邏輯提煉成函式**：`if`, `else`, `while` 等控制結構的區塊 (body) 應該只包含**一個函式呼叫**。這並非指大括號 `{}` 內只能寫一行程式碼，而是鼓勵將區塊內的複雜邏輯提煉 (extract) 成一個新的、有清楚命名的函式。
-   **避免深度巢狀結構**：函式內的縮排層級應盡量保持在 1-2 層。過深的巢狀結構通常意味著函式承擔了過多職責，應考慮重構。

**範例：提煉 `if` 區塊的邏輯**

-   ❌ **不好的寫法 (區塊內有多行邏輯)**
    ```typescript
    function processPayment(payment: Payment, user: User) {
      if (payment.status === 'SUCCESS') {
        // 1. 更新訂單歷史
        user.orderHistory.push(payment.orderId);
        // 2. 增加紅利點數
        user.points += calculateBonus(payment.amount);
        // 3. 發送確認郵件
        emailService.send(user.email, 'Payment Successful');
      } else {
        // ... 處理失敗的邏輯
      }
    }
    ```

-   ✅ **好的寫法 (區塊內只有一個函式呼叫)**
    ```typescript
    // 提煉出來的新函式，職責單一
    function handleSuccessfulPayment(payment: Payment, user: User) {
      user.orderHistory.push(payment.orderId);
      user.points += calculateBonus(payment.amount);
      emailService.send(user.email, 'Payment Successful');
    }

    // 原本的函式變得非常簡潔易讀
    function processPayment(payment: Payment, user: User) {
      if (payment.status === 'SUCCESS') {
        // if 區塊內只有一行，清楚表達了意圖
        handleSuccessfulPayment(payment, user);
      } else {
        handleFailedPayment(payment, user); // 同理，失敗的邏輯也應提煉
      }
    }
    ```

#### 4. 避免輸出型參數 (Avoid Output Parameters)

「輸出型參數」是指函式不透過 `return` 回傳結果，而是直接修改傳入的參數（通常是物件或陣列）來達成目的。這種作法會產生「副作用」(Side Effect)，讓程式碼變得不直觀且難以追蹤。

**為什麼要避免？**
-   **違反直覺**：我們通常預期函式是接收輸入 (`input`)，然後回傳輸出 (`output`)。如果一個函式偷偷修改了傳入的參數，會讓呼叫者感到困惑。
-   **降低可讀性**：看到 `append(element, list)` 這樣的呼叫，我們無法確定 `list` 是否被改變了，必須去讀函式內部的實作。
-   **難以測試與除錯**：有副作用的函式不易進行單元測試，因為需要驗證傳入的參數狀態是否被正確修改。

**範例：**

-   ❌ **不好的寫法 (修改傳入的陣列)**
    ```typescript
    // 這個函式會直接修改傳入的 `items` 陣列
    function addToList(items: string[], newItem: string) {
      items.push(newItem);
    }
    
    const userList = ["Alice", "Bob"];
    addToList(userList, "Charlie"); // userList 現在變成 ["Alice", "Bob", "Charlie"]
    // 呼叫者可能沒預期到 userList 被改變了
    ```

-   ✅ **好的寫法 1：回傳一個新的陣列 (Immutability)**
    ```typescript
    // 回傳一個包含新項目的全新陣列，不修改原始陣列
    function addToList(items: string[], newItem: string): string[] {
      return [...items, newItem];
    }
    
    const userList = ["Alice", "Bob"];
    const newUserList = addToList(userList, "Charlie");
    // userList 仍然是 ["Alice", "Bob"]
    // newUserList 是 ["Alice", "Bob", "Charlie"]，意圖非常明確
    ```

-   ✅ **好的寫法 2：使用物件導向方法**
    如果操作的對象是一個核心物件，應將操作封裝成該物件的方法。這樣，呼叫者就能清楚地知道是在操作該物件的狀態。
    ```typescript
    class ShoppingCart {
      private items: string[] = [];
    
      // 方法的意圖很清楚：就是要修改這個購物車的內部狀態
      addItem(item: string): void {
        this.items.push(item);
      }
    
      getItems(): readonly string[] {
        return this.items;
      }
    }
    
    const cart = new ShoppingCart();
    cart.addItem("Apple"); // 呼叫者明確知道 cart 的狀態會被改變
    ```

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
function processUser(user: User): void {
  validateUser(user);
  saveUser(user);
  notifyUser(user);
}

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
- ❌ **壞例子**：「建立資料庫連線,取得被插旗的地雷格子，並寫入日誌」- 包含不同抽象層次的步驟

### 6. 函式參數的數量 (Number of Arguments)

函式的參數數量是衡量其複雜性的重要指標。

**核心原則：**
-   **0-2 個參數是理想的**：這是最容易理解和測試的情況。
-   **3 個參數需要有充分理由**：應謹慎使用。
-   **超過 3 個參數是個警訊 (Code Smell)**：通常代表函式可能承擔了過多職責，或是可以進行更好的封裝。

**為什麼過多的參數是個問題？**
-   **降低可讀性**：`createUser(name, age, email, address, role, isActive)` 這樣的函式呼叫，很難記住每個參數的順序和意義。
-   **增加測試複雜度**：你需要為參數的各種組合撰寫測試案例，數量會呈指數級增長。
-   **容易出錯**：傳遞參數時很容易搞錯順序，尤其當它們的型別相同時 (例如多個 `string` 參數)。

**解決方案：引入參數物件 (Introduce Parameter Object)**

當你發現多個參數總是結伴出現時，就應該將它們封裝成一個獨立的類別或介面。

**範例：**

-   ❌ **不好的寫法 (過多參數)**
    ```typescript
    function createChart(
      type: 'line' | 'bar', 
      data: number[], 
      color: string, 
      width: number, 
      height: number, 
      showLegend: boolean
    ) {
      // ...
    }
    ```

-   ✅ **好的寫法 (使用參數物件)**

    **TypeScript**
    ```typescript
    // 1. 定義參數物件
    interface ChartOptions {
      type: 'line' | 'bar';
      data: number[];
      color?: string;
      width?: number;
      height?: number;
      showLegend?: boolean;
    }

    // 2. 函式只接收一個物件
    function createChart(options: ChartOptions) {
      const { type, data, color = '#000000', width = 600, height = 400, showLegend = true } = options;
      // ...
    }

    // 呼叫時更清晰，且順序無關
    createChart({
      type: 'bar',
      data: [1, 2, 3],
      color: 'blue'
    });
    ```

    **C#**
    ```csharp
    // 1. 定義參數物件
    public class ChartOptions
    {
        public string Type { get; set; }
        public IEnumerable<int> Data { get; set; }
        public string Color { get; set; } = "#000000";
        public int Width { get; set; } = 600;
        public int Height { get; set; } = 400;
        public bool ShowLegend { get; set; } = true;
    }

    // 2. 函式接收參數物件
    public void CreateChart(ChartOptions options)
    {
        // ... 使用 options.Type, options.Data ...
    }

    // 呼叫時使用物件初始化器，非常清晰
    CreateChart(new ChartOptions 
    {
        Type = "bar",
        Data = new[] { 1, 2, 3 },
        Color = "blue"
    });
    ```

    **Python**
    ```python
    from dataclasses import dataclass
    from typing import List

    # 1. 定義參數物件 (使用 dataclass)
    @dataclass
    class ChartOptions:
        type: str
        data: List[int]
        color: str = "#000000"
        width: int = 600
        height: int = 400
        show_legend: bool = True

    # 2. 函式接收參數物件
    def create_chart(options: ChartOptions):
        # ... 使用 options.type, options.data ...
        pass

    # 呼叫時更具可讀性
    create_chart(ChartOptions(
        type='bar',
        data=[1, 2, 3],
        color='blue'
    ))
    ```

---

## 三、條件判斷 (Conditionals)

條件判斷是程式中最常見的邏輯分支，但如果寫得不好，很容易變成複雜、難以閱讀的巢狀結構 (nested structure)。當 `if` 判斷不斷向右延伸，程式碼在視覺上會形成一個類似箭頭或金字塔的形狀，這種結構也常被戲稱為「波動拳」，大幅降低可讀性。

Clean Code 的核心原則是**讓程式碼路徑盡可能保持線性、扁平**，使讀者能像閱讀一般文章一樣順暢地理解。

### ✅ 核心原則：使用衛述句 (Guard Clauses) 提早回傳

**衛述句**是一種程式設計模式，核心思想是在函式開頭先處理所有的**邊界條件 (edge cases)** 或**無效情況**，並立即回傳或拋出錯誤。

這樣做的好處是：
1.  **減少巢狀**：避免了 `if-else` 的層層包裹，讓主邏輯（Happy Path）不再需要縮排。
2.  **提高可讀性**：讀者可以先在函式開頭看到所有例外情況，然後專注於後面的核心業務邏輯。
3.  **職責清晰**：將「檢查」與「執行」分離。

#### 範例：透過「衛述句」和「提煉函式」來扁平化巢狀結構

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
    // 1. 使用衛述句 (Guard Clauses) 提早回傳，處理錯誤或邊界情況
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
// ✅ 使用衛述句
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
# ✅ 使用衛述句
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

## 六、理解與管理副作用 (Side Effects)

在函式設計中，最核心也最容易被忽略的概念之一就是「副作用」。理解並妥善管理副作用，是區分普通程式碼與高品質、可維護程式碼的關鍵。

### ❓ 什麼是副作用 (Side Effect)？

**副作用**是指函式在執行過程中，除了回傳一個值之外，還對**函式外部的可見狀態**產生了任何影響。

常見的副作用包括：
-   修改全域變數或靜態變數。
-   修改傳入的參數（如物件或陣列）。
-   執行任何 I/O 操作，例如：
    -   寫入檔案或資料庫。
    -   呼叫外部 API。
    -   在控制台（Console）上印出日誌。
-   觸發一個事件或訊息。

### ✨ 什麼是純函式 (Pure Function)？

與副作用相對的概念是「純函式」。一個函式如果滿足以下兩個條件，就是純函式：
1.  **相同的輸入永遠得到相同的輸出**：函式的回傳值只依賴於其輸入參數，不受任何外部狀態影響。
2.  **沒有可觀察的副作用**：函式不會修改任何外部狀態。

純函式就像一個可靠的數學公式，例如 `sum(2, 3)` 永遠會回傳 `5`，無論你呼叫它多少次，也不會影響到系統的其他部分。

### 💔 為什麼副作用是個問題？

-   **不可預測性**：有副作用的函式很難預測其行為。`calculateTotal()` 可能這次回傳 `100`，下次因為某個全域折扣變數被修改而回傳 `80`。
-   **難以測試**：測試有副作用的函式很麻煩。你需要模擬（Mock）資料庫、API，並驗證外部狀態是否被正確修改。而測試純函式只需要給定輸入並斷言輸出即可。
-   **降低可讀性與可維護性**：當你看到一個函式呼叫時，如果它是純函式，你只需要關心它的回傳值。如果它有副作用，你還必須追蹤它可能對系統產生的所有潛在影響。
-   **併發問題**：在多執行緒環境下，如果多個執行緒同時呼叫一個會修改共享狀態的函式，就會產生競爭條件 (Race Condition)，導致不可預期的錯誤。

### 🛠️ 如何管理副作用？

副作用是不可避免的，例如我們總需要將資料存入資料庫。關鍵不是完全消滅副作用，而是**將它們與核心業務邏輯分離**。

**策略：將純邏輯與不純的行為分離**

讓大部分的程式碼（特別是複雜的業務邏輯）保持純粹，並將副作用推向系統的邊緣（例如 Controller、主程式進入點）。

#### 範例：計算購物車總價

-   ❌ **不好的寫法 (副作用與邏輯混合)**
    ```typescript
    // 全域變數，可能在任何地方被修改
    let taxRate = 0.05; 

    // 這個函式有副作用：讀取了全域變數 taxRate
    function calculateTotal(items: { price: number }[]): number {
      const subtotal = items.reduce((sum, item) => sum + item.price, 0);
      const tax = subtotal * taxRate; // 依賴外部狀態
      return subtotal + tax;
    }
    ```

-   ✅ **好的寫法 (分離副作用)**
    ```typescript
    // 1. 純函式：核心計算邏輯
    // 不再依賴任何外部狀態，所有需要的資訊都透過參數傳入
    function calculateTotal(items: { price: number }[], taxRate: number): number {
      const subtotal = items.reduce((sum, item) => sum + item.price, 0);
      const tax = subtotal * taxRate;
      return subtotal + tax;
    }

    // 2. 不純的程式碼 (系統邊緣)
    // 負責獲取外部狀態，並呼叫純函式
    function main() {
      const items = [{ price: 100 }, { price: 200 }];
      const currentTaxRate = database.getTaxRate(); // 副作用：讀取資料庫
      
      // 呼叫純函式來執行核心邏輯
      const total = calculateTotal(items, currentTaxRate);
      
      console.log(total); // 副作用：寫入控制台
    }
    ```

#### C# 範例
```csharp
// ❌ 不純的函式
public class ImpureCalculator
{
    public static double TaxRate = 0.2; // 靜態變數 (外部狀態)

    public double Calculate(double price)
    {
        return price * (1 + TaxRate); // 依賴外部狀態
    }
}

// ✅ 純函式
public class PureCalculator
{
    // 所有依賴都透過參數傳入
    public double Calculate(double price, double taxRate)
    {
        return price * (1 + taxRate);
    }
}
```

#### Python 範例
```python
# ❌ 不純的函式
user_list = [] # 全域變數

def add_user(name):
    # 副作用：修改了全域變數
    user_list.append({"name": name})

# ✅ 純函式
def add_user_pure(users, name):
    # 回傳一個新的 list，不修改原始的 users
    return users + [{"name": name}]

# --- 使用 ---
initial_users = []
new_users = add_user_pure(initial_users, "Alice")
# initial_users 仍然是 []
# new_users 是 [{"name": "Alice"}]
```

---

## 七、類別與物件 (Classes & Objects)

> 「類別應該簡短！」— Robert C. Martin

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
- 類別的成員變數應該盡量少（建議 5 個以內）

**判斷標準**：如果你需要「向下捲動才能看完一個類別」，它可能太大了。

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
3. **成員變數應該少**：超過 5-7 個成員變數通常代表職責過多
4. **公開方法應該少**：一個類別不應該暴露超過 10 個公開方法
5. **避免「資料類別」**：只有 getter/setter 沒有行為的類別通常是設計不良的徵兆
6. **避免「神級類別」(God Class)**：一個類別不應該知道或控制太多事情

---

## 八、錯誤處理 (Error Handling)

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

## 九、魔術數字/字串處理 (Magic Numbers & Strings)
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

## 十、撰寫符合語言風格的程式碼 (Idiomatic Code)

Clean Code 不僅僅是遵循通用原則，更深一層的境界是**用該語言最自然、最被社群認可的方式來寫程式**。這就是所謂的「Idiomatic Code」。

這不只是「能動」的程式碼，而是「說著流利母語」的程式碼。這樣寫的好處是：
-   **可讀性極高**：同語言的工程師一看就懂，無需轉換思維。
-   **維護成本低**：符合語言的慣例，更容易被工具支持，也更容易找到解決方案。
-   **更安全與精簡**：語言的慣用寫法通常是經過千錘百鍊，能避開許多常見陷阱。

### 🐍 Python: Pythonic Code

「Pythonic」是指符合《The Zen of Python》精神的程式碼風格。它強調簡潔、可讀性和直接。

**範例：遍歷一個列表**

-   ❌ **不那麼 Pythonic (C-style loop)**
    ```python
    items = ["apple", "banana", "cherry"]
    for i in range(len(items)):
        print(items[i])
    ```

-   ✅ **Pythonic**
    ```python
    items = ["apple", "banana", "cherry"]
    # 直接遍歷元素，更直觀
    for item in items:
        print(item)
    ```

**範例：列表生成式 (List Comprehensions)**

-   ❌ **不那麼 Pythonic**
    ```python
    squares = []
    for i in range(10):
        squares.append(i * i)
    ```

-   ✅ **Pythonic**
    ```python
    # 一行程式碼表達意圖
    squares = [i * i for i in range(10)]
    ```

### 💠 C#: Idiomatic C# / .NET-style Code

現代 C# 的慣用風格大量運用 LINQ、`async/await` 和表達式主體成員 (Expression-bodied members) 來撰寫流暢、聲明式的程式碼。

**範例：篩選集合**

-   ❌ **不那麼 Idiomatic (傳統 `foreach`)**
    ```csharp
    var adults = new List<User>();
    foreach (var user in users)
    {
        if (user.Age >= 18)
        {
            adults.Add(user);
        }
    }
    ```

-   ✅ **Idiomatic C# (使用 LINQ)**
    ```csharp
    // 使用 LINQ，意圖清晰，程式碼更簡潔
    var adults = users.Where(user => user.Age >= 18).ToList();
    ```

**範例：非同步程式碼**

-   ❌ **不那麼 Idiomatic (手動處理 Task)**
    ```csharp
    public Task<string> GetData()
    {
        return Task.Run(() => {
            // ... 複雜的同步操作 ...
            return "data";
        });
    }
    ```

-   ✅ **Idiomatic C# (使用 async/await)**
    ```csharp
    public async Task<string> GetDataAsync()
    {
        // 讓編譯器處理非同步的複雜性
        var result = await _httpClient.GetStringAsync("url");
        return result;
    }
    ```

### 📜 TypeScript: Idiomatic TypeScript

Idiomatic TypeScript 的核心在於充分運用其強大的型別系統和現代 JavaScript 特性（如 ES6+ 語法）來增強程式碼的健壯性和可讀性。

**範例：善用型別與可選鏈 (Optional Chaining)**

-   ❌ **不那麼 Idiomatic (手動檢查 null/undefined)**
    ```typescript
    if (user && user.profile && user.profile.address) {
      console.log(user.profile.address.city);
    }
    ```

-   ✅ **Idiomatic TypeScript**
    ```typescript
    const city = user?.profile?.address?.city ?? "Default City";
    console.log(city);
    ```

**範例：使用 `map` 和 `filter` 而非 `for` 迴圈**

-   ❌ **不那麼 Idiomatic (命令式)**
    ```typescript
    const activeUserNames: string[] = [];
    for (const user of users) {
      if (user.isActive) {
        activeUserNames.push(user.name);
      }
    }
    ```

-   ✅ **Idiomatic TypeScript (聲明式)**
    ```typescript
    // 鏈式呼叫，清楚表達轉換過程
    const activeUserNames = users
      .filter(user => user.isActive)
      .map(user => user.name);
    ```

---

## 十一、一致性與風格工具 (Consistency)

| 工具              | 功能               | 適用語言   |
| ----------------- | ------------------ | ---------- |
| Prettier          | 排版格式化         | TypeScript |
| ESLint            | 語法檢查           | TypeScript |
| EditorConfig      | 統一縮排與換行     | 全語言     |
| StyleCop / Roslyn | 程式風格檢查       | C#         |
| Black / flake8    | 自動排版、靜態檢查 | Python     |

---

## 十二、抽象層次與依賴反轉 (Abstraction & Dependency)

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

## 十三、模組化與測試性 (Modularization & Testability)

```
src/
 ├── domain/      # 業務邏輯
 ├── infra/       # 外部資源 (DB, API)
 ├── app/         # 控制流程
 ├── tests/       # 單元測試
```

---

## 十四、不可變性與型別安全 (Immutability & Type Safety)

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

## 十六、附錄：Clean Code Checklist

- [ ] **命名**
  - [ ] 名稱清楚表達意圖
  - [ ] 避免縮寫與無意義的名稱
  - [ ] 變數、函式、類別使用正確的詞性

- [ ] **函式設計**
  - [ ] 函式簡短
  - [ ] 函式只做一件事
  - [ ] 區塊內只有一個函式呼叫
  - [ ] 避免深度巢狀結構
  - [ ] 避免輸出型參數
  - [ ] 函式名稱反映目的

- [ ] **副作用管理**
  - [ ] 理解副作用與純函式
  - [ ] 副作用與邏輯分離

- [ ] **條件判斷**
  - [ ] 使用早期回傳
  - [ ] 避免巢狀結構

- [ ] **類別與物件**
  - [ ] 單一職責原則
  - [ ] 用封裝保護內部狀態

- [ ] **註解**
  - [ ] 說明「為什麼」而非「做什麼」

- [ ] **重構與避免重複**
  - [ ] 適時重構
  - [ ] 避免程式碼重複

- [ ] **錯誤處理**
  - [ ] 使用例外處理機制
  - [ ] 定義自訂例外類別

- [ ] **一致性與風格工具**
  - [ ] 使用風格檢查工具
  - [ ] 團隊內部風格一致

- [ ] **撰寫符合語言風格的程式碼**
  - [ ] Pythonic Code
  - [ ] Idiomatic C# / .NET-style Code
  - [ ] Idiomatic TypeScript

- [ ] **進階：抽象層次與依賴反轉**
  - [ ] 依賴抽象而非具體實作
  - [ ] 使用介面與抽象類別

- [ ] **進階：模組化與測試性**
  - [ ] 適當模組化
  - [ ] 易於測試

- [ ] **進階：不可變性與型別安全**
  - [ ] 使用不可變資料結構
  - [ ] 強型別檢查

- [ ] **進階：魔術數字/字串處理**
  - [ ] 使用具名常數
  - [ ] 使用列舉

- [ ] **進階：「高內聚、低耦合」的黃金法則：S.O.L.I.D. 架構思維**
  - [ ] 單一職責原則
  - [ ] 開放封閉原則
  - [ ] 里氏替換原則
  - [ ] 介面隔離原則
  - [ ] 依賴反轉原則
