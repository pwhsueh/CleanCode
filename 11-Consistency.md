## 十一、一致性與風格工具 (Consistency)

> 「程式碼的風格應該像是出自同一人之手，即使實際上是由一個團隊完成的。」

### ❓ 為什麼一致性如此重要？

在團隊協作中，**一致性 (Consistency)** 往往比「誰的風格最好」更重要。不一致的程式碼風格會造成：

#### 1. **認知負荷增加**
當每個檔案的縮排、命名、結構都不同時，開發者需要不斷適應不同的風格，消耗心智能量。

```typescript
// ❌ 風格不一致的程式碼
// File A: 使用單引號、2 空格縮排
function getUserName(user) {
  return user.name;
}

// File B: 使用雙引號、4 空格縮排
function getUserEmail(user) {
    return user.email;
}

// File C: 使用 tab 縮排、結尾分號不一致
function getUserAge(user) {
	return user.age
}
```

#### 2. **程式碼審查困難**
在 Pull Request 中，如果有人「順手」調整了格式，會產生大量無意義的 diff，掩蓋真正的邏輯變更。

```diff
// ❌ 格式變更掩蓋邏輯變更
- function calculateTotal(items) {
-   return items.reduce((sum, item) => sum + item.price, 0);
+ function calculateTotal(items) {
+     return items.reduce((sum, item) => sum + item.price * 1.1, 0);  // 加了稅
}
```

#### 3. **合併衝突頻繁**
不同開發者使用不同的格式化工具或設定，導致同一段程式碼被反覆重新格式化，造成不必要的 Git 衝突。

#### 4. **降低專業形象**
對外開源專案如果風格混亂，會給人「不專業」、「缺乏維護」的印象。

---

### ✅ 解決方案：自動化風格工具

**核心原則**：
- **不要依賴人工檢查**：人會疲勞、會忘記、會有主觀偏好
- **自動化一切**：讓工具在儲存檔案或提交程式碼時自動格式化
- **團隊共識**：選定工具和配置後，所有成員都必須遵守

---

### 🛠️ 各語言的風格工具生態系統

#### **TypeScript / JavaScript**

| 工具             | 功能                        | 特色                                         | 配置複雜度 |
| ---------------- | --------------------------- | -------------------------------------------- | ---------- |
| **Prettier**     | 程式碼格式化                | 零配置、固執己見 (Opinionated)、支援多種語言 | ⭐ 低       |
| **ESLint**       | 語法與風格檢查              | 可高度自訂、可自動修復部分問題               | ⭐⭐⭐ 中高   |
| **Biome**        | 格式化 + Linting (新興工具) | 極快速度、取代 Prettier + ESLint             | ⭐⭐ 中      |
| **EditorConfig** | 統一編輯器設定              | 跨語言、跨編輯器                             | ⭐ 低       |

**推薦組合**：Prettier + ESLint + EditorConfig

**1. Prettier：專注於格式化**

Prettier 的哲學是「不要浪費時間爭論格式」，它提供極少的配置選項，強制統一風格。

```json
// .prettierrc.json
{
  "semi": true,              // 結尾加分號
  "singleQuote": true,       // 使用單引號
  "trailingComma": "es5",    // 物件/陣列最後一項加逗號
  "printWidth": 100,         // 每行最大字元數
  "tabWidth": 2,             // 縮排寬度
  "arrowParens": "always"    // 箭頭函式參數永遠加括號
}
```

**使用方式**：
```bash
# 安裝
npm install --save-dev prettier

# 格式化所有檔案
npx prettier --write .

# 檢查格式（CI 用）
npx prettier --check .
```

**VS Code 整合**：
```json
// .vscode/settings.json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "editor.formatOnPaste": true
}
```

**2. ESLint：專注於程式品質與風格檢查**

ESLint 不只檢查格式，更重要的是檢查潛在錯誤、反模式、最佳實踐。

