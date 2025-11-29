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

[← 回目錄](./00-README.md) | [下一章：條件判斷 →](./03-Conditionals.md)
