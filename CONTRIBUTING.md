# 貢獻指南 (Contributing Guide)

感謝您對 WebRTC 視訊聊天室專案的貢獻興趣！本文件將協助您了解如何為專案做出貢獻。

## 開發環境設置

### 前置要求
- Node.js 16+ 或 Python 3.7+ (用於本地伺服器)
- Git
- 現代瀏覽器 (Chrome 60+, Firefox 55+, Safari 12+)
- Janus Gateway 伺服器 (用於完整測試)

### 本地開發設置

1. **Fork 並克隆專案**
   ```bash
   git clone https://github.com/your-username/lab-webrtc-client-sample.git
   cd lab-webrtc-client-sample
   ```

2. **設置開發伺服器**
   ```bash
   # 使用 Python
   python -m http.server 8080
   
   # 或使用 Node.js
   npx serve .
   ```

3. **配置測試環境**
   - 修改 `index.html` 中的 Janus Gateway 地址
   - 確保瀏覽器允許攝影機和麥克風權限

## 專案架構

### 文件結構
```
lab-webrtc-client-sample/
├── index.html          # 主應用程序 (HTML + CSS + JavaScript)
├── janus.js           # Janus Gateway 客戶端庫
├── requirement.md     # 需求分析與實現狀態
├── README.md          # 專案說明文件
└── CONTRIBUTING.md    # 貢獻指南 (本文件)
```

### 核心組件

#### 1. 連接管理
- `initJanus()` - 初始化 Janus Gateway 連接
- `startJanusConnection()` - 建立 WebSocket 連接
- `joinRoom()` - 加入視訊聊天室
- `leaveRoom()` - 離開房間並清理資源

#### 2. 媒體處理
- `navigator.mediaDevices.getUserMedia()` - 本地媒體擷取
- `onlocalstream` - 本地媒體流處理
- `onremotestream` - 遠程媒體流處理
- `toggleMute()` - 音訊控制
- `toggleVideo()` - 視訊控制

#### 3. 聊天系統
- `sendChatMessage()` - 發送文字訊息
- `sendChatViaVideoRoom()` - 透過 VideoRoom display 傳送
- `processChatMessage()` - 解析並顯示聊天訊息
- `addChatMessage()` - 添加訊息到聊天界面

## 代碼規範

### JavaScript 編碼標準

1. **命名慣例**
   ```javascript
   // 變數和函數使用 camelCase
   let currentUserId = null;
   function sendChatMessage() { }
   
   // 常數使用 UPPER_SNAKE_CASE
   const LOG_LEVELS = { ERROR: 0, WARN: 1, INFO: 2, DEBUG: 3 };
   
   // DOM 元素變數使用描述性名稱
   const joinBtn = document.getElementById('joinBtn');
   ```

2. **函數結構**
   ```javascript
   // 使用 JSDoc 註解說明複雜函數
   /**
    * 處理 VideoRoom 訊息
    * @param {Object} msg - Janus VideoRoom 訊息
    * @param {Object} jsep - WebRTC 會話描述
    */
   function handleVideoRoomMessage(msg, jsep) {
       // 實作內容
   }
   ```

3. **錯誤處理**
   ```javascript
   // 統一使用 log() 函數記錄錯誤
   try {
       // 危險操作
   } catch (error) {
       log('操作失敗: ' + error.message, LOG_LEVELS.ERROR);
   }
   ```

### HTML/CSS 規範

1. **HTML 結構**
   - 使用語意化標籤
   - 保持適當的縮排 (4 空格)
   - 添加必要的 `id` 和 `class` 屬性

2. **CSS 樣式**
   - 使用 BEM 命名慣例 (如適用)
   - 響應式設計優先
   - 避免內聯樣式

## 功能開發流程

### 1. 新增功能

1. **創建功能分支**
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **實作功能**
   - 遵循現有代碼結構
   - 添加適當的日誌輸出
   - 更新相關 UI 元素

3. **測試功能**
   - 本地測試基本功能
   - 多瀏覽器相容性測試
   - 多用戶場景測試

### 2. 修復 Bug

1. **重現問題**
   - 在本地環境重現 bug
   - 檢查瀏覽器控制台錯誤
   - 查看系統日誌輸出

2. **修復實作**
   - 修正根本原因
   - 添加防護性程式碼
   - 更新錯誤處理

3. **驗證修復**
   - 確認 bug 已解決
   - 檢查是否引入新問題

## 技術細節

### WebRTC 相關

#### 信令流程
```javascript
// 1. 加入房間
videoRoomHandle.send({ message: joinMsg });

// 2. 創建 Offer (Publisher)
videoRoomHandle.createOffer({
    media: { videoRecv: false, audioRecv: false },
    stream: stream,
    success: function(jsep) {
        const publishMsg = { request: 'configure', audio: true, video: true };
        videoRoomHandle.send({ message: publishMsg, jsep: jsep });
    }
});

// 3. 處理 Answer (Subscriber)
subscriberHandle.createAnswer({
    jsep: jsep,
    media: { audioSend: false, videoSend: false },
    success: function(answerJsep) {
        const startMsg = { request: 'start', room: roomId };
        subscriberHandle.send({ message: startMsg, jsep: answerJsep });
    }
});
```

