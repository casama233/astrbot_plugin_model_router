# AstrBot Plugin Model Router — 智慧多模型路由

[简体中文](README.md) · **繁體中文** · [English](README.en.md)

依訊息**意圖、分類與難度**，使用較快的 Router Model 評分，再選擇最合適的 AstrBot LLM Provider／模型。目標是一般問題使用小模型降低延遲與成本、複雜問題使用高能力模型。

## 功能與版本

v0.6.0（2025-12-15）加入多任務快照、`code/math/roleplay/chat` 等多維度評分、`context_relation`（continue/downgrade/unrelated）、`continued_task_id`、可配置的上下文長度與任務過期輪數，並強化任務延續判斷。

- **意圖分類**：聊天、程式、角色扮演、數學等自訂分類。
- **難度三級**：Low（1–3）、Mid（4–6）、High（7–9）。
- **上下文感知**：參考近期對話，避免將延續工作誤判為新工作。
- **黑／白名單**：依使用者或群組的 Session ID 決定是否路由。
- **可配置**：自訂分類規則、Provider、模型、難度閾值及 Prompt。
- **調試**：顯示意圖、評分和路由原因。

## 安裝

將 `astrbot_plugin_model_router` 放入 AstrBot 的 `data/plugins/`，重啟後於 WebUI 配置。

## 主要設定

| 參數 | 用途 | 預設 |
| --- | --- | --- |
| `plugin_enabled` | 插件總開關 | `true` |
| `router_config.router_provider` | 路由分析服務商 | 必須配置 |
| `router_config.router_model` | 指定路由模型，留空用預設 | 空 |
| `router_config.debug_mode` | 詳細偵錯 | `false` |
| `router_config.context_turns` | 參考最近幾輪及任務快照有效期 | `4` |
| `router_config.context_max_chars` | 每則訊息文字長度上限，`0` 不截斷 | `500` |

**Low Tier**：難度 ≤ 3，閒聊、簡單指令、單工具工作。**Mid Tier**：3–6 的一般推理與摘要。**High Tier**：複雜程式與深度推理。各 Tier 有自己的 Global Model，並可設 `r1`–`r6` 六組分類規則（`name`、`desc`、`provider`、`model`）。`desc` 會用於 Router Model 的意圖說明。

`filter_type=blacklist` 時，黑名單內 Session ID 略過路由，使用 AstrBot 預設模型；`filter_type=whitelist` 時，**只對名單內 Session ID** 啟用路由。

## 指令

| 指令 | 用途 |
| --- | --- |
| `/router config` | 顯示現行路由設定 |
| `/router status` | 顯示狀態及路由模型 |
| `/router debug [on/off]` | 啟停偵錯；不帶參數切換 |
| `/router list` | 顯示黑／白名單 |
| `/router add [sid]` | 將目前／指定 Session 加入名單 |
| `/router remove [sid]` | 移除目前／指定 Session |

## 技術路徑

AstrBot Pipeline 有九階段：`WakingCheckStage` → `WhitelistCheckStage` → `SessionStatusCheckStage` → `RateLimitStage` → `ContentSafetyCheckStage` → `PreProcessStage` → `ProcessStage` → `ResultDecorateStage` → `RespondStage`。

插件以 `@filter.event_message_type(ALL, priority=9999)` 在唤醒階段優先取得事件。檢查會話名單、讀取上下文、請 Router Model 評分，之後透過 `event.set_extra("selected_provider", ...)`、`event.set_extra("selected_model", ...)` 將選擇交給後續 `ProcessStage`；如未指定，就使用原預設模型。詳細 Pipeline 圖與核心碼位於[簡中原文件](README.md)。

建議 Router Model 選低延遲模型。路由不準時執行 `/router debug on`，查看 `reasoning` 後調整各分類的 `desc`。

## 授權與上游來源

**本倉庫沒有專案級 LICENSE 檔案，GitHub 可見不代表可以自由複製、修改或再分發。** 未經相關著作權人明確授權，不能將原始碼併入其他作品。本倉庫是 [yukikazechan/astrbot_plugin_model_router](https://github.com/yukikazechan/astrbot_plugin_model_router) 的 fork，而上游亦未提供 LICENSE；**不能假定為 MIT、Apache 或任何開放授權**。AstrBot 擴充介面相容性也不代表官方背書或自動套用 AstrBot 的授權。保留全部第三方權利。
