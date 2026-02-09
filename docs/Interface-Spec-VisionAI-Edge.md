# 設備介面規格文件

## VisionAI Edge × WISE-IoTSuite × Mac 整合介面

**文件編號**: SMAI-IFS-2024-001
**專案名稱**: 製程視覺 AI 分析系統 (Process Vision AI)
**版本**: 1.2
**日期**: 2025-02-09
**單位**: 再生廠智慧製造

---

## 文件資訊

| 項目 | 內容 |
|------|------|
| **需求單位** | 再生廠智慧製造 |
| **撰寫人** | |
| **審核人** | |

---

## 修訂紀錄

| 版本 | 日期 | 修訂內容 | 作者 |
|------|------|---------|------|
| 1.0 | 2024-01-15 | 初版 | |
| 1.1 | 2025-01-30 | 文件審閱更新 | |
| 1.2 | 2025-02-05 | 介面規格細化、安全性規格補充 | |

---

## 1. 系統架構概述

### 1.1 整合架構圖

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              OT 網路 (192.168.1.0/24)                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐  │
│   │  IPCam × 20                                                     │  │
│   │  RTSP: rtsp://{ip}:{port}/stream                               │  │
│   └─────────────────────────────┬───────────────────────────────────┘  │
│                                 │                                       │
│   ┌─────────────────────────────┴───────────────────────────────────┐  │
│   │  VisionAI Edge (MIC-730AI / MIC-720AI)                          │  │
│   │  IP: 192.168.1.101 ~ 192.168.1.106                             │  │
│   └─────────────────────────────┬───────────────────────────────────┘  │
│                                 │ MQTT (Port 1883)                      │
│                                 │                                       │
└─────────────────────────────────┼───────────────────────────────────────┘
                                  │
                        ┌─────────┴─────────┐
                        │   Firewall/Router │
                        │   NAT + ACL       │
                        └─────────┬─────────┘
                                  │
┌─────────────────────────────────┼───────────────────────────────────────┐
│                         IT 網路 (192.168.10.0/24)                        │
├─────────────────────────────────┼───────────────────────────────────────┤
│                                 │                                       │
│   ┌─────────────────────────────┴───────────────────────────────────┐  │
│   │  WISE-IoTSuite                                                  │  │
│   │  IP: 192.168.10.50                                              │  │
│   │  MQTT Broker: mqtt://192.168.10.50:1883                        │  │
│   │  REST API: https://192.168.10.50:443/api                       │  │
│   └─────────────────────────────┬───────────────────────────────────┘  │
│                                 │                                       │
│   ┌─────────────────────────────┴───────────────────────────────────┐  │
│   │  Mac Studio M2 Max                                              │  │
│   │  IP: 192.168.10.100                                             │  │
│   │  API: https://192.168.10.100:8000/api                          │  │
│   └─────────────────────────────────────────────────────────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 1.2 介面清單

| 介面編號 | 介面名稱 | 來源 | 目的 | 協定 |
|---------|---------|------|------|------|
| IF-01 | 攝影機串流 | IPCam | VisionAI Edge | RTSP |
| IF-02 | 推論結果 | VisionAI Edge | WISE-IoTSuite | MQTT |
| IF-03 | 設備狀態 | VisionAI Edge | WISE-IoTSuite | MQTT |
| IF-04 | 告警事件 | VisionAI Edge | WISE-IoTSuite | MQTT |
| IF-05 | 時序資料查詢 | Mac | WISE-IoTSuite | REST |
| IF-06 | 分析結果回寫 | Mac | WISE-IoTSuite | REST |
| IF-07 | 設備管理 | Mac | WISE-IoTSuite | REST |
| IF-08 | 跨源查詢 | Mac | DataInsight | REST |
| IF-09 | 模型部署 | Mac | VisionAI Edge | SSH/SCP |

---

## 2. 介面規格詳細

### 2.1 IF-01: 攝影機串流介面

#### 2.1.1 基本資訊

| 項目 | 內容 |
|------|------|
| 介面類型 | 視訊串流 |
| 協定 | RTSP over TCP |
| 編碼 | H.264 / H.265 |
| 解析度 | 1920×1080 |
| 幀率 | 30 FPS |

#### 2.1.2 連線格式

```
rtsp://{username}:{password}@{ip_address}:{port}/{stream_path}
```

**範例**:
```
rtsp://admin:password@192.168.1.11:554/stream1
```

#### 2.1.3 攝影機清單

| 攝影機 ID | IP 位址 | 連接 Edge | 工站 |
|----------|---------|-----------|------|
| CAM-A01 | 192.168.1.11 | MIC730AI-001 | 組裝線 A |
| CAM-A02 | 192.168.1.12 | MIC730AI-001 | 組裝線 A |
| CAM-A03 | 192.168.1.13 | MIC730AI-001 | 組裝線 A |
| CAM-B01 | 192.168.1.21 | MIC730AI-002 | 組裝線 B |
| ... | ... | ... | ... |

---

### 2.2 IF-02: 推論結果介面 (MQTT)

#### 2.2.1 基本資訊

| 項目 | 內容 |
|------|------|
| 介面類型 | 訊息佇列 |
| 協定 | MQTT v3.1.1 |
| QoS | 1 (At least once) |
| Broker | WISE-IoTSuite MQTT Broker |
| Port | 1883 (TCP) / 8883 (TLS) |

#### 2.2.2 Topic 設計

```
edge/{edge_id}/inference
```

**範例**:
```
edge/MIC730AI-001/inference
edge/MIC720AI-003/inference
```

#### 2.2.3 訊息格式

```json
{
  "edge_id": "MIC730AI-001",
  "camera_id": "CAM-A01",
  "timestamp": "2024-01-15T10:30:45.123Z",
  "frame_id": 12345,
  "detections": [
    {
      "class": "person",
      "class_id": 0,
      "confidence": 0.95,
      "bbox": [120, 80, 450, 520],
      "track_id": 1
    },
    {
      "class": "hand",
      "class_id": 15,
      "confidence": 0.88,
      "bbox": [200, 150, 280, 220],
      "action": "picking"
    }
  ],
  "scene_type": "operation",
  "inference_time_ms": 45,
  "metadata": {
    "model_version": "yolo11n-v1.0",
    "gpu_usage": 65,
    "temperature": 52
  }
}
```

#### 2.2.4 欄位說明

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| edge_id | string | Y | Edge 設備 ID |
| camera_id | string | Y | 攝影機 ID |
| timestamp | string | Y | ISO 8601 時間戳記 |
| frame_id | integer | Y | 影格序號 |
| detections | array | Y | 偵測結果陣列 |
| detections[].class | string | Y | 物件類別名稱 |
| detections[].class_id | integer | Y | 物件類別 ID |
| detections[].confidence | float | Y | 置信度 (0-1) |
| detections[].bbox | array[4] | Y | 邊界框 [x1,y1,x2,y2] |
| detections[].track_id | integer | N | 追蹤 ID |
| detections[].action | string | N | 動作類別 |
| scene_type | string | Y | 場景類型 |
| inference_time_ms | integer | Y | 推論耗時 (ms) |
| metadata | object | N | 附加資訊 |

#### 2.2.5 scene_type 列舉

| 值 | 說明 |
|----|------|
| idle | 空閒（無人/無動作） |
| preparation | 準備中 |
| operation | 作業中 |
| inspection | 檢查中 |
| transition | 過渡/移動 |
| anomaly | 異常 |

---

### 2.3 IF-03: 設備狀態介面 (MQTT)

#### 2.3.1 Topic 設計

```
edge/{edge_id}/status
```

#### 2.3.2 訊息格式

```json
{
  "edge_id": "MIC730AI-001",
  "timestamp": "2024-01-15T10:30:00.000Z",
  "status": "online",
  "system": {
    "cpu_usage": 45.2,
    "memory_usage": 68.5,
    "disk_usage": 32.1,
    "uptime_seconds": 86400
  },
  "gpu": {
    "gpu_usage": 65.0,
    "memory_used": 4096,
    "memory_total": 8192,
    "temperature": 52
  },
  "cameras": [
    {
      "camera_id": "CAM-A01",
      "status": "active",
      "fps": 28.5,
      "resolution": "1920x1080"
    },
    {
      "camera_id": "CAM-A02",
      "status": "active",
      "fps": 29.1,
      "resolution": "1920x1080"
    }
  ],
  "inference": {
    "model_version": "yolo11n-v1.0",
    "avg_inference_ms": 45,
    "total_frames_processed": 1234567
  }
}
```

#### 2.3.3 發送頻率

| 項目 | 頻率 |
|------|------|
| 正常狀態 | 每 60 秒 |
| 狀態變更 | 即時 |

---

### 2.4 IF-04: 告警事件介面 (MQTT)

#### 2.4.1 Topic 設計

```
edge/{edge_id}/alert
```

#### 2.4.2 訊息格式

```json
{
  "edge_id": "MIC730AI-001",
  "alert_id": "ALT-20240115-001",
  "timestamp": "2024-01-15T10:30:45.123Z",
  "severity": "warning",
  "category": "anomaly_detection",
  "title": "異常動作偵測",
  "description": "CAM-A01 偵測到非預期的動作模式",
  "details": {
    "camera_id": "CAM-A01",
    "frame_id": 12345,
    "detected_action": "unexpected_movement",
    "confidence": 0.82
  },
  "snapshot_url": "https://storage.example.com/snapshots/12345.jpg"
}
```

#### 2.4.3 severity 列舉

| 值 | 說明 | 處理方式 |
|----|------|---------|
| info | 資訊 | 記錄 |
| warning | 警告 | 記錄 + 通知 |
| error | 錯誤 | 記錄 + 立即通知 |
| critical | 嚴重 | 記錄 + 立即通知 + 告警 |

#### 2.4.4 category 列舉

| 值 | 說明 |
|----|------|
| anomaly_detection | 異常偵測 |
| device_error | 設備錯誤 |
| camera_disconnect | 攝影機斷線 |
| high_temperature | 溫度過高 |
| inference_timeout | 推論逾時 |

---

### 2.5 IF-05: 時序資料查詢介面 (REST)

#### 2.5.1 基本資訊

| 項目 | 內容 |
|------|------|
| 介面類型 | RESTful API |
| 協定 | HTTPS |
| Base URL | https://{iotsuite_host}/api/v1 |
| 認證 | Bearer Token |

#### 2.5.2 API: 查詢推論結果

**Endpoint**: `GET /timeseries/query`

**Request Headers**:
```
Authorization: Bearer {api_token}
Content-Type: application/json
```

**Request Parameters**:
| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| measurement | string | Y | 量測名稱 |
| start | string | Y | 開始時間 (ISO 8601) |
| end | string | Y | 結束時間 (ISO 8601) |
| tags | object | N | 篩選條件 |
| limit | integer | N | 筆數限制 (預設 1000) |

**Request Example**:
```http
GET /api/v1/timeseries/query?measurement=vision_inference&start=2024-01-15T09:00:00Z&end=2024-01-15T10:00:00Z&tags[edge_id]=MIC730AI-001
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Response Example**:
```json
{
  "success": true,
  "data": [
    {
      "timestamp": "2024-01-15T09:30:45.123Z",
      "edge_id": "MIC730AI-001",
      "camera_id": "CAM-A01",
      "scene_type": "operation",
      "person_count": 1,
      "inference_time_ms": 45
    },
    {
      "timestamp": "2024-01-15T09:30:45.156Z",
      "edge_id": "MIC730AI-001",
      "camera_id": "CAM-A01",
      "scene_type": "operation",
      "person_count": 1,
      "inference_time_ms": 43
    }
  ],
  "meta": {
    "total": 3600,
    "limit": 1000,
    "offset": 0
  }
}
```

---

### 2.6 IF-06: 分析結果回寫介面 (REST)

#### 2.6.1 API: 寫入分析結果

**Endpoint**: `POST /timeseries/write`

**Request Headers**:
```
Authorization: Bearer {api_token}
Content-Type: application/json
```

**Request Body**:
```json
{
  "measurement": "process_analysis",
  "tags": {
    "video_id": "abc123",
    "work_station": "A01"
  },
  "fields": {
    "cycle_time": 45.2,
    "segment_count": 5,
    "efficiency_rate": 0.92
  },
  "timestamp": "2024-01-15T10:30:00.000Z"
}
```

**Response Example**:
```json
{
  "success": true,
  "message": "Data written successfully"
}
```

---

### 2.7 IF-07: 設備管理介面 (REST)

#### 2.7.1 API: 取得設備列表

**Endpoint**: `GET /devices`

**Response Example**:
```json
{
  "success": true,
  "data": [
    {
      "device_id": "MIC730AI-001",
      "name": "Edge-A01",
      "model": "MIC-730AI",
      "status": "online",
      "ip_address": "192.168.1.101",
      "last_seen": "2024-01-15T10:30:00.000Z"
    },
    {
      "device_id": "MIC720AI-003",
      "name": "Edge-C01",
      "model": "MIC-720AI",
      "status": "online",
      "ip_address": "192.168.1.103",
      "last_seen": "2024-01-15T10:30:00.000Z"
    }
  ]
}
```

#### 2.7.2 API: 取得設備遙測資料

**Endpoint**: `GET /devices/{device_id}/telemetry`

**Response Example**:
```json
{
  "success": true,
  "data": {
    "device_id": "MIC730AI-001",
    "timestamp": "2024-01-15T10:30:00.000Z",
    "cpu_usage": 45.2,
    "memory_usage": 68.5,
    "gpu_usage": 65.0,
    "temperature": 52,
    "uptime": 86400
  }
}
```

---

### 2.8 IF-08: DataInsight 跨源查詢介面 (REST)

#### 2.8.1 API: 執行 SQL 查詢

**Endpoint**: `POST /datainsight/query`

**Request Body**:
```json
{
  "sql": "SELECT p.work_order, p.product_name, v.cycle_time FROM mes.production p LEFT JOIN vision.analysis v ON p.work_order = v.work_order WHERE p.date = :date",
  "params": {
    "date": "2024-01-15"
  }
}
```

**Response Example**:
```json
{
  "success": true,
  "data": [
    {
      "work_order": "WO-20240115-001",
      "product_name": "Product A",
      "cycle_time": 45.2
    },
    {
      "work_order": "WO-20240115-002",
      "product_name": "Product B",
      "cycle_time": 42.8
    }
  ],
  "meta": {
    "execution_time_ms": 125,
    "row_count": 2
  }
}
```

---

### 2.9 IF-09: 模型部署介面 (SSH/SCP)

#### 2.9.1 基本資訊

| 項目 | 內容 |
|------|------|
| 協定 | SSH / SCP |
| Port | 22 |
| 認證 | SSH Key |

#### 2.9.2 部署流程

```bash
# 1. 上傳模型檔案
scp model.engine admin@192.168.1.101:/opt/models/

# 2. 更新模型配置
ssh admin@192.168.1.101 "echo 'MODEL_PATH=/opt/models/model.engine' >> /opt/config/inference.conf"

# 3. 重啟推論服務
ssh admin@192.168.1.101 "sudo systemctl restart ai-inference"

