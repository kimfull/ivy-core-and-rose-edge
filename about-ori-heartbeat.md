# about-ori-heartbeat.md — OpenClaw HEARTBEAT.md 功能介紹

## 什麼是 Heartbeat？

Heartbeat 是 OpenClaw 的**定期輪詢機制**。Gateway 會按照設定的間隔（通常約每 30 分鐘）自動觸發一次 heartbeat，讓 Agent 有機會主動檢查環境、處理待辦、或向使用者回報狀態。

> 如果 BOOT.md 是「開機時做一次的事」，那 Heartbeat 就是「持續運轉時定期做的事」。

---

## HEARTBEAT.md 的角色

`HEARTBEAT.md` 是一份**任務清單檔案**，放在 Agent 的 workspace 裡。

- Gateway 觸發 heartbeat 時，會指示 Agent 讀取 `HEARTBEAT.md` 並執行裡面的任務
- 如果檔案是空的（或只有註解），Agent 不做任何事，回覆 `HEARTBEAT_OK`
- 使用者或 Agent 自己可以編輯這份檔案，加入需要定期檢查的事項

**出廠預設內容：**

```markdown
# HEARTBEAT.md

# Keep this file empty (or with only comments) to skip heartbeat API calls.

# Add tasks below when you want the agent to check something periodically.
```

意即：出廠時不啟用任何定期任務，等使用者自行填入。

---

## 觸發流程

```
Gateway 每 ~30 分鐘觸發
       ↓
Agent 收到 heartbeat prompt：
「Read HEARTBEAT.md if it exists. Follow it strictly.
 Do not infer or repeat old tasks from prior chats.
 If nothing needs attention, reply HEARTBEAT_OK.」
       ↓
Agent 讀取 HEARTBEAT.md
       ↓
有任務 → 執行並回報
沒任務 → 回覆 HEARTBEAT_OK（不消耗額外 token）
```

---

## Heartbeat vs Cron

OpenClaw 同時支援 Heartbeat 和 Cron 兩種定期機制：

| | Heartbeat | Cron |
|---|---|---|
| **觸發方式** | Gateway 輪詢，時間有彈性 | 精確排程（如「每週一 09:00」） |
| **上下文** | 在主 Session 中執行，有對話上下文 | 獨立 Session，與主對話隔離 |
| **適合** | 批次檢查多個項目、需要對話記憶 | 精確定時任務、一次性提醒 |
| **模型** | 用主 Session 的模型 | 可指定不同模型或思考級別 |
| **輸出** | 在主 Session 中回覆 | 可直接送到特定頻道 |

**建議：** 把多個類似的定期檢查合併寫進 `HEARTBEAT.md`，而不是建多個 cron job。用 cron 處理需要精確時間或獨立執行的任務。

---

## 使用範例

### 範例一：基本定期檢查

```markdown
# HEARTBEAT.md

## 定期檢查（每天輪替 2-4 次）

- 檢查 Email 是否有緊急信件
- 檢查行事曆未來 24 小時有無事件
- 檢查社群媒體是否有 @mention
```

### 範例二：搭配狀態追蹤

Agent 可以用 `memory/heartbeat-state.json` 追蹤上次檢查的時間，避免短時間內重複檢查同一項目：

```json
{
  "lastChecks": {
    "email": 1703275200,
    "calendar": 1703260800,
    "weather": null
  }
}
```

### 範例三：Ivy 的系統監控

```markdown
# HEARTBEAT.md

## 定期檢查

- 檢查 token 使用紀錄，是否接近月度上限
- 檢查容器儲存空間使用率
- 檢查已上線服務是否正常運作
- 是否有 Rose 提交的待整合產出
```

### 範例四：Rose 的專案追蹤

```markdown
# HEARTBEAT.md

## 定期檢查

- 檢查手上專案進度，是否有卡住的
- 確認開發工具是否正常
- 是否有草稿該提交給 Ivy 了
- 查看競品或產業有無新動態
```

---

## 行為準則（何時主動、何時安靜）

### 主動聯繫使用者的時機

- 發現重要或緊急的事項
- 行事曆事件即將到來（< 2 小時）
- 發現使用者可能感興趣的資訊
- 超過 8 小時沒有任何互動

### 保持安靜（回覆 HEARTBEAT_OK）的時機

- 深夜 23:00-08:00（除非緊急）
- 使用者明顯在忙
- 沒有任何新狀況
- 距離上次檢查不到 30 分鐘

### 可以不問就自己做的事

- 整理記憶檔案
- 檢查專案狀態（git status 等）
- 更新文件
- commit 和 push 自己的變更
- 整理、更新 MEMORY.md

---

## 記憶維護（透過 Heartbeat）

每隔幾天，利用一次 heartbeat 進行記憶維護：

1. 讀取近期的 `memory/YYYY-MM-DD.md` 日記
2. 找出值得長期保留的事件、教訓、洞察
3. 更新 `MEMORY.md`（策展記憶）
4. 移除 `MEMORY.md` 中已過時的內容

> 像人類翻閱日記、更新心智模型一樣。每日筆記是原始紀錄；MEMORY.md 是策展過的智慧。

---

## 注意事項

- `HEARTBEAT.md` 保持精簡，避免每次 heartbeat 消耗過多 token
- 太多任務會拖慢 heartbeat 回應速度，建議控制在 5-8 個檢查項目以內
- Agent 可以自行編輯 `HEARTBEAT.md`，根據需求動態調整檢查項目
