# Ollama-Groq 視覺分析助手

使用 Streamlit 建立的多模態影像分析 APP，整合 Ollama 本機模型、Ollama Cloud 與 Groq API，讓使用者上傳圖片並透過自訂提示詞進行影像描述與問答。

## 系統功能

- 支援 JPG、JPEG、PNG 圖片上傳。
- 支援自訂 Prompt，預設以繁體中文描述圖片及觀光特色。
- 自動修正 EXIF 方向、轉換 RGB 並進行 JPEG Base64 編碼。
- 並排顯示原始圖片與分析結果。

## 支援模型

| 服務 | 程式指定模型 | 執行位置 |
|---|---|---|
| Ollama | `gemma4:12b` | 本機 |
| Ollama Cloud | `gemma4:31b-cloud` | 雲端 |
| Groq | `qwen/qwen3.8-27b` | 雲端 |

以上為原始程式設定，模型是否仍可使用、是否收費及是否支援圖片，請以各服務官方資訊為準。

## 專案結構

```text
img_recog.py
README.md
requirements.txt
```

## 安裝方式

建議使用 Python 3.11 或 3.12，並建立獨立環境：

```bash
conda create -n img_recog python=3.11 -y
conda activate img_recog
pip install -r requirements.txt
```

## API Key 設定

- Ollama Cloud：https://ollama.com/settings/keys
- Groq：https://console.groq.com/keys

Windows Anaconda Prompt（CMD）：

```bat
set OLLAMA_API_KEY=your_ollama_api_key
set GROQ_API_KEY=your_groq_api_key
```

Windows PowerShell：

```powershell
$env:OLLAMA_API_KEY="your_ollama_api_key"
$env:GROQ_API_KEY="your_groq_api_key"
```

macOS / Linux：

```bash
export OLLAMA_API_KEY="your_ollama_api_key"
export GROQ_API_KEY="your_groq_api_key"
```

請勿將真實 API Key 上傳至 GitHub。

### 本機 Ollama 設定

從 https://ollama.com/download 安裝 Ollama，並下載程式指定模型（若該模型可用）：

```bash
ollama pull gemma4:12b
```

確認 Ollama 服務已啟動；若尚未啟動，可執行 `ollama serve`。

### 原始程式注意事項

原始程式在啟動時即建立 Ollama Cloud Client，若未設定 `OLLAMA_API_KEY`，可能因字串與 None 相加而發生 TypeError，即使選擇其他 API 亦然。建議移除全域 `client = Client(...)`，改在 `ollama_cloud_analyze_image()` 的 `try` 區塊內建立 Client：

```python
api_key = os.environ.get("OLLAMA_API_KEY")
if not api_key:
    return "請先設定 OLLAMA_API_KEY。"
client = Client(
    host="https://ollama.com",
    headers={"Authorization": "Bearer " + api_key}
)
```

如此本機 Ollama 或 Groq 模式便不必依賴 Ollama Cloud Key。此建議尚未修改原始 Python 檔案。

## 啟動 APP

```bash
streamlit run img_recog.py
```

瀏覽器開啟 http://localhost:8501 。選擇 API、上傳圖片、輸入提示詞，再按下「分析影像」。

## 部署至 Streamlit Community Cloud

1. 將三個專案檔案上傳至 GitHub repository。
2. 登入 https://share.streamlit.io/ 並建立 APP。
3. 選擇 repository、branch，將 Main file path 設為 `img_recog.py`。
4. 在 Advanced settings 的 Secrets 中設定：

```toml
OLLAMA_API_KEY = "your_ollama_api_key"
GROQ_API_KEY = "your_groq_api_key"
```

5. 按下 Deploy。根層 Secrets 可作為環境變數供程式讀取。

雲端部署建議使用 Ollama-Cloud 或 Groq。本機 Ollama 模式預設連線至 APP 執行主機上的 Ollama 服務，無法直接連線至使用者電腦的 localhost。

## 常見問題

- API Key 錯誤：確認環境變數或 Secrets 是否正確設定。
- Model not found：確認模型名稱及服務是否仍提供該模型。
- Connection refused：確認本機 Ollama 服務是否啟動。
- API 額度不足：確認帳戶額度、速率限制及付費設定。
- 隱私：使用雲端 API 時，圖片及提示詞會傳送至外部服務，請避免上傳敏感資料。

## 使用套件

Streamlit、Ollama Python SDK、Groq Python SDK、Pillow。`base64`、`io`、`os` 為 Python 標準函式庫，不需另外安裝。
