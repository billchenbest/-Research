# Z24（FVT）FRF/傳遞函數：Python 實作與使用方式

## 你要達到的目標（對應你整理的研究會議重點）

- **選用 FVT（強迫振動）資料**：以 `pdt_*/fvt/*.mat` 為主
- **計算 FRF（頻率響應函數）並繪圖**：X 軸 Hz、Y 軸 FRF 幅值（以及相位、相干度）
- **健康基準 vs 損傷**：通常用 `setup01` 當健康基準，`setup02~setup09` 當不同狀態
- **複數 FRF**：腳本輸出為複數 \(H(f)\)，繪圖採用 \(|H(f)| = \sqrt{Re^2 + Im^2}\)
- **採樣頻率已釐清**：
  - deltaT = 0.01 sec
  - fs = 1/deltaT = 100 Hz
  - Nyquist = fs/2 = 50 Hz

---

## 重要限制（務必先知道）

**「真正的 FRF」需要已知/量測到的輸入（常見是力 F）。**

目前我們在 `pdt_01-08/01/fvt/01setup01.mat` 看到的通道標籤像：
`139V ... R3V, DP2V`

**通道命名規則：**
- 數字部分（如 139, 239）可能對應測量網格的行號
- 字母部分（V, T, L）對應方向：V=垂直, T=橫向, L=縱向
- `DP2V`：可能為激振輸入通道（建議作為參考通道）
- `R1V/R2V/R3V`：參考通道
- `139V/239V`：測量點通道
- **詳細通道定義請參考：** `Z24_點位分布與通道定義.md`

資料本身未必有明確「力」通道。因此本工具預設用一個 **reference channel**（例如 `DP2V`）來算：

- **傳遞函數 / Transmissibility（相對於 reference 的頻域比值）**
- 同時輸出 **Coherence**（相干度）來檢查可靠頻段

如果你後續在 DOC/PDF 找到真正的「力」通道，或能確認 `DP2V` 就是激振輸入，那就可以把它當作 input。

---

## 程式已補上的「研究會議重點」

- **(1) 訊號預處理與濾波**：`scripts/z24_frf.py` 與 GUI 都支援 `detrend` + `low/high/bandpass`（用於降低雜訊、避免時域差分過度敏感）
- **(2) 損傷指標加總（Summation）**：支援用兩個 `.mat`（健康 vs 損傷）計算 DI，並在指定頻帶內做「多頻率加總」
- **(3) Healthy vs Damaged 標籤提醒**：PDT 的 `setup01` 通常視為基準，但仍**建議 double check 原始文件**（不同測試可能有差異）

---

## 安裝套件（已安裝可跳過）

在 `D:\\Z24`：

```bash
python -m pip install --no-input -r requirements.txt
```

---

## 0) 一鍵流程（推薦）：GUI

直接雙擊 `run_frf_gui.bat`，或用命令列：

```bash
python scripts\z24_gui.py
```

GUI 會做的事：
- 自動載入你選的 `.mat`
- 自動推薦 **輸入通道**（預設會選 `DP2V`，若不存在會改挑 `R2V/R1V/R3V` 等）
- 自動推薦一組「通道順序」（用於 theta/kappa），你也可以用 **上移/下移** 改順序
- 你按下 **執行（產生 FRF 圖）** 後，就會輸出 PNG + NPZ

---

## 1) 先列出該 FVT 檔案的通道

```bash
python scripts/z24_frf.py --mat pdt_01-08/01/fvt/01setup01.mat --list-channels
```

你會看到類似：

- `DP2V`
- `R1V` / `R2V` / `R3V`
- `139V`, `239T` ...（不同測點/方向）

---

## 2) 計算 FRF/傳遞函數並輸出圖

### (A) 健康基準 setup01（建議先跑）

```bash
python scripts/z24_frf.py --mat pdt_01-08/01/fvt/01setup01.mat --input DP2V --out out/frf_01_setup01
```

輸出：
- `out/frf_01_setup01/*.png`：三張子圖（幅值/相位/相干度）
- `out/frf_01_setup01/*.npz`：保存頻率向量、複數 H(f)、相干度、通道標籤

### (B) 損傷狀態 setup02（用來跟 setup01 比）

```bash
python scripts/z24_frf.py --mat pdt_01-08/01/fvt/01setup02.mat --input DP2V --out out/frf_01_setup02
```

---

## 2.5) 可選：濾波/預處理（建議用 bandpass 抑制雜訊）

```bash
python scripts/z24_frf.py --mat pdt_01-08/01/fvt/01setup01.mat --input DP2V ^
  --detrend constant --filter bandpass --f-low 0.5 --f-high 45 ^
  --out out/frf_filtered
```

---

## 2.6) 可選：健康 vs 損傷 DI（多頻率加總）

### 用 setup01 當健康、setup02 當損傷（你可自行換成 setup09）

```bash
python scripts/z24_frf.py --mat pdt_01-08/01/fvt/01setup01.mat ^
  --compare-mat pdt_01-08/01/fvt/01setup02.mat ^
  --input DP2V --detrend constant --filter bandpass --f-low 0.5 --f-high 45 ^
  --di-fmin 0.5 --di-fmax 45 --di-metric l1 ^
  --out out/frf_compare_di
```

輸出 `...__DI.csv`：每個通道一個 DI（頻帶內多頻率加總）。

---

## 3) 只挑某幾個輸出通道（讓圖更清楚）

```bash
python scripts/z24_frf.py --mat pdt_01-08/01/fvt/01setup01.mat --input DP2V --outputs 139V 239V 343V R3V --out out/frf_demo
```

---

## 4) 參數建議

- **nperseg**：頻率解析度與穩定性折衷，預設 auto（常落在 8192/16384）
- **window**：預設 `hann`
- **coherence**：如果某頻段 coherence < 0.7，該頻段的 FRF/傳遞函數可信度較差

---

## 5) 轉角與曲率（你說老師想要的那一步）

我已先把「有限差分」的通用函式寫在：
- `scripts/z24_rotation_curvature.py`

但要做到「由 Z24 資料算轉角/曲率」，你還需要補兩件關鍵資訊：

1. **哪幾個通道對應到同一條梁線上的空間點位**（要有順序）
2. **相鄰點位距離 dx（公尺）**

一旦你把「通道順序 + dx」定下來，我可以把它接到 `z24_frf.py` 裡：
- 由加速度（或位移）時間序列 → 位移（可用頻域積分）→ rotation/curvature（時域差分）→ 再做 FRF → 出圖

---

## 建議你下一步給我的資訊（我就能把轉角/曲率整合完成）

請你回覆其中一項即可：

- **(1)** 你老師/文件指定的「激振 input」是哪個通道？（例如：是否就是 `DP2V`？）
- **(2)** 你要算轉角/曲率的空間點位通道順序（例如：`139V 140V 141V 142V 143V`）以及 dx（m）

**參考資料：**
- **EMS加速計位置**：根據 Fig. 37，共有17個加速計（通道01-16），詳細位置請參考 `Z24_點位分布與通道定義.md`
- **PDT測量網格**：完整的測量網格定義應參考 Appendix K 中的 "Progressive Damage Tests Measurement Grid"（Page 120）
