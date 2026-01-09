## 一、命名規則 (Naming)

### ✅ 核心原則

1.  **名稱應清楚表達意圖 (Self-Explanatory)**
    -   變數名稱應該能回答所有關於它的問題：它為何存在？它的作用是什麼？它如何被使用？
    -   好的命名讓程式碼讀起來像一篇流暢的文章，而不是需要解密的謎題。

    **範例對比：**

    ❌ **不好的範例**
    ```typescript
    // 無法理解變數用途
    let d = new Date();
    let temp = user.name;
    let data = fetchData();

    // 需要註解才能理解
    let elapsed = 0; // elapsed time in days

    // 縮寫讓人困惑
    let usrMgr = new UserManager();
    let prjCfg = getProjectConfig();

    // 函式名稱不清楚
    function process(items) { ... }
    function handle(data) { ... }
    function doStuff() { ... }
    ```

    ```python
    # 單字母變數（非迴圈）
    a = calculate_discount()
    x = get_user_data()

    # 無意義的名稱
    def get_them():
        list1 = []
        for x in the_list:
            if x[0] == 4:
                list1.append(x)
        return list1
    ```

    ```csharp
    // 模糊不清的命名
    int value;
    string str;
    List<int> list;

    // 過度縮寫
    var usrAcc = GetUsrAcc();
    var ordDtl = FetchOrdDtl();
    ```

    ✅ **好的範例**
    ```typescript
    // 清楚表達意圖
    let currentDate = new Date();
    let userName = user.name;
    let customerOrders = fetchCustomerOrders();

    // 名稱本身就是文件
    let elapsedTimeInDays = 0;

    // 使用完整名稱
    let userManager = new UserManager();
    let projectConfig = getProjectConfig();

    // 函式名稱明確動作
    function calculateDiscountedPrice(items: Item[]): number { ... }
    function validateUserInput(input: string): boolean { ... }
    function sendConfirmationEmail(recipient: string): void { ... }
    ```

    ```python
    # 描述性的變數名
    discount_percentage = calculate_discount()
    user_profile = get_user_data()

    # 清楚的業務邏輯
    def get_flagged_cells():
        flagged_cells = []
        for cell in game_board:
            if cell.is_flagged:
                flagged_cells.append(cell)
        return flagged_cells
    ```

    ```csharp
    // 明確的商業概念
    decimal totalPrice;
    string customerEmail;
    List<Order> pendingOrders;

    // 完整且有意義的名稱
    var userAccount = GetUserAccount();
    var orderDetails = FetchOrderDetails();
    ```

    **💡 關鍵差異**
    
    好的命名讓你能直接理解：
    - **為什麼**這個變數存在
    - **什麼**資料被儲存
    - **如何**使用這個變數

    不好的命名會讓你問：「這是什麼？」「這用來做什麼？」

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

[← 回目錄](./README.md) | [下一章：函式設計 →](./Functions.md)