#### 聊天系統實作
```javascript
// VideoRoom display 屬性聊天方案
function sendChatViaVideoRoom(message) {
    const chatPayload = {
        type: 'chat',
        message: message,
        sender: currentUserName,
        timestamp: Date.now(),
        messageId: generateUUIDv7()  // 去重機制
    };
    
    const newDisplay = `${currentUserName}|CHAT:${JSON.stringify(chatPayload)}`;
    
    // 發送 configure 請求更新 display
    videoRoomHandle.send({
        message: { request: 'configure', display: newDisplay },
        success: function(result) {
            // 處理發送結果
        }
    });
}
```

### 除錯技巧

#### 1. 日誌系統使用
```javascript
// 設定日誌等級 (index.html 第 256 行)
const CURRENT_LOG_LEVEL = LOG_LEVELS.DEBUG; // 開發時使用 DEBUG

// 記錄不同等級的訊息
log('一般資訊', LOG_LEVELS.INFO);
log('調試訊息', LOG_LEVELS.DEBUG);
log('錯誤訊息', LOG_LEVELS.ERROR);
```

#### 2. WebRTC 狀態檢查
```javascript
// 檢查 PeerConnection 狀態
console.log('Connection State:', videoRoomHandle.webrtcStuff.pc.connectionState);
console.log('ICE State:', videoRoomHandle.webrtcStuff.pc.iceConnectionState);

// 檢查媒體軌道狀態
localStream.getTracks().forEach(track => {
    console.log(`${track.kind} track enabled:`, track.enabled);
});
```

## 測試指南

### 基本功能測試

1. **單用戶測試**
   - 攝影機和麥克風存取
   - 連接到 Janus Gateway
   - 房間加入和離開
   - 媒體控制 (靜音/視訊開關)

2. **多用戶測試**
   - 同時多個瀏覽器標籤頁
   - 不同瀏覽器測試
   - 不同裝置測試
   - 聊天訊息傳遞

3. **邊界條件測試**
   - 網路中斷恢復
   - 權限拒絕處理
   - 大量用戶 (性能測試)
   - 長時間運行穩定性

### 瀏覽器相容性

支援的瀏覽器版本：
- Chrome 60+
- Firefox 55+
- Safari 12+
- Edge 79+

測試重點：
- WebRTC API 支援
- getUserMedia 權限處理
- WebSocket 連接
- 媒體編解碼器支援

## 提交規範

### Commit 訊息格式

```
<type>(<scope>): <subject>

<body>

<footer>
```

#### Type 類型
- `feat`: 新功能
- `fix`: 錯誤修復
- `docs`: 文件更新
- `style`: 代碼格式調整
- `refactor`: 重構
- `test`: 測試相關
- `chore`: 建置工具或輔助工具的變動

#### 範例
```
feat(chat): implement UUID v7 message deduplication

- Add generateUUIDv7() function for unique message IDs
- Implement processedMessages Map to track sent messages
- Add automatic cleanup for expired message records
- Improve chat reliability with retry mechanism

Fixes #123
```

### Pull Request 流程

1. **提交前檢查**
   ```bash
   # 確保代碼沒有明顯錯誤
   # 檢查瀏覽器控制台是否有錯誤
   # 驗證基本功能正常運作
   ```

2. **創建 Pull Request**
   - 提供清楚的標題和說明
   - 列出變更的功能或修復的問題
   - 包含測試結果或截圖 (如適用)
   - 參考相關的 Issue

3. **代碼審查**
   - 回應審查者的意見
   - 進行必要的修改
   - 確保通過所有檢查

## 常見問題

### Q: 如何添加新的 UI 組件？
A: 在 `index.html` 中添加 HTML 結構，在 `<style>` 區段添加 CSS，在 JavaScript 中處理事件邏輯。

### Q: 如何修改 Janus Gateway 配置？
A: 修改 `index.html` 第 401 行和 242 行的伺服器地址，或在 `startJanusConnection()` 函數中調整連接參數。

### Q: 聊天訊息不顯示怎麼處理？
A: 檢查 `processChatMessage()` 函數的解析邏輯，確認 VideoRoom display 格式正確，檢查去重機制是否正常運作。

### Q: 如何添加新的媒體控制功能？
A: 參考現有的 `toggleMute()` 和 `toggleVideo()` 函數，添加相應的 UI 按鈕和事件處理器。

## 聯繫資訊

- 提交 Issue: [GitHub Issues](https://github.com/your-repo/issues)
- 討論: [GitHub Discussions](https://github.com/your-repo/discussions)
- 文件問題: 直接修改相關 Markdown 文件

---

感謝您對專案的貢獻！每個貢獻都讓這個專案變得更好。