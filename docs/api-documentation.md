# API 接口文檔

## 📋 概述

本文檔提供Quectel AI功能的所有API接口說明，包括REST API、WebSocket API和SDK接口。

## 🔧 基礎信息

### 基礎URL
- **生產環境**: `https://api.quectel.com/v1`
- **測試環境**: `https://api-test.quectel.com/v1`
- **開發環境**: `https://api-dev.quectel.com/v1`

### 認證方式
```bash
# API Key認證
Authorization: Bearer YOUR_API_KEY

# JWT Token認證
Authorization: Bearer YOUR_JWT_TOKEN
```

## 📡 MCP API

### Cellular MCP API

#### 獲取網絡狀態
```http
GET /api/mcp/cellular/status
```

**響應示例**:
```json
{
  "signal_strength": -85,
  "network_type": "5G",
  "operator": "China Mobile",
  "connection_status": "connected",
  "data_speed": {
    "download": "150Mbps",
    "upload": "50Mbps"
  }
}
```

#### 獲取SIM卡信息
```http
GET /api/mcp/cellular/sim
```

**響應示例**:
```json
{
  "iccid": "89860012345678901234",
  "imsi": "460001234567890",
  "operator": "China Mobile",
  "status": "active"
}
```

### WiFi MCP API

#### 掃描WiFi網絡
```http
GET /api/mcp/wifi/scan
```

**響應示例**:
```json
{
  "networks": [
    {
      "ssid": "Quectel_Office",
      "signal_strength": -45,
      "security": "WPA2",
      "channel": 6,
      "frequency": "2.4GHz"
    }
  ]
}
```

#### 獲取連接狀態
```http
GET /api/mcp/wifi/status
```

**響應示例**:
```json
{
  "connected": true,
  "ssid": "Quectel_Office",
  "ip_address": "192.168.1.100",
  "signal_strength": -45,
  "connected_devices": 8
}
```

### GPS MCP API

#### 獲取當前位置
```http
GET /api/mcp/gps/location
```

**響應示例**:
```json
{
  "latitude": 31.2304,
  "longitude": 121.4737,
  "altitude": 4.5,
  "accuracy": 3.2,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### 獲取軌跡記錄
```http
GET /api/mcp/gps/track?time_range=24h
```

**響應示例**:
```json
{
  "track_points": [
    {
      "latitude": 31.2304,
      "longitude": 121.4737,
      "timestamp": "2024-01-15T10:30:00Z"
    }
  ]
}
```

### Bluetooth MCP API

#### 掃描藍牙設備
```http
GET /api/mcp/bluetooth/scan
```

**響應示例**:
```json
{
  "devices": [
    {
      "name": "Quectel_Module",
      "address": "00:11:22:33:44:55",
      "rssi": -65,
      "services": ["serial", "audio"]
    }
  ]
}
```

#### 獲取連接設備
```http
GET /api/mcp/bluetooth/connections
```

**響應示例**:
```json
{
  "connected_devices": [
    {
      "name": "Quectel_Module",
      "address": "00:11:22:33:44:55",
      "services": ["serial"]
    }
  ]
}
```

## 🎤 語音服務 API

### 語音喚醒 API

#### 開始語音喚醒
```http
POST /api/voice/wakeup/start
```

**請求體**:
```json
{
  "keywords": ["你好小Q", "Hey Quectel"],
  "sensitivity": 0.8,
  "timeout": 30
}
```

#### 停止語音喚醒
```http
POST /api/voice/wakeup/stop
```

### 語音識別 API

#### 語音轉文本
```http
POST /api/voice/asr/recognize
Content-Type: multipart/form-data
```

**請求體**:
```json
{
  "audio_file": "audio_data",
  "language": "zh-CN",
  "model": "default"
}
```

**響應示例**:
```json
{
  "text": "你好，我想查詢網絡狀態",
  "confidence": 0.95,
  "language": "zh-CN"
}
```

### 文本轉語音 API

#### 文本轉語音
```http
POST /api/voice/tts/synthesize
```

**請求體**:
```json
{
  "text": "網絡狀態正常",
  "voice": "female",
  "speed": 1.0,
  "language": "zh-CN"
}
```

**響應**: 音頻文件 (MP3/WAV)

## 🤖 AI Agent API

### 對話API

#### 發送消息
```http
POST /api/ai/chat
```

**請求體**:
```json
{
  "message": "幫我檢查網絡狀態",
  "context": {
    "device_id": "QT_001",
    "session_id": "session_123"
  },
  "model": "gpt-4"
}
```

**響應示例**:
```json
{
  "response": "好的，我來為您檢查網絡狀態：\n- 蜂窩網絡：5G信號強度-85dBm\n- WiFi：已連接，信號良好\n所有網絡正常！",
  "confidence": 0.9,
  "actions": [
    {
      "type": "mcp_call",
      "service": "cellular",
      "method": "get_status"
    }
  ]
}
```

## 📊 設備管理 API

### 設備註冊
```http
POST /api/devices/register
```

**請求體**:
```json
{
  "device_id": "QT_001",
  "device_type": "RG500Q",
  "firmware_version": "1.2.3",
  "capabilities": ["cellular", "wifi", "gps"]
}
```

### 設備狀態查詢
```http
GET /api/devices/{device_id}/status
```

**響應示例**:
```json
{
  "device_id": "QT_001",
  "online": true,
  "last_seen": "2024-01-15T10:30:00Z",
  "cellular_signal": -85,
  "wifi_connected": true,
  "gps_fix": true
}
```

## 📈 數據分析 API

### 性能數據查詢
```http
GET /api/analytics/performance
```

**查詢參數**:
- `device_id`: 設備ID
- `metric`: 指標名稱
- `time_range`: 時間範圍
- `aggregation`: 聚合方式

**響應示例**:
```json
{
  "device_id": "QT_001",
  "metric": "cellular_signal",
  "data": [
    {
      "timestamp": "2024-01-15T10:00:00Z",
      "value": -85
    }
  ]
}
```

## 🔐 安全 API

### 獲取訪問令牌
```http
POST /api/auth/token
```

**請求體**:
```json
{
  "client_id": "your_client_id",
  "client_secret": "your_client_secret",
  "grant_type": "client_credentials"
}
```

**響應示例**:
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

## 📋 錯誤碼

| 錯誤碼 | 說明 | 解決方案 |
|--------|------|----------|
| 400 | 請求參數錯誤 | 檢查請求參數格式 |
| 401 | 認證失敗 | 檢查API Key或Token |
| 403 | 權限不足 | 檢查API權限設置 |
| 404 | 資源不存在 | 檢查資源ID是否正確 |
| 429 | 請求頻率過高 | 降低請求頻率 |
| 500 | 服務器內部錯誤 | 聯繫技術支持 |

## 📋 TODO 清單

### 高優先級
- [ ] 完善API文檔
- [ ] 實現所有MCP API
- [ ] 建立API測試框架
- [ ] 實現API版本控制

### 中優先級
- [ ] 添加API限流
- [ ] 實現API監控
- [ ] 建立API文檔生成
- [ ] 實現API SDK

### 低優先級
- [ ] 實現GraphQL API
- [ ] 建立API市場
- [ ] 實現API分析
- [ ] 建立API治理

## 🔗 相關文檔

- [MCP Server實現](./05-mcp-server.md)
- [SDK使用指南](./sdk-guide.md)
- [部署指南](./deployment-guide.md)

---

*最後更新：2024年* 