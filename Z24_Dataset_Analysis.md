# Z24 橋樑結構健康監測資料集完整分析報告

## 📋 目錄

- [一、資料集概述](#一資料集概述)
- [二、資料夾結構與檔案類型](#二資料夾結構與檔案類型)
- [三、資料檔案格式詳解](#三資料檔案格式詳解)
- [四、可用資料與參數](#四可用資料與參數)
- [五、動態模態形狀與頻率響應函數](#五動態模態形狀與頻率響應函數)
- [六、資料使用方法](#六資料使用方法)
- [七、資料集特點](#七資料集特點)
- [八、應用建議](#八應用建議)
- [九、注意事項](#九注意事項)

---

## 一、資料集概述

**Z24 橋樑結構健康監測資料集**是一個完整的結構動力學測試資料集，包含以下主要內容：

| 測試類型 | 縮寫 | 說明 |
|---------|------|------|
| 漸進損傷測試 | PDT | Progressive Damage Tests |
| 環境振動測試 | AVT | Ambient Vibration Tests |
| 強迫振動測試 | FVT | Forced Vibration Tests |
| 環境監測系統 | EMS | Environmental Monitoring System |

---

## 二、資料夾結構與檔案類型

### 2.1 PDT 測試資料

**資料夾位置：**
- `pdt_01-08/` - 測試 01 至 08
- `pdt_09-17/` - 測試 09 至 17

**結構說明：**

```
pdt_XX/
├── avt/          # 環境振動測試
│   ├── XXsetup01.mat
│   ├── XXsetup02.mat
│   ├── ...
│   └── XXsetup09.mat
└── fvt/          # 強迫振動測試
    ├── XXsetup01.mat
    ├── XXsetup02.mat
    ├── ...
    └── XXsetup09.mat
```

**檔案格式：**

| 檔案類型 | 副檔名 | 說明 |
|---------|--------|------|
| Matlab 資料 | `.mat` | 原始時間資料（Matlab 格式） |
| ASCII 時間歷程 | `.aaa` | 文字格式的時間歷程資料（部分測試） |
| 測試說明文件 | `.DOC` | 測試配置說明文件（二進制格式） |

### 2.2 EMS 環境監測資料

**資料夾位置：**
- `Z24ems1/` - 第一組環境監測資料
- `Z24ems2/` - 第二組環境監測資料
- `Z24ems3/` - 第三組環境監測資料

**結構說明：**

```
Z24emsX/
└── XXYYZZ/       # 測量點編號（網格座標）
    ├── XXYYZZ03.aaa
    ├── XXYYZZ05.aaa
    ├── ...
    ├── XXYYZZcar.aaa
    ├── XXYYZZPRE.env   # 測試前環境資料
    └── XXYYZZPOS.env   # 測試後環境資料
```

**測量點命名規則：**
- 格式：`XXYYZZ`（如 `01C14`、`12B22`）
- `XX`：行號（01-36）
- `YY`：列號（A-G）
- `ZZ`：子位置（00-23）

---

## 三、資料檔案格式詳解

### 3.1 `.mat` 檔案（Matlab 格式）

根據 `readme.txt` 說明：

> Z24 PDT tests.  
> The original files have been compressed to Matlab compatible format.  
> Each *.mat file containes a variable 'data' with the raw time data (each column corresponds to a channel),  
> and a variable 'labelshulp' which contains the channel information.  
> The sampling frequency is always 100 Hz.

**包含的變數：**

| 變數名稱 | 類型 | 說明 |
|---------|------|------|
| `data` | 矩陣 | 原始時間資料，每列對應一個通道 |
| `labelshulp` | 結構/字串 | 通道資訊標籤 |
| 採樣頻率 | 固定值 | **100 Hz** |

### 3.2 `.aaa` 檔案（時間歷程資料）

**檔案結構：**

| 行號 | 內容 | 說明 |
|-----|------|------|
| 1 | 檔案名稱和平均次數 | 例如：`100V.aaa\|8Averages` |
| 2 | 樣本數量 | 通常為 `65536` |
| 3 | 時間間隔 | 通常為 `0.01` 秒（對應 100 Hz） |
| 4 開始 | 加速度數值 | 單位：**g**（重力加速度） |
| 結尾 | 元數據資訊 | 包含採集參數和統計資訊 |

**元數據包含的資訊：**

| 參數 | 說明 |
|------|------|
| 單位 | g（重力加速度） |
| 感測器靈敏度 | 1 V/g |
| 平均次數 | 通常為 8 |
| 段（Segments）資訊 | 每個段包含：<br>- 開始/結束時間<br>- 採集時間（約 82 秒）<br>- 前濾波增益（Prefilter Gain）<br>- 後濾波增益（Postfilter Gain）<br>- 最小值/最大值/平均值/標準差<br>- ADC 使用位數（動態範圍） |

**範例檔案結尾：**

```
Timehistories end here
Additinal Information:
Samples are in g's
Sensor sensitivity: 1 [V/g]
Number of Averages : 8
Segment #1 Start :Mon Nov 10 14:01:53 1997
 Stop :Mon Nov 10 14:03:15 1997
 Acquisition time : 82.398
Number of Restarts : 0 Number of autoranges : 0
Prefilter Gain: 316.228 Postfilter Gain: 2
Minimum value : -0.00113737 Maximum Value : 0.00132078 
Mean Value : 3.16228e-06 Std Deviation : 0.000131662
ADC usage : 11 Bits dynamic range
```

### 3.3 `.env` 檔案（環境監測資料）

**包含的環境參數：**

| 參數代碼 | 單位 | 說明 |
|---------|------|------|
| `WS` | [V] | 風速 |
| `WD` | [V] | 風向 |
| `AT` | [V] | 環境溫度 |
| `R` | [V] | 降雨量 |
| `H` | [V] | 濕度 |
| `TE` | [V] | 溫度 |
| `ADU` | [V] | 其他環境參數 |
| `ADK` | [V] | 其他環境參數 |

**溫度感測器（多個測點）：**

| 感測器類型 | 數量 | 說明 |
|-----------|------|------|
| `TSPU1-3` | 3 | 上部結構溫度感測器 |
| `TSAU1-3` | 3 | 上部結構環境溫度感測器 |
| `TSPK1-3` | 3 | 下部結構溫度感測器 |
| `TSAK1-3` | 3 | 下部結構環境溫度感測器 |
| `TBC1-2` | 2 | 橋樑基礎溫度感測器 |
| `TSWS1-2` | 2 | 西南側溫度感測器 |
| `TSWN1-2` | 2 | 西北側溫度感測器 |
| `TWS1-3` | 3 | 西側溫度感測器 |
| `TWC1-3` | 3 | 西中側溫度感測器 |
| `TWN1-3` | 3 | 西北側溫度感測器 |
| `TP1-3` | 3 | 點溫度感測器 |
| `TDT1-3` | 3 | 數據溫度感測器 |
| `TDS1-3` | 3 | 數據系統溫度感測器 |
| `TS1-3` | 3 | 系統溫度感測器 |

**檔案格式：**
- 第一行：參數標題（空格分隔）
- 後續行：時間序列資料（每行對應一個時間點）
- 檔案結尾：包含採集時間和掃描次數資訊

---

## 四、可用資料與參數

### 4.1 時間歷程資料（Time History Data）

**用途：**
- 結構動力響應分析
- 模態參數識別
- 損傷檢測
- 頻率響應分析

**技術參數：**

| 參數 | 數值/說明 |
|------|----------|
| 採樣頻率 | **100 Hz**（fs = 1 / deltaT = 1 / 0.01 = 100 Hz） |
| 時間間隔 | **0.01 秒**（deltaT） |
| 奈奎斯特頻率 | **50 Hz**（f_nyquist = fs / 2 = 100 / 2 = 50 Hz） |
| 樣本長度 | 通常 **65536** 點（約 655.36 秒） |
| 單位 | **g**（重力加速度） |
| 通道數 | 依測試配置而定 |
| 資料格式 | 浮點數（科學記號格式） |

### 4.2 環境監測資料（Environmental Data）

**用途：**
- 環境因素對結構響應的影響分析
- 溫度補償
- 長期監測趨勢分析

**監測參數：**

| 參數類型 | 測量項目 |
|---------|---------|
| 氣象參數 | 風速、風向、降雨量 |
| 溫度參數 | 多個測點的結構溫度 |
| 環境參數 | 濕度、環境溫度 |

### 4.3 測試配置

**PDT 測試配置：**

| 測試編號 | 設置數量 | 說明 |
|---------|---------|------|
| 01-17 | 每個測試 9 個設置 | setup01 至 setup09，對應不同的測試條件或損傷狀態 |

**EMS 監測配置：**

| 監測組 | 測量點數量 | 說明 |
|-------|-----------|------|
| Z24ems1 | 大量測量點 | 第一組環境監測資料 |
| Z24ems2 | 大量測量點 | 第二組環境監測資料 |
| Z24ems3 | 大量測量點 | 第三組環境監測資料 |

---

## 五、動態模態形狀與頻率響應函數

### 5.1 模態參數識別

從時間歷程資料可以提取以下模態參數：

| 模態參數 | 說明 |
|---------|------|
| **自然頻率** | Natural Frequencies |
| **模態阻尼比** | Modal Damping Ratios |
| **模態形狀** | Mode Shapes |
| **模態參與因子** | Modal Participation Factors |

**常用的識別方法：**

| 方法 | 縮寫 | 說明 |
|------|------|------|
| 頻域分解 | FDD | Frequency Domain Decomposition |
| 隨機子空間識別 | SSI | Stochastic Subspace Identification |
| 峰值拾取法 | Peak Picking | 基於功率譜密度的簡單方法 |
| 增強頻域分解 | EFDD | Enhanced Frequency Domain Decomposition |

### 5.2 頻率響應函數（FRF）

可以計算的頻率域函數：

| 函數類型 | 說明 |
|---------|------|
| **傳遞函數** | Transfer Functions |
| **頻率響應函數** | Frequency Response Functions (FRF) |
| **功率譜密度** | Power Spectral Density (PSD) |
| **交叉功率譜密度** | Cross Power Spectral Density (CPSD) |

### 5.3 動態特性分析

可以分析的動態特性：

| 特性類型 | 說明 |
|---------|------|
| **結構剛度變化** | 通過頻率變化反映 |
| **阻尼特性** | 通過模態阻尼比反映 |
| **共振頻率偏移** | 損傷檢測的重要指標 |
| **模態形狀變化** | 損傷定位的重要指標 |

---

## 六、資料使用方法

### 6.1 讀取 `.mat` 檔案（Matlab）

```matlab
% 載入資料
load('pdt_01-08/01/avt/01setup01.mat');

% 使用資料
% data: 時間資料矩陣（每列對應一個通道）
% labelshulp: 通道資訊

% 設定採樣頻率
fs = 100; % 採樣頻率 100 Hz

% 計算時間向量
t = (0:size(data,1)-1) / fs;
```

### 6.2 讀取 `.aaa` 檔案（Python）

```python
import numpy as np

def read_aaa_file(filename):
    """
    讀取 .aaa 格式的時間歷程檔案
    
    參數:
        filename: 檔案路徑
    
    返回:
        data: 加速度資料陣列
        dt: 時間間隔（秒）
        metadata: 元數據字典
    """
    with open(filename, 'r') as f:
        lines = f.readlines()
    
    # 讀取樣本數和時間間隔
    n_samples = int(lines[1].strip())
    dt = float(lines[2].strip())
    
    # 讀取加速度資料
    data = []
    metadata_start = False
    metadata = {}
    
    for i in range(3, len(lines)):
        line = lines[i].strip()
        
        # 檢查是否到達元數據部分
        if line.startswith('Timehistories end here'):
            metadata_start = True
            continue
        
        if not metadata_start:
            # 讀取資料
            if line:
                try:
                    data.append(float(line))
                except ValueError:
                    break
        else:
            # 解析元數據
            if ':' in line:
                key, value = line.split(':', 1)
                metadata[key.strip()] = value.strip()
    
    return np.array(data), dt, metadata

# 使用範例
data, dt, metadata = read_aaa_file('Z24ems1/01C14/01C1403.aaa')
fs = 1 / dt  # 計算採樣頻率
t = np.arange(len(data)) * dt  # 時間向量
```

### 6.3 讀取 `.env` 檔案（Python）

```python
def read_env_file(filename):
    """
    讀取 .env 格式的環境監測檔案
    
    參數:
        filename: 檔案路徑
    
    返回:
        headers: 參數名稱列表
        data: 環境資料矩陣
    """
    with open(filename, 'r') as f:
        lines = f.readlines()
    
    # 第一行是標題
    headers = lines[0].split()
    
    # 讀取資料
    data = []
    for line in lines[1:]:
        line = line.strip()
        if line and not line.startswith('EnvScan') and not line.startswith('Job'):
            values = line.split()
            if len(values) > 0:
                try:
                    data.append([float(v) for v in values])
                except ValueError:
                    continue
    
    return headers, np.array(data)

# 使用範例
headers, env_data = read_env_file('Z24ems1/01C14/01C14POS.env')
```

### 6.4 計算頻率響應函數（Python）

```python
from scipy import signal
import numpy as np

def calculate_frf(input_data, output_data, fs=100, nperseg=None):
    """
    計算頻率響應函數
    
    參數:
        input_data: 輸入訊號（激勵）
        output_data: 輸出訊號（響應）
        fs: 採樣頻率
        nperseg: 每段的長度（預設為資料長度的 1/4）
    
    返回:
        frequencies: 頻率向量
        frf: 頻率響應函數（複數）
    """
    if nperseg is None:
        nperseg = len(input_data) // 4
    
    # 計算交叉功率譜密度和輸入功率譜密度
    f, Pxy = signal.csd(input_data, output_data, fs, nperseg=nperseg)
    f, Pxx = signal.welch(input_data, fs, nperseg=nperseg)
    
    # 計算 FRF
    frf = Pxy / Pxx
    
    return f, frf
```

### 6.5 模態參數識別範例（Python）

```python
from scipy.signal import find_peaks
import numpy as np

def identify_modal_parameters(psd, frequencies, n_modes=5):
    """
    使用峰值拾取法識別模態參數
    
    參數:
        psd: 功率譜密度
        frequencies: 頻率向量
        n_modes: 要識別的模態數量
    
    返回:
        natural_frequencies: 自然頻率列表
        peak_indices: 峰值索引
    """
    # 尋找峰值
    peaks, properties = find_peaks(psd, height=np.max(psd) * 0.1, distance=10)
    
    # 按峰值高度排序
    peak_heights = psd[peaks]
    sorted_indices = np.argsort(peak_heights)[::-1]
    
    # 選擇前 n_modes 個峰值
    selected_peaks = peaks[sorted_indices[:n_modes]]
    natural_frequencies = frequencies[selected_peaks]
    
    return natural_frequencies, selected_peaks
```

---

## 七、資料集特點

| 特點 | 說明 |
|------|------|
| **多測試條件** | 17 個 PDT 測試，每個有 9 個設置 |
| **多測量點** | EMS 系統覆蓋大量測量位置 |
| **長期監測** | 包含環境監測資料，可進行長期趨勢分析 |
| **損傷漸進** | PDT 測試模擬漸進損傷過程 |
| **多種激勵** | 包含環境振動和強迫振動兩種激勵方式 |
| **完整記錄** | 包含時間歷程、環境參數、測試配置等完整資訊 |

---

## 八、應用建議

### 8.1 主要應用領域

| 應用領域 | 說明 |
|---------|------|
| **結構健康監測（SHM）** | 長期監測結構狀態變化 |
| **損傷檢測與定位** | 識別結構損傷位置和程度 |
| **模態參數識別** | 提取結構動態特性參數 |
| **環境影響分析** | 分析環境因素對結構響應的影響 |
| **長期性能評估** | 評估結構長期性能變化 |
| **機器學習與模式識別** | 使用 ML 方法進行損傷識別和預測 |

### 8.2 研究建議

1. **損傷檢測算法開發**
   - 基於頻率變化的損傷檢測
   - 基於模態形狀變化的損傷定位
   - 基於機器學習的損傷識別

2. **環境影響研究**
   - 溫度對結構頻率的影響
   - 環境因素補償方法
   - 長期監測資料分析

3. **模態識別方法比較**
   - 比較不同模態識別方法的準確性
   - 評估不同方法的適用性

---

## 九、注意事項

### 9.1 資料處理注意事項

| 注意事項 | 說明 |
|---------|------|
| **採樣頻率** | 所有資料統一為 **100 Hz** |
| **資料單位** | 時間歷程資料單位為 **g**（重力加速度） |
| **多段資料** | 部分檔案包含多個段（segments），需分別處理 |
| **時間同步** | 環境監測資料需與振動資料時間同步 |
| **檔案格式** | DOC 檔案為二進制格式，可能需要特定軟體讀取 |

### 9.2 資料品質檢查

建議在處理資料前進行以下檢查：

1. **資料完整性檢查**
   - 檢查檔案是否完整
   - 檢查資料長度是否符合預期

2. **異常值檢測**
   - 檢查是否有異常的數值
   - 檢查是否有缺失資料

3. **採樣頻率驗證**
   - 確認採樣頻率為 100 Hz
   - 檢查時間間隔是否正確

4. **通道一致性**
   - 檢查不同通道的資料是否同步
   - 檢查通道標籤是否正確

---

## 十、參考資料

### 10.1 檔案說明

- `readme.txt` - 基本資料格式說明
- `C11-Appendices.pdf` - 附錄文件（可能包含詳細說明）
- `Z24-EMS-readme.pdf` - EMS 系統說明文件

### 10.2 資料集資訊

- **測試對象**：Z24 橋樑
- **測試時間**：1997-1998 年
- **資料格式**：Matlab (.mat)、ASCII (.aaa)、環境資料 (.env)
- **採樣頻率**：100 Hz
- **資料單位**：加速度 (g)

---

## 📝 更新記錄

- **2024** - 初始分析報告建立

---

## 📧 聯絡資訊

如有任何問題或建議，請透過 GitHub Issues 提出。

---

**注意：** 本報告基於對 Z24 資料集檔案結構的分析，實際使用時請參考原始文件說明。

