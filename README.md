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
- **Janus Gateway 伺服器** (已配置 VideoRoom 插件)
- **現代瀏覽器** (Chrome 60+, Firefox 55+, Safari 12+)
- **HTTPS 環境** (攝影機權限要求)
- **Docker & Docker Compose** (推薦，用於快速啟動 Janus Gateway)

### 方法一：使用 Docker (推薦)

1. **克隆專案**
   ```bash
   git clone <repository-url>
   cd lab-webrtc-client-sample
   ```

2. **啟動 Janus Gateway 服務**
   ```bash
   # 使用 Docker Compose 啟動 Janus Gateway
   docker-compose up -d janus
   
   # 檢查服務狀態
   docker-compose ps
   ```

3. **啟動客戶端應用**
   ```bash
   # 使用 Docker 啟動靜態網頁服務器
   docker-compose up -d web
   
   # 或手動啟動 HTTP 伺服器
   python -m http.server 8080
   ```

4. **開啟應用程式**
   - 瀏覽器開啟 `http://localhost:8080`
   - 允許攝影機和麥克風權限
   - 輸入房間 ID 並加入房間

### 方法二：手動安裝 Janus Gateway

如果您偏好手動安裝，請參考 [CONTRIBUTING.md](CONTRIBUTING.md) 中的詳細說明。

#### 快速手動設置
1. **安裝 Janus Gateway**
   ```bash
   # Ubuntu/Debian
   sudo apt-get update
   sudo apt-get install janus
   
   # 或從源碼編譯 (詳見官方文件)
   ```

2. **配置 VideoRoom 插件**
   - 確保 Janus Gateway 運行在 `localhost:8188`
   - 或修改 `index.html` 中的伺服器地址 (第 401 行和 242 行)

3. **啟動服務**
   ```bash
   # 啟動 Janus Gateway
   janus
   
   # 啟動客戶端 HTTP 伺服器
   python -m http.server 8080
   ```

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

## Docker 配置說明

### 服務架構
本專案提供完整的 Docker 環境，包含：
- **Janus Gateway**: SFU 媒體伺服器 (端口: 8188, 8081)
- **Web Server**: 靜態網頁服務器 (端口: 8080)
- **Admin Interface**: Janus 管理界面 (端口: 7088)

### 環境變數
```yaml
# docker-compose.yml 中可配置的環境變數
JANUS_WS_PORT=8188          # WebSocket 端口
JANUS_HTTP_PORT=8081        # HTTP 端口
JANUS_ADMIN_PORT=7088       # 管理界面端口
WEB_PORT=8080               # 客戶端網頁端口
```

### 客戶端配置
```javascript
// index.html 中的 Janus Gateway 配置
server: 'ws://localhost:8188'              // WebSocket 連接地址
src: 'http://localhost:8081/janus.js'      // Janus 客戶端庫地址
```

### 自定義配置
如需修改服務器地址，請更新 `index.html` 中的以下行：
- **第 401 行**: WebSocket 連接地址
- **第 242 行**: Janus JavaScript 庫地址

```javascript
// 修改為您的 Janus Gateway 地址
server: 'ws://your-janus-server:8188'
src: 'http://your-janus-server:8081/janus.js'
```

### Janus Gateway 配置文件

Docker 環境會自動配置以下 Janus 設定：

```ini
# janus.jcfg - 主要配置
[general]
configs_folder = /opt/janus/etc/janus
plugins_folder = /opt/janus/lib/janus/plugins
transports_folder = /opt/janus/lib/janus/transports

[nat]
ice_enforce_list = "eth0"
ice_ignore_list = "vmnet"

[media]
rtp_port_range = "20000-40000"
```

```ini
# janus.transport.websockets.jcfg - WebSocket 傳輸
[general]
ws = true
ws_port = 8188
wss = false

[admin]
admin_ws = true
admin_ws_port = 7188
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

### Docker 相關問題

1. **Docker 服務啟動失敗**
   ```bash
   # 檢查 Docker 服務狀態
   docker-compose logs janus
   
   # 重新啟動服務
   docker-compose down
   docker-compose up -d
   ```

2. **端口衝突**
   ```bash
   # 檢查端口占用
   netstat -tulpn | grep :8188
   
   # 修改 docker-compose.yml 中的端口映射
   ports:
     - "8189:8188"  # 改用其他端口
   ```

3. **Janus Gateway 連接問題**
   ```bash
   # 進入 Janus 容器檢查狀態
   docker-compose exec janus bash
   
   # 查看 Janus 日誌
   docker-compose logs -f janus
   ```

### 應用程式問題

1. **無法連接到伺服器**
   - 檢查 Docker 容器是否正常運行: `docker-compose ps`
   - 確認防火牆設定允許相關端口
   - 驗證客戶端配置中的伺服器地址

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
- **單一房間建議上限**: 50 人
- **視訊延遲目標**: < 200ms
- **聊天訊息保留時間**: 5 分鐘
- **Docker 資源需求**: 
  - 最小記憶體: 512MB
  - 推薦記憶體: 2GB+
  - CPU: 2 核心以上推薦

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