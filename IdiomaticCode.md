## 十、撰寫符合語言風格的程式碼 (Idiomatic Code)

Clean Code 不僅僅是遵循通用原則，更深一層的境界是**用該語言最自然、最被社群認可的方式來寫程式**。這就是所謂的「Idiomatic Code」。

這不只是「能動」的程式碼，而是「說著流利母語」的程式碼。這樣寫的好處是：
-   **可讀性極高**：同語言的工程師一看就懂，無需轉換思維。
-   **維護成本低**：符合語言的慣例，更容易被工具支持，也更容易找到解決方案。
-   **更安全與精簡**：語言的慣用寫法通常是經過千錘百鍊，能避開許多常見陷阱。

### 🐍 Python: Pythonic Code

「Pythonic」是指符合《The Zen of Python》精神的程式碼風格。它強調簡潔、可讀性和直接。

**範例：列表生成式 (List Comprehensions) - 過濾與轉換**

-   ❌ **不那麼 Pythonic**
    ```python
    # 傳統迴圈：找出所有大於 5 的數字並平方
    numbers = [1, 3, 5, 7, 9, 2, 4, 6, 8]
    result = []
    for num in numbers:
        if num > 5:
            result.append(num ** 2)
    ```

-   ✅ **Pythonic (使用 List Comprehension)**
    ```python
    # 一行程式碼清晰表達意圖：過濾並轉換
    numbers = [1, 3, 5, 7, 9, 2, 4, 6, 8]
    result = [num ** 2 for num in numbers if num > 5]
    ```

**範例：字典生成式 (Dict Comprehensions)**

-   ❌ **不那麼 Pythonic**
    ```python
    # 建立姓名與長度的對應字典
    names = ["Alice", "Bob", "Charlie"]
    name_lengths = {}
    for name in names:
        name_lengths[name] = len(name)
    ```

-   ✅ **Pythonic (使用 Dict Comprehension)**
    ```python
    # 簡潔的字典生成
    names = ["Alice", "Bob", "Charlie"]
    name_lengths = {name: len(name) for name in names}
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

[← 回目錄](./README.md) | [下一章：一致性 →](./Consistency.md)
