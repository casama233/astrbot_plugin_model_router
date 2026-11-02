# AstrBot Plugin Model Router — Intent-Aware Multi-Model Routing

[简体中文](README.md) · [繁體中文](README.zh-TW.md) · **English**

Use a lightweight **Router Model** to classify messages by intent and difficulty, then select the best configured AstrBot LLM provider/model. Simple questions can use fast/cheap models and difficult tasks can use stronger ones.

## Features and version

v0.6.0 (2025-12-15) added multi-task snapshots, dimension-specific scoring for `code/math/roleplay/chat`, `context_relation` (continue/downgrade/unrelated), `continued_task_id`, configurable context length and snapshot expiry, plus stricter task-continuation decisions.

- **Intent classification:** chat, programming, roleplay, math and custom categories.
- **Three difficulty tiers:** Low (1–3), Mid (4–6), High (7–9).
- **Context awareness:** retain recent conversation to avoid misclassifying continued tasks.
- **Session controls:** black/whitelists of user and group Session IDs.
- **Custom routing:** categories, prompts, score thresholds, providers and models.
- **Debugging:** print the router's reasoning, scores and selected models.

## Installation

Copy `astrbot_plugin_model_router` into AstrBot's `data/plugins/` directory, restart AstrBot and configure the plugin in WebUI.

## Key configuration

| Setting | Meaning | Default |
| --- | --- | --- |
| `plugin_enabled` | Master switch | `true` |
| `router_config.router_provider` | Provider used to analyze intent | Required |
| `router_config.router_model` | Optional specific model | Empty |
| `router_config.debug_mode` | Debug mode | `false` |
| `router_config.context_turns` | Conversation turns / task snapshot validity | `4` |
| `router_config.context_max_chars` | Per-message context limit; `0` disables truncation | `500` |

**Low**: score ≤3, basic chat, single-tool requests. **Mid**: score 4–6, ordinary logic and summaries. **High**: 7–9, advanced code and reasoning. Each tier has a Global Model and up to six category rules `r1`–`r6`. Each rule defines `name`, `desc` (prompted intent description), `provider` and `model`.

With `filter_type=blacklist`, listed Session IDs bypass routing and use the AstrBot default model. With `filter_type=whitelist`, **only listed IDs** are routed.

## Commands

| Command | Meaning |
| --- | --- |
| `/router config` | Show current route configuration |
| `/router status` | Show enabled state and router model |
| `/router debug [on/off]` | Toggle or set debug mode |
| `/router list` | Show session blacklist/whitelist |
| `/router add [sid]` | Add current/specified session |
| `/router remove [sid]` | Remove current/specified session |

## Technical mechanism

AstrBot has nine pipeline stages: `WakingCheckStage`, `WhitelistCheckStage`, `SessionStatusCheckStage`, `RateLimitStage`, `ContentSafetyCheckStage`, `PreProcessStage`, `ProcessStage`, `ResultDecorateStage`, and `RespondStage`.

This plugin registers `@filter.event_message_type(ALL, priority=9999)` to act early during wake checks. It checks session permissions, reads context, asks the router model for `category` and `difficulty`, then sets `event.set_extra("selected_provider", ...)` and `event.set_extra("selected_model", ...)` for the later `ProcessStage`. If no override is set, AstrBot uses the original default. See the [Simplified Chinese README](README.md) for the full sequence diagram and source-level pipeline table.

Use a fast Router Model to reduce overhead. If classification is poor, run `/router debug on`, examine `reasoning`, and refine the category `desc` text.

## License and upstream provenance

**There is no project-level LICENSE file in this repository. Public GitHub visibility does not grant general permission to copy, modify, redistribute or reuse the code.** The upstream project [yukikazechan/astrbot_plugin_model_router](https://github.com/yukikazechan/astrbot_plugin_model_router) also has no LICENSE. **Do not assume MIT, Apache or any other open-source permission.** AstrBot plugin API compatibility does not imply official endorsement or automatic use of AstrBot's license. All authors' and third parties' rights remain reserved unless permission is expressly established.
