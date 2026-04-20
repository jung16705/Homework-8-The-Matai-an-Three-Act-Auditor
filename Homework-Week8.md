# Week 8 Assignment: ARIA v5.0 — The Matai'an Three-Act Auditor

# 第8週作業：ARIA v5.0 — 馬太鞍三幕稽核員

Submission deadline: Before next class

提交截止日期：下次上課前

---

## 1. Scenario

## 1. 情境

After seven weeks of *modeling* disaster risk, the commander now demands **physical evidence**:

> "We have predictions. Now show me what actually happened on the ground — and tell me whether a lake that should not exist was visible before it killed 18 people."

Your task is to upgrade the ARIA system from **ARIA v4.0 (Network Accessibility, W7)** to **ARIA v5.0 (Matai'an Three-Act Auditor, W8)** by integrating **Sentinel-2 L2A** optical imagery via the **STAC API**. You will produce an optical forensic audit of the **2025 Matai'an Creek barrier lake event（馬太鞍溪堰塞湖事件）** — the most consequential disaster in Hualien County in 2025 — using three satellite scenes spanning the full life cycle of the barrier lake.

**The Matai'an timeline in one sentence**: On Jul 21, 2025 Typhoon Wipha's rainfall triggered a massive landslide in upper Wanrong that dammed the Matai'an Creek and formed a ~200 m deep barrier lake; the lake existed for 64 days and then breached on Sep 23, 2025 at 14:50, releasing 15.4 million m³ of water in 30 minutes and burying downstream Guangfu township (光復鄉). 18+ lives lost.

**Key differences from the in-class lab**:

- The lab used one fingerprint point per land cover type → **this assignment samples a full 20-point grid** along the Matai'an valley from upstream to downstream
- The lab picked one Mid and one Post scene with default thresholds → **this assignment tunes thresholds via confusion matrix + F1 score** against a ground-truth set
- The lab cross-referenced W3/W7/Guangfu visually → **this assignment produces a formal Eyewitness Impact Table** with quantitative audit metrics for all three acts

經過七週的*災害風險建模*，指揮官現在要求**實地證據**：

> 「我們有預測。現在讓我看看地面上實際發生了什麼——並告訴我一個不應該存在的湖泊是否在殺死18人之前就已經可見。」

您的任務是將ARIA系統從**ARIA v4.0（網路可及性，W7）**升級到**ARIA v5.0（馬太鞍三幕稽核員，W8）**，通過**STAC API**整合**Sentinel-2 L2A**光學影像。您將使用涵蓋堰塞湖完整生命週期的三個衛星場景，製作**2025年馬太鞍溪堰塞湖事件**的光學取證稽核——這是2025年花蓮縣最具影響力的災害。

**馬太鞍時間軸一言以蔽之**：2025年7月21日颱風韋帕的降雨引發萬榮上游大規模崩塌，堵塞馬太鞍溪並形成約200公尺深的堰塞湖；該湖存在64天，然後於2025年9月23日14:50潰決，在30分鐘內釋放1540萬立方公尺的水，淹沒下游光復鄉。18+人喪生。

**與課堂實驗室的主要差異**：

- 實驗室使用每種土地覆蓋類型一個指紋點 → **此作業沿馬太鞍谷從上游到下游取樣完整的20點網格**
- 實驗室選擇一個中間和一個後期場景，使用預設閾值 → **此作業通過混淆矩陣 + F1分數針對地面真實集調整閾值**
- 實驗室視覺交叉參考W3/W7/光復 → **此作業產生正式的目擊者影響表**，包含所有三幕的量化稽核指標

---

## 2. Data Sources

## 2. 資料來源

### A. Satellite Imagery (STAC)

### A. 衛星影像（STAC）

- **Source**: Microsoft Planetary Computer
- **Collection**: `sentinel-2-l2a`
- **Area of interest**: Matai'an bounding box `121.28, 23.56, 121.52, 23.76` (covers upper Wanrong barrier lake site → downstream Guangfu township)
- **Three-act date windows**:
  - **Pre** (original forest): 2025-06-01 to 2025-07-15, cloud cover < 20%
  - **Mid** (lake present, pre-breach): 2025-08-01 to 2025-09-20, cloud cover < 40% (monsoon season — relax clouds)
  - **Post** (lake drained, debris in Guangfu): 2025-09-25 to 2025-11-15, cloud cover < 30%
- **Required bands**: B02, B03, B04, B08, B11, B12
- **Tool**: `pystac-client` + `stackstac` + `rioxarray` (see Pre-lab)
- **Recommended pattern**: Use the `robust_search()` helper from the Demo notebook (client-side cloud filtering with exponential backoff retry) for more reliable results than server-side `query={"eo:cloud_cover": {"lt": N}}`
- **COG token refresh**: Always call `pc.sign(item)` before reading band assets — SAS tokens expire after ~1 hour

- **來源**：Microsoft Planetary Computer
- **集合**：`sentinel-2-l2a`
- **興趣區域**：馬太鞍邊界框 `121.28, 23.56, 121.52, 23.76`（涵蓋萬榮上游堰塞湖地點 → 下游光復鄉）
- **三幕日期窗口**：
  - **前**（原始森林）：2025-06-01 到 2025-07-15，雲覆蓋 < 20%
  - **中**（湖泊存在，潰決前）：2025-08-01 到 2025-09-20，雲覆蓋 < 40%（季風季節——放寬雲層）
  - **後**（湖泊排空，光復下游殘骸）：2025-09-25 到 2025-11-15，雲覆蓋 < 30%
- **所需波段**：B02, B03, B04, B08, B11, B12
- **工具**：`pystac-client` + `stackstac` + `rioxarray`（參見預實驗室）
- **推薦模式**：使用演示筆記本中的 `robust_search()` 輔助函數（客戶端雲過濾，具有指數退避重試）以獲得比伺服器端 `query={"eo:cloud_cover": {"lt": N}}` 更可靠的結果
- **COG權杖刷新**：在讀取波段資產之前始終呼叫 `pc.sign(item)` —— SAS權杖在大約1小時後過期

### B. Outputs from Previous Weeks

### B. 前幾週的輸出

- **Week 3 Shelters** GeoDataFrame — including `river_risk` (Hualien City range — geographically north of Matai'an; you will document this coverage gap)
- **Week 4 Terrain Risk** — `mean_elevation`, `max_slope`, `terrain_risk`
- **Week 5 Rainfall** — Wipha simulation (`wipha_202507.json`) *or* the teacher-provided historical rainfall record
- **Week 6 Kriging Output** — `kriging_rainfall.tif` (recommended for validation of the upstream landslide source)
- **Week 7 Top-5 Bottlenecks** — `top5_bottlenecks.gpkg`
- **Week 7 Road Network** — `hualien_network.graphml`

- **第3週庇護所** GeoDataFrame —— 包括 `river_risk`（花蓮市範圍——地理上位於馬太鞍以北；您將記錄此覆蓋差距）
- **第4週地形風險** —— `mean_elevation`, `max_slope`, `terrain_risk`
- **第5週降雨** —— 韋帕模擬（`wipha_202507.json`）*或* 教師提供的歷史降雨記錄
- **第6週克里金輸出** —— `kriging_rainfall.tif`（推薦用於驗證上游崩塌源頭）
- **第7週前5個瓶頸** —— `top5_bottlenecks.gpkg`
- **第7週道路網路** —— `hualien_network.graphml`

### C. New for Week 8 — Guangfu Overlay (built by you in Pre-lab)

### C. 第8週新內容 —— 光復疊加層（由您在預實驗室中建置）

- **`guangfu_overlay.gpkg`** — You built this yourself in **Pre-lab Step 7b**. Five required nodes (光復火車站、光復國小、光復鄉公所、台9線馬太鞍溪橋、佛祖街沉積區中心), schema `name / cn_name / node_type / priority / geometry`, saved as EPSG:3826. If your file is missing or malformed at submission time, the Demo notebook's Cell [17] has a synthetic fallback that will keep Part D runnable — but Part D full credit requires your own valid file with the 5 required nodes plus at least 2 optional nodes of your own choosing.

- **`guangfu_overlay.gpkg`** —— 您在**預實驗室步驟7b**中自行建置。五個必需節點（光復火車站、光復國小、光復鄉公所、台9線馬太鞍溪橋、佛祖街沉積區中心），架構 `name / cn_name / node_type / priority / geometry`，儲存為EPSG:3826。如果您的檔案在提交時遺失或格式錯誤，演示筆記本的Cell [17]有一個合成後備方案，將保持第D部分可運行——但第D部分滿分需要您自己的有效檔案，包含5個必需節點加上至少2個您自己選擇的可選節點。

### D. Ground Truth & Verification

### D. 地面真實與驗證

- **Esri Sentinel-2 Explorer**: Use [ArcGIS Sentinel-2 Explorer](https://sentinel2explorer.esri.com/) to visually verify your scene selection and compare spectral indices (NDWI, NDVI, NDMI) at the barrier lake location. This is an essential human-verification step.
- **NCDR reference data**: The barrier lake peaked at ~86 ha on Sep 11, and shrank to ~12 ha by Oct 16 after partial breach. Use these as calibration benchmarks for your lake mask area.
- Official post-event landslide reports from 農業部林業署花蓮分署 or 中央地質調查所 (if available)
- News photos of the Guangfu flood with geotags (you may georeference 5–10 of them for the ground-truth subset)
- The teacher will provide `reference_mataian_truth.gpkg` as a fallback

- **Esri Sentinel-2 Explorer**：使用 [ArcGIS Sentinel-2 Explorer](https://sentinel2explorer.esri.com/) 視覺驗證您的場景選擇，並比較堰塞湖位置的光譜指標（NDWI、NDVI、NDMI）。這是必不可少的人類驗證步驟。
- **NCDR參考資料**：堰塞湖於9月11日達到峰值約86公頃，並於10月16日部分潰決後縮小到約12公頃。使用這些作為湖泊遮罩面積的校準基準。
- 官方災後崩塌報告來自農業部林業署花蓮分署或中央地質調查所（如果可用）
- 帶有地理標籤的光復洪水新聞照片（您可以為地面真實子集地理參考5–10張）
- 教師將提供 `reference_mataian_truth.gpkg` 作為後備

---

## 3. Core Requirements

## 3. 核心要求

Submit as a single **`.ipynb`** (Jupyter Notebook).

以單個**`.ipynb`**（Jupyter Notebook）提交。

### A. Three-Act STAC Scene Selection + TCI Quick-QA

### A. 三幕STAC場景選擇 + TCI快速QA

1. Connect to Planetary Computer via `pystac_client.Client.open()` + `planetary_computer.sign_inplace`
2. Query `sentinel-2-l2a` for each of the three date windows
3. **For each window**, preview the top-3 candidates using the `visual` (TCI) asset and **write a 2-sentence justification** in a markdown cell explaining why you picked each scene (pay attention to cloud cover over the Matai'an valley specifically, not just the tile-level percentage)
4. Save the chosen item IDs to your notebook metadata so results are reproducible — you should end up with exactly **three item IDs**: `PRE_ITEM_ID`, `MID_ITEM_ID`, `POST_ITEM_ID`

1. 通過 `pystac_client.Client.open()` + `planetary_computer.sign_inplace` 連接到Planetary Computer
2. 為每個三個日期窗口查詢 `sentinel-2-l2a`
3. **對於每個窗口**，使用 `visual`（TCI）資產預覽前3個候選，並在markdown單元格中**寫下2句理由**解釋為什麼選擇每個場景（特別注意馬太鞍谷的雲覆蓋，而不是僅僅圖塊級百分比）
4. 將選擇的項目ID儲存到您的筆記本元資料中，以便結果可重現 —— 您應該最終得到精確**三個項目ID**：`PRE_ITEM_ID`, `MID_ITEM_ID`, `POST_ITEM_ID`

### B. Four Change Metrics (Three-Act aware)

### B. 四個變化指標（三幕感知）

Implement the following as reusable functions that take two `xarray.DataArray` cubes and return a single-band change raster:

1. **NIR Drop**: `nir_drop(pre, post) = pre_B08 - post_B08`
2. **SWIR Post Brightness**: `swir_post(post) = post_B12`
3. **Bare Soil Index change**: `bsi_change(pre, post) = bsi(post) - bsi(pre)` where `BSI = ((B11 + B04) - (B08 + B02)) / ((B11 + B04) + (B08 + B02))`
4. **NDVI Change**: `ndvi_change(pre, post) = ndvi(pre) - ndvi(post)` where `NDVI = (B08 - B04) / (B08 + B04)`

Apply these to two comparisons and save the resulting eight change maps as PNGs in `output/`:
- **Pre → Mid** (to capture the birth of the barrier lake)
- **Pre → Post** (to capture the landslide source + downstream debris flow)

將以下實作為可重用函數，接受兩個 `xarray.DataArray` 立方體並返回單波段變化柵格：

1. **NIR下降**：`nir_drop(pre, post) = pre_B08 - post_B08`
2. **SWIR後亮度**：`swir_post(post) = post_B12`
3. **裸土指標變化**：`bsi_change(pre, post) = bsi(post) - bsi(pre)` 其中 `BSI = ((B11 + B04) - (B08 + B02)) / ((B11 + B04) + (B08 + B02))`
4. **NDVI變化**：`ndvi_change(pre, post) = ndvi(pre) - ndvi(post)` 其中 `NDVI = (B08 - B04) / (B08 + B04)`

將這些應用於兩個比較，並將產生的八個變化地圖儲存為 `output/` 中的PNG：
- **前 → 中**（捕捉堰塞湖的誕生）
- **前 → 後**（捕捉崩塌源頭 + 下游泥石流）

### C. Three Detection Masks with Tuned Thresholds

### C. 三個檢測遮罩與調整閾值

You will produce **three** separate detection masks, each with its own physical justification:

#### C1. Barrier lake mask (Pre→Mid)
Baseline rule: `(nir_pre > 0.25) & (nir_mid < 0.18) & (blue_mid > 0.03) & (green_mid > nir_mid)`
- **Important**: The barrier lake water is **turbid** (loaded with suspended sediment), so its NIR is 0.10–0.18, NOT < 0.05 like clear water. Using 0.08 will return zero lake pixels.
- **Spatial gate**: Restrict detection to west of ~121.33°E to exclude downstream river flooding false positives
- Tune the `nir_mid` upper bound — pick 3 candidate values (e.g. 0.12, 0.15, 0.18) and report which best matches the NCDR reference ~0.86 km² peak lake area (Sep 11)
- **NCDR-verified lake center**: approximately (121.292, 23.696)
- Vectorize with `rasterio.features.shapes`

#### C2. Landslide source scar mask (Pre→Post)
Baseline rule: `(nir_drop > 0.15) & (swir_post > 0.25) & (nir_pre > 0.25)`
- Build a ground-truth set of at least **10 confirmed landslide pixels** + **10 stable-vegetation pixels** (from the reference gpkg, your own georeferencing of news photos, or hand-drawn in QGIS)
- Tune `nir_drop` and `swir_post` thresholds by computing the confusion matrix (TP, FP, TN, FN) at **5 candidate threshold pairs**
- Report the best F1-score and the pair you picked. Save as a markdown table in the notebook.
- Vectorize the final mask and drop polygons with area < 2000 m²

#### C3. Debris flow footprint mask (Pre→Post, downstream only)
Baseline rule: `(ndvi_change > 0.25) & (bsi_change > 0.10) & (nir_pre > 0.20)` — AND must be downstream of the Matai'an Creek mouth (lon > 121.35 as a simple gate)
- Drop polygons with area < 5000 m²
- **Explain in a markdown cell** why this rule is different from the C2 landslide rule (the physics of fresh mud sheet vs. exposed rock headwall)

您將產生**三個**單獨的檢測遮罩，每個都有自己的物理理由：

#### C1. 堰塞湖遮罩（前→中）
基準規則：`(nir_pre > 0.25) & (nir_mid < 0.18) & (blue_mid > 0.03) & (green_mid > nir_mid)`
- **重要**：堰塞湖水是**渾濁的**（載有懸浮沉積物），因此其NIR為0.10–0.18，**不是**像清澈水一樣 < 0.05。使用0.08將返回零湖泊像素。
- **空間閘**：將檢測限制在約121.33°E以西，以排除下游河流洪水假陽性
- 調整 `nir_mid` 上限 —— 選擇3個候選值（例如0.12、0.15、0.18），並報告哪個最符合NCDR參考約0.86平方公里峰值湖泊面積（9月11日）
- **NCDR驗證湖泊中心**：大約（121.292, 23.696）
- 使用 `rasterio.features.shapes` 向量化

#### C2. 崩塌源頭疤痕遮罩（前→後）
基準規則：`(nir_drop > 0.15) & (swir_post > 0.25) & (nir_pre > 0.25)`
- 建置至少**10個確認崩塌像素** + **10個穩定植被像素**的地面真實集（來自參考gpkg、您自己的新聞照片地理參考，或在QGIS中手繪）
- 通過計算混淆矩陣（TP、FP、TN、FN）在**5個候選閾值對**調整 `nir_drop` 和 `swir_post` 閾值
- 報告最佳F1分數和您選擇的對。儲存為筆記本中的markdown表格。
- 向量化最終遮罩並丟棄面積 < 2000平方公尺的多邊形

#### C3. 泥石流足跡遮罩（前→後，下游僅）
基準規則：`(ndvi_change > 0.25) & (bsi_change > 0.10) & (nir_pre > 0.20)` —— 並且必須位於馬太鞍溪口下游（lon > 121.35作為簡單閘）
- 丟棄面積 < 5000平方公尺的多邊形
- **在markdown單元格中解釋**為什麼此規則與C2崩塌規則不同（新鮮泥板與暴露岩石頭牆的物理）

### D. Multi-Layer Audit — The Eyewitness Impact Table

### D. 多層稽核 —— 目擊者影響表

Produce an **Eyewitness Impact Table** as a DataFrame in the notebook:

| Asset | Type | Location | W4 Terrain Risk | W7 Centrality Rank | Barrier Lake Hit (Y/N) | Landslide Hit (Y/N) | Debris Flow Hit (Y/N) | Notes |
|-------|------|----------|-----------------|---------------------|-----------------------|---------------------|----------------------|-------|
| Shelter_H001 | W3 Shelter | Hualien City | 3 | — | N | N | N | outside event area |
| Node_H42 | W7 Bottleneck | Suhua Hwy | 2 | 1 | N | N | N | outside event area |
| Guangfu_Elementary | W8 Guangfu Overlay | 光復國小 | — | — | N | N | Y | buried in 20 cm debris |
| Matai'an_Bridge | W8 Guangfu Overlay | 台9線馬太鞍溪橋 | — | — | N | Y | Y | collapsed Sep 23 16:00 |

Rules:
- Include **all W3 shelters**, **all W7 Top-5 bottlenecks**, and **all nodes from the Guangfu overlay**
- "Hit" = inside (debris flow, lake) or within 200 m (landslide scar) of the detected polygon
- Sort by "Debris Flow Hit" first, then "Landslide Hit", then W4 terrain risk descending
- **Crucial teaching moment**: Include a markdown cell analyzing the **coverage gap** — how many W3/W7 Hualien City assets were hit (probably zero) vs. how many Guangfu overlay assets were hit. Discuss what this says about ARIA's pre-event coverage and what the county should do about it.

在筆記本中將**目擊者影響表**產生為DataFrame：

| 資產 | 類型 | 位置 | W4地形風險 | W7中心性排名 | 堰塞湖擊中（Y/N） | 崩塌擊中（Y/N） | 泥石流擊中（Y/N） | 註記 |
|-------|------|----------|-----------------|---------------------|-----------------------|---------------------|----------------------|-------|
| Shelter_H001 | W3庇護所 | 花蓮市 | 3 | — | N | N | N | 事件區域外 |
| Node_H42 | W7瓶頸 | 蘇花公路 | 2 | 1 | N | N | N | 事件區域外 |
| Guangfu_Elementary | W8光復疊加層 | 光復國小 | — | — | N | N | Y | 埋在20公分殘骸中 |
| Matai'an_Bridge | W8光復疊加層 | 台9線馬太鞍溪橋 | — | — | N | Y | Y | 於9月23日16:00崩塌 |

規則：
- 包括**所有W3庇護所**、**所有W7前5個瓶頸**，以及**光復疊加層中的所有節點**
- 「擊中」= 內部（泥石流、湖泊）或檢測多邊形內200公尺（崩塌疤痕）
- 首先按「泥石流擊中」排序，然後「崩塌擊中」，然後W4地形風險降序
- **關鍵教學時刻**：包括一個markdown單元格分析**覆蓋差距** —— W3/W7花蓮市資產有多少被擊中（可能為零）與光復疊加層資產有多少被擊中。討論這對ARIA預事件覆蓋意味著什麼，以及縣政府應該做什麼。

### E. AI Advisor Operational Brief (Bonus)

### E. AI顧問作戰簡報（獎勵）

1. Feed the Impact Table + a text summary of the ARIA chain (W3→W4→W5→W6→W7→W8) + the **three-act timeline** into **any LLM you prefer** (ChatGPT / Gemini / Claude / local model — pick the tool you're most comfortable with)
2. Prompt specification: The AI should act as a *Hualien County Disaster Prevention Command Center — Chief of Operations* writing for the county magistrate
3. The AI brief must cover:
   - **Confirmed timeline**: What the imagery proves about each of the three acts
   - **The pre-breach window**: Between Jul 21 (lake formed) and Sep 23 (breach), what could ARIA v5.0 have warned about if it had been operational? Be specific — which satellite revisits had clean scenes?
   - **Coverage gap**: Why did the W3/W7 pre-event coverage miss Guangfu? What expansion does ARIA need?
   - **Next 24-hour orders** (as of the Post-event scene): priority clearance, shelter resupply, UAV tasking
   - **Model refinement**: One concrete suggestion to extend ARIA before the next barrier-lake event

```python
# Bonus Prompt Example (adapt to your AI of choice)
prompt = f"""You are the Chief of Operations at the Hualien County Disaster
Prevention Command Center, writing a brief for the county magistrate. ARIA v5.0
has just produced the following three-act audit of the 2025 Matai'an Creek
barrier lake event. Write a 250-word operational brief covering:

1. Three-act timeline — what the imagery proves
2. The pre-breach warning window (Jul 21 to Sep 23, 64 days) — what ARIA could
   have caught if it had been operational during that window
3. Coverage gap — why did W3/W7 miss Guangfu?
4. Next-24-hour orders: priority clearance, shelter resupply, UAV tasking
5. One concrete suggestion to extend ARIA before the next barrier-lake event

IMPACT TABLE:
{impact_table.to_markdown()}

THREE-ACT DETECTION SUMMARY:
- Act 1 (Pre, {PRE_ITEM_ID}): forested Matai'an valley, no lake
- Act 2 (Mid, {MID_ITEM_ID}): barrier lake {lake_area_km2:.3f} km² detected
- Act 3 (Post, {POST_ITEM_ID}): lake drained; landslide source {ls_area_km2:.3f} km²;
  debris flow footprint {db_area_km2:.3f} km² over Guangfu

ARIA CHAIN SUMMARY:
- W3: {len(shelters)} Hualien City shelters
- W4: {(shelters['terrain_risk'] > 2).sum()} shelters high terrain risk
- W7: Top 5 bottlenecks (all in Hualien City corridor)
- W8: 3 hazard masks produced, {n_guangfu_hits} Guangfu overlay nodes hit
"""
```

1. 將影響表 + ARIA鏈的文字摘要（W3→W4→W5→W6→W7→W8）+ **三幕時間軸**輸入到**您偏好的任何LLM**（ChatGPT / Gemini / Claude / 本地模型 —— 選擇您最熟悉的工具）
2. 提示規範：AI應該扮演*花蓮縣災害防治指揮中心 —— 作戰主任*為縣長撰寫
3. AI簡報必須涵蓋：
   - **確認時間軸**：影像證明每個三幕的什麼
   - **潰決前窗口**：7月21日（湖泊形成）到9月23日（潰決）之間，如果ARIA v5.0已經運作，能警告什麼？具體說明 —— 哪些衛星重訪有清潔場景？
   - **覆蓋差距**：為什麼W3/W7預事件覆蓋遺漏光復？ARIA需要什麼擴展？
   - **下24小時命令**（根據後事件場景）：優先清理、庇護所補給、UAV任務
   - **模型精煉**：一個具體建議，在下一個堰塞湖事件前擴展ARIA

```python
# 獎勵提示範例（適應您的AI選擇）
prompt = f"""You are the Chief of Operations at the Hualien County Disaster
Prevention Command Center, writing a brief for the county magistrate. ARIA v5.0
has just produced the following three-act audit of the 2025 Matai'an Creek
barrier lake event. Write a 250-word operational brief covering:

1. Three-act timeline — what the imagery proves
2. The pre-breach warning window (Jul 21 to Sep 23, 64 days) — what ARIA could
   have caught if it had been operational during that window
3. Coverage gap — why did W3/W7 miss Guangfu?
4. Next-24-hour orders: priority clearance, shelter resupply, UAV tasking
5. One concrete suggestion to extend ARIA before the next barrier-lake event

IMPACT TABLE:
{impact_table.to_markdown()}

THREE-ACT DETECTION SUMMARY:
- Act 1 (Pre, {PRE_ITEM_ID}): forested Matai'an valley, no lake
- Act 2 (Mid, {MID_ITEM_ID}): barrier lake {lake_area_km2:.3f} km² detected
- Act 3 (Post, {POST_ITEM_ID}): lake drained; landslide source {ls_area_km2:.3f} km²;
  debris flow footprint {db_area_km2:.3f} km² over Guangfu

ARIA CHAIN SUMMARY:
- W3: {len(shelters)} Hualien City shelters
- W4: {(shelters['terrain_risk'] > 2).sum()} shelters high terrain risk
- W7: Top 5 bottlenecks (all in Hualien City corridor)
- W8: 3 hazard masks produced, {n_guangfu_hits} Guangfu overlay nodes hit
"""
```

### F. Professional Standards (Infrastructure First)

### F. 專業標準（基礎設施優先）

1. **Environment Variables**: `.env` must contain `STAC_ENDPOINT`, `S2_COLLECTION`, `MATAIAN_BBOX`, `PRE_EVENT_START/END`, `MID_EVENT_START/END`, `POST_EVENT_START/END`, `TARGET_EPSG`
2. **Reproducible Scene IDs**: Save `PRE_ITEM_ID`, `MID_ITEM_ID`, `POST_ITEM_ID` as Python constants at the top of the notebook
3. **Captain's Log Markdown Cells**: Before each code block, write 1–2 sentences explaining *why* this step exists
4. **AI Diagnostic Log in README**: Describe how you solved at least one of:
   - "Mid-event STAC window returned only cloudy scenes — how did I find a usable one?"
   - "Barrier lake mask picked up river shadows as false positives — how did I filter them out?"
   - "Landslide mask had false positives on river sandbars — how did I use `pre_NIR > 0.3` or a slope gate?"
   - "Debris flow mask overlapped with the landslide mask — how did I decide which to keep?"
   - "Threshold tuning gave me F1 < 0.5 — what went wrong and how did I fix it?"

1. **環境變數**：`.env` 必須包含 `STAC_ENDPOINT`, `S2_COLLECTION`, `MATAIAN_BBOX`, `PRE_EVENT_START/END`, `MID_EVENT_START/END`, `POST_EVENT_START/END`, `TARGET_EPSG`
2. **可重現場景ID**：將 `PRE_ITEM_ID`, `MID_ITEM_ID`, `POST_ITEM_ID` 儲存為筆記本頂部的Python常數
3. **船長日誌Markdown單元格**：在每個程式碼區塊前，寫下1–2句解釋*為什麼*此步驟存在
4. **README中的AI診斷日誌**：描述您如何解決至少一個：
   - 「中事件STAC窗口只返回雲層場景 —— 我如何找到可用的？」
   - 「堰塞湖遮罩拾取河流陰影作為假陽性 —— 我如何過濾它們？」
   - 「崩塌遮罩在河流沙洲上有假陽性 —— 我如何使用 `pre_NIR > 0.3` 或坡度閘？」
   - 「泥石流遮罩與崩塌遮罩重疊 —— 我如何決定保留哪個？」
   - 「閾值調整給我F1 < 0.5 —— 出了什麼錯，如何修復？」

---

## 4. Recommended Coding Prompt

## 4. 推薦編碼提示

> "I need to build ARIA v5.0 — a satellite-based eyewitness auditor for the 2025 Matai'an Creek barrier lake event. I have:
> 1. `shelters_hualien.gpkg` from W3 (Hualien City range) with river_risk
> 2. `hualien_terrain.gpkg` from W4 with terrain_risk
> 3. `kriging_rainfall.tif` from W6
> 4. `top5_bottlenecks.gpkg` and `hualien_network.graphml` from W7
> 5. `guangfu_overlay.gpkg` from W8 (new Guangfu township critical nodes)
>
> Help me in separate Jupyter cells:
> 1. Connect to Planetary Computer STAC, query Sentinel-2 L2A for all three acts (Pre Jun / Mid Aug / Post Oct 2025)
> 2. TCI-preview the top-3 candidates in each window; let me pick and save item IDs
> 3. Stream B02/B03/B04/B08/B11/B12 via stackstac with Matai'an bbox (121.28, 23.56, 121.52, 23.76), EPSG:32651, 10 m
> 4. Compute nir_drop, swir_post, BSI change, and NDVI change as xarray DataArrays
> 5. Build three detection masks: (a) barrier lake Pre→Mid, (b) landslide source Pre→Post, (c) debris flow downstream Pre→Post
> 6. Ground-truth tune the landslide thresholds: 10+10 points, 5 threshold pairs, report F1
> 7. Vectorize each mask with rasterio.features.shapes (drop tiny polygons)
> 8. Spatial join with W3 shelters (100 m buffer), W7 top-5 bottlenecks (200 m buffer), and Guangfu overlay (within/100 m buffer)
> 9. Produce the Eyewitness Impact Table DataFrame with columns for all three hits + a coverage-gap discussion
> 10. Build the AI Advisor prompt (three-act timeline + impact table + coverage gap) and call my preferred LLM API
> 11. Save all figures to `output/` and write the README AI diagnostic log"

> 「我需要建置ARIA v5.0 —— 一個基於衛星的2025年馬太鞍溪堰塞湖事件的目擊者稽核員。我有：
> 1. `shelters_hualien.gpkg` 來自W3（花蓮市範圍）帶有river_risk
> 2. `hualien_terrain.gpkg` 來自W4帶有terrain_risk
> 3. `kriging_rainfall.tif` 來自W6
> 4. `top5_bottlenecks.gpkg` 和 `hualien_network.graphml` 來自W7
> 5. `guangfu_overlay.gpkg` 來自W8（新的光復鄉關鍵節點）
>
> 在單獨的Jupyter單元格中幫助我：
> 1. 連接到Planetary Computer STAC，為所有三幕查詢Sentinel-2 L2A（前6月 / 中8月 / 後10月2025）
> 2. 在每個窗口中TCI預覽前3個候選；讓我挑選並儲存項目ID
> 3. 通過stackstac串流B02/B03/B04/B08/B11/B12與馬太鞍bbox（121.28, 23.56, 121.52, 23.76），EPSG:32651，10公尺
> 4. 計算nir_drop、swir_post、BSI變化，以及NDVI變化作為xarray DataArrays
> 5. 建置三個檢測遮罩：（a）堰塞湖前→中，（b）崩塌源頭前→後，（c）下游泥石流前→後
> 6. 地面真實調整崩塌閾值：10+10點，5個閾值對，報告F1
> 7. 使用rasterio.features.shapes向量化每個遮罩（丟棄小多邊形）
> 8. 空間連接W3庇護所（100公尺緩衝區）、W7前5個瓶頸（200公尺緩衝區），以及光復疊加層（內部/100公尺緩衝區）
> 9. 產生目擊者影響表DataFrame，包含所有三個擊中的欄位 + 覆蓋差距討論
> 10. 建置AI顧問提示（三幕時間軸 + 影響表 + 覆蓋差距）並呼叫我偏好的LLM API
> 11. 將所有圖形儲存到 `output/` 並寫README AI診斷日誌」

---

## 5. Deliverables

## 5. 交付物

1. **GitHub Repo URL**
2. **`ARIA_v5_mataian.ipynb`** — Complete analysis notebook with all outputs executed
3. **`mataian_detections.gpkg`** — Multi-layer GeoPackage with three layers: `barrier_lake`, `landslide_source`, `debris_flow`
4. **`impact_table.csv`** — The Eyewitness Impact Table
5. **`output/` folder** — At minimum: three-act TCI panel, four change-metric maps, three detection masks, final impact map with all layers overlaid
6. **`README.md`** — Including AI diagnostic log, the three chosen STAC item IDs, and a coverage-gap discussion

1. **GitHub Repo URL**
2. **`ARIA_v5_mataian.ipynb`** —— 包含所有執行輸出的完整分析筆記本
3. **`mataian_detections.gpkg`** —— 多層GeoPackage，包含三個層：`barrier_lake`, `landslide_source`, `debris_flow`
4. **`impact_table.csv`** —— 目擊者影響表
5. **`output/` 資料夾** —— 至少：三幕TCI面板、四個變化指標地圖、三個檢測遮罩、最終影響地圖疊加所有層
6. **`README.md`** —— 包括AI診斷日誌、三個選擇的STAC項目ID，以及覆蓋差距討論

---

## 6. Grading Rubric

## 6. 評分標準

| Item | Weight |
|------|--------|
| Three-act STAC scene selection + TCI quick-QA + reproducible item IDs | 15% |
| Four change metric functions (nir_drop, swir_post, BSI change, NDVI change) implemented correctly | 15% |
| Three detection masks (barrier lake + landslide source + debris flow) with threshold tuning and F1 reporting | 25% |
| Multi-layer audit (Impact Table with W3/W4/W7/Guangfu overlay join) + coverage-gap discussion | 20% |
| Professional standards (.env + README + Captain's logs + AI diagnostic log) | 15% |
| Visualization quality (three-act panel + detection overlays + final impact map) | 10% |
| Bonus: AI advisor three-act operational brief (any LLM) | +10% |

| 項目 | 權重 |
|------|--------|
| 三幕STAC場景選擇 + TCI快速QA + 可重現項目ID | 15% |
| 四個變化指標函數（nir_drop、swir_post、BSI變化、NDVI變化）正確實作 | 15% |
| 三個檢測遮罩（堰塞湖 + 崩塌源頭 + 泥石流）帶有閾值調整和F1報告 | 25% |
| 多層稽核（帶有W3/W4/W7/光復疊加層連接的影響表）+ 覆蓋差距討論 | 20% |
| 專業標準（.env + README + 船長日誌 + AI診斷日誌） | 15% |
| 可視化品質（三幕面板 + 檢測疊加 + 最終影響地圖） | 10% |
| 獎勵：AI顧問三幕作戰簡報（任何LLM） | +10% |

---

## 7. Tips and Notes

## 7. 提示與註記

- **CRS consistency**: Sentinel-2 native = EPSG:32651 (UTM 51N). W3/W4/W7/Guangfu vectors = EPSG:3826 (TWD97). Always reproject *vectors to raster CRS* when sampling pixel values (faster than resampling the raster)
- **Memory safety**: `stackstac.stack()` with `bounds_latlon=MATAIAN_BBOX` keeps the cube under 200 MB. Without it you will get an OOM
- **Monsoon cloud traps**: August and September 2025 were peak monsoon. You may need `cloud_cover < 50%` and then hand-pick via TCI preview. This is normal and is exactly why Step A (TCI Quick-QA) exists
- **Turbid water NIR**: The Matai'an barrier lake is **turbid** (sediment-laden), so its NIR reflectance is 0.10–0.18, NOT < 0.05 like clear water. If you use a clear-water threshold, you will get zero lake pixels. Use `nir_mid < 0.18` as a starting point and add `green_mid > nir_mid` to confirm water-like spectral shape
- **Lake false-positive traps**: Deep shadows (especially on north-facing slopes in the afternoon) have low NIR *and* low blue. The `blue_mid > 0.03` gate handles most of them, but check near steep headwalls
- **Landslide false-positive traps**: Rivers, sandbars, bare harvested fields all trigger NIR-drop + SWIR-high. Use `pre_NIR > 0.3` (was actually vegetated) or a slope > 15° gate from W4 to clean it up
- **Debris flow false-positive traps**: Fresh wet rice paddies after harvest also have NDVI drop + moderate BSI. Use the `lon > 121.35` gate to restrict to downstream Guangfu, or cross-reference against a harvested-paddy mask
- **Cloud remnants**: Even cloud_cover < 20% scenes have scattered cloud shadows. Use the Sentinel-2 SCL (Scene Classification Layer, asset name `SCL`) and mask out classes 3 (cloud shadow), 8, 9, 10 (clouds of varying confidence)
- **TCI trick**: The TCI is served as 8-bit RGB JPEG — it loads in < 2 seconds even at full resolution. Use it for every visual check, not just the initial QA
- **Item ID format**: Sentinel-2 item IDs look like `S2A_MSIL2A_20250610T022601_R046_T51RTH_20250610T061802`. The 8-digit number in the middle is the acquisition date — use it to verify the scene falls inside your intended window
- **If you get zero lake pixels**: First check the `nir_mid` threshold — turbid water NIR is 0.10–0.18, not < 0.05 like clear water. If still zero, your Mid scene may be wrong (too early, lake hadn't formed, or too late, already breached). Check the item date is between 2025-08-01 and 2025-09-22. Also verify your spatial gate is not too aggressive — the lake center is at approximately (121.292, 23.696)
- **RAW vs GATED comparison**: The Demo notebook shows both unfiltered (`lake_mask_raw`) and spatially gated (`lake_mask`) results. This is good practice — always inspect raw results before applying spatial filters, to understand what the filter is doing

- **CRS一致性**：Sentinel-2原生 = EPSG:32651（UTM 51N）。W3/W4/W7/光復向量 = EPSG:3826（TWD97）。當取樣像素值時，始終將*向量重新投影到柵格CRS*（比重新取樣柵格更快）
- **記憶體安全**：帶有 `bounds_latlon=MATAIAN_BBOX` 的 `stackstac.stack()` 將立方體保持在200 MB以下。沒有它您會得到OOM
- **季風雲陷阱**：2025年8月和9月是高峰季風。您可能需要 `cloud_cover < 50%` 然後通過TCI預覽手動挑選。這是正常的，這正是步驟A（TCI快速QA）存在的原因
- **渾濁水NIR**：馬太鞍堰塞湖是**渾濁的**（沉積物載荷），因此其NIR反射率為0.10–0.18，**不是**像清澈水一樣 < 0.05。如果您使用清澈水閾值，您將得到零湖泊像素。使用 `nir_mid < 0.18` 作為起點，並添加 `green_mid > nir_mid` 以確認水樣光譜形狀
- **湖泊假陽性陷阱**：深陰影（尤其在下午北向坡）有低NIR *和* 低藍。 `blue_mid > 0.03` 閘處理大多數，但檢查陡峭頭牆附近
- **崩塌假陽性陷阱**：河流、沙洲、裸收穫田都觸發NIR下降 + SWIR高。使用 `pre_NIR > 0.3`（實際上是植被）或W4的坡度 > 15°閘來清理
- **泥石流假陽性陷阱**：收穫後新鮮濕稻田也有NDVI下降 + 中等BSI。使用 `lon > 121.35` 閘限制到下游光復，或交叉參考收穫稻田遮罩
- **雲殘餘**：即使cloud_cover < 20%場景也有散佈雲陰影。使用Sentinel-2 SCL（場景分類層，資產名稱 `SCL`）並遮罩類別3（雲陰影）、8、9、10（不同信心度的雲）
- **TCI技巧**：TCI作為8位元RGB JPEG提供 —— 即使在全解析度下也載入 < 2秒。將其用於每個視覺檢查，而不僅僅是初始QA
- **項目ID格式**：Sentinel-2項目ID看起來像 `S2A_MSIL2A_20250610T022601_R046_T51RTH_20250610T061802`。中間的8位數字是獲取日期 —— 使用它來驗證場景落在您的預期窗口內
- **如果您得到零湖泊像素**：首先檢查 `nir_mid` 閾值 —— 渾濁水NIR為0.10–0.18，不是 < 0.05清澈水。如果仍然零，您的中場景可能錯了（太早，湖泊尚未形成，或太晚，已潰決）。檢查項目日期在2025-08-01和2025-09-22之間。也驗證您的空間閘不夠激進 —— 湖泊中心在大約（121.292, 23.696）
- **RAW vs GATED比較**：演示筆記本顯示未過濾（`lake_mask_raw`）和空間閘（`lake_mask`）結果。這是良好實務 —— 始終檢查原始結果在應用空間過濾前，以了解過濾在做什麼

---

*"A risk model predicts the disaster. A network analysis tells you if you can still reach it. A satellite tells you whether your predictions were right — and whether there was a 200 m deep lake growing in the mountains for 64 days that nobody noticed."*

*"風險模型預測災害。網路分析告訴您是否仍能到達。衛星告訴您您的預測是否正確 —— 以及山中是否有一個200公尺深的湖泊在64天內成長，沒人注意到。"*