```javascript
// .eslintrc.js
module.exports = {
  extends: [
    'eslint:recommended',           // ESLint 推薦規則
    'plugin:@typescript-eslint/recommended',  // TypeScript 規則
    'prettier'                      // 關閉與 Prettier 衝突的規則
  ],
  rules: {
    // 禁止使用 var
    'no-var': 'error',
    
    // 優先使用 const
    'prefer-const': 'error',
    
    // 禁止未使用的變數
    'no-unused-vars': ['error', { 
      argsIgnorePattern: '^_'  // 允許 _開頭的未使用參數
    }],
    
    // 強制使用 === 而非 ==
    'eqeqeq': ['error', 'always'],
    
    // 禁止 console.log (生產環境)
    'no-console': process.env.NODE_ENV === 'production' ? 'error' : 'warn',
    
    // 函式複雜度限制
    'complexity': ['warn', 10],
    
    // 最大巢狀深度
    'max-depth': ['warn', 4],
    
    // 函式最大行數
    'max-lines-per-function': ['warn', { max: 50, skipBlankLines: true }]
  }
};
```

**自動修復**：
```bash
# 自動修復可修復的問題
npx eslint --fix .

# 只檢查不修復
npx eslint .
```

**3. EditorConfig：統一基本編輯器設定**

確保團隊成員無論使用 VS Code、IntelliJ、Vim，都有一致的基本設定。

```ini
# .editorconfig
root = true

[*]
charset = utf-8
end_of_line = lf               # 統一使用 LF (Unix)
insert_final_newline = true    # 檔案結尾加空行
trim_trailing_whitespace = true # 移除行尾空白

[*.{js,ts,jsx,tsx,json}]
indent_style = space
indent_size = 2

[*.{md,mdx}]
trim_trailing_whitespace = false

[*.py]
indent_size = 4
```

**完整 TypeScript 專案設定範例**：

```json
// package.json
{
  "scripts": {
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "lint": "eslint . --ext .ts,.tsx",
    "lint:fix": "eslint . --ext .ts,.tsx --fix",
    "check": "npm run format:check && npm run lint"
  },
  "devDependencies": {
    "@typescript-eslint/eslint-plugin": "^6.0.0",
    "@typescript-eslint/parser": "^6.0.0",
    "eslint": "^8.0.0",
    "eslint-config-prettier": "^9.0.0",
    "prettier": "^3.0.0"
  }
}
```

**Git Hooks (自動化檢查)**：
```bash
# 安裝 husky 和 lint-staged
npm install --save-dev husky lint-staged

# 初始化 husky
npx husky init
```

```json
// package.json
{
  "lint-staged": {
    "*.{js,ts,tsx,jsx}": [
      "prettier --write",
      "eslint --fix"
    ],
    "*.{json,md,yml}": [
      "prettier --write"
    ]
  }
}
```

```bash
# .husky/pre-commit
#!/bin/sh
npx lint-staged
```

---

#### **C#**

| 工具                       | 功能           | 特色                        | 配置複雜度 |
| -------------------------- | -------------- | --------------------------- | ---------- |
| **StyleCop**               | 程式風格檢查   | 強制 C# 編碼規範            | ⭐⭐ 中      |
| **Roslyn Analyzers**       | 靜態分析       | 內建於 .NET SDK、可自訂規則 | ⭐⭐ 中      |
| **EditorConfig**           | 統一編輯器設定 | 跨 IDE 支援                 | ⭐ 低       |
| **.editorconfig for .NET** | .NET 專用配置  | 定義 C# 命名與格式規則      | ⭐⭐ 中      |

**推薦組合**：Roslyn Analyzers + EditorConfig

**1. EditorConfig for .NET**

