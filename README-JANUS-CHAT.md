# 使用 Janus Gateway TextRoom 插件實現聊天功能

## 概述

現在聊天功能直接使用 Janus Gateway 的 TextRoom 插件，無需額外的聊天服務器。這樣所有功能（視訊、音訊、聊天）都通過同一個 Janus Gateway 服務器處理。

## 啟用 Janus TextRoom 插件

### 方法 1: 檢查插件是否已啟用

1. 查看 Janus Gateway 配置文件 `janus.plugin.textroom.jcfg`
2. 確保插件未被禁用

### 方法 2: Docker 環境（推薦）

如果你使用 Docker 運行 Janus Gateway，TextRoom 插件通常已經包含：

```bash
docker run -it --rm \
  --name janus \
  -p 8088:8088 \
  -p 8089:8089 \
  -p 8188:8188 \
  -p 10000-10200:10000-10200/udp \
  canyan/janus-gateway:latest
```

### 方法 3: 從源碼編譯

如果從源碼安裝 Janus Gateway：

```bash
# 確保編譯時包含 TextRoom 插件
./configure --enable-plugin-textroom
make && make install
```

## 功能特色

✅ **統一服務器**：視訊、音訊、聊天都使用同一個 Janus Gateway  
✅ **真正跨用戶**：不同電腦、不同瀏覽器之間的即時聊天同步  
✅ **房間隔離**：聊天訊息只在同一視訊房間內同步  
✅ **自動降級**：如果 TextRoom 插件不可用，會自動使用本地聊天模式  
✅ **無需額外配置**：不需要運行額外的聊天服務器  

## 使用流程

1. **啟動 Janus Gateway**（確保包含 TextRoom 插件）
2. **打開 index.html**
3. **加入視訊房間** 
4. **聊天功能自動啟用**

## 故障排除

### 如果看到 "TextRoom 插件連接失敗" 錯誤：

1. **檢查 Janus 日誌**：
   ```bash
   # 查看 Janus Gateway 日誌
   tail -f /var/log/janus/janus.log
   ```

2. **檢查可用插件**：
   訪問 `http://your-janus-server:8088/janus/info` 查看已載入的插件

3. **使用 WebSocket API 測試**：
   ```javascript
   // 在瀏覽器控制台測試
   fetch('ws://192.168.6.70:8188')
   .then(response => console.log('Janus WebSocket 可用'))
   .catch(error => console.error('連接失敗:', error));
   ```

### ⚠️ TextRoom API 兼容性問題：

**常見錯誤及解決方案**：
- `"Missing mandatory element (secret)"` (錯誤代碼 413)：TextRoom 需要密鑰驗證
- `"Unknown request 'message'"` (錯誤代碼 415)：你的 Janus Gateway 版本的 TextRoom API 不兼容
- `"Unknown request 'announcement'"` (錯誤代碼 415)：需要升級 Janus Gateway
- `"TextRoom 插件連接失敗"`：插件未啟用或不存在

**修復密鑰問題（錯誤 413）**：
1. 編輯 `index.html` 文件，找到 `textRoomSecret` 變數
2. 如果你的 Janus TextRoom 配置需要密鑰，設置正確的值：
   ```javascript
   const textRoomSecret = "your-secret-here"; // 替換為你的密鑰
   ```
3. 如果 TextRoom 配置為不需要密鑰，檢查 `janus.plugin.textroom.jcfg` 配置文件

**自動降級機制**：
- 系統會自動檢測 TextRoom API 兼容性
- 如果出現 415 錯誤，會顯示 "此 Janus Gateway 版本的 TextRoom API 不兼容"
- 自動切換到備用聊天模式

### 如果 TextRoom 不可用，系統會：

- 自動顯示 "使用備用聊天方案" 訊息
- 切換到本地 BroadcastChannel + localStorage 模式
- 同一台機器的多個標籤頁仍可聊天
- 提供模擬回覆功能用於測試

## 技術細節

### Janus TextRoom 插件 API：

```javascript
// 創建聊天房間
{
  "request": "create",
  "room": 房間ID,
  "description": "聊天房間描述"
}

// 加入聊天房間
{
  "request": "join", 
  "room": 房間ID,
  "username": "用戶名"
}

// 發送聊天訊息
{
  "request": "message",
  "room": 房間ID, 
  "text": "聊天內容"
}
```

### 事件處理：

- `join`: 用戶加入聊天室
- `leave`: 用戶離開聊天室  
- `message`: 接收到聊天訊息

## 優勢

1. **簡化部署**：只需一個 Janus Gateway 服務器
2. **統一管理**：所有 WebRTC 功能集中管理
3. **更好性能**：減少網路連接數和延遲
4. **原生支援**：使用 Janus 的原生聊天功能
5. **自動備援**：插件不可用時自動降級

這樣的設計讓你的 WebRTC 聊天室更加穩定和易於維護！