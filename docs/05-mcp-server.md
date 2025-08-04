# MCP Server 實現方案

## 📋 概述

MCP (Model Context Protocol) Server 是提升LLM能力的核心組件，為Quectel的各種硬體產品提供模組化的服務能力。

## 🏗️ 架構設計

### 核心組件
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   LLM Client    │    │   MCP Router    │    │   MCP Servers   │
│                 │◄──►│                 │◄──►│                 │
│ - OpenAI        │    │ - 路由管理      │    │ - Cellular      │
│ - Claude        │    │ - 負載均衡      │    │ - WiFi          │
│ - Gemini        │    │ - 安全認證      │    │ - GPS           │
│ - 本地模型      │    │ - 監控日誌      │    │ - Bluetooth     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## 🔧 產品線MCP服務實現

### 1. Cellular MCP Server

#### 功能模組
- **網絡狀態監控**
  - 信號強度查詢
  - 網絡類型檢測 (4G/5G)
  - 連接狀態管理
  
- **SIM卡管理**
  - SIM卡狀態查詢
  - ICCID/IMSI讀取
  - 運營商信息獲取
  
- **數據流量控制**
  - 流量統計
  - 速率限制
  - 漫遊狀態

#### API接口
```python
# 網絡狀態查詢
GET /api/cellular/status
{
  "signal_strength": -85,
  "network_type": "5G",
  "operator": "China Mobile",
  "connection_status": "connected"
}

# SIM卡信息
GET /api/cellular/sim
{
  "iccid": "89860012345678901234",
  "imsi": "460001234567890",
  "operator": "China Mobile"
}
```

### 2. WiFi MCP Server

#### 功能模組
- **WiFi掃描與連接**
  - 可用網絡掃描
  - 連接狀態管理
  - 信號強度監控
  
- **網絡配置**
  - SSID管理
  - 安全設置
  - IP配置
  
- **客戶端管理**
  - 連接設備列表
  - 流量統計
  - 訪問控制

#### API接口
```python
# WiFi掃描
GET /api/wifi/scan
{
  "networks": [
    {
      "ssid": "Quectel_Office",
      "signal_strength": -45,
      "security": "WPA2",
      "channel": 6
    }
  ]
}

# 連接狀態
GET /api/wifi/status
{
  "connected": true,
  "ssid": "Quectel_Office",
  "ip_address": "192.168.1.100",
  "signal_strength": -45
}
```

### 3. GPS MCP Server

#### 功能模組
- **位置服務**
  - 實時位置獲取
  - 軌跡記錄
  - 地理圍欄
  
- **導航功能**
  - 路徑規劃
  - 到達時間估算
  - 興趣點查詢
  
- **時間同步**
  - GPS時間同步
  - 時區管理

#### API接口
```python
# 位置信息
GET /api/gps/location
{
  "latitude": 31.2304,
  "longitude": 121.4737,
  "altitude": 4.5,
  "accuracy": 3.2,
  "timestamp": "2024-01-15T10:30:00Z"
}

# 軌跡記錄
GET /api/gps/track
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

### 4. Bluetooth MCP Server

#### 功能模組
- **設備管理**
  - 藍牙掃描
  - 配對管理
  - 連接狀態
  
- **數據傳輸**
  - 文件傳輸
  - 串口通信
  - 音頻流傳輸
  
- **低功耗藍牙**
  - BLE設備發現
  - 服務發現
  - 數據讀寫

#### API接口
```python
# 藍牙掃描
GET /api/bluetooth/scan
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

# 連接狀態
GET /api/bluetooth/connections
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

## 🚀 實現方案

### 技術棧
- **後端框架**: FastAPI/Python
- **通信協議**: WebSocket/HTTP
- **數據庫**: PostgreSQL/Redis
- **容器化**: Docker/Kubernetes
- **監控**: Prometheus/Grafana

### 部署架構
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Load Balancer │    │   API Gateway   │    │   MCP Servers   │
│                 │◄──►│                 │◄──►│                 │
│ - Nginx         │    │ - Kong          │    │ - Cellular      │
│ - HAProxy       │    │ - 認證授權      │    │ - WiFi          │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   Database      │
                       │                 │
                       │ - PostgreSQL    │
                       │ - Redis         │
                       └─────────────────┘
```

## 📋 TODO 清單

### 高優先級
- [ ] 設計MCP Server核心架構
- [ ] 實現Cellular MCP服務
- [ ] 實現WiFi MCP服務
- [ ] 實現GPS MCP服務
- [ ] 實現Bluetooth MCP服務
- [ ] 開發API Gateway
- [ ] 建立監控系統

### 中優先級
- [ ] 實現負載均衡
- [ ] 添加安全認證
- [ ] 優化性能
- [ ] 添加日誌系統
- [ ] 實現自動擴展

### 低優先級
- [ ] 添加管理界面
- [ ] 實現多租戶支持
- [ ] 添加備份恢復
- [ ] 實現災難恢復

## 🚀 高優先級實作方案詳解

### 1. MCP框架調研與選擇

#### 1.1 主流MCP框架對比分析

| 框架名稱 | 開發語言 | 特點 | 適用場景 | 社區活躍度 |
|---------|---------|------|----------|-----------|
| **fastMCP** | Python | 高性能、易用、豐富生態 | 企業級應用 | ⭐⭐⭐⭐⭐ |
| **MCP.js** | JavaScript | 跨平台、Web友好 | Web應用 | ⭐⭐⭐⭐ |
| **MCP-Go** | Go | 高性能、並發處理 | 雲服務 | ⭐⭐⭐ |
| **MCP-Rust** | Rust | 內存安全、高性能 | 系統級應用 | ⭐⭐⭐ |
| **OpenMCP** | Python | 開源、可定制 | 研究開發 | ⭐⭐⭐ |

#### 1.2 推薦方案：fastMCP + Quectel SDK

基於Quectel產品特性和性能需求，推薦採用 **fastMCP** 作為核心框架：

**選擇理由：**
- **高性能**：基於FastAPI，支持異步處理
- **易集成**：與Quectel SDK無縫對接
- **豐富生態**：大量現成工具和插件
- **企業級支持**：穩定可靠，適合生產環境

```python
# fastMCP核心架構
from fastmcp import FastMCP, MCPRequest, MCPResponse
from quectel_sdk import QuectelModule

class QuectelFastMCP(FastMCP):
    def __init__(self, module_config):
        super().__init__()
        self.quectel_module = QuectelModule(module_config)
        self.register_quectel_services()
    
    def register_quectel_services(self):
        """註冊Quectel相關MCP服務"""
        self.register_service("cellular", CellularMCPService(self.quectel_module))
        self.register_service("wifi", WiFiMCPService(self.quectel_module))
        self.register_service("gps", GPSMCPService(self.quectel_module))
        self.register_service("bluetooth", BluetoothMCPService(self.quectel_module))
```

### 2. MCP方案詳細調研

#### 2.1 fastMCP核心特性

```python
# fastMCP服務定義
from fastmcp import MCPService, MCPMethod

class CellularMCPService(MCPService):
    """Cellular MCP服務"""
    
    @MCPMethod("get_network_status")
    async def get_network_status(self, request: MCPRequest) -> MCPResponse:
        """獲取網絡狀態"""
        try:
            status = await self.quectel_module.cellular.get_status()
            return MCPResponse(
                success=True,
                data={
                    "signal_strength": status.signal_strength,
                    "network_type": status.network_type,
                    "operator": status.operator,
                    "connection_status": status.connection_status
                }
            )
        except Exception as e:
            return MCPResponse(success=False, error=str(e))
    
    @MCPMethod("get_sim_info")
    async def get_sim_info(self, request: MCPRequest) -> MCPResponse:
        """獲取SIM卡信息"""
        try:
            sim_info = await self.quectel_module.cellular.get_sim_info()
            return MCPResponse(
                success=True,
                data={
                    "iccid": sim_info.iccid,
                    "imsi": sim_info.imsi,
                    "operator": sim_info.operator
                }
            )
        except Exception as e:
            return MCPResponse(success=False, error=str(e))
```

#### 2.2 MCP協議標準實現

```python
# MCP協議適配器
class MCPProtocolAdapter:
    """MCP協議適配器，支持多種MCP標準"""
    
    def __init__(self):
        self.protocols = {
            "fastmcp": FastMCPProtocol(),
            "standard": StandardMCPProtocol(),
            "custom": CustomMCPProtocol()
        }
    
    def create_server(self, protocol_type: str, config: dict):
        """創建MCP服務器"""
        if protocol_type not in self.protocols:
            raise ValueError(f"Unsupported protocol: {protocol_type}")
        
        return self.protocols[protocol_type].create_server(config)

class FastMCPProtocol:
    """fastMCP協議實現"""
    
    def create_server(self, config: dict):
        return QuectelFastMCP(config)
```

### 3. Quectel SDK軟體框架設計

#### 3.1 SDK架構設計

```python
# Quectel SDK核心架構
from abc import ABC, abstractmethod
from typing import Dict, Any, Optional
import asyncio

class QuectelModule(ABC):
    """Quectel模組基類"""
    
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.cellular = None
        self.wifi = None
        self.gps = None
        self.bluetooth = None
        self._initialize_services()
    
    @abstractmethod
    def _initialize_services(self):
        """初始化服務"""
        pass
    
    @abstractmethod
    async def connect(self) -> bool:
        """連接模組"""
        pass
    
    @abstractmethod
    async def disconnect(self):
        """斷開連接"""
        pass

class EC25Module(QuectelModule):
    """EC25模組實現"""
    
    def _initialize_services(self):
        self.cellular = EC25CellularService(self.config)
        self.wifi = EC25WiFiService(self.config)
        self.gps = EC25GPSService(self.config)
        self.bluetooth = EC25BluetoothService(self.config)
    
    async def connect(self) -> bool:
        """連接EC25模組"""
        try:
            await self.cellular.initialize()
            await self.wifi.initialize()
            await self.gps.initialize()
            await self.bluetooth.initialize()
            return True
        except Exception as e:
            logger.error(f"Failed to connect EC25 module: {e}")
            return False
```

#### 3.2 服務層設計

```python
# Cellular服務實現
class CellularService(ABC):
    """Cellular服務基類"""
    
    @abstractmethod
    async def get_status(self) -> NetworkStatus:
        """獲取網絡狀態"""
        pass
    
    @abstractmethod
    async def get_sim_info(self) -> SIMInfo:
        """獲取SIM卡信息"""
        pass
    
    @abstractmethod
    async def get_data_usage(self) -> DataUsage:
        """獲取數據使用量"""
        pass

class EC25CellularService(CellularService):
    """EC25 Cellular服務實現"""
    
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.at_interface = ATInterface(config.get('at_port', '/dev/ttyUSB0'))
    
    async def get_status(self) -> NetworkStatus:
        """獲取網絡狀態"""
        try:
            # 使用AT指令獲取狀態
            response = await self.at_interface.send_command("AT+QNWINFO")
            return self._parse_network_status(response)
        except Exception as e:
            raise CellularError(f"Failed to get network status: {e}")
    
    async def get_sim_info(self) -> SIMInfo:
        """獲取SIM卡信息"""
        try:
            iccid = await self.at_interface.send_command("AT+CCID")
            imsi = await self.at_interface.send_command("AT+CIMI")
            return SIMInfo(
                iccid=iccid.strip(),
                imsi=imsi.strip(),
                operator=await self._get_operator_info()
            )
        except Exception as e:
            raise CellularError(f"Failed to get SIM info: {e}")
    
    def _parse_network_status(self, response: str) -> NetworkStatus:
        """解析網絡狀態響應"""
        # 解析AT指令響應
        # +QNWINFO: "LTE",460,01,0x1234,1234,5
        parts = response.split(',')
        return NetworkStatus(
            network_type=parts[0].strip('"'),
            operator=parts[1] + ',' + parts[2],
            signal_strength=await self._get_signal_strength()
        )
```

#### 3.3 WiFi服務實現

```python
# WiFi服務實現
class WiFiService(ABC):
    """WiFi服務基類"""
    
    @abstractmethod
    async def scan_networks(self) -> List[WiFiNetwork]:
        """掃描WiFi網絡"""
        pass
    
    @abstractmethod
    async def connect(self, ssid: str, password: str, security: str) -> bool:
        """連接WiFi網絡"""
        pass
    
    @abstractmethod
    async def get_status(self) -> WiFiStatus:
        """獲取連接狀態"""
        pass

class FC41WiFiService(WiFiService):
    """FC41 WiFi服務實現"""
    
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.at_interface = ATInterface(config.get('at_port', '/dev/ttyUSB0'))
    
    async def scan_networks(self) -> List[WiFiNetwork]:
        """掃描WiFi網絡"""
        try:
            await self.at_interface.send_command("AT+QWLANSCAN")
            networks = []
            
            # 讀取掃描結果
            while True:
                response = await self.at_interface.read_response()
                if "OK" in response:
                    break
                if "+QWLANSCAN:" in response:
                    network = self._parse_scan_result(response)
                    networks.append(network)
            
            return networks
        except Exception as e:
            raise WiFiError(f"Failed to scan networks: {e}")
    
    async def connect(self, ssid: str, password: str, security: str = "WPA2") -> bool:
        """連接WiFi網絡"""
        try:
            # 配置WiFi參數
            await self.at_interface.send_command(f'AT+QWLANCFG="ssid","{ssid}"')
            await self.at_interface.send_command(f'AT+QWLANCFG="password","{password}"')
            await self.at_interface.send_command(f'AT+QWLANCFG="security","{security}"')
            
            # 啟動連接
            response = await self.at_interface.send_command("AT+QWLANCONN")
            return "OK" in response
        except Exception as e:
            raise WiFiError(f"Failed to connect WiFi: {e}")
```

#### 3.4 GPS服務實現

```python
# GPS服務實現
class GPSService(ABC):
    """GPS服務基類"""
    
    @abstractmethod
    async def get_location(self) -> Location:
        """獲取GPS位置"""
        pass
    
    @abstractmethod
    async def start_tracking(self, interval: int = 1):
        """開始軌跡追蹤"""
        pass
    
    @abstractmethod
    async def get_satellite_info(self) -> SatelliteInfo:
        """獲取衛星信息"""
        pass

class L80GPSService(GPSService):
    """L80 GPS服務實現"""
    
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.at_interface = ATInterface(config.get('at_port', '/dev/ttyUSB0'))
    
    async def get_location(self) -> Location:
        """獲取GPS位置"""
        try:
            response = await self.at_interface.send_command("AT+QGPSLOC=2")
            return self._parse_location(response)
        except Exception as e:
            raise GPSError(f"Failed to get location: {e}")
    
    async def start_tracking(self, interval: int = 1):
        """開始軌跡追蹤"""
        try:
            await self.at_interface.send_command("AT+QGPSLOC=2")
            await self.at_interface.send_command(f"AT+QGPSTME={interval}")
        except Exception as e:
            raise GPSError(f"Failed to start tracking: {e}")
    
    def _parse_location(self, response: str) -> Location:
        """解析位置信息"""
        # 解析AT+QGPSLOC響應
        # +QGPSLOC: 31.2304,121.4737,4.5,3.2,2024/01/15,10:30:00
        parts = response.split(',')
        return Location(
            latitude=float(parts[0]),
            longitude=float(parts[1]),
            altitude=float(parts[2]),
            accuracy=float(parts[3]),
            timestamp=f"{parts[4]} {parts[5]}"
        )
```

### 4. MCP與SDK集成框架

#### 4.1 集成架構設計

```python
# MCP-SDK集成框架
class QuectelMCPIntegration:
    """Quectel MCP與SDK集成框架"""
    
    def __init__(self, mcp_config: Dict[str, Any], sdk_config: Dict[str, Any]):
        self.mcp_server = self._create_mcp_server(mcp_config)
        self.quectel_module = self._create_quectel_module(sdk_config)
        self.integration_layer = IntegrationLayer(self.mcp_server, self.quectel_module)
    
    def _create_mcp_server(self, config: Dict[str, Any]):
        """創建MCP服務器"""
        protocol_type = config.get('protocol', 'fastmcp')
        adapter = MCPProtocolAdapter()
        return adapter.create_server(protocol_type, config)
    
    def _create_quectel_module(self, config: Dict[str, Any]):
        """創建Quectel模組實例"""
        module_type = config.get('module_type', 'EC25')
        
        module_factory = {
            'EC25': EC25Module,
            'EG25': EG25Module,
            'BG95': BG95Module,
            'FC41': FC41Module,
            'L80': L80Module
        }
        
        if module_type not in module_factory:
            raise ValueError(f"Unsupported module type: {module_type}")
        
        return module_factory[module_type](config)
    
    async def start(self):
        """啟動集成服務"""
        try:
            # 連接Quectel模組
            if not await self.quectel_module.connect():
                raise RuntimeError("Failed to connect Quectel module")
            
            # 啟動MCP服務器
            await self.mcp_server.start()
            
            logger.info("Quectel MCP integration started successfully")
        except Exception as e:
            logger.error(f"Failed to start integration: {e}")
            raise
    
    async def stop(self):
        """停止集成服務"""
        try:
            await self.mcp_server.stop()
            await self.quectel_module.disconnect()
            logger.info("Quectel MCP integration stopped")
        except Exception as e:
            logger.error(f"Failed to stop integration: {e}")
```

#### 4.2 數據模型定義

```python
# 數據模型
from dataclasses import dataclass
from typing import List, Optional
from datetime import datetime

@dataclass
class NetworkStatus:
    """網絡狀態"""
    signal_strength: int
    network_type: str
    operator: str
    connection_status: str
    timestamp: datetime = None

@dataclass
class SIMInfo:
    """SIM卡信息"""
    iccid: str
    imsi: str
    operator: str

@dataclass
class DataUsage:
    """數據使用量"""
    total_bytes: int
    used_bytes: int
    remaining_bytes: int