```ini
# .editorconfig
root = true

# C# 檔案規則
[*.cs]
indent_style = space
indent_size = 4
end_of_line = crlf
charset = utf-8-bom

# 命名規則
dotnet_naming_rule.interfaces_should_be_prefixed_with_i.severity = error
dotnet_naming_rule.interfaces_should_be_prefixed_with_i.symbols = interface
dotnet_naming_rule.interfaces_should_be_prefixed_with_i.style = begins_with_i

dotnet_naming_symbols.interface.applicable_kinds = interface

dotnet_naming_style.begins_with_i.required_prefix = I
dotnet_naming_style.begins_with_i.capitalization = pascal_case

# 程式碼風格規則
csharp_prefer_braces = true:warning
csharp_prefer_simple_using_statement = true:suggestion
csharp_style_namespace_declarations = file_scoped:warning

# var 使用
csharp_style_var_for_built_in_types = false:suggestion
csharp_style_var_when_type_is_apparent = true:suggestion
csharp_style_var_elsewhere = false:suggestion

# 表達式偏好
dotnet_style_prefer_auto_properties = true:warning
dotnet_style_prefer_conditional_expression_over_assignment = true:suggestion

# 空值檢查
dotnet_style_coalesce_expression = true:warning
dotnet_style_null_propagation = true:warning

# 程式碼品質規則
dotnet_code_quality_unused_parameters = all:warning
```

**2. StyleCop.Analyzers**

```xml
<!-- .csproj -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="StyleCop.Analyzers" Version="1.2.0-beta.435">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
    </PackageReference>
  </ItemGroup>
</Project>
```

```json
// stylecop.json
{
  "$schema": "https://raw.githubusercontent.com/DotNetAnalyzers/StyleCopAnalyzers/master/StyleCop.Analyzers/StyleCop.Analyzers/Settings/stylecop.schema.json",
  "settings": {
    "documentationRules": {
      "companyName": "YourCompany",
      "copyrightText": "Copyright (c) {companyName}. All rights reserved."
    },
    "orderingRules": {
      "usingDirectivesPlacement": "outsideNamespace"
    }
  }
}
```

**使用 dotnet format**：
```bash
# 格式化專案
dotnet format

# 檢查格式（CI 用）
dotnet format --verify-no-changes

# 只檢查特定規則
dotnet format --diagnostics IDE0005
```

**Visual Studio / Rider 整合**：
```json
// .vscode/settings.json (VS Code with C# extension)
{
  "omnisharp.enableEditorConfigSupport": true,
  "omnisharp.enableRoslynAnalyzers": true,
  "editor.formatOnSave": true
}
```

---

#### **Python**

| 工具       | 功能                 | 特色                             | 配置複雜度 |
| ---------- | -------------------- | -------------------------------- | ---------- |
| **Black**  | 程式碼格式化         | 零妥協、完全自動化               | ⭐ 低       |
| **isort**  | import 排序          | 自動整理 import 順序             | ⭐ 低       |
| **flake8** | 語法與風格檢查       | PEP 8 合規性檢查                 | ⭐⭐ 中      |
| **pylint** | 深度靜態分析         | 更嚴格的程式碼品質檢查           | ⭐⭐⭐ 高     |
| **mypy**   | 型別檢查             | 靜態型別分析                     | ⭐⭐ 中      |
| **Ruff**   | 超快速 Linter (新興) | 取代 flake8/pylint/isort，極快速 | ⭐⭐ 中      |

**推薦組合**：Black + isort + flake8 + mypy (或直接用 Ruff)

**1. Black：不妥協的格式化工具**

Black 的口號是「任何顏色，只要是黑色」。它幾乎沒有配置選項，強制統一風格。

```toml
# pyproject.toml
[tool.black]
line-length = 100
target-version = ['py311']
include = '\.pyi?$'
extend-exclude = '''
/(
  # 排除的目錄
  \.git
  | \.venv
  | build
  | dist
)/
'''
```

```bash
# 格式化所有 Python 檔案
black .

# 檢查格式
black --check .

# 只顯示 diff 不修改
black --diff .
```

**2. isort：自動整理 import**

```toml
# pyproject.toml
[tool.isort]
profile = "black"  # 與 Black 相容
line_length = 100
multi_line_output = 3
include_trailing_comma = true
force_grid_wrap = 0
use_parentheses = true
ensure_newline_before_comments = true

# import 順序
sections = ["FUTURE", "STDLIB", "THIRDPARTY", "FIRSTPARTY", "LOCALFOLDER"]
known_first_party = ["myapp"]
```

```bash
# 整理 import
isort .

# 檢查順序
isort --check-only .
```

**3. flake8：PEP 8 合規性檢查**

```ini
# .flake8
[flake8]
max-line-length = 100
extend-ignore = E203, W503  # 與 Black 相容
exclude =
    .git,
    __pycache__,
    .venv,
    build,
    dist

# 每個函式最大複雜度
max-complexity = 10

# 每個檔案最大行數
max-line-length = 100

# 特定規則
per-file-ignores =
    __init__.py:F401  # 允許未使用的 import (用於 re-export)
    tests/*:S101      # 允許測試中使用 assert
```

```bash
# 執行檢查
flake8 .
```

**4. mypy：型別檢查**

```toml
# pyproject.toml
[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true
no_implicit_optional = true
warn_redundant_casts = true
warn_unused_ignores = true
warn_no_return = true
strict_equality = true

# 第三方套件沒有型別提示時忽略
[[tool.mypy.overrides]]
module = "untyped_library.*"
ignore_missing_imports = true
```

```bash
# 型別檢查
mypy .
```

**5. Ruff：全方位超快速工具（推薦）**

Ruff 使用 Rust 撰寫，比傳統工具快 10-100 倍，可以取代 flake8、pylint、isort。

```toml
# pyproject.toml
[tool.ruff]
line-length = 100
target-version = "py311"

# 啟用的規則集
select = [
    "E",   # pycodestyle errors
    "W",   # pycodestyle warnings
    "F",   # pyflakes
    "I",   # isort
    "C90", # mccabe (複雜度)
    "N",   # pep8-naming
    "UP",  # pyupgrade
    "B",   # flake8-bugbear
    "A",   # flake8-builtins
    "C4",  # flake8-comprehensions
    "PT",  # flake8-pytest-style
]

ignore = [
    "E501",  # 行長度（交給 Black 處理）
]

# 自動修復
fix = true
fixable = ["I", "F401", "UP"]

[tool.ruff.per-file-ignores]
"__init__.py" = ["F401"]
"tests/*" = ["S101"]
```

```bash
# 檢查並自動修復
ruff check --fix .

# 只檢查
ruff check .

# 同時進行格式化（Ruff 也內建格式化功能）
ruff format .
```

**完整 Python 專案設定範例**：

```toml
# pyproject.toml
[tool.poetry]
# ... 專案資訊

[tool.poetry.group.dev.dependencies]
black = "^23.0.0"
ruff = "^0.1.0"
mypy = "^1.7.0"
pytest = "^7.4.0"

[tool.black]
line-length = 100

[tool.ruff]
line-length = 100
select = ["E", "F", "I", "UP", "B"]

[tool.mypy]
strict = true

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

```bash
# Makefile
.PHONY: format lint type-check test

format:
	black .
	ruff check --fix .

lint:
	ruff check .

type-check:
	mypy .

test:
	pytest

check: format lint type-check test
```

**Git Hooks (pre-commit)**：
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/psf/black
    rev: 23.11.0
    hooks:
      - id: black

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.6
    hooks:
      - id: ruff
        args: [--fix, --exit-non-zero-on-fix]

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.7.1
    hooks:
      - id: mypy
        additional_dependencies: [types-requests]
```

```bash
# 安裝 pre-commit
pip install pre-commit
pre-commit install

# 手動執行
pre-commit run --all-files
```

---

### 🎯 團隊實踐指南

#### **1. 選擇工具的原則**

- ✅ **選擇「固執己見」(Opinionated) 的工具**：如 Prettier、Black，減少無意義的爭論
- ✅ **自動化優先**：能自動修復的問題，不要依賴人工檢查
- ✅ **整合到 CI/CD**：在 Pull Request 階段就擋下不合規範的程式碼
- ✅ **漸進式導入**：新專案嚴格執行，舊專案逐步改善

#### **2. 導入步驟**

```
階段 1: 團隊共識
├─ 討論並選定工具與配置
├─ 記錄在 README 或 CONTRIBUTING.md
└─ 所有成員安裝必要的編輯器外掛

階段 2: 配置專案
├─ 添加配置檔（.prettierrc, .eslintrc, pyproject.toml）
├─ 添加 npm scripts 或 Makefile 指令
└─ 設定 Git hooks (husky, pre-commit)

階段 3: 整合 CI/CD
├─ 在 GitHub Actions / GitLab CI 添加格式檢查
├─ PR 必須通過檢查才能合併
└─ 每日自動化報告

階段 4: 持續改善
├─ 定期審視規則是否合理
├─ 根據團隊反饋調整配置
└─ 分享最佳實踐
```

