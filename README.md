# WebRTC 視訊聊天室

一個基於 Janus Gateway 的多人視訊聊天室應用程式，支援高品質音視訊通話和即時文字聊天。

## 功能特色

### 核心功能
- 🎥 **多人視訊通話** - 支援 20-50 人同時在線視訊通話
- 🎵 **高品質音訊** - 低延遲音訊傳輸 (< 200ms)
- 💬 **即時文字聊天** - 房間內即時文字訊息交流
- 🔧 **媒體控制** - 靜音/取消靜音、開/關攝影機功能
- 👥 **用戶管理** - 動態用戶加入/離開，實時用戶計數
- 📱 **響應式設計** - 適配桌面和行動裝置

### 技術特色
- ⚡ **SFU 架構** - 使用 Selective Forwarding Unit 優化性能
- 🔄 **動態品質調整** - 根據網路狀況自動調整視訊品質
- 🛡️ **訊息去重機制** - 使用 UUID v7 確保聊天訊息可靠傳遞
- 🔍 **分級日誌系統** - INFO/DEBUG/ERROR 多層級日誌記錄

## 系統架構

```
[用戶A] ←→ [Janus Gateway SFU] ←→ [用戶B]
[用戶C] ←→      (媒體轉發)      ←→ [用戶D]
                     |
                [WebSocket 信令]
```

### 架構優勢
- **連接簡化**: 每用戶僅需一個上行和下行連接到 SFU
- **資源優化**: 降低客戶端 CPU 和記憶體消耗
- **網路效率**: 減少整體網路流量和延遲
- **可擴展性**: 支援大規模併發用戶

## 快速開始

### 前置要求
- Janus Gateway 伺服器 (已配置 VideoRoom 插件)
- 現代瀏覽器 (Chrome 60+, Firefox 55+, Safari 12+)
- HTTPS 環境 (攝影機權限要求)

### 安裝與運行

1. **克隆專案**
   ```bash
   git clone <repository-url>
   cd lab-webrtc-client-sample
   ```

2. **配置 Janus Gateway**
   - 確保 Janus Gateway 運行在 `192.168.6.70:8188`
   - 或修改 `index.html` 中的伺服器地址 (第 401 行和 242 行)

3. **啟動 HTTP 伺服器**
   ```bash
   # 使用 Python
   python -m http.server 8080
   
   # 或使用 Node.js
   npx serve .
   ```

4. **開啟應用程式**
   - 瀏覽器開啟 `http://localhost:8080`
   - 允許攝影機和麥克風權限
   - 輸入房間 ID 並加入房間

## 使用方法

### 基本操作
1. **加入房間**: 輸入房間 ID (預設: 1234)，點擊「加入房間」
2. **媒體控制**: 使用「靜音」、「關閉視訊」按鈕控制本地媒體
3. **文字聊天**: 在右側聊天區域輸入訊息，按 Enter 發送
4. **離開房間**: 點擊「離開房間」結束通話

### 房間管理
- **房間 1234**: 預設房間，設有管理員密碼 ("adminpwd")
- **其他房間**: 可使用任意數字 ID 創建新房間
- **用戶顯示**: 系統自動生成用戶名稱 (格式: 用戶_XXX)

## 技術規格

### 前端技術
- **HTML5** - 結構和樣式
- **Vanilla JavaScript** - 核心邏輯
- **WebRTC API** - 媒體處理
- **WebSocket** - 信令通訊

### 後端服務
- **Janus Gateway** - SFU 媒體伺服器
- **VideoRoom Plugin** - 視訊聊天室功能
- **WebRTC Adapter** - 瀏覽器相容性

### 聊天系統
- **主要方案**: VideoRoom display 屬性更新機制
- **備用方案**: BroadcastChannel + localStorage
- **訊息格式**: JSON 結構化數據
- **去重機制**: UUID v7 + Map 記錄

## 專案結構

```
lab-webrtc-client-sample/
├── index.html          # 主應用程序
├── janus.js           # Janus Gateway 客戶端庫
├── requirement.md     # 需求分析與實現狀態
├── README.md          # 專案說明文件
└── CONTRIBUTING.md    # 貢獻指南
```

## 配置說明

### 伺服器配置
```javascript
// Janus Gateway WebSocket 地址
server: 'ws://192.168.6.70:8188'

// Janus JavaScript 庫地址
src: 'http://192.168.6.70:8081/janus.js'
```

### 媒體設定
```javascript
// 用戶媒體請求
navigator.mediaDevices.getUserMedia({
    video: true,
    audio: true
})

// 房間配置
{
    publishers: 50,         // 最大發布者數量
    is_private: false,      // 公開房間
    description: `房間 ${roomId}`
}
```


## 故障排除

### 常見問題

1. **無法連接到伺服器**
   - 檢查 Janus Gateway 是否正常運行
   - 確認網路防火牆設定
   - 驗證 WebSocket 連接地址

2. **攝影機權限被拒絕**
   - 使用 HTTPS 協定提供服務
   - 在瀏覽器設定中允許攝影機權限
   - 檢查硬體攝影機是否可用

3. **聊天訊息不顯示**
   - 檢查瀏覽器控制台錯誤訊息
   - 確認房間 ID 一致
   - 驗證 VideoRoom display 更新機制

### 日誌分析
系統提供分級日誌輸出：
- `ERROR` - 錯誤訊息
- `WARN` - 警告訊息  
- `INFO` - 一般資訊
- `DEBUG` - 調試訊息

## 效能優化

### 最佳實踐
- 使用適當的視訊解析度 (720p 推薦)
- 監控網路頻寬使用情況
- 定期清理過期的聊天訊息記錄
- 適時調整日誌等級減少輸出

### 系統限制
- 單一房間建議上限: 50 人
- 視訊延遲目標: < 200ms
- 聊天訊息保留時間: 5 分鐘

## 貢獻

歡迎貢獻代碼、報告問題或提出建議！請查看 [CONTRIBUTING.md](CONTRIBUTING.md) 了解詳細的貢獻指南。

## 授權聲明

本專案為學習和示範用途，請遵守相關開源軟體授權條款。

## 支援與反饋

如有問題或建議，請通過以下方式聯繫：
- 提交 [GitHub Issue](https://github.com/your-repo/issues)
- 查看 [CONTRIBUTING.md](CONTRIBUTING.md) 開發指南
- 參考 [Janus Gateway 官方文件](https://janus.conf.meetecho.com/)

---

**注意**: 此專案需要配置 Janus Gateway 媒體伺服器才能正常運行。請確保您的網路環境支援 WebRTC 連接。