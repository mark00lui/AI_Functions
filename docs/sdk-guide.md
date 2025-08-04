# SDK 使用指南

## 📋 概述

Quectel AI SDK提供完整的開發工具包，支持多平台開發，包括C/C++、Python、Java等語言。

## 🛠️ 支持的平台

### 硬件平台
- **Linux**: 嵌入式Linux系統
- **Android**: Android應用開發
- **Windows**: Windows桌面應用
- **macOS**: macOS桌面應用

### 開發語言
- **C/C++**: 底層驅動和應用
- **Python**: 快速原型開發
- **Java**: Android應用開發
- **JavaScript**: Web應用開發

## 📦 SDK 安裝

### Python SDK
```bash
# 安裝Python SDK
pip install quectel-ai-sdk

# 或者從源碼安裝
git clone https://github.com/quectel/ai-sdk-python
cd ai-sdk-python
pip install -e .
```

### C/C++ SDK
```bash
# 下載SDK
wget https://github.com/quectel/ai-sdk-c/releases/latest/download/quectel-ai-sdk.tar.gz
tar -xzf quectel-ai-sdk.tar.gz
cd quectel-ai-sdk

# 編譯SDK
mkdir build && cd build
cmake ..
make
make install
```

## 🔧 核心功能

### 1. 語音喚醒

#### Python SDK
```python
from quectel_ai import WakeWordDetector

# 初始化語音喚醒
wake_word = WakeWordDetector(
    model_path="wake_word_model.bin",
    keywords=["你好小Q", "Hey Quectel"]
)

# 開始檢測
def on_wake_word_detected(keyword, confidence):
    print(f"檢測到喚醒詞: {keyword}, 置信度: {confidence}")

wake_word.start_detection(callback=on_wake_word_detected)
```

#### C/C++ SDK
```c
#include "quectel_ai_wake_word.h"

// 初始化語音喚醒
quectel_wake_word_t wake_word;
quectel_wake_word_init(&wake_word, "wake_word_model.bin");

// 設置喚醒詞
const char* keywords[] = {"你好小Q", "Hey Quectel"};
quectel_wake_word_set_keywords(&wake_word, keywords, 2);

// 開始檢測
quectel_wake_word_start(&wake_word, on_wake_word_callback);
```

### 2. MCP 服務調用

#### Python SDK
```python
from quectel_ai import MCPClient

# 初始化MCP客戶端
mcp_client = MCPClient("https://api.quectel.com/v1")

# 調用Cellular服務
cellular_status = mcp_client.call("cellular", "get_status")
print(f"信號強度: {cellular_status['signal_strength']}")

# 調用WiFi服務
wifi_networks = mcp_client.call("wifi", "scan_networks")
for network in wifi_networks['networks']:
    print(f"SSID: {network['ssid']}, 信號: {network['signal_strength']}")
```

#### C/C++ SDK
```c
#include "quectel_ai_mcp.h"

// 初始化MCP客戶端
quectel_mcp_client_t mcp_client;
quectel_mcp_init(&mcp_client, "https://api.quectel.com/v1");

// 調用Cellular服務
quectel_mcp_response_t response;
quectel_mcp_call(&mcp_client, "cellular", "get_status", NULL, &response);

// 解析響應
json_object* json = json_tokener_parse(response.data);
json_object* signal_strength;
json_object_object_get_ex(json, "signal_strength", &signal_strength);
printf("信號強度: %d\n", json_object_get_int(signal_strength));
```

### 3. AI 對話

#### Python SDK
```python
from quectel_ai import AIClient

# 初始化AI客戶端
ai_client = AIClient(
    api_key="your_api_key",
    model="gpt-4"
)

# 發送消息
response = ai_client.chat("幫我檢查網絡狀態")
print(f"AI回應: {response['text']}")

# 流式對話
for chunk in ai_client.chat_stream("講個笑話"):
    print(chunk, end="", flush=True)
```

#### C/C++ SDK
```c
#include "quectel_ai_chat.h"

// 初始化AI客戶端
quectel_ai_client_t ai_client;
quectel_ai_init(&ai_client, "your_api_key", "gpt-4");

// 發送消息
quectel_ai_response_t response;
quectel_ai_chat(&ai_client, "幫我檢查網絡狀態", &response);
printf("AI回應: %s\n", response.text);
```

## 📱 應用示例

### 1. 智能車載助手