#### **3. GitHub Actions 範例**

**TypeScript 專案**：
```yaml
# .github/workflows/lint.yml
name: Lint & Format Check

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Check formatting
        run: npm run format:check
      
      - name: Run ESLint
        run: npm run lint
      
      - name: Run type check
        run: npm run type-check
```

**Python 專案**：
```yaml
# .github/workflows/lint.yml
name: Python Lint & Format

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          pip install black ruff mypy
      
      - name: Check formatting (Black)
        run: black --check .
      
      - name: Run Ruff
        run: ruff check .
      
      - name: Type check (mypy)
        run: mypy .
```

---

### 💡 最佳實踐總結

| 原則                 | 說明                                       | 範例                                  |
| -------------------- | ------------------------------------------ | ------------------------------------- |
| **自動化一切**       | 不要依賴人工，讓工具在儲存或提交時自動執行 | VS Code 的 `formatOnSave`、Git hooks  |
| **團隊共識**         | 所有成員必須使用相同的工具和配置           | 統一的 `.prettierrc`、`.editorconfig` |
| **CI 強制執行**      | 在 PR 階段就擋下不合規範的程式碼           | GitHub Actions 自動檢查               |
| **配置檔案納入版控** | 所有配置檔都應提交到 Git                   | `.eslintrc.js`、`pyproject.toml`      |
| **提供清晰的文件**   | 在 README 說明如何設定開發環境             | 包含安裝指令和編輯器設定              |
| **定期審視規則**     | 根據團隊反饋調整不合理的規則               | 每季檢討一次                          |

---

### 🚀 快速開始範本

**TypeScript 專案初始化**：
```bash
# 安裝工具
npm install --save-dev prettier eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin eslint-config-prettier

# 建立配置檔
npx eslint --init
echo '{"semi": true, "singleQuote": true}' > .prettierrc.json

# 設定 Git hooks
npx husky init
npx husky add .husky/pre-commit "npx lint-staged"
```

**Python 專案初始化**：
```bash
# 安裝工具
pip install black ruff mypy pre-commit

# 建立配置
ruff init
pre-commit sample-config > .pre-commit-config.yaml

# 安裝 Git hooks
pre-commit install
```

**C# 專案初始化**：
```bash
# 建立 .editorconfig
dotnet new editorconfig

# 安裝 StyleCop
dotnet add package StyleCop.Analyzers

# 格式化專案
dotnet format
```

---

### 📚 工具比較表

| 語言           | 格式化        | Linter                      | 型別檢查            | Import 排序  | 推薦組合              |
| -------------- | ------------- | --------------------------- | ------------------- | ------------ | --------------------- |
| **TypeScript** | Prettier      | ESLint                      | TypeScript Compiler | ESLint       | Prettier + ESLint     |
| **C#**         | dotnet format | Roslyn Analyzers / StyleCop | 內建                | -            | EditorConfig + Roslyn |
| **Python**     | Black         | Ruff / flake8               | mypy                | isort / Ruff | Black + Ruff + mypy   |

---

### 🔑 核心要點

1. **一致性比個人偏好重要**：選定工具後，團隊必須統一遵守
2. **自動化優於人工檢查**：讓工具處理格式，人類專注於邏輯
3. **整合到開發流程**：編輯器、Git hooks、CI 三管齊下
4. **文件與範例齊全**：降低新成員的上手成本
5. **持續改善**：定期檢討規則的合理性

**記住**：程式碼風格的目標不是「誰的風格最好」，而是「團隊的風格統一」。當團隊花時間爭論要用 2 個空格還是 4 個空格時，不如讓工具決定，把精力放在解決真正的問題上。

---

[← 回目錄](./00-README.md) | [下一章：抽象層次 →](./12-Abstraction.md)
