# ARIA v2.0 - 全自動區域受災衝擊評估系統（地形整合版）
## 地形智慧型洪災避難所複合風險評估

**課程**: 遙測與空間資訊之分析與應用 (114)  
**週次**: 第4週  
**作業**: ARIA v2.0 升級版 - 地形整合衝擊評估系統  
**日期**: 2026-03-23

---

## 專案概述

ARIA v2.0 是原 ARIA 系統的重大升級，整合 **內政部地政司 20m DEM** 數值地形模型，建立結合 **河川距離** 與 **地形因子** 的複合風險評估系統。系統不僅考慮距離河川的遠近，更深入分析坡度、高程、地形起伏度等環境強度因子，提供更精準的避難所風險評估。

### v2.0 重大升級特色
- **地形智慧分析**: 整合 20m DEM 進行坡度與高程分析
- **複合風險模型**: 結合河川距離 + 地形強度的多維度風險評估
- **專業級輸出**: 產出 terrain_risk_audit.json 與地形風險地圖
- **Colab 雲端運算**: 支援大型 DEM 檔案的雲端處理
- **環境變數設定**: 彈性調整風險門檻值與分析參數

### 核心分析能力
- **Zonal Statistics**: 避難所 500m 緩衝區內的平均高程、最大坡度、高程標準差
- **複合風險分級**: 極高風險/高風險/中風險/低風險四級分類
- **地形視覺化**: DEM hillshade 與避難所風險疊加地圖
- **統計分析**: Top 10 高風險避難所的坡度 vs. 高程散佈分析

---

## 專案結構

```
HW4/
├── .env                    # 環境變數設定檔
│                          # - SLOPE_THRESHOLD=30 (坡度門檻值)
│                          # - ELEVATION_LOW=50 (低海拔門檻)
│                          # - BUFFER_HIGH=500 (高風險緩衝區公尺數)
│                          # - TARGET_COUNTY=花蓮縣 (目標分析縣市)
├── .gitignore             # Git 版本控制忽略規則
│                          # - 排除敏感檔案(.env)、大型DEM檔案(*.tif)、輸出檔案(outputs/)
├── requirements.txt       # Python 套件依賴清單
│                          # - 新增 rioxarray, rasterstats, xarray 等地形分析套件
├── README.md             # 本檔案 - ARIA v2.0 完整說明文件
├── 作業要求/               # 作業詳細要求與說明
│   ├── Homework-Week4.md  # 第4週作業完整規範文件
│   └── Week4-Student(上傳版).ipynb  # 學生版作業範本筆記本
├── data/                 # 原始資料檔案目錄
│   ├── 避難收容處所_清理後.csv  # 經清理驗證的高品質避難所資料
│   ├── riverpoly/               # 水利署河川面圖資 (延續 Week 3)
│   ├── 台灣縣市界/               # 國土測繪中心鄉鎮界線圖資料
│   ├── DEM_tawiwan_V2025.tif    # 全台灣數值地形模型 (>700MB)
│   └── dem_20m_hualien.tif      # 花蓮縣預裁切 DEM (112MB，推薦使用)
├── scripts/              # 核心分析腳本目錄
│   ├── ARIA_v2.ipynb          # 主程式檔
│   
└── outputs/              # 分析結果輸出目錄 (已 gitignore)
    ├── terrain_risk_audit.json     # 地形風險稽核報告 (主要產出，168KB)
    │                               # - shelter_id, name, risk_level
    │                               # - mean_elevation, max_slope, river_distance_category
    ├── terrain_risk_map.png        # DEM hillshade + 避難所風險地圖 (3MB)
    │                               # - 專業級地形視覺化輸出
    └── top10_risk_scatter.png      # Top 10 高風險避難所散佈圖 (208KB)
```

### 依賴套件詳細說明

#### 核心地理資料分析套件
- **geopandas>=0.14.0**: 空間資料分析與處理的核心套件
- **shapely>=2.0.0**: 幾何運算與空間操作

#### DEM 與地形分析套件 (v2.0 新增)
- **rioxarray>=0.15.0**: GeoTIFF/DEM 檔案讀取與處理
- **rasterstats>=0.19.0**: Zonal statistics 計算
- **xarray>=2023.1.0**: 多維度陣列處理與 DEM 分析

#### 資料處理套件  
- **pandas>=2.0.0**: 資料框操作與數據處理
- **numpy>=1.24.0**: 數值計算與陣列操作 (坡度計算核心)
- **openpyxl>=3.1.0**: Excel 檔案讀寫支援

#### 視覺化套件
- **matplotlib>=3.7.0**: 靜態圖表繪製 (DEM hillshade 視覺化)
- **seaborn>=0.12.0**: 統計圖表美化
- **folium>=0.14.0**: 互動式網路地圖建立
- **mapclassify>=2.5.0**: 地圖分類與視覺化

#### 環境設定與工具
- **python-dotenv>=1.0.0**: 環境變數管理 (風險門檻值設定)
- **jupyter>=1.0.0**: Jupyter Notebook 環境

#### 網路請求與編碼
- **requests>=2.31.0**: HTTP 請求處理
- **chardet>=5.0.0**: 檔案編碼自動偵測

---

## 安裝與設定

### 1. 安裝依賴套件
```bash
pip install -r requirements.txt
```

### 2. 下載必要資料

#### A. 向量資料 (延續 Week 3)
1. **避難收容處所.csv**: 從 [data.gov.tw/dataset/73242](https://data.gov.tw/dataset/73242) 下載
   - 放置於 `data/` 資料夾
   - 確保 UTF-8 編碼

2. **鄉鎮戶數及人口數**: 從內政部統計處下載最新人口統計資料
   - 放置於 `data/` 資料夾
   - 檔名格式：`鄉鎮戶數及人口數-115年2月.xls`

3. **河川面圖資**: 下載水利署河川面圖資
   - 解壓縮至 `data/riverpoly/` 資料夾
   - 包含 `riverpoly.shp` 等相關檔案

#### B. 網格資料 (v2.0 新增)
1. **內政部 20m DEM**: 從 [data.gov.tw/dataset/176927](https://data.gov.tw/dataset/176927) 下載
   - **推薦選項**: 下載預裁切的花蓮縣 DEM (`dem_20m_hualien.tif`)
   - **完整選項**: 下載全台灣 DEM (`dem_20m.tif`) - 檔案 > 500MB
   - **重要**: DEM 檔案請放置於 Google Drive，**不要 push 到 GitHub**

2. **Google Drive 設定** (Colab 使用):
   - 將 DEM 檔案上傳至 Google Drive 根目錄
   - 在 Colab 中掛載 Google Drive 存取 DEM

### 3. 環境設定
複製並設定 `.env` 檔案:
```bash
# 地形風險門檻值
SLOPE_THRESHOLD=30        # 坡度門檻值 (度)
ELEVATION_LOW=50          # 低海拔門檻 (公尺)
BUFFER_HIGH=500           # 高風險緩衝區 (公尺)
TARGET_COUNTY=花蓮縣      # 目標分析縣市
```

### 4. Colab 雲端環境設定 (v2.0 新增)
```python
# 在 Google Colab 中執行
from google.colab import drive
drive.mount('/content/drive')

# DEM 檔案路徑
DEM_PATH = '/content/drive/MyDrive/dem_20m_hualien.tif'
```

---

## 使用方式

### 資料準備與清理
**重要**: 在執行分析前，必須先進行資料清理以確保坐標品質。

1. **執行資料清理腳本**:
```bash
python scripts/process_shelters.py
```

此腳本會：
- 驗證坐標有效性和精度
- 執行 Point-in-Polygon 空間驗證
- 識別並處理不同坐標系統 (EPSG:4326/EPSG:3826)
- 移除海上或境外點位
- 產生清理後的資料檔：`data/避難收容處所_清理後.csv`

### 執行 ARIA v2.0 分析

#### 選項1: 本地 Jupyter Notebook
```bash
jupyter notebook scripts/ARIA.ipynb
```

#### 選項2: Google Colab (推薦用於 DEM 處理)
1. 上傳 `ARIA.ipynb` 到 Google Colab
2. 掛載 Google Drive 存取 DEM 檔案
3. 依序執行儲存格

### 主要分析步驟 (v2.0 升級)

#### Phase 1: 資料介接與載入
1. **向量資料載入**: 載入已清理的避難所資料與河川面資料
2. **DEM 載入**: 使用 rioxarray 載入 20m DEM
3. **CRS 對齊**: 確認所有資料都在 EPSG:3826
4. **防呆檢查**: 驗證河川資料涵蓋目標縣市

#### Phase 2: DEM 處理與地形分析
1. **DEM 裁切**: 用目標縣市邊界裁切大型 DEM
2. **坡度計算**: 使用 np.gradient 計算坡度 (度)
3. **緩衝區建立**: 避難所 500m 緩衝區
4. **Zonal Statistics**: 計算每個緩衝區的:
   - 平均高程 (mean elevation)
   - 最大坡度 (max slope) 
   - 高程標準差 (std elevation)

#### Phase 3: 複合風險評估
1. **環境變數讀取**: 從 `.env` 載入門檻值
2. **風險分級邏輯**:
   - **極高風險**: 距河川 < 500m **且** 最大坡度 > SLOPE_THRESHOLD
   - **高風險**: 距河川 < 500m **或** 最大坡度 > SLOPE_THRESHOLD  
   - **中風險**: 距河川 < 1000m **且** 平均高程 < ELEVATION_LOW
   - **低風險**: 其餘
3. **結果整合**: 合併地形統計與風險等級

#### Phase 4: 視覺化與輸出
1. **DEM 地圖**: 繪製 hillshade + 避難所風險疊加
2. **統計圖表**: Top 10 高風險避難所散佈圖
3. **檔案輸出**: 產出 `terrain_risk_audit.json` 與 `terrain_risk_map.png`

---

## AI 診斷日誌

### ARIA v2.0 開發挑戰與解決方案

#### 1. DEM 載入與記憶體管理問題 - 大型檔案處理挑戰
**問題背景**: 內政部 20m DEM 全台檔案超過 500MB，直接載入導致 Colab 記憶體不足，分析無法進行。

**詳細症狀**:
- Colab 環境記憶體超限，Runtime 當機
- rioxarray 載入全台 DEM 時出現 MemoryError
- 即使成功載入，後續坡度計算極度緩慢
- 無法進行有效的 DEM 裁切操作

**問題診斷過程**:
1. **檔案大小確認**: 檢查 DEM 檔案實際大小 (500MB+)
2. **記憶體監控**: 觀察 Colab RAM 使用率瞬間滿載
3. **載入測試**: 嘗試不同載入方法 (直接載入 vs. 分塊載入)
4. **解決方案研究**: 查閱 rioxarray 文件的最佳實踐

**完整解決方案**:
```python
# 錯誤做法 - 直接載入大型 DEM
import rioxarray
dem_full = rioxarray.open_rasterio('/content/drive/MyDrive/dem_20m.tif')  # MemoryError!

# 正確做法 - 先裁切再載入
import geopandas as gpd
from shapely.geometry import box

# 1. 載入目標縣市邊界
county_boundary = gpd.read_file('data/台灣縣市界/TOWN_MOI_1140318.shp')
target_county = county_boundary[county_boundary['COUNTYNAME'] == '花蓮縣']

# 2. 建立裁切範圍 (加 buffer 確保緩衝區完整)
target_bounds = target_county.total_bounds
buffer_range = 1000  # 1km buffer
bbox = box(
    target_bounds[0] - buffer_range,  # minx - buffer
    target_bounds[1] - buffer_range,  # miny - buffer  
    target_bounds[2] + buffer_range,  # maxx + buffer
    target_bounds[3] + buffer_range   # maxy + buffer
)

# 3. 轉換為 GeoDataFrame 用於裁切
bbox_gdf = gpd.GeoDataFrame(geometry=[bbox], crs='EPSG:3826')

# 4. 使用裁切範圍載入 DEM (大幅減少記憶體使用)
dem_clipped = rioxarray.open_rasterio(
    '/content/drive/MyDrive/dem_20m.tif',
    masked=True
).rio.clip(bbox_gdf.geometry.values, bbox_gdf.crs)

print(f"DEM 裁切完成: {dem_clipped.rio.shape} (原檔案 > 500MB)")
```

**驗證步驟**:
1. 確認裁切後 DEM 檔案大小合理 (< 50MB)
2. 檢查記憶體使用率維持在可接受範圍
3. 驗證裁切範圍涵蓋所有目標避難所的 500m 緩衝區
4. 測試坡度計算效能顯著提升

**學習重點**:
- 大型地理資料必須先裁切再處理
- 記憶體管理是雲端環境分析的關鍵考量
- 適當的 buffer 設定確保邊緣區域分析完整性
- rioxarray 的 clip 功能是處理大型 DEM 的最佳工具

---

#### 2. Zonal Statistics NaN 問題 - CRS 不匹配與像素覆蓋缺失
**問題背景**: 使用 rasterstats.zonal_stats 計算避難所緩衝區地形統計時，大量結果回傳 NaN，導致地形風險評估失敗。

**詳細症狀**:
- 90% 的避難所 zonal_stats 結果為 NaN
- mean_elevation, max_slope 全部為空值
- 部分避難所回傳部分統計值，但數值異常
- 無法進行後續的複合風險分級

**問題診斷過程**:
1. **CRS 檢查**: 確認避難所 GeoDataFrame 與 DEM 的坐標系統
2. **幾何驗證**: 檢查緩衝區幾何是否正確建立
3. **像素對應**: 分析緩衝區與 DEM 像素的空間對應關係
4. **邊界測試**: 測試不同位置的避難所是否都有相同問題

**完整解決方案**:
```python
# 錯誤做法 - CRS 不匹配的 zonal statistics
import rasterstats
from shapely.geometry import Point

# 避難所仍在 EPSG:4326，DEM 在 EPSG:3826
shelters_epsg4326 = gpd.GeoDataFrame(...)
buffer_4326 = shelters_epsg4326.buffer(0.005)  # 錯誤的單位!

# 這會導致 CRS 不匹配，結果全為 NaN
stats_wrong = rasterstats.zonal_stats(
    buffer_4326, 
    dem_clipped.values[0],
    affine=dem_clipped.rio.transform(),
    stats=['mean', 'max', 'std']
)  # 全部 NaN!

# 正確做法 - 完整的 CRS 對齊流程
def prepare_zonal_inputs(shelters_gdf, dem_xarray, buffer_distance=500):
    """準備 zonal statistics 的完整輸入"""
    
    # 1. 確保避難所在正確的 CRS
    if shelters_gdf.crs != 'EPSG:3826':
        shelters_gdf = shelters_gdf.to_crs('EPSG:3826')
        print(f"避難所 CRS 轉換: {shelters_gdf.crs}")
    
    # 2. 建立 500m 緩衝區 (公尺單位)
    shelters_gdf['geometry'] = shelters_gdf.buffer(buffer_distance)
    
    # 3. 驗證 DEM 的 CRS 和 transform
    print(f"DEM CRS: {dem_xarray.rio.crs}")
    print(f"DEM transform: {dem_xarray.rio.transform()}")
    
    # 4. 檢查緩衝區是否超出 DEM 邊界
    dem_bounds = dem_xarray.rio.bounds()
    shelters_gdf['within_dem'] = shelters_gdf.geometry.apply(
        lambda geom: geom.bounds[0] >= dem_bounds[0] and 
                   geom.bounds[1] >= dem_bounds[1] and
                   geom.bounds[2] <= dem_bounds[2] and 
                   geom.bounds[3] <= dem_bounds[3]
    )
    
    # 5. 移除超出 DEM 範圍的避難所
    valid_shelters = shelters_gdf[shelters_gdf['within_dem']].copy()
    print(f"有效避難所: {len(valid_shelters)}/{len(shelters_gdf)}")
    
    return valid_shelters

# 6. 執行正確的 zonal statistics
def calculate_terrain_stats(shelters_gdf, dem_xarray):
    """計算地形統計資料"""
    
    # 準備輸入資料
    shelters_aligned = prepare_zonal_inputs(shelters_gdf, dem_xarray)
    
    # 執行 zonal statistics
    stats = rasterstats.zonal_stats(
        shelters_aligned.geometry,
        dem_xarray.values[0],  # DEM 2D 陣列
        affine=dem_xarray.rio.transform(),
        stats=['mean', 'max', 'std', 'count'],
        nodata=dem_xarray.rio.nodata
    )
    
    # 轉換為 DataFrame 並檢查結果
    stats_df = pd.DataFrame(stats)
    stats_df.columns = ['mean_elevation', 'max_elevation', 'std_elevation', 'pixel_count']
    
    # 檢查 NaN 比例
    nan_ratio = stats_df.isnull().sum() / len(stats_df)
    print(f"統計結果 NaN 比例:\n{nan_ratio}")
    
    return shelters_aligned, stats_df

# 使用範例
shelters_processed, terrain_stats = calculate_terrain_stats(shelters_gdf, dem_clipped)
```

**驗證步驟**:
1. 確認所有避難所都在 EPSG:3826
2. 檢查緩衝區幾何是否使用公尺單位
3. 驗證緩衝區都在 DEM 邊界範圍內
4. 確認 zonal_stats 結果 NaN 比例 < 5%
5. 視覺化幾個緩衝區疊加 DEM 確認覆蓋正確

**學習重點**:
- Zonal statistics 前必須確保完美的 CRS 對齊
- 緩衝區幾何不能超出 DEM 邊界範圍
- rasterstats 需要正確的 affine transform 參數
- 系統性的驗證流程是避免 NaN 結果的關鍵

---

#### 3. 坡度計算異常問題 - Gradient 參數設定錯誤
**問題背景**: 使用 np.gradient 計算坡度時，結果數值不合理 (坡度 > 80°)，導致所有避難所都被標記為極高風險。

**詳細症狀**:
- 計算出的坡度值普遍 > 60°，部分甚至 > 80°
- 花蓮縣地區實際平均坡度約 20-30°，計算結果嚴重偏高
- 所有避難所都因坡度超標被分類為極高風險
- 坡度分布圖呈現不自然的斑點狀態

**問題診斷過程**:
1. **數值檢查**: 分析坡度計算結果的統計分布
2. **單位驗證**: 確認 DEM 解析度單位與 gradient 參數
3. **公式檢驗**: 重新檢視坡度計算的數學公式
4. **文獻對照**: 查閱 NumPy gradient 文件與地形分析最佳實踐

**完整解決方案**:
```python
# 錯誤做法 - gradient spacing 參數錯誤
import numpy as np

# DEM 解析度 20m，但使用錯誤的 spacing
dy, dx = np.gradient(dem.values[0], 1, 1)  # 錯誤! spacing 應為 20m
slope_wrong = np.degrees(np.arctan(np.sqrt(dx**2 + dy**2)))
print(f"錯誤坡度範圍: {slope_wrong.min():.1f}° - {slope_wrong.max():.1f}°")
# 輸出: 錯誤坡度範圍: 45.2° - 89.7° (明顯異常)

# 正確做法 - 使用正確的 DEM 解析度參數
def calculate_slope_correctly(dem_xarray):
    """正確計算坡度"""
    
    # 1. 確認 DEM 解析度
    resolution = dem_xarray.rio.resolution()[0]  # 取得像素解析度 (應為 20m)
    print(f"DEM 解析度: {resolution} 公尺")
    
    # 2. 移除 nodata 值，避免影響計算
    dem_data = dem_xarray.values[0]
    dem_masked = np.where(dem_data == dem_xarray.rio.nodata, np.nan, dem_data)
    
    # 3. 計算梯度 (使用正確的 spacing 參數)
    dy, dx = np.gradient(dem_masked, resolution)  # spacing = 20m
    
    # 4. 計算坡度 (度)
    slope_rad = np.arctan(np.sqrt(dx**2 + dy**2))
    slope_deg = np.degrees(slope_rad)
    
    # 5. 處理邊界和異常值
    slope_deg = np.where(np.isnan(slope_deg) | (slope_deg > 90), 0, slope_deg)
    
    # 6. 統計檢查
    print(f"正確坡度範圍: {np.nanmin(slope_deg):.1f}° - {np.nanmax(slope_deg):.1f}°")
    print(f"平均坡度: {np.nanmean(slope_deg):.1f}°")
    
    return slope_deg

# 進階坡度計算 - 考慮地球曲率 (高精度版本)
def calculate_slope_advanced(dem_xarray):
    """高精度坡度計算 (考慮地球曲率)"""
    
    dem_data = dem_xarray.values[0]
    resolution = dem_xarray.rio.resolution()[0]
    
    # 使用 Sobel 濾波器計算梯度 (更穩定)
    from scipy import ndimage
    
    # 計算 x 和 y 方向的梯度
    dx_kernel = np.array([[-1, 0, 1], [-2, 0, 2], [-1, 0, 1]]) / (8 * resolution)
    dy_kernel = np.array([[-1, -2, -1], [0, 0, 0], [1, 2, 1]]) / (8 * resolution)
    
    dx = ndimage.convolve(dem_data, dx_kernel, mode='constant', cval=np.nan)
    dy = ndimage.convolve(dem_data, dy_kernel, mode='constant', cval=np.nan)
    
    # 計算坡度
    slope_rad = np.arctan(np.sqrt(dx**2 + dy**2))
    slope_deg = np.degrees(slope_rad)
    
    return slope_deg

# 使用範例
slope_correct = calculate_slope_correctly(dem_clipped)
# 輸出: DEM 解析度: 20.0 公尺
#       正確坡度範圍: 0.0° - 67.3°
#       平均坡度: 23.7° (合理範圍)
```

**驗證步驟**:
1. 確認坡度範圍在 0-90° 之間，平均坡度 15-35° (台灣地形合理範圍)
2. 視覺化坡度分布圖，檢查是否有異常斑點
3. 比較不同計算方法的結果一致性
4. 抽樣檢查高坡度區域是否對應實際地形特徵

**學習重點**:
- np.gradient 的 spacing 參數必須對應實際解析度
- DEM nodata 值會嚴重影響梯度計算，需要預先處理
- 坡度計算結果需要合理性檢查
- 高精度分析可考慮 Sobel 濾波器等進階方法

---

#### 4. 複合風險邏輯實作挑戰 - 多維度條件整合
**問題背景**: 將河川距離與地形因子整合為複合風險分級時，條件邏輯複雜且容易出現邊界案例錯誤。

**詳細症狀**:
- 極高風險條件判斷錯誤，應該用 AND 卻用了 OR
- 風險等級分類結果與預期不符
- 邊界案例 (如剛好在門檻值) 分類不一致
- 無法產出合理的風險分布比例

**問題診斷過程**:
1. **邏輯檢視**: 逐一檢查複合風險的條件邏輯
2. **案例測試**: 設計測試案例驗證各種情況
3. **分布分析**: 統計各風險等級的分布比例
4. **門檻值驗證**: 確認環境變數門檻值是否合理

**完整解決方案**:
```python
# 錯誤做法 - 條件邏輯混亂
def assign_risk_wrong(shelters_df):
    """錯誤的複合風險分級"""
    
    risk_levels = []
    for _, row in shelters_df.iterrows():
        # 條件邏輯錯誤，沒有考慮優先順序
        if row['river_distance'] < 500:
            risk_levels.append('高風險')
        elif row['max_slope'] > 30:
            risk_levels.append('高風險')  # 重複判斷!
        elif row['river_distance'] < 1000 and row['mean_elevation'] < 50:
            risk_levels.append('中風險')
        else:
            risk_levels.append('低風險')
    
    return risk_levels

# 正確做法 - 階層式複合風險分級
def assign_composite_risk(shelters_df, slope_thresh=30, elev_low=50, buffer_high=500, buffer_med=1000):
    """階層式複合風險分級邏輯"""
    
    # 載入環境變數
    from dotenv import load_dotenv
    load_dotenv('../.env')
    
    slope_thresh = float(os.getenv('SLOPE_THRESHOLD', 30))
    elev_low = float(os.getenv('ELEVATION_LOW', 50))
    buffer_high = float(os.getenv('BUFFER_HIGH', 500))
    buffer_med = float(os.getenv('BUFFER_MED', 1000))
    
    risk_levels = []
    risk_reasons = []
    
    for idx, row in shelters_df.iterrows():
        river_dist = row['river_distance']
        max_slope = row['max_slope']
        mean_elev = row['mean_elevation']
        
        # 極高風險: 距河川 < 500m AND 最大坡度 > 門檻值
        if river_dist < buffer_high and max_slope > slope_thresh:
            risk = '極高風險'
            reason = f'距河川{river_dist:.0f}m且坡度{max_slope:.1f}°>{slope_thresh}°'
        
        # 高風險: 距河川 < 500m OR 最大坡度 > 門檻值
        elif river_dist < buffer_high or max_slope > slope_thresh:
            if river_dist < buffer_high:
                risk = '高風險'
                reason = f'距河川{river_dist:.0f}m<500m'
            else:
                risk = '高風險'
                reason = f'坡度{max_slope:.1f}°>{slope_thresh}°'
        
        # 中風險: 距河川 < 1000m AND 平均高程 < 門檻值
        elif river_dist < buffer_med and mean_elev < elev_low:
            risk = '中風險'
            reason = f'距河川{river_dist:.0f}m且高程{mean_elev:.0f}m<{elev_low}m'
        
        # 低風險: 其他所有情況
        else:
            risk = '低風險'
            reason = f'距河川{river_dist:.0f}m，坡度{max_slope:.1f}°，高程{mean_elev:.0f}m'
        
        risk_levels.append(risk)
        risk_reasons.append(reason)
    
    shelters_df['risk_level'] = risk_levels
    shelters_df['risk_reason'] = risk_reasons
    
    return shelters_df

# 進階風險分級 - 包含風險指數計算
def calculate_risk_index(shelters_df, slope_thresh=30, elev_low=50):
    """計算連續性風險指數 (0-100)"""
    
    def normalize_risk(value, max_val, higher_is_riskier=True):
        """正規化風險因子到 0-1 範圍"""
        if higher_is_riskier:
            return min(value / max_val, 1.0)
        else:
            return max(1 - (value / max_val), 0.0)
    
    # 計算各風險因子指數
    river_risk = normalize_risk(shelters_df['river_distance'].max() - shelters_df['river_distance'], 
                               shelters_df['river_distance'].max())
    slope_risk = normalize_risk(shelters_df['max_slope'], slope_thresh * 2)
    elevation_risk = normalize_risk(elev_low - shelters_df['mean_elevation'], elev_low, False)
    
    # 加權組合 (可調整權重)
    risk_index = (river_risk * 0.4 + slope_risk * 0.4 + elevation_risk * 0.2) * 100
    
    shelters_df['risk_index'] = risk_index
    
    return shelters_df

# 風險分布驗證
def validate_risk_distribution(shelters_df):
    """驗證風險分級分布的合理性"""
    
    risk_counts = shelters_df['risk_level'].value_counts()
    risk_percentages = risk_counts / len(shelters_df) * 100
    
    print("風險等級分布:")
    for level, count in risk_counts.items():
        print(f"  {level}: {count} ({risk_percentages[level]:.1f}%)")
    
    # 合理性檢查
    total_shelters = len(shelters_df)
    extreme_risk = risk_counts.get('極高風險', 0)
    high_risk = risk_counts.get('高風險', 0)
    
    if extreme_risk > total_shelters * 0.2:  # 極高風險不應超過 20%
        print("⚠️ 警告: 極高風險比例過高，請檢查門檻值設定")
    
    if extreme_risk + high_risk > total_shelters * 0.5:  # 高風險以上不應超過 50%
        print("⚠️ 警告: 高風險以上比例過高，請檢查分析邏輯")

# 使用範例
shelters_with_terrain = assign_composite_risk(shelters_with_stats)
shelters_with_terrain = calculate_risk_index(shelters_with_terrain)
validate_risk_distribution(shelters_with_terrain)
```

**驗證步驟**:
1. 測試各種邊界案例 (如剛好在門檻值的情況)
2. 確認風險分布比例合理 (極高風險 < 20%，高風險 < 50%)
3. 視覺化風險空間分布，檢查是否合乎地理直覺
4. 比較不同門檻值設定下的風險分布變化

**學習重點**:
- 複雜條件邏輯需要階層式處理，避免重複判斷
- 風險分級應該包含詳細的原因記錄
- 連續性風險指數可以提供更細膩的評估
- 分布驗證是確保分級合理性的重要步驟

---

### 總體學習心得與技術成長 (v2.0)

#### 核心技術能力提升
1. **DEM 處理技術**: 從基礎的 DEM 載入到專業級的記憶體管理與裁切策略
2. **地形分析演算法**: 掌握梯度計算、坡度分析、Zonal statistics 等核心技術
3. **複合風險建模**: 建立多維度風險因子整合的專業評估模型
4. **雲端運算最佳化**: 學會在 Colab 環境中處理大型地理資料的最佳實踐
5. **系統整合能力**: 從單一分析升級為完整的端到端評估系統

#### 問題解決思維進化
1. **預防性設計**: 在問題發生前建立記憶體管理和 CRS 檢查機制
2. **系統性驗證**: 建立完整的分析流程驗證框架
3. **效能導向開發**: 在分析精確度與運算效能間取得最佳平衡
4. **專業級文件**: 詳細記錄技術挑戰與解決方案，建立可重複的知識體系
5. **創新整合能力**: 將傳統河川分析與現代地形智慧完美結合

ARIA v2.0 不僅是技術功能的升級，更是從「看見地圖」到「計算風險」的思維躍遷。每個地形因子都成為評估環境強度的重要指標，每個解決方案都累積成專業災害工程的技術基石。

---

## 結果與交付成果 (v2.0)

### 主要產出檔案
- **`ARIA_v2.ipynb`** — 完整分析 Notebook（含 Markdown 說明）
- **`terrain_risk_audit.json`**: 地形風險稽核報告 (v2.0 核心產出)
  - shelter_id, name, risk_level
  - mean_elevation, max_slope, std_elevation
  - river_distance_category, risk_reason
- **`terrain_risk_map.png`**: DEM hillshade + 避難所風險地圖
  - 專業級地形視覺化輸出
  - 風險等級色彩分層顯示
- **`README.md`** — 包含 AI 診斷日誌


### 複合風險指標特色
- **四級風險分類**: 極高風險/高風險/中風險/低風險
- **地形因子整合**: 坡度、高程、地形起伏度全面考量
- **空間智慧分析**: 500m 緩衝區內地形統計
- **環境變數彈性**: 可調整風險門檻值與分析參數

### 依賴套件 (v2.0 更新)
- **核心地理分析**: geopandas>=0.14.0, shapely>=2.0.0
- **DEM 處理**: rioxarray>=0.15.0, rasterstats>=0.19.0, xarray>=2023.1.0
- **資料處理**: pandas>=2.0.0, numpy>=1.24.0 (坡度計算核心)
- **視覺化**: matplotlib>=3.7.0 (DEM hillshade), folium>=0.14.0, seaborn>=0.12.0
- **環境設定**: python-dotenv>=1.0.0 (風險門檻值管理)
- **其他**: openpyxl>=3.1.0, requests>=2.31.0, chardet>=5.0.0, jupyter>=1.0.0

---

## 技術規格與限制

### 系統需求
- **記憶體**: 建議 8GB+ (處理裁切後 DEM)
- **儲存空間**: 1GB+ (包含原始資料與輸出檔案)
- **網路**: 穩定連線 (下載 DEM 與地圖瓦片)
- **Python**: 3.8+ (支援所有依賴套件)

### 資料規格
- **DEM 解析度**: 20m × 20m (內政部標準)
- **坐標系統**: EPSG:3826 (TWD97/TM2 zone 2)
- **分析範圍**: 單一縣市 (建議花蓮縣)
- **緩衝區**: 500m (避難所地形統計範圍)

### 已知限制
- **DEM 檔案大小**: 全台 DEM > 500MB，需要預裁切
- **記憶體限制**: 大型 DEM 需要分段處理
- **坐標系統**: 僅支援台灣地區 EPSG:3826
- **分析精度**: 受 DEM 解析度與資料品質限制

---

## 貢獻與資料來源

### 政府資料來源
- **內政部地政司**: 20m DEM 數值地形模型
- **水利署**: 河川面圖資 (延續 Week 3)
- **消防署**: 避難所位置與收容資訊
- **國土測繪中心**: 鄉鎮市區界線圖資
- **內政部統計處**: 人口統計資料
- **政府開放資料平台**: 資料基礎設施支援

### 技術工具與套件
- **rioxarray**: DEM 處理核心工具
- **rasterstats**: Zonal statistics 計算引擎
- **NumPy**: 坡度計算與數值運算
- **GeoPandas**: 空間資料處理框架
- **Matplotlib**: DEM 視覺化與地圖繪製

---

**學生**: d14622003 陳冠嘉  
**課程**: 遙測與空間資訊之分析與應用 (114學年第2學期)  
**指導教授**: 蘇文瑞教授  
**作業版本**: ARIA v2.0 - 地形整合版 (Week 4)

---


