# 路線 B：開源 Wan2.2-Animate

## 已查證

- 模型：[Wan-AI/Wan2.2-Animate-14B](https://hf.co/Wan-AI/Wan2.2-Animate-14B)，Hugging Face 標籤授權 apache-2.0。有 [GGUF 量化版](https://hf.co/QuantStack/Wan2.2-Animate-14B-GGUF)。
- 較新版本：[Wan-AI/Wan2.2-Animate-2-14B](https://hf.co/Wan-AI/Wan2.2-Animate-2-14B)（及 Diffusers 版）。**與第一代的差異未查證**，使用前先讀模型卡。
- 兩種模式（ComfyUI-wiki 教學）：
  - **Mix / Replacement**：替換既有影片中的人物，保留原場景。
  - **Move**：把動作遷移到靜態人物圖。
- ComfyUI 官方範本使用 DWPose（姿態）與 SAM2（遮罩），模型檔名 `Wan2_2-Animate-14B_fp8_e4m3fn_scaled_KJ.safetensors`。範本：https://comfy.org/workflows/video_wan2_2_14B_animate-c2c4b45b7df5/
- HF Space：`alexnasa/Wan2.2-Animate-ZEROGPU`（標籤含 gradio、mcp-server），可透過 `dynamic_space` 呼叫。

## 實測狀況（2026-11-05）

- 用 `gradio_client` 從沙盒呼叫 Space `alexnasa/Wan2.2-Animate-ZEROGPU` 的 `/animate_scene`，需要先放行 `*.hf.space` 與 `huggingface.co`。
- 匿名呼叫回應「You have exceeded your ZeroGPU runs limit」，需要 `HF_TOKEN`（放在環境變數，**不要貼在聊天**，且新設定要開新工作階段才讀得到）。
- Hugging Face MCP 的 `dynamic_space` 在這個環境 `invoke` 被停用，只能 `view_parameters`。
- 因為上述阻礙，路線 B **尚未實際產出任何影片**，下方所有品質問題仍未驗證。

## 未驗證（必須實測或查證）

- 各量化版所需顯存。
- Move 模式是否保留起始圖裡的場景。
- HF Space 的時間、解析度、排隊限制。
- 「Replacement 後再換背景」的效果。

## 要「換人＋換場景」怎麼做

首選：**Move 模式**。
1. 做「人物＋新場景」起始圖（見 SKILL.md 步驟 2）。
2. 以起始圖 + 動作影片跑 Move 模式。
3. 檢查場景是否保留；若背景被動作影片影響或漂移，改備案。

備案：**Replacement 後換背景**。
1. 先用 Replacement 模式得到「新人物、原場景」的影片。
2. 用 SAM2 或其他去背工具取得人物遮罩，再合成到新場景（ffmpeg 或剪輯軟體）。需要自行處理光線與邊緣，品質要實測。

## 執行方式（三選一）

1. **HF Space（最快試）**：`dynamic_space` 先 `view_parameters` 看欄位，再 `invoke`。Space 可能排隊或有時間上限，用短片段。
2. **Comfy Cloud**：有官方範本，但需付費方案與點數（來自搜尋結果，費用未查證）。
3. **租雲端 GPU 跑 ComfyUI**：使用者自行選供應商。幫使用者列出需下載的模型與節點（見上方範本），並先請使用者確認 GPU 顯存足夠；不要自己猜數字。

## 步驟

1. 先讀 Space／模型卡，確認輸入格式與限制。
2. 前處理（SKILL.md 步驟 2）。
3. 先跑 3–5 秒測試，檢查臉、動作、場景。
4. 通過後再跑完整長度，必要時分段生成再用 ffmpeg 合併。
5. 後製與回報（SKILL.md 步驟 4）。
