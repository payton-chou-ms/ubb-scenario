# GitHub Copilot Usage-Based Billing — 情境分析與管理要點

> 適用於 2026/6/1 起之 Usage-Based Billing 新制

## 前提假設

| 項目 | 值 |
|------|---|
| 方案 | Copilot Business |
| Seat 月費 | $19/user/month |
| Included AI Credits（促銷期 6/1–9/1） | **3,000**/user/month = **$30**/user |
| Included AI Credits（標準期） | 1,900/user/month = $19/user |
| 範例人數 | 10 位使用者 |
| Pool 總量（促銷期） | 10 × 3,000 = **30,000 credits（$300）** |
| 1 AI Credit | = $0.01 USD |
| Code completions / Next edit suggestions | **不計入** AI Credits，維持無限制 |

## 關鍵概念

| 概念 | 說明 |
|------|------|
| **AI Credits** | 計費單位，1 credit = $0.01 USD |
| **Pool（池）** | 所有 seat 的 included credits **合併為共享池**，非個人 bucket。新增 license 立即增加 pool；移除 license 於下個帳期才縮減 |
| **ULB（User-Level Budget）** | 追蹤每位使用者的**總消耗**（含 pool + overage），到上限即停用。設為 $0 = 完全無法使用 |
| **Org Budget** | 控制 org 層級的 **additional usage（超額）** 花費上限 |
| **Enterprise Budget** | 控制整個 enterprise 的超額花費上限，跨所有 org |
| **Additional Usage Policy** | Org 層級開關：Pool 用完後是否允許繼續使用（和 budget 是不同的控制） |

---

## 總結表

| 情境 | ULB | Pool 共享 | 超額 | 最大額外費用 | 風險 |
|------|-----|-----------|------|-------------|------|
| 1 未設 ULB | 無上限 | ✅ | ✅ | 依 budget | 一人吃光 pool |
| 2 ULB=included | $30 | ❌ | ❌ | $0 | 無法利用 pooling 優勢 |
| 3 ULB>included, no overage | $40 | ✅ | ❌ | $0 | Power user 排擠他人 |
| 4 ULB>included + overage | $40 | ✅ | ✅ (org) | $100 | 可控 |
| 5 Enterprise 多 Org | $40 | ✅ | ✅ (org+ent) | $1,000 | Enterprise 是全局剎車 |
| 6 差異化 ULB | Mixed | ✅ | Depends | Varies | 靈活但管理複雜 |

---

## 情境分析

### 情境 1：未設定 ULB（預設值）

| 設定 | 值 |
|------|---|
| ULB | 未設定（無每人上限） |
| Org Budget | $100 |

**行為：**

- 沒有 per-user 上限，任一使用者可消耗大量 pool
- 一個 power user 使用 agent mode 跑整天，可能吃掉大部分 pool
- Pool 耗盡後，依 org budget 允許最多 $100 overage
- **最大風險情境：缺乏個人消耗控制**

**適用場景：** 小團隊、高信任度環境

---

### 情境 2：ULB = $30, Org Budget = $0 — 完全不可用到別人的 Pool

| 設定 | 值 |
|------|---|
| ULB | $30 = 3,000 credits/user |
| Org Budget | $0（不允許超額） |

**行為：**

- 每位使用者從 pool 取用，上限 3,000 credits
- 10 人 × 3,000 = 30,000 = pool 總量
- 即使 User A 只用 1,000，User B **仍被 ULB 卡在 3,000**，無法取用多餘的
- **等同於每人有獨立 bucket**，pooling 效果被消除
- 額外費用：$0
- 月費：$190（seat cost only）

---

### 情境 3：ULB = $40, Org Budget = $0 — 可共享 Pool，無超額

| 設定 | 值 |
|------|---|
| ULB | $40 = 4,000 credits/user |
| Org Budget | $0（不允許超額） |

**行為：**

- 每人上限 4,000，但 pool 只有 30,000
- Power user 可從 pool 取超過 3,000（最多 4,000），前提是其他人用得少
- Pool 耗盡後 → 所有人 block（org budget = $0）
- 即使某使用者 ULB 還沒到 4,000，pool 空了就停
- 額外費用：$0

**風險：** 一個 power user 大量消耗可能導致其他人提前被 block

---

### 情境 4：ULB = $40, Org Budget = $100 — 共享 Pool + 超額預算

| 設定 | 值 |
|------|---|
| ULB | $40 = 4,000 credits/user |
| Org Budget | $100 = 10,000 credits 超額 |

