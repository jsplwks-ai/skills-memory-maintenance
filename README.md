# memory-maintenance

一個 Agent Skill，用來稽核、整理、驗證和清理 AI coding agent 的持久記憶（memory）。支援 Claude Code 與 Codex，其他遵循 [Agent Skills 規格](https://agentskills.io/specification) 的工具也能使用。

## 專案目的

AI agent 的 memory 用久了會累積問題：

- 重複或互相矛盾的記錄
- 已被新實作取代的過時資訊
- 除錯紀錄、對話碎片這類沒有長期價值的雜訊
- 只在某個 Git 分支成立的內容，切換分支後被當成既定事實

這個 Skill 讓 agent 依照固定流程檢查 memory、比對程式碼與 Git 歷史確認有效性，再提出整理計畫。設計重點：

- **預設唯讀：** 只掃描和提出建議，任何修改都要使用者核准。
- **不確定就保留：** 無法確認是否過時的記憶標記 `needs-review`，不刪除。舊不等於過時，找不到引用的類別也不等於可以刪。
- **遵守環境閘門：** 環境已有 memory 寫入規則或 hook 時以它為準；工具被擋就停下，不繞道。
- **範圍明確：** `CLAUDE.md`、`AGENTS.md`、rules、Skills、hooks、設定檔都不算 memory，不會被當成清理對象。

## 使用方式

安裝後，用自然語言要求即可觸發，例如：

- 「幫我整理 memory」
- 「檢查有沒有過時或重複的記憶」
- 「切到 develop 了，看看 memory 還適不適用」
- 「這個 feature 已經 merge，哪些分支記憶可以提升成專案記憶？」

Claude Code 也可以直接輸入 `/memory-maintenance`；Codex 可用 `$memory-maintenance` 或 `/skills` 指定。

### 模式

沒有指定時，從 `scan` 和 `review` 開始。

| 模式 | 用途 | 會修改檔案 |
|---|---|---|
| `scan` | 盤點 memory 檔案並分類 | 否 |
| `review` | 找出重複、衝突、過時、雜訊，列出可提升的記憶與建議去處，產出附證據的整理計畫 | 否 |
| `cleanup` | 執行已核准的計畫 | 是，需核准 |
| `branch-sync` | 切換分支或 worktree 後，回報哪些記憶不適用目前分支 | 否，只提出建議 |
| `promote` | 驗證分支記憶已合併且仍成立後，提升為專案記憶 | 是，需核准 |
| `archive` | 封存已確認過時的記憶，保留來源 | 是，需核准 |

每次完成後會回報掃描了哪些檔案、保留、合併、更新、封存、刪除了什麼、哪些待人工確認、哪些是提升候選，以及修改後的驗證結果。

### 各平台的差異

| 平台 | memory 位置 | 行為 |
|---|---|---|
| Claude Code | `~/.claude/projects/<project-slug>/memory/` | 完整支援；增刪檔案時一併維護 `MEMORY.md` 索引與 `[[name]]` 連結 |
| Codex | `$CODEX_HOME/memories_v2/` 或 `memories/`（預設 `~/.codex`） | memory 由 Codex 自動產生，官方不建議手動編輯；`scan`、`review` 照常，修改類模式預設改為提議使用官方設定（如 `memories.use_memories`） |
| 其他 | 不固定 | 先詢問 memory 位置與格式，確認前維持唯讀 |

## 安裝

依 Agent Skills 規格，**安裝後的資料夾名稱必須是 `memory-maintenance`**，與 `SKILL.md` 的 `name` 一致。本 repo 名稱不同，請不要直接把 clone 下來的資料夾放進 skills 目錄。

先取得原始碼：

```bash
git clone https://github.com/jsplwks-ai/skills-memory-maintenance.git
```

### Claude Code

使用者層級（所有專案可用）：

```bash
mkdir -p ~/.claude/skills/memory-maintenance
cp -r skills-memory-maintenance/SKILL.md skills-memory-maintenance/references ~/.claude/skills/memory-maintenance/
```

只給單一專案使用時，改放到專案內的 `.claude/skills/memory-maintenance/`。

### Codex

使用者層級：

```bash
mkdir -p ~/.agents/skills/memory-maintenance
cp -r skills-memory-maintenance/SKILL.md skills-memory-maintenance/references ~/.agents/skills/memory-maintenance/
```

只給單一 repo 使用時，改放到 repo 內的 `.agents/skills/memory-maintenance/`。Codex 會自動偵測新的 Skill；沒出現的話重新啟動 Codex。

Codex 的 memory 功能預設關閉，要在 `config.toml` 的 `[features]` 設定 `memories = true` 才會有 memory 可以整理。

### 更新

重新執行上面的 `cp` 指令覆寫即可。新版本通常要開新的 session 才會生效。

## 檔案結構

```text
SKILL.md                            Skill 主體：模式、規則、安全要求、報告格式
references/
  execution-principles.md           review / cleanup 時讀取的判斷清單
  platform-claude-code.md           Claude Code 的 memory 位置與格式
  platform-codex.md                 Codex 的 memory 位置與限制
```

Agent 只在需要時讀取 `references/` 裡的檔案，平台檔只讀符合目前平台的那一份。

## 授權

Apache License 2.0，詳見 [LICENSE.txt](LICENSE.txt) 與 [NOTICE.txt](NOTICE.txt)。
