# Hour 6：系統整合、效能瓶頸與 AIoT 設計方法
（課程時間：60 分鐘）

---

## 一、本小時課程目標（Learning Objectives）

完成本小時課程後，學生應能：

- 從「功能實作」提升到「**系統整合與設計思維**」
- 能分析 AIoT 系統在 MCU 上的 **效能瓶頸**
- 理解 CPU / Memory / Task / FPS 之間的關係
- 看懂專案中與效能、整合相關的重要程式碼
- 能將本課架構 **遷移到其他 AIoT 應用場景**

---

## 二、為什麼最後一小時談「整合」？

前五小時我們已經學會：

- AIoT 是什麼（Hour 1）
- 系統架構與事件流（Hour 2）
- 資料流與 Camera Pipeline（Hour 3）
- Edge AI 與模型部署（Hour 4）
- UI、事件、決策與儲存（Hour 5）

但真實世界的 AIoT 專案，**80% 的問題出現在「整合」階段**。

---

## 三、AIoT 系統的四大資源

在 MCU 上，所有問題最終都會回到：

1. **CPU**
2. **Memory（SRAM / PSRAM / Flash）**
3. **Task / Thread**
4. **I/O（Camera / Display / SD）**

本小時將用實際專案說明如何平衡這四件事。

---

## 四、CPU：誰在吃時間？

### 4.1 主要 CPU 消耗來源

在本專案中，CPU 時間主要被：

- Camera Pipeline
- AI Inference
- UI Rendering
- Storage I/O

---

### 4.2 AI 推論時間的影響因素

影響 AI 推論速度的不是只有模型：

- Input Resolution（640 vs 320）
- Preprocess
- Postprocess
- 記憶體存取速度

> 教學重點：**FPS ≠ 模型速度**

---

## 五、Memory：為什麼 P4 才能跑得動？

### 5.1 記憶體種類回顧

| 類型 | 特性 |
|---|---|
| SRAM | 快、少 |
| PSRAM | 大、較慢 |
| Flash | 最慢、用來存檔 |

---

### 5.2 模型與記憶體的關係

**相關程式碼位置：**
- `components/coco_detect/coco_detect.cpp`

```cpp
dl::Model(..., param_copy);
```

- `param_copy = true`：模型參數拷貝到 PSRAM
- `param_copy = false`：直接從 Flash 存取

> 教學重點：速度與記憶體的 trade-off

---

### 5.3 minimize() 的實際效果

```cpp
m_model->minimize();
```

- 釋放初始化 buffer
- 降低 peak memory

這一步常被忽略，但對 MCU 極為重要。

---

## 六、Task：誰在背景跑？

### 6.1 本專案的主要任務

- Video Stream Task
- AI Detect Task
- UI Task
- IMU Task
- Control Task

---

### 6.2 任務設計原則

- AI 不可阻塞 UI
- I/O 不可阻塞 AI
- UI 必須即時回應

> 任務分離是穩定系統的關鍵

---

## 七、效能瓶頸實例分析

### 7.1 問題情境

「畫面卡、AI 偵測慢」

### 7.2 可能原因

- 解析度過高
- 模型過大
- Storage 同時寫入
- UI 與 AI 在同一 task

---

### 7.3 解法方向

- 降低 input size
- 使用 320x320 模型
- 延後寫入 SD
- 調整 task priority

---

## 八、模型生命週期回顧（總整理）

### 8.1 模型檔案類型總覽

| 階段 | 檔案 | 來源 |
|---|---|---|
| 訓練 | `.pt` | PyTorch |
| 交換 | `.onnx` | export_onnx.py |
| 部署 | `.espdl` | esp-dl toolchain |

---

### 8.2 ONNX 生成方式回顧

**工具：**
- PyTorch
- YOLOv11

**腳本：**
- `export_onnx.py`

**指令範例：**

```bash
python export_onnx.py   --weights yolo11n.pt   --imgsz 320   --output yolo11n_320.onnx
```

---

## 九、從範例到產品：如何套用這個架構？

### 9.1 可遷移的設計元素

- Video Stream 中樞
- AI Service Layer（COCODetect）
- UI 狀態機
- Event-driven Control
- 分層 Storage 設計

---

### 9.2 可應用場景

- 智慧攝影機
- 智慧門鈴
- 工業視覺檢測
- 校園 / 醫療監控

---

## 十、課堂總練習（15 分鐘）

請學生設計一個 AIoT 產品，並回答：

1. 感測器是什麼？
2. 哪些 AI 在 Edge？
3. 哪些資料上雲？
4. UI 怎麼設計？
5. 哪裡可能成為瓶頸？

---

## 十一、本小時重點整理（也是全課總結）

- AIoT 的難點在「整合」
- 效能瓶頸永遠存在，只能取捨
- 好的架構讓 AI、UI、硬體共存
- 本專案是一個完整 AIoT Reference Design

---

## 十二、課程結語

如果你能：

- 看懂這個專案
- 解釋每一層的責任
- 知道效能瓶頸在哪

那你已經具備 **AIoT 系統工程師** 的基本能力。

---

（6 小時 AIoT 課程結束）