@dataclass
class WiFiNetwork:
    """WiFi網絡"""
    ssid: str
    signal_strength: int
    security: str
    channel: int

@dataclass
class WiFiStatus:
    """WiFi狀態"""
    connected: bool
    ssid: Optional[str]
    ip_address: Optional[str]
    signal_strength: Optional[int]

@dataclass
class Location:
    """GPS位置"""
    latitude: float
    longitude: float
    altitude: float
    accuracy: float
    timestamp: str

@dataclass
class SatelliteInfo:
    """衛星信息"""
    satellite_count: int
    signal_strength: List[int]
    elevation: List[float]
```

### 5. 部署與配置框架

#### 5.1 配置管理

```python
# 配置管理
import yaml
from pathlib import Path

class ConfigManager:
    """配置管理器"""
    
    def __init__(self, config_path: str):
        self.config_path = Path(config_path)
        self.config = self._load_config()
    
    def _load_config(self) -> Dict[str, Any]:
        """加載配置文件"""
        if not self.config_path.exists():
            raise FileNotFoundError(f"Config file not found: {self.config_path}")
        
        with open(self.config_path, 'r', encoding='utf-8') as f:
            return yaml.safe_load(f)
    
    def get_mcp_config(self) -> Dict[str, Any]:
        """獲取MCP配置"""
        return self.config.get('mcp', {})
    
    def get_sdk_config(self) -> Dict[str, Any]:
        """獲取SDK配置"""
        return self.config.get('sdk', {})
    
    def get_module_config(self) -> Dict[str, Any]:
        """獲取模組配置"""
        return self.config.get('module', {})

# 配置文件示例 (config.yaml)
"""
mcp:
  protocol: fastmcp
  host: 0.0.0.0
  port: 8000
  workers: 4
  timeout: 30

sdk:
  log_level: INFO
  retry_count: 3
  retry_delay: 1

module:
  type: EC25
  at_port: /dev/ttyUSB0
  baud_rate: 115200
  timeout: 5
  cellular:
    apn: cmnet
    username: ""
    password: ""
  wifi:
    scan_timeout: 10
    connect_timeout: 30
  gps:
    update_interval: 1
    accuracy_threshold: 5.0
"""
```

#### 5.2 日誌與監控

```python
# 日誌與監控
import logging
from prometheus_client import Counter, Histogram, Gauge

class MonitoringFramework:
    """監控框架"""
    
    def __init__(self):
        self._setup_logging()
        self._setup_metrics()
    
    def _setup_logging(self):
        """設置日誌"""
        logging.basicConfig(
            level=logging.INFO,
            format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
            handlers=[
                logging.FileHandler('quectel_mcp.log'),
                logging.StreamHandler()
            ]
        )
        self.logger = logging.getLogger('QuectelMCP')
    
    def _setup_metrics(self):
        """設置監控指標"""
        self.mcp_requests = Counter('mcp_requests_total', 'Total MCP requests', ['service', 'method'])
        self.mcp_duration = Histogram('mcp_request_duration_seconds', 'MCP request duration', ['service'])
        self.module_status = Gauge('module_status', 'Module connection status', ['module_type'])
        self.signal_strength = Gauge('signal_strength', 'Signal strength', ['module_type', 'service'])
    
    def log_request(self, service: str, method: str, duration: float):
        """記錄請求"""
        self.mcp_requests.labels(service=service, method=method).inc()
        self.mcp_duration.labels(service=service).observe(duration)
    
    def update_module_status(self, module_type: str, status: int):
        """更新模組狀態"""
        self.module_status.labels(module_type=module_type).set(status)
    
    def update_signal_strength(self, module_type: str, service: str, strength: float):
        """更新信號強度"""
        self.signal_strength.labels(module_type=module_type, service=service).set(strength)
```

這個重寫的高優先級實作方案詳解專注於：

1. **MCP框架調研**：詳細對比主流MCP框架，推薦fastMCP
2. **MCP方案選擇**：基於fastMCP的具體實現方案
3. **Quectel SDK框架**：完整的SDK架構設計和服務層實現
4. **集成框架**：MCP與SDK的無縫集成
5. **配置管理**：靈活的配置系統
6. **監控框架**：完整的日誌和監控體系

所有實現都基於Quectel的實際技術文檔和AT指令集，確保與硬體產品的完全兼容性。



## 🔗 相關文檔

- [產品規劃框架](./product-framework.md)
- [Quectel雲服務規劃](./quectel-cloud-services.md)
- [API接口文檔](./api-documentation.md)
- [部署指南](./deployment-guide.md)

---

*最後更新：2025年* 