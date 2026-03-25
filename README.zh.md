[English](README.md) | [繁體中文](README.zh.md)

# _ref-review-criteria

Workshop 程式碼庫的 Code Review 標準與品質檢查清單參考技能。

## 說明

此參考技能提供 Workshop 專案 code review 過程中使用的結構化品質檢查清單，涵蓋安全性、架構、程式碼品質、效能與測試標準。它作為 context 被其他技能和 agent 注入使用，確保 review 結果一致，避免在多個技能檔案中重複定義標準。

## 功能

- 安全性檢查清單：機密資料、輸入驗證、SQL 注入防護、auth 強制執行
- 架構規則：模組邊界、僅透過 EventBus 進行跨模組寫入、路由層保持精簡
- 程式碼品質標準：函式長度限制、命名慣例、型別提示
- 效能指南：N+1 防護、分頁、Redis 快取、embedding 併發控制
- 測試需求：測試資料硬刪除、事件處理器冪等性

## 使用方式

此技能為參考用途（`user-invocable: false`），不可直接呼叫。由 reviewer agent 和 code review 技能作為 context 注入使用。

## 運作原理

檢查清單依 Workshop 架構分為五個類別。當 reviewer agent 或技能需要評估標準時，會讀取此參考以套用一致的規範。每個類別直接對應 Workshop 技術棧的一個層次（安全層、模組邊界規則、服務層模式、資料層、測試層）。

## 需求

- Claude Code CLI
- 作為參考使用的技能：`reviewer` agent、code review 相關技能

## 授權

MIT
