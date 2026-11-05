# 路線 A：Higgsfield 雲端 API

## 已查證（來自 Higgsfield MCP 的 `models_explore` 與工具說明）

- `hf_mult_motion_control`（Genjutsu）：把參考影片的動作轉移到參考圖的主體。輸入角色：`image_references`、`video_references`。解析度 480p / 720p / 1080p。工具說明：1–49 張主體圖時，用 `generate_video` 並指定此 model。
- `hf_mult_motion_control_flash`（Genjutsu Lite）：同類，說明寫「Wan 3.0，最長 15 秒」。
- `motion_control` 工具：單一角色的 Kling 重製，參數為 `image_id` 加 `motion_video_id`。
- `kling3_0`：標籤有 motion-transfer，但 `models_explore` 顯示它的媒體角色只有 start_image / end_image，**是否能吃動作影片未確認**。
- `seedance_2_5`：`omni_reference` 模式可帶圖片、影片、音訊參考，4–30 秒，另有 `video_edit` 模式。**動作貼合程度未驗證。**
- 伺服器說明另提到 `hf_mult_replace_object`（物件替換）。**能否用來換人未查證。**

## 實測結果（2026-11-05，使用者目視回報，Claude 因網路限制未能自行抽幀檢查）

| 測試 | 模型與設定 | 花費 | 使用者回報 |
|---|---|---|---|
| 小豬（卡通）＋森林場景起始圖，動作取自 10.8 秒家庭舞蹈影片 | Genjutsu（`hf_mult_motion_control`），720p，整支約 11 秒 | 77 點 | 保留了原始起始圖（含場景） |
| 同上素材 | Seedance 2.5 `omni_reference`，720p，11 秒，無聲 | 77 點 | 使用者只說 Genjutsu 較能保留原始圖片；兩者動作貼合度的比較**未回報** |
| 真人半身照＋舞蹈教室動作影片（兩位，已確認本人同意、僅私下測試） | Seedance 2.5 `omni_reference`，720p，5 秒，無聲 | 每支 35 點 | 臉像、戴眼鏡特徵保留；動作與舞蹈影片一致；下半身（照片沒有）沒有明顯變形；提示詞要求的舞蹈教室場景有出現 |

25 秒完整版（Claude 自行抽 7 幀，與原動作影片同時間點對照）：Seedance 2.5 `omni_reference`，720p，25 秒，花費 175 點，輸出 720×1280、24 fps、約 25.04 秒。
- 7 幀中人物的眼鏡、領帶、白袍都一致；動作與原影片對得上（舉手過頭、交叉手臂、手放頭後、手叉腰、比 V 字貼眼等）。
- 背景舞蹈教室與招牌有出現，但燈光顏色與原影片同一時間點不同步。
- 第 9 秒畫面右緣出現另一位穿白袍的人；第 24 秒左右兩邊露出類似原影片背景舞者的人影。原影片背景舞者的影響會被帶進結果。
- 只抽了 7 幀，沒有逐幀檢查閃爍或臉部漂移；臉與原照片的相似度是目視印象，不是量化比較。

結論：
- 要「保留起始圖場景」：優先用 Genjutsu。
- 要「真人照片＋動作影片」：Seedance 2.5 `omni_reference` 在這次 5 秒測試表現良好；只有一次測試，不代表穩定。
- 只測了 5 秒與 11 秒；Seedance 取用動作影片的哪一段**未確認**。

## 仍未驗證

- 各模型的影片長度與格式上限。
- Genjutsu 與 Seedance 在動作貼合度上的差異。
- 長時間（25 秒以上）的穩定性與花費。

## 操作經驗

- `generate_video` 的媒體角色要寫 `image_references` / `video_references`（寫 `image` / `video` 會被自動改寫，但直接用正確名稱較清楚）。
- 報價：Genjutsu 720p 約 11 秒 = 77 點；Seedance 2.5 `omni_reference` 720p 11 秒 = 77 點、5 秒 = 35 點。先用 `get_cost: true` 預覽。
- Google Drive 匯入：`https://drive.google.com/uc?export=download&id=ID` 有一支 15.7 MB 影片連續兩次匯入失敗；改用 `https://drive.usercontent.google.com/download?id=ID&export=download` 成功。失敗時換這個格式再試。
- 生成通常要數分鐘，用 `jobs_wait`（每次最多 15 秒）輪詢。
- 沙盒網路：下載結果影片（`*.cloudfront.net`）和直接上傳到 Higgsfield（`upload.higgsfield.ai`）預設被擋。要自行檢查畫面，需在環境設定放行這些網域；放行未生效時，請使用者把影片放到 Google Drive 或自行檢視。

## 步驟

1. `balance` 查餘額。餘額不足就先告知使用者，不要硬跑。
2. 若要處理剪輯類需求，先呼叫 `get_workflow_instructions`（伺服器指示在剪輯影片時要先呼叫）。
3. 上傳素材：能在沙盒讀到的檔案用 `media_upload` → PUT → `media_confirm`；否則用 `media_upload_widget`；有公開網址用 `media_import_url`。
4. 換場景時：用 `generate_image` 做「人物＋新場景」起始圖，讓使用者確認。
5. `models_explore`（`action: get`）確認所選模型參數，向使用者報告將使用的模型、解析度、預估花費（查不到就說查不到），取得同意。
6. 用 `generate_video`（model: `hf_mult_motion_control`）送出，媒體帶起始圖（image_references）與動作影片（video_references）。先用短片段。
7. `jobs_wait`（timeout_seconds ≤ 15，需要時重複呼叫），完成後用 `show_generation_by_ids` 顯示。
8. 若場景沒保留，備案：改試 `seedance_2_5` 的 `omni_reference`，或改走路線 B。