#### Python 實現
```python
import asyncio
from quectel_ai import WakeWordDetector, MCPClient, AIClient

class CarAssistant:
    def __init__(self):
        self.wake_word = WakeWordDetector()
        self.mcp_client = MCPClient()
        self.ai_client = AIClient()
    
    async def start(self):
        # 開始語音喚醒
        self.wake_word.start_detection(callback=self.on_wake_word)
        
        # 保持運行
        while True:
            await asyncio.sleep(1)
    
    def on_wake_word(self, keyword, confidence):
        print(f"喚醒詞: {keyword}")
        # 開始語音識別和AI對話
        self.start_conversation()
    
    def start_conversation(self):
        # 語音識別
        text = self.recognize_speech()
        
        # AI處理
        response = self.ai_client.chat(text)
        
        # 語音合成
        self.synthesize_speech(response)

# 運行車載助手
assistant = CarAssistant()
asyncio.run(assistant.start())
```

### 2. 智能家居控制

#### Python 實現
```python
from quectel_ai import MCPClient, AIClient

class SmartHomeController:
    def __init__(self):
        self.mcp_client = MCPClient()
        self.ai_client = AIClient()
    
    def check_network_status(self):
        """檢查網絡狀態"""
        cellular = self.mcp_client.call("cellular", "get_status")
        wifi = self.mcp_client.call("wifi", "get_status")
        
        return {
            "cellular": cellular,
            "wifi": wifi
        }
    
    def control_device(self, device_type, action, params=None):
        """控制智能設備"""
        if device_type == "light":
            return self.mcp_client.call("home", "control_light", {
                "action": action,
                "params": params
            })
        elif device_type == "thermostat":
            return self.mcp_client.call("home", "control_thermostat", {
                "action": action,
                "params": params
            })
    
    def process_voice_command(self, command):
        """處理語音命令"""
        # AI理解命令
        intent = self.ai_client.analyze_intent(command)
        
        # 執行相應動作
        if intent["action"] == "check_network":
            return self.check_network_status()
        elif intent["action"] == "control_device":
            return self.control_device(
                intent["device_type"],
                intent["action"],
                intent["params"]
            )

# 使用示例
controller = SmartHomeController()
status = controller.process_voice_command("檢查家裡的網絡狀態")
print(status)
```

## 🔧 配置管理

### 配置文件
```yaml
# config.yaml
api:
  base_url: "https://api.quectel.com/v1"
  api_key: "your_api_key"
  timeout: 30

wake_word:
  model_path: "models/wake_word.bin"
  keywords: ["你好小Q", "Hey Quectel"]
  sensitivity: 0.8

mcp:
  services:
    - cellular
    - wifi
    - gps
    - bluetooth

ai:
  model: "gpt-4"
  max_tokens: 1000
  temperature: 0.7
```

### 加載配置
```python
from quectel_ai import load_config

# 加載配置文件
config = load_config("config.yaml")

# 使用配置初始化SDK
wake_word = WakeWordDetector(**config["wake_word"])
mcp_client = MCPClient(**config["api"])
ai_client = AIClient(**config["ai"])
```

## 🐛 調試和日誌

### 啟用調試模式
```python
import logging
from quectel_ai import set_log_level

# 設置日誌級別
set_log_level(logging.DEBUG)

# 或者使用環境變量
import os
os.environ["QUECTEL_AI_LOG_LEVEL"] = "DEBUG"
```

### 性能監控
```python
from quectel_ai import PerformanceMonitor

# 創建性能監控器
monitor = PerformanceMonitor()

# 監控API調用
with monitor.track("api_call"):
    response = mcp_client.call("cellular", "get_status")

# 獲取性能報告
report = monitor.get_report()
print(f"平均響應時間: {report['avg_response_time']}ms")
```

## 📋 TODO 清單

### 高優先級
- [ ] 完善SDK文檔
- [ ] 實現所有核心功能
- [ ] 建立示例代碼
- [ ] 實現錯誤處理

### 中優先級
- [ ] 添加單元測試
- [ ] 實現性能優化
- [ ] 建立CI/CD流程
- [ ] 實現版本管理

### 低優先級
- [ ] 實現高級功能
- [ ] 建立插件系統
- [ ] 實現自動更新
- [ ] 建立社區支持

## 🔗 相關文檔

- [API接口文檔](./api-documentation.md)
- [部署指南](./deployment-guide.md)
- [MCP Server實現](./05-mcp-server.md)

---

*最後更新：2024年* 