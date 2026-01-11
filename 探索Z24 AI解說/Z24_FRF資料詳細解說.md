# Z24資料集FRF（頻率響應函數）資料詳細解說

## 📋 目錄

- [一、FRF概述](#一frf概述)
- [二、Z24資料集中FRF相關資料](#二z24資料集中frf相關資料)
- [三、FRF計算所需的原始資料](#三frf計算所需的原始資料)
- [四、FRF計算方法](#四frf計算方法)
- [五、資料格式與結構](#五資料格式與結構)
- [六、FRF計算實作範例](#六frf計算實作範例)
- [七、FRF在損傷檢測中的應用](#七frf在損傷檢測中的應用)
- [八、注意事項與建議](#八注意事項與建議)

---

## 一、FRF概述

### 1.1 什麼是FRF（頻率響應函數）？

**頻率響應函數（Frequency Response Function, FRF）**是描述線性系統在頻域中輸入與輸出關係的函數。在結構動力學中，FRF表示結構在特定頻率下的響應特性。

### 1.2 FRF的數學定義

對於單輸入單輸出（SISO）系統：

\[
H(\omega) = \frac{Y(\omega)}{X(\omega)} = \frac{S_{XY}(\omega)}{S_{XX}(\omega)}
\]

其中：
- \(H(\omega)\)：頻率響應函數
- \(Y(\omega)\)：輸出信號的傅立葉變換
- \(X(\omega)\)：輸入信號的傅立葉變換
- \(S_{XY}(\omega)\)：輸入-輸出交叉功率譜密度
- \(S_{XX}(\omega)\)：輸入功率譜密度

### 1.3 FRF的重要性

FRF在結構健康監測中的應用：

| 應用領域 | 說明 |
|---------|------|
| **模態參數識別** | 從FRF中提取自然頻率、模態阻尼比、模態形狀 |
| **損傷檢測** | 比較健康與損傷狀態的FRF差異 |
| **結構特性分析** | 評估結構剛度、阻尼、質量分佈 |
| **模型驗證** | 驗證有限元模型的準確性 |

---

## 二、Z24資料集中FRF相關資料

### 2.1 資料集概述

**重要發現：Z24資料集本身不包含預先計算的FRF資料，而是提供計算FRF所需的原始時間序列資料。**

### 2.2 可用於FRF計算的資料類型

#### 2.2.1 環境振動測試（AVT）資料

**位置：** `pdt_XX/avt/` 和 `Z24emsX/`

**資料格式：**
- `.mat` 檔案：Matlab格式的多通道時間序列資料
- `.aaa` 檔案：ASCII格式的單通道時間序列資料

**特點：**
- ✅ 只有輸出響應資料（無已知輸入）
- ✅ 使用環境激勵（風、交通等）
- ✅ 適合使用輸出-輸出方法計算FRF

#### 2.2.2 強迫振動測試（FVT）資料

**位置：** `pdt_XX/fvt/`

**資料格式：**
- `.mat` 檔案：Matlab格式的多通道時間序列資料

**特點：**
- ✅ 包含輸入（激勵）和輸出（響應）資料
- ✅ 使用已知的激勵信號
- ✅ 可直接計算傳統的輸入-輸出FRF

### 2.3 資料統計

| 資料類型 | 檔案數量 | 採樣頻率 | 樣本長度 | 單位 |
|---------|---------|---------|---------|------|
| **PDT測試** | 每個測試9個設置 | 100 Hz | 依檔案而定 | g（重力加速度） |
| **EMS監測** | 大量測量點 | 100 Hz | 通常65536點 | g（重力加速度） |
| **詳細測試** | 案例06和17有291-309個檔案 | 100 Hz | 65536點 | g（重力加速度） |

---

## 三、FRF計算所需的原始資料

### 3.1 資料格式詳解

#### 3.1.1 `.mat` 檔案格式

**檔案位置範例：**
```
pdt_01-08/01/avt/01setup01.mat
pdt_01-08/01/fvt/01setup01.mat
```

**包含的變數：**
- `data`：時間資料矩陣
  - 每**列**（row）對應一個時間點
  - 每**行**（column）對應一個測量通道
  - 單位：g（重力加速度）
- `labelshulp`：通道資訊標籤
  - 包含每個通道的測量位置資訊

**技術參數：**
- 採樣頻率：**100 Hz**（fs = 1 / deltaT = 1 / 0.01 = 100 Hz，固定）
- 時間間隔：**0.01 秒**（deltaT）
- 奈奎斯特頻率：**50 Hz**（f_nyquist = fs / 2 = 100 / 2 = 50 Hz，最大可解析頻率）

#### 3.1.2 `.aaa` 檔案格式

**檔案位置範例：**
```
Z24ems1/01D00/01D0003.aaa
pdt_01-08/06/avt/01/100V.aaa
```

**檔案結構：**

```
行號  內容                    說明
1     100V.aaa|8Averages      檔案名稱和平均次數
2     65536                   樣本數量
3     0.010000                時間間隔（秒）
4+    加速度數值             時間序列資料（單位：g）
...   ...                     ...
結尾  元數據資訊              包含採集參數
```

**元數據包含：**
- 單位：g（重力加速度）
- 感測器靈敏度：1 V/g
- 平均次數：通常為8
- 採集時間：約82.398秒
- 前濾波增益和後濾波增益
- 統計資訊（最小值、最大值、平均值、標準差）

### 3.2 資料讀取方法

#### 3.2.1 讀取`.mat`檔案（Matlab）

```matlab
% 載入資料
load('pdt_01-08/01/avt/01setup01.mat');

% 資料結構
% data: 時間資料矩陣（每列對應一個時間點，每行對應一個通道）
% labelshulp: 通道資訊

% 設定參數
fs = 100;  % 採樣頻率（Hz）
dt = 1/fs; % 時間間隔（秒）
N = size(data, 1);  % 樣本數量
t = (0:N-1) * dt;    % 時間向量

% 提取特定通道的資料
channel_1 = data(:, 1);  % 第一個通道
channel_2 = data(:, 2);  % 第二個通道
```

#### 3.2.2 讀取`.mat`檔案（Python）

```python
import scipy.io as sio
import numpy as np

# 載入資料
mat_data = sio.loadmat('pdt_01-08/01/avt/01setup01.mat')

# 提取資料
data = mat_data['data']  # 時間資料矩陣
labelshulp = mat_data['labelshulp']  # 通道資訊

# 設定參數
fs = 100  # 採樣頻率（Hz）
dt = 1/fs  # 時間間隔（秒）
N = data.shape[0]  # 樣本數量
t = np.arange(N) * dt  # 時間向量

# 提取特定通道的資料
channel_1 = data[:, 0]  # 第一個通道
channel_2 = data[:, 1]  # 第二個通道
```

#### 3.2.3 讀取`.aaa`檔案（Python）

```python
import numpy as np

def read_aaa_file(filename):
    """
    讀取.aaa格式的時間歷程檔案
    
    參數:
        filename: 檔案路徑
    
    返回:
        data: 加速度資料陣列
        dt: 時間間隔（秒）
        fs: 採樣頻率（Hz）
        metadata: 元數據字典
    """
    with open(filename, 'r') as f:
        lines = f.readlines()
    
    # 讀取檔案名稱和平均次數
    header = lines[0].strip()
    
    # 讀取樣本數和時間間隔
    n_samples = int(lines[1].strip())
    dt = float(lines[2].strip())
    fs = 1.0 / dt
    
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
                parts = line.split(':', 1)
                if len(parts) == 2:
                    key = parts[0].strip()
                    value = parts[1].strip()
                    metadata[key] = value
    
    return np.array(data), dt, fs, metadata

# 使用範例
data, dt, fs, metadata = read_aaa_file('Z24ems1/01D00/01D0003.aaa')
print(f"樣本數量: {len(data)}")
print(f"採樣頻率: {fs} Hz")
print(f"時間間隔: {dt} 秒")
```

---

## 四、FRF計算方法

### 4.1 強迫振動測試（FVT）的FRF計算

**適用情況：** 有已知的輸入激勵信號

#### 4.1.1 基本方法：H1估計器

\[
H_1(\omega) = \frac{S_{XY}(\omega)}{S_{XX}(\omega)}
\]

**優點：**
- 對輸出雜訊不敏感
- 適合輸入信號雜訊較小的情況

#### 4.1.2 替代方法：H2估計器

\[
H_2(\omega) = \frac{S_{YY}(\omega)}{S_{YX}(\omega)}
\]

**優點：**
- 對輸入雜訊不敏感
- 適合輸出信號雜訊較小的情況

#### 4.1.3 實作範例（Python）

```python
import numpy as np
from scipy import signal

def calculate_frf_h1(input_signal, output_signal, fs, nperseg=None):
    """
    使用H1估計器計算FRF
    
    參數:
        input_signal: 輸入信號（激勵）
        output_signal: 輸出信號（響應）
        fs: 採樣頻率
        nperseg: 每個段的長度（預設為信號長度的1/4）
    
    返回:
        frequencies: 頻率陣列
        frf: 頻率響應函數（複數）
        coherence: 相干函數
    """
    if nperseg is None:
        nperseg = len(input_signal) // 4
    
    # 計算交叉功率譜密度和功率譜密度
    frequencies, Pxy = signal.csd(input_signal, output_signal, 
                                   fs=fs, nperseg=nperseg)
    frequencies, Pxx = signal.welch(input_signal, fs=fs, 
                                     nperseg=nperseg)
    
    # 計算FRF (H1估計器)
    frf = Pxy / Pxx
    
    # 計算相干函數
    frequencies, Pyy = signal.welch(output_signal, fs=fs, 
                                     nperseg=nperseg)
    coherence = np.abs(Pxy)**2 / (Pxx * Pyy)
    
    return frequencies, frf, coherence

# 使用範例
# 假設已載入輸入和輸出資料
frequencies, frf, coherence = calculate_frf_h1(input_data, output_data, fs=100)

# 繪製FRF幅值和相位
import matplotlib.pyplot as plt

fig, axes = plt.subplots(3, 1, figsize=(10, 8))

# 幅值
axes[0].semilogy(frequencies, np.abs(frf))
axes[0].set_xlabel('頻率 (Hz)')
axes[0].set_ylabel('FRF 幅值')
axes[0].set_title('頻率響應函數 - 幅值')
axes[0].grid(True)

# 相位
axes[1].plot(frequencies, np.angle(frf) * 180 / np.pi)
axes[1].set_xlabel('頻率 (Hz)')
axes[1].set_ylabel('相位 (度)')
axes[1].set_title('頻率響應函數 - 相位')
axes[1].grid(True)

# 相干函數
axes[2].plot(frequencies, coherence)
axes[2].set_xlabel('頻率 (Hz)')
axes[2].set_ylabel('相干函數')
axes[2].set_title('相干函數')
axes[2].set_ylim([0, 1])
axes[2].grid(True)

plt.tight_layout()
plt.show()
```

### 4.2 環境振動測試（AVT）的FRF計算

**適用情況：** 只有輸出響應資料，無已知輸入

#### 4.2.1 輸出-輸出方法：功率譜密度法

對於環境振動測試，可以使用輸出-輸出方法計算等效FRF：

\[
H_{ij}(\omega) = \frac{S_{Y_i Y_j}(\omega)}{S_{Y_j Y_j}(\omega)}
\]

其中：
- \(Y_i\)：第i個測量點的響應
- \(Y_j\)：參考測量點的響應（通常選擇響應較大的點）

#### 4.2.2 實作範例（Python）

```python
import numpy as np
from scipy import signal

def calculate_frf_avt(reference_signal, response_signal, fs, nperseg=None):
    """
    從環境振動測試資料計算FRF
    
    參數:
        reference_signal: 參考通道信號
        response_signal: 響應通道信號
        fs: 採樣頻率
        nperseg: 每個段的長度
    
    返回:
        frequencies: 頻率陣列
        frf: 頻率響應函數（複數）
        coherence: 相干函數
    """
    if nperseg is None:
        nperseg = len(reference_signal) // 4
    
    # 計算交叉功率譜密度
    frequencies, Pxy = signal.csd(response_signal, reference_signal, 
                                   fs=fs, nperseg=nperseg)
    
    # 計算參考通道的功率譜密度
    frequencies, Prr = signal.welch(reference_signal, fs=fs, 
                                     nperseg=nperseg)
    
    # 計算響應通道的功率譜密度
    frequencies, Pyy = signal.welch(response_signal, fs=fs, 
                                     nperseg=nperseg)
    
    # 計算FRF
    frf = Pxy / Prr
    
    # 計算相干函數
    coherence = np.abs(Pxy)**2 / (Prr * Pyy)
    
    return frequencies, frf, coherence

# 使用範例
# 從.mat檔案載入多通道資料
mat_data = sio.loadmat('pdt_01-08/01/avt/01setup01.mat')
data = mat_data['data']

# 選擇參考通道（例如第一個通道）
reference_channel = data[:, 0]

# 計算其他通道相對於參考通道的FRF
frf_results = []
for i in range(1, data.shape[1]):
    response_channel = data[:, i]
    frequencies, frf, coherence = calculate_frf_avt(
        reference_channel, response_channel, fs=100
    )
    frf_results.append({
        'channel': i,
        'frequencies': frequencies,
        'frf': frf,
        'coherence': coherence
    })
```

### 4.3 多輸入多輸出（MIMO）FRF計算

對於多通道資料，可以計算完整的FRF矩陣：

\[
[H(\omega)] = [S_{YY}(\omega)] [S_{XX}(\omega)]^{-1}
\]

**實作範例：**

```python
def calculate_mimo_frf(input_data, output_data, fs, nperseg=None):
    """
    計算多輸入多輸出FRF矩陣
    
    參數:
        input_data: 輸入資料矩陣（時間 x 輸入通道）
        output_data: 輸出資料矩陣（時間 x 輸出通道）
        fs: 採樣頻率
        nperseg: 每個段的長度
    
    返回:
        frequencies: 頻率陣列
        frf_matrix: FRF矩陣（頻率 x 輸出通道 x 輸入通道）
    """
    n_inputs = input_data.shape[1]
    n_outputs = output_data.shape[1]
    
    if nperseg is None:
        nperseg = input_data.shape[0] // 4
    
    # 初始化FRF矩陣
    frequencies = None
    frf_matrix = None
    
    # 計算每個輸入-輸出對的FRF
    for i in range(n_inputs):
        for j in range(n_outputs):
            freq, frf, _ = calculate_frf_h1(
                input_data[:, i], output_data[:, j], fs, nperseg
            )
            
            if frequencies is None:
                frequencies = freq
                frf_matrix = np.zeros(
                    (len(freq), n_outputs, n_inputs), 
                    dtype=complex
                )
            
            frf_matrix[:, j, i] = frf
    
    return frequencies, frf_matrix
```

---

## 五、資料格式與結構

### 5.1 資料組織結構

#### 5.1.1 PDT測試資料結構

```
pdt_01-08/
├── 01/
│   ├── avt/              # 環境振動測試
│   │   ├── 01setup01.mat
│   │   ├── 01setup02.mat
│   │   └── ...
│   └── fvt/              # 強迫振動測試
│       ├── 01setup01.mat
│       ├── 01setup02.mat
│       └── ...
├── 02/
│   └── ...
└── ...
```

**命名規則：**
- `XXsetupYY.mat`：XX為測試編號，YY為設置編號（01-09）
- setup01通常為健康基準狀態
- setup02-09為不同的損傷狀態或測試條件

#### 5.1.2 EMS監測資料結構

```
Z24ems1/
├── 01D00/                # 健康基準：測試01，位置D
│   ├── 01D0003.aaa
│   ├── 01D0005.aaa
│   ├── ...
│   ├── 01D00PRE.env
│   └── 01D00POS.env
├── 01D01/                # 損傷狀態1：測試01，位置D
│   ├── 01D0103.aaa
│   ├── ...
│   ├── 01D01PRE.env
│   └── 01D01POS.env
└── ...
```

**命名規則：**
- `XXYYZZ`：XX為測試編號，YY為位置（A-G），ZZ為狀態（00=健康，01-23=損傷）
- `.aaa`檔案：時間序列資料
  - 檔案名稱中的通道號碼（兩位數）對應到 Fig. 37 中的加速計位置
  - 例如：`13A2305.AAA` = 第13週，星期一，23時，通道05（Zurich方向最左側外緣）
- `PRE.env`和`POS.env`：環境監測資料
  - **詳細通道定義請參考：** `Z24_點位分布與通道定義.md`

### 5.2 資料品質指標

#### 5.2.1 相干函數（Coherence）

相干函數用於評估FRF計算的可靠性：

\[
\gamma^2(\omega) = \frac{|S_{XY}(\omega)|^2}{S_{XX}(\omega) S_{YY}(\omega)}
\]

**解釋：**
- \(\gamma^2 = 1\)：完全相干，FRF可靠
- \(\gamma^2 < 0.7\)：相干性較差，FRF可能不可靠
- \(\gamma^2 < 0.5\)：相干性很差，不建議使用該頻段的FRF

#### 5.2.2 信噪比（SNR）

評估信號品質：

\[
SNR = 10 \log_{10} \frac{P_{signal}}{P_{noise}}
\]

---

## 六、FRF計算實作範例

### 6.1 完整範例：從Z24資料計算FRF

```python
import numpy as np
import scipy.io as sio
from scipy import signal
import matplotlib.pyplot as plt

def load_z24_mat_file(filepath):
    """載入Z24的.mat檔案"""
    mat_data = sio.loadmat(filepath)
    data = mat_data['data']
    labelshulp = mat_data.get('labelshulp', None)
    return data, labelshulp

def calculate_frf_from_z24(filepath, reference_channel=0, 
                           response_channel=1, fs=100, 
                           nperseg=None, window='hann'):
    """
    從Z24資料計算FRF
    
    參數:
        filepath: .mat檔案路徑
        reference_channel: 參考通道索引
        response_channel: 響應通道索引
        fs: 採樣頻率（預設100 Hz）
        nperseg: 每個段的長度
        window: 窗函數類型
    
    返回:
        frequencies: 頻率陣列
        frf: 頻率響應函數
        coherence: 相干函數
    """
    # 載入資料
    data, labelshulp = load_z24_mat_file(filepath)
    
    # 提取通道資料
    ref_signal = data[:, reference_channel]
    resp_signal = data[:, response_channel]
    
    # 設定參數
    if nperseg is None:
        nperseg = len(ref_signal) // 4
    
    # 計算功率譜密度
    frequencies, Pxy = signal.csd(
        resp_signal, ref_signal, 
        fs=fs, nperseg=nperseg, 
        window=window, noverlap=nperseg//2
    )
    
    frequencies, Prr = signal.welch(
        ref_signal, fs=fs, nperseg=nperseg,
        window=window, noverlap=nperseg//2
    )
    
    frequencies, Pyy = signal.welch(
        resp_signal, fs=fs, nperseg=nperseg,
        window=window, noverlap=nperseg//2
    )
    
    # 計算FRF
    frf = Pxy / Prr
    
    # 計算相干函數
    coherence = np.abs(Pxy)**2 / (Prr * Pyy)
    
    return frequencies, frf, coherence

# 使用範例
# 計算健康基準狀態的FRF
healthy_file = 'pdt_01-08/01/avt/01setup01.mat'
freq_healthy, frf_healthy, coh_healthy = calculate_frf_from_z24(
    healthy_file, reference_channel=0, response_channel=1
)

# 計算損傷狀態的FRF
damaged_file = 'pdt_01-08/01/avt/01setup02.mat'
freq_damaged, frf_damaged, coh_damaged = calculate_frf_from_z24(
    damaged_file, reference_channel=0, response_channel=1
)

# 繪製比較圖
fig, axes = plt.subplots(3, 1, figsize=(12, 10))

# FRF幅值比較
axes[0].semilogy(freq_healthy, np.abs(frf_healthy), 
                 label='健康狀態', linewidth=2)
axes[0].semilogy(freq_damaged, np.abs(frf_damaged), 
                 label='損傷狀態', linewidth=2)
axes[0].set_xlabel('頻率 (Hz)')
axes[0].set_ylabel('FRF 幅值')
axes[0].set_title('FRF幅值比較')
axes[0].legend()
axes[0].grid(True, alpha=0.3)

# FRF相位比較
axes[1].plot(freq_healthy, np.angle(frf_healthy) * 180 / np.pi, 
             label='健康狀態', linewidth=2)
axes[1].plot(freq_damaged, np.angle(frf_damaged) * 180 / np.pi, 
             label='損傷狀態', linewidth=2)
axes[1].set_xlabel('頻率 (Hz)')
axes[1].set_ylabel('相位 (度)')
axes[1].set_title('FRF相位比較')
axes[1].legend()
axes[1].grid(True, alpha=0.3)

# 相干函數比較
axes[2].plot(freq_healthy, coh_healthy, 
             label='健康狀態', linewidth=2)
axes[2].plot(freq_damaged, coh_damaged, 
             label='損傷狀態', linewidth=2)
axes[2].axhline(y=0.7, color='r', linestyle='--', 
                label='相干性閾值 (0.7)')
axes[2].set_xlabel('頻率 (Hz)')
axes[2].set_ylabel('相干函數')
axes[2].set_title('相干函數比較')
axes[2].set_ylim([0, 1])
axes[2].legend()
axes[2].grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

### 6.2 批量處理多個檔案

```python
import os
import glob

def batch_calculate_frf(data_dir, output_dir, reference_channel=0, 
                        response_channel=1, fs=100):
    """
    批量計算FRF
    
    參數:
        data_dir: 資料目錄
        output_dir: 輸出目錄
        reference_channel: 參考通道
        response_channel: 響應通道
        fs: 採樣頻率
    """
    # 尋找所有.mat檔案
    mat_files = glob.glob(os.path.join(data_dir, '**/*.mat'), 
                          recursive=True)
    
    results = []
    
    for mat_file in mat_files:
        try:
            frequencies, frf, coherence = calculate_frf_from_z24(
                mat_file, reference_channel, response_channel, fs
            )
            
            # 儲存結果
            result = {
                'filepath': mat_file,
                'frequencies': frequencies,
                'frf': frf,
                'coherence': coherence,
                'mean_coherence': np.mean(coherence)
            }
            
            results.append(result)
            
            # 儲存到檔案
            output_file = os.path.join(
                output_dir, 
                os.path.basename(mat_file).replace('.mat', '_frf.npy')
            )
            np.save(output_file, result)
            
        except Exception as e:
            print(f"處理檔案 {mat_file} 時發生錯誤: {e}")
    
    return results

# 使用範例
results = batch_calculate_frf(
    'pdt_01-08/01/avt',
    'frf_results',
    reference_channel=0,
    response_channel=1
)
```

---

## 七、FRF在損傷檢測中的應用

### 7.1 損傷指標

#### 7.1.1 FRF幅值變化

比較健康與損傷狀態的FRF幅值差異：

\[
DI_{magnitude}(\omega) = \frac{|H_{damaged}(\omega)| - |H_{healthy}(\omega)|}{|H_{healthy}(\omega)|}
\]

#### 7.1.2 FRF相位變化

比較健康與損傷狀態的FRF相位差異：

\[
DI_{phase}(\omega) = |\angle H_{damaged}(\omega) - \angle H_{healthy}(\omega)|
\]

#### 7.1.3 FRF實部與虛部變化

分析FRF的實部和虛部變化：

\[
DI_{real}(\omega) = \frac{Re[H_{damaged}(\omega)] - Re[H_{healthy}(\omega)]}{Re[H_{healthy}(\omega)]}
\]

\[
DI_{imaginary}(\omega) = \frac{Im[H_{damaged}(\omega)] - Im[H_{healthy}(\omega)]}{Im[H_{healthy}(\omega)]}
\]

### 7.2 實作範例：損傷檢測

```python
def calculate_damage_indicator(frf_healthy, frf_damaged, frequencies):
    """
    計算損傷指標
    
    參數:
        frf_healthy: 健康狀態的FRF
        frf_damaged: 損傷狀態的FRF
        frequencies: 頻率陣列
    
    返回:
        damage_indicators: 損傷指標字典
    """
    # FRF幅值變化
    magnitude_change = (
        np.abs(frf_damaged) - np.abs(frf_healthy)
    ) / np.abs(frf_healthy)
    
    # FRF相位變化
    phase_change = np.abs(
        np.angle(frf_damaged) - np.angle(frf_healthy)
    ) * 180 / np.pi
    
    # FRF實部變化
    real_change = (
        np.real(frf_damaged) - np.real(frf_healthy)
    ) / np.real(frf_healthy)
    
    # FRF虛部變化
    imag_change = (
        np.imag(frf_damaged) - np.imag(frf_healthy)
    ) / np.imag(frf_healthy)
    
    # 計算總體損傷指標（RMS）
    rms_magnitude = np.sqrt(np.mean(magnitude_change**2))
    rms_phase = np.sqrt(np.mean(phase_change**2))
    rms_real = np.sqrt(np.mean(real_change**2))
    rms_imag = np.sqrt(np.mean(imag_change**2))
    
    return {
        'frequencies': frequencies,
        'magnitude_change': magnitude_change,
        'phase_change': phase_change,
        'real_change': real_change,
        'imaginary_change': imag_change,
        'rms_magnitude': rms_magnitude,
        'rms_phase': rms_phase,
        'rms_real': rms_real,
        'rms_imaginary': rms_imag
    }

# 使用範例
damage_indicators = calculate_damage_indicator(
    frf_healthy, frf_damaged, freq_healthy
)

# 繪製損傷指標
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

# 幅值變化
axes[0, 0].plot(damage_indicators['frequencies'], 
                damage_indicators['magnitude_change'])
axes[0, 0].set_xlabel('頻率 (Hz)')
axes[0, 0].set_ylabel('幅值變化')
axes[0, 0].set_title('FRF幅值變化')
axes[0, 0].grid(True, alpha=0.3)
axes[0, 0].axhline(y=0, color='k', linestyle='--')

# 相位變化
axes[0, 1].plot(damage_indicators['frequencies'], 
                damage_indicators['phase_change'])
axes[0, 1].set_xlabel('頻率 (Hz)')
axes[0, 1].set_ylabel('相位變化 (度)')
axes[0, 1].set_title('FRF相位變化')
axes[0, 1].grid(True, alpha=0.3)

# 實部變化
axes[1, 0].plot(damage_indicators['frequencies'], 
                damage_indicators['real_change'])
axes[1, 0].set_xlabel('頻率 (Hz)')
axes[1, 0].set_ylabel('實部變化')
axes[1, 0].set_title('FRF實部變化')
axes[1, 0].grid(True, alpha=0.3)
axes[1, 0].axhline(y=0, color='k', linestyle='--')

# 虛部變化
axes[1, 1].plot(damage_indicators['frequencies'], 
                damage_indicators['imaginary_change'])
axes[1, 1].set_xlabel('頻率 (Hz)')
axes[1, 1].set_ylabel('虛部變化')
axes[1, 1].set_title('FRF虛部變化')
axes[1, 1].grid(True, alpha=0.3)
axes[1, 1].axhline(y=0, color='k', linestyle='--')

plt.tight_layout()
plt.show()

# 輸出總體損傷指標
print(f"RMS幅值變化: {damage_indicators['rms_magnitude']:.4f}")
print(f"RMS相位變化: {damage_indicators['rms_phase']:.4f} 度")
print(f"RMS實部變化: {damage_indicators['rms_real']:.4f}")
print(f"RMS虛部變化: {damage_indicators['rms_imaginary']:.4f}")
```

---

## 八、注意事項與建議

### 8.1 資料品質檢查

#### 8.1.1 檢查清單

在計算FRF之前，建議檢查：

- ✅ **資料完整性**：確認資料沒有缺失值或異常值
- ✅ **採樣頻率一致性**：確認所有資料使用相同的採樣頻率（100 Hz）
- ✅ **時間同步**：確認多通道資料時間同步
- ✅ **信號品質**：檢查信號是否有明顯的雜訊或干擾

#### 8.1.2 資料預處理

```python
def preprocess_signal(signal, fs, remove_dc=True, 
                      detrend=True, filter_noise=True):
    """
    信號預處理
    
    參數:
        signal: 原始信號
        fs: 採樣頻率
        remove_dc: 是否移除直流分量
        detrend: 是否去趨勢
        filter_noise: 是否濾除雜訊
    
    返回:
        processed_signal: 處理後的信號
    """
    processed = signal.copy()
    
    # 移除直流分量
    if remove_dc:
        processed = processed - np.mean(processed)
    
    # 去趨勢
    if detrend:
        from scipy import signal as scipy_signal
        processed = scipy_signal.detrend(processed)
    
    # 濾除雜訊（可選的低通濾波）
    if filter_noise:
        from scipy import signal as scipy_signal
        # 設計低通濾波器（截止頻率為Nyquist頻率的0.8倍）
        nyquist = fs / 2
        cutoff = 0.8 * nyquist
        b, a = scipy_signal.butter(4, cutoff / nyquist, 'low')
        processed = scipy_signal.filtfilt(b, a, processed)
    
    return processed
```

### 8.2 FRF計算參數選擇

#### 8.2.1 窗函數選擇

| 窗函數 | 優點 | 缺點 | 適用場景 |
|--------|------|------|---------|
| **Hanning** | 頻率解析度好，旁瓣抑制好 | 頻譜洩漏 | 一般用途（推薦） |
| **Hamming** | 旁瓣抑制好 | 頻率解析度稍差 | 信號較弱時 |
| **Blackman** | 旁瓣抑制最好 | 頻率解析度較差 | 強雜訊環境 |
| **矩形** | 頻率解析度最好 | 頻譜洩漏嚴重 | 不推薦 |

#### 8.2.2 段長度（nperseg）選擇

- **較長的段**：頻率解析度高，但統計穩定性差
- **較短的段**：統計穩定性好，但頻率解析度低
- **建議**：使用信號長度的1/4到1/8

#### 8.2.3 重疊率選擇

- **建議重疊率**：50%（noverlap = nperseg // 2）
- **優點**：提高統計穩定性，減少資料浪費

### 8.3 常見問題與解決方案

#### 8.3.1 問題：FRF在某些頻段不可靠

**原因：**
- 相干函數過低
- 信號強度不足
- 雜訊干擾

**解決方案：**
- 檢查相干函數，只使用相干函數 > 0.7 的頻段
- 增加平均次數
- 改善信號品質

#### 8.3.2 問題：FRF幅值異常

**原因：**
- 輸入信號過小（接近雜訊水平）
- 頻率解析度不足
- 窗函數選擇不當

**解決方案：**
- 檢查輸入信號的功率譜密度
- 調整段長度
- 嘗試不同的窗函數

#### 8.3.3 問題：多通道資料同步問題

**原因：**
- 採樣時鐘不同步
- 資料讀取錯誤

**解決方案：**
- 確認所有通道使用相同的時間向量
- 檢查資料長度是否一致

### 8.4 最佳實踐建議

1. **資料驗證**
   - 始終檢查相干函數
   - 驗證FRF的合理性（幅值、相位連續性）

2. **參數選擇**
   - 使用Hanning窗函數（一般情況）
   - 段長度設為信號長度的1/4
   - 重疊率設為50%

3. **結果驗證**
   - 比較不同估計器（H1 vs H2）的結果
   - 檢查FRF的物理合理性
   - 驗證模態頻率是否在合理範圍內

4. **損傷檢測**
   - 使用多個損傷指標的組合
   - 考慮環境因素的影響
   - 建立統計基準

---

## 附錄：參考資料

### A. 相關檔案位置

- **PDT測試資料**：`pdt_01-08/` 和 `pdt_09-17/`
- **EMS監測資料**：`Z24ems1/`, `Z24ems2/`, `Z24ems3/`
- **說明文件**：`readme.txt`, `Z24-EMS-readme.pdf`
- **點位分布與通道定義**：`Z24_點位分布與通道定義.md`（新增）

### B. 技術規格摘要

| 參數 | 數值 | 說明 |
|------|------|------|
| 採樣頻率 | 100 Hz | fs = 1 / deltaT = 1 / 0.01 = 100 Hz |
| 時間間隔 | 0.01 秒 | deltaT |
| 奈奎斯特頻率 | 50 Hz | f_nyquist = fs / 2 = 100 / 2 = 50 Hz（最大可解析頻率） |
| 樣本長度（.aaa檔案） | 通常 65536 點 | 約 655.36 秒 |
| 資料單位 | g（重力加速度） | 重力加速度單位 |
| 感測器靈敏度 | 1 V/g | 感測器靈敏度 |

### C. 常用Python函式庫

```python
import numpy as np              # 數值計算
import scipy.io as sio          # 讀取.mat檔案
from scipy import signal         # 信號處理
import matplotlib.pyplot as plt  # 繪圖
```

---

**報告生成日期：** 2024年  
**資料集版本：** Z24 Bridge Benchmark Dataset  
**適用範圍：** 結構健康監測、損傷檢測、模態分析
