# 🧹 Clean Code 實戰指南  
> **撰寫乾淨、可維護、可測試的程式碼 — For C#, TypeScript, Python Developers**

---

## 🤖 AI Coding 時代,為何更需要 Clean Code?

在 AI 輔助編程的時代,許多人可能會問:「既然 AI 可以幫我寫程式,為什麼還需要學習 Clean Code?」事實上,**正是因為 AI Coding 的普及,Clean Code 的重要性反而更加凸顯**。

我們團隊面臨的現實挑戰:
- **新進同仁缺乏經驗**:無法判斷 AI 生成程式碼的品質是否符合標準
- **資深同仁未受過訓練**:即使經驗豐富,若未接受 Clean Code 訓練,同樣難以識別程式碼品質問題
- **Brown Field 專案的困境**:大部分同仁使用 Agent Mode 在現有專案(Brown field)進行開發,許多 Legacy Code 本身就寫得不好

這導致一個嚴重問題:**垃圾進,垃圾出(Garbage In, Garbage Out)**。當 AI 學習和模仿不良的程式碼素材時,它生成的程式碼品質也會跟著下降。AI 不會自動識別什麼是「好的程式碼」,它只會複製它所看到的模式。

### 人才是正駕駛,AI 是副駕駛

在 AI Coding 的時代,**工程師的角色從「程式碼撰寫者」轉變為「程式碼品質把關者」**。你必須:
- 理解 Clean Code 的本質與原則
- 能夠識別 AI 生成程式碼的品質問題
- 懂得如何引導 AI 產生高品質的程式碼
- 具備重構與優化程式碼的能力

提升自我基本能力,掌握 Clean Code 的核心概念,才能真正領導 AI 寫出高品質的程式碼。**別忘了:人才是正駕駛,AI 是副駕駛。** 副駕駛可以協助你開車,但方向盤永遠掌握在你手中。

---

## 🎯 Clean Code 的本質

  「任何傻瓜都能寫出電腦能理解的程式碼。優秀的程式設計師寫出人類能理解的程式碼。」(Martin Fowler)
  
  「 即便身處 AI 浪潮，高品質的程式碼依然源於人類的導引與裁量——它融入了資深工程師的直覺與美學考量，使 AI 產生的雛形蛻變為洗鍊且具備生命力的架構。這種**『看不出是 AI 寫的』**細膩感，正是經驗與匠心的最高體現。」(Bowen Hsueh)

  Clean Code 不只是「能運行的程式碼」,更是**專業開發者對品質的承諾與責任**。它代表:
  - 可讀性高:其他開發者(包括6個月後的自己)能快速理解
  - 可維護性強:修改功能時不會牽一髮動全身
  - 可測試:容易撰寫單元測試
  - 表達清晰的意圖:程式碼本身就是最好的文件
  - 專業態度:展現對團隊、對產品、對未來維護者的尊重與責任感

---

## 📖 目錄

### 基礎篇:建立良好習慣
1. [命名規則 (Naming)](./Naming.md)
2. [函式設計 (Functions)](./Functions.md)
3. [條件判斷 (Conditionals)](./Conditionals.md)
4. [註解原則 (Comments)](./Comments.md)
5. [DRY 原則:避免重複 (Don't Repeat Yourself)](./DRY.md)
6. [魔術數字/字串處理 (Magic Numbers & Strings)](./MagicNumbers.md)

### 進階篇:深化設計思維
7. [理解與管理副作用 (Side Effects)](./SideEffects.md)
8. [類別與物件 (Classes & Objects)](./Classes.md)
9. [錯誤處理 (Error Handling)](./ErrorHandling.md)

### 架構篇:系統化思考
10. [撰寫符合語言風格的程式碼 (Idiomatic Code)](./IdiomaticCode.md)
11. [一致性與風格工具 (Consistency)](./Consistency.md)
12. [抽象層次與依賴反轉 (Abstraction & Dependency)](./Abstraction.md)
13. [模組化與測試性 (Modularization & Testability)](./Modularization.md)
14. [不可變性與型別安全 (Immutability & Type Safety)](./Immutability.md)
15. [「高內聚、低耦合」的黃金法則:S.O.L.I.D. 架構思維](./SOLID.md)

### 附錄
16. [Clean Code Checklist](./Checklist.md)

---