# 4. 驗證部署
ssh admin@192.168.1.101 "curl -s http://localhost:8080/health"
```

#### 2.9.3 模型檔案格式

| 格式 | 引擎 | 說明 |
|------|------|------|
| .engine | TensorRT | NVIDIA GPU 最佳化 |
| .onnx | ONNX Runtime | 通用格式 |
| .pt | PyTorch | 開發測試用 |

---

## 3. 安全性規格

### 3.1 網路安全

| 項目 | 規格 |
|------|------|
| OT/IT 隔離 | 防火牆 + NAT |
| 傳輸加密 | TLS 1.3 |
| MQTT 加密 | TLS (Port 8883) |
| API 認證 | Bearer Token |

### 3.2 防火牆規則

| 來源 | 目的 | Port | 協定 | 動作 |
|------|------|------|------|------|
| VisionAI Edge | IoTSuite MQTT | 8883 | TCP | Allow |
| Mac | IoTSuite API | 443 | TCP | Allow |
| Mac | VisionAI Edge | 22 | TCP | Allow |
| Any | Any | Any | Any | Deny |

### 3.3 認證機制

| 介面 | 認證方式 |
|------|---------|
| MQTT | Username/Password + TLS |
| REST API | Bearer Token (JWT) |
| SSH | SSH Key |

---

## 4. 錯誤處理

### 4.1 MQTT 錯誤處理

| 錯誤 | 處理方式 |
|------|---------|
| Broker 連線失敗 | 自動重連 (指數退避) |
| 訊息發送失敗 | 本地快取 + 重試 |
| Topic 訂閱失敗 | 記錄錯誤 + 告警 |

### 4.2 REST API 錯誤代碼

| HTTP Code | 錯誤類型 | 說明 |
|-----------|---------|------|
| 400 | Bad Request | 請求格式錯誤 |
| 401 | Unauthorized | 認證失敗 |
| 403 | Forbidden | 權限不足 |
| 404 | Not Found | 資源不存在 |
| 429 | Too Many Requests | 請求頻率過高 |
| 500 | Internal Server Error | 伺服器錯誤 |
| 503 | Service Unavailable | 服務不可用 |

### 4.3 錯誤回應格式

```json
{
  "success": false,
  "error": {
    "code": "AUTH_001",
    "message": "Invalid or expired token",
    "details": "Token expired at 2024-01-15T10:30:00Z"
  }
}
```

---

## 5. 效能規格

### 5.1 MQTT 效能

| 指標 | 規格 |
|------|------|
| 訊息大小上限 | 256 KB |
| 發送頻率上限 | 30 msg/sec/edge |
| 延遲 | < 100ms (p99) |

### 5.2 REST API 效能

| 指標 | 規格 |
|------|------|
| 回應時間 | < 500ms (p95) |
| 並發連線 | 100 |
| 請求頻率上限 | 100 req/sec |

---

## 6. 附錄

### 6.1 Edge 設備清單

| Edge ID | 型號 | IP 位址 | 連接攝影機 |
|---------|------|---------|-----------|
| MIC730AI-001 | MIC-730AI | 192.168.1.101 | CAM-A01~A06 |
| MIC730AI-002 | MIC-730AI | 192.168.1.102 | CAM-B01~B06 |
| MIC720AI-003 | MIC-720AI | 192.168.1.103 | CAM-C01~C04 |
| MIC720AI-004 | MIC-720AI | 192.168.1.104 | CAM-D01~D04 |

### 6.2 YOLO 類別對照表

| Class ID | Class Name | 說明 |
|----------|------------|------|
| 0 | person | 人員 |
| 1 | hand | 手部 |
| 2 | tool | 工具 |
| 3 | product | 產品 |
| 4 | container | 容器 |
| ... | ... | ... |

### 6.3 API Token 申請流程

1. 登入 WISE-IoTSuite 管理介面
2. 進入「API 管理」頁面
3. 點擊「建立 API Token」
4. 設定 Token 名稱與權限
5. 複製 Token 並妥善保存

---

## 簽核

| 角色 | 姓名 | 簽名 | 日期 |
|------|------|------|------|
| 撰寫人 | | | |
| 審核人 | | | |
| 核准人 | | | |
