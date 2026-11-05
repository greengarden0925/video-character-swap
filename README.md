# video-character-swap

Claude skill：把動作影片裡的人換成你指定的人物，並可同時換場景、保留原動作（角色替換／動作遷移）。

可選兩條路線：

- **路線 A**：Higgsfield 雲端 API（Genjutsu、Seedance 2.5）。
- **路線 B**：開源 Wan2.2-Animate（HF Space、ComfyUI 或雲端 GPU）。

## 內容

| 檔案 | 說明 |
|---|---|
| `SKILL.md` | skill 主檔：流程、授權檢查、路線選擇 |
| `references/route-a-higgsfield.md` | 路線 A 的工具、步驟、報價與實測結果 |
| `references/route-b-open-source.md` | 路線 B 的模型、執行方式與限制 |
| `video-character-swap.skill` | 打包好的安裝檔 |

## 安裝

- Claude 介面：開啟 `video-character-swap.skill`，使用「Save skill」。
- Claude Code：把 `SKILL.md` 與 `references/` 放進 `~/.claude/skills/video-character-swap/`。

## 使用前提醒

- 人物若是真人，必須取得每一位本人的同意；動作影片也要確認來源與授權。
- 說明檔中標為「未驗證」的項目都尚未實測，使用前請先用短片段測試。
- 路線 A 需要可用的 Higgsfield 點數；路線 B 的 HF Space 需要 `HF_TOKEN` 環境變數（不要寫進任何檔案或貼在聊天裡）。
- 實測結果是 2026-11-05 的單次測試，不代表穩定表現。
