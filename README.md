# ARIA v3.0 — Typhoon Fung-wong Dynamic Risk Integration

---

## 📌 專案說明

本專案整合 ARIA v3.0，結合三個來源的風險資訊：

- **W3**：避難所與河川距離（淹水風險）
- **W4**：地形坡度與高程（崩塌風險）
- **Week 5**：2025 鳳凰颱風（Typhoon Fung-wong）雨量站資料（動態風險）

系統支援兩種模式：

- **LIVE**：呼叫 CWA API 取得即時雨量
- **SIMULATION**：讀取本地 `fungwong_202511.json`

所有資料皆透過 `normalize_cwa_json()` 統一格式與座標（WGS84），確保後續分析流程一致。

若 API 呼叫失敗，系統會自動 fallback 至本地快照。

---

## 🔍 ARIA v3.0 核心邏輯

本系統整合：

- 靜態風險（河川距離 + 地形坡度）
- 動態風險（颱風降雨）

透過空間分析與條件判斷，將避難所分級為：

- **CRITICAL**
- **URGENT**
- **WARNING**
- **SAFE**

其中降雨條件可覆蓋靜態風險（event-driven risk）。

---

## 🚀 執行方式

### 1️⃣ 安裝套件

pip install geopandas pandas numpy requests folium branca matplotlib rioxarray rasterstats python-dotenv shapely

---

### 2️⃣ 建立 `.env`

APP_MODE=SIMULATION
CWA_API_KEY=your_api_key_here
TARGET_COUNTY=花蓮縣

ROOT_DIR=.

TOWNSHIP_PATH=./TOWN_MOI_1140318.shp
SHELTER_CSV_PATH=./避難收容處所點位檔案v9.csv
DEM_PATH=./dem_20m_hualien.tif
SIMULATION_DATA=./fungwong_202511.json

OUTPUT_DIR=./output

BUFFER_RAIN_M=5000
BUFFER_SHELTER_M=500
SLOPE_THRESHOLD=30
ELEVATION_LOW=50
RAIN_WARNING_MM=40
RAIN_CRITICAL_MM=80

---

### 3️⃣ 執行 Notebook

jupyter notebook ARIA_v3_Fungwong_final.ipynb

---

### 4️⃣ 輸出結果

- ARIA_v3_Fungwong.html
- ARIA_v3_shelters.csv
- ARIA_v3_shelters.geojson

---

## 📁 建議資料夾結構

project/
├── ARIA_v3_Fungwong_final.ipynb
├── .env
├── TOWN_MOI_1140318.shp
├── 避難收容處所點位檔案v9.csv
├── dem_20m_hualien.tif
├── fungwong_202511.json
└── output/

---

## AI 診斷日記

### 1️⃣ Lat / Lon 順序

問題：Folium 使用 `[lat, lon]`，但 GeoDataFrame 為 `[lon, lat]`

解法：

location=[row.geometry.y, row.geometry.x]

---

### 2️⃣ -998 異常值處理（與實作一致）

問題：CWA 使用 `-998` 表示缺測資料

實作方式：

def _to_float_or_nan(x):
    try:
        x = float(x)
        return np.nan if x <= -998 else x
    except:
        return np.nan

df["rain_1hr"] = df["rain_1hr"].apply(_to_float_or_nan)
df = df.dropna(subset=["lon", "lat", "rain_1hr"])

---

### 3️⃣ CRS mismatch

assert str(shelters.crs) == str(rain_stations.crs)

---

### 4️⃣ Fallback 機制（與實作一致）

data = fetch_api()

if data is None:
    data = load_local_json()
    source = "FALLBACK_LOCAL_SNAPSHOT"
else:
    source = "LIVE"

---

### 5️⃣ WGS84

coordinates[1]

---

## ⚙️ terrain_risk

very_high / high → HIGH  
medium → MEDIUM  
low → LOW  

---

## 🔐 安全

- 不上傳 .env
- API key 不寫死

---