**行為：**

- Pool 共享使用，每人上限 4,000
- Pool 耗盡後 → additional usage 啟動，org 最多再花 $100
- 總容量 = 30,000（pool）+ 10,000（overage）= **40,000 credits**
- 10 × 4,000 ULB = 40,000 → 數字對齊
- 最差月費：$190（seats）+ $100（overage）= **$290**

**最常見配法：** 允許彈性但設天花板

---

### 情境 5：Enterprise 多 Org + Enterprise Budget

| 設定 | 值 |
|------|---|
| ULB | $40/user |
| Org A Budget | $500 |
| Org B Budget | $500 |
| Enterprise Budget | $1,000 |

**行為：**

- 每個 Org 有自己的 pool（依各自 seat 數計算）
- Overage 消耗**同時**計入 Org budget 和 Enterprise budget
- Org A 用完 $500 → Org A 被 block，Org B 不受影響
- Enterprise budget 到 $1,000 → **所有 Org 被 block**，即使個別 Org budget 還沒滿
- **Enterprise budget 是全局剎車**

**關鍵風險：** 如果 Org A 大量消耗，Enterprise budget 提前到限，連帶影響 Org B

---

### 情境 6：不同人不同 ULB

| 使用者 | ULB |
|--------|-----|
| Senior Dev A, B | $60 |
| Junior Dev C–J | $20 |

**行為：**

- 所有人從同一 pool 取用
- Senior devs 可使用更多 credits（上限 6,000），Junior 受限較嚴（上限 2,000）
- 最靈活的配法，但管理複雜度最高

---

## 管理要點

### 要點 1：務必勾選「Stop usage when budget limit is reached」

設定 budget 時，必須勾選此選項才能 hard stop。

**若未勾選，budget 只是 alert，不會阻止 usage**，可能導致帳單超出預期。這是最常見的管理疏忽。

> 📖 參考連結：
> - [Setting up budgets — Stop usage](https://docs.github.com/en/billing/how-tos/set-up-budgets)
> - [Budgets and alerts — Stopping usage](https://docs.github.com/en/billing/concepts/budgets-and-alerts#stopping-usage)

---

### 要點 2：確認 Additional Usage Policy 設定

Org 層級有獨立的「Additional usage allowed / not allowed」開關，和 budget 是不同的控制：

- **Not allowed**：Pool 用完就全停，不管 budget 怎麼設
- **Allowed**：Pool 用完後繼續使用，費用依 published rates 計算，受 budget 限制

> 📖 參考連結：
> - [What happens if I exceed my included AI credits?](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises#what-happens-if-i-exceed-my-included-ai-credits)

---

### 要點 3：啟用 Included Usage Alerts

可在 Budgets & Alerts 頁面啟用 included usage alerts：

- Pool 到 **90%** 和 **100%** 時寄 email 通知
- 這和 budget alerts 是**不同的機制**：一個追蹤 included usage 消耗比例，一個追蹤 dollar budget 消耗
- **建議兩者都啟用**

> 📖 參考連結：
> - [Included usage alerts](https://docs.github.com/en/billing/concepts/budgets-and-alerts#included-usage-alerts)
> - [Managing included usage alerts](https://docs.github.com/en/billing/how-tos/set-up-budgets#managing-included-usage-alerts)

---

## 相關連結

| 主題 | 連結 |
|------|------|
| Usage-Based Billing 主頁 | [usage-based-billing-for-organizations-and-enterprises](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises) |
| Models and Pricing | [models-and-pricing](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing) |
| 設定 Budget | [set-up-budgets](https://docs.github.com/en/billing/how-tos/set-up-budgets) |
| Budget 概念與 Alerts | [budgets-and-alerts](https://docs.github.com/en/billing/concepts/budgets-and-alerts) |
| 準備轉換指南 | [prepare-for-usage-based-billing](https://docs.github.com/en/copilot/how-tos/manage-and-track-spending/prepare-for-usage-based-billing) |
| Pool 耗盡行為 | [What happens if I exceed my included AI credits?](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises#what-happens-if-i-exceed-my-included-ai-credits) |
| Budget 控制層級 | [How can I control costs with budgets?](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises#how-can-i-control-costs-with-budgets) |
| Included Usage Alerts | [included-usage-alerts](https://docs.github.com/en/billing/concepts/budgets-and-alerts#included-usage-alerts) |
| Stopping Usage | [stopping-usage](https://docs.github.com/en/billing/concepts/budgets-and-alerts#stopping-usage) |
