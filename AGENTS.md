# 給 AI 工具的修改說明（Claude / ChatGPT / Gemini / Copilot 等通用）

臺南特教校園 3D 互動地圖。整個專案就是 `index.html` 一個檔案（Three.js 0.147.0 + OrbitControls，從 CDN 載入，無需建置）。線上版：https://hsiang0422-star.github.io/campus-3d-map/

## 工作方式
- 直接編輯 `index.html`，commit 並 push 到 `main`，GitHub Pages 1–2 分鐘後更新。
- 檔案約 2MB，因為危險角落照片以 base64 內嵌（`data:image/jpeg;base64,...`）。用搜尋定位，不要整檔讀入；用字串取代修改。
- 改完先用 `node --check` 檢查 `<script>` 內容語法。注意 `const` 宣告順序（用到前必須先宣告，否則整頁空白）。
- 沒有測試環境；位置與尺寸皆為依照片估算。

## 重要資料位置（用關鍵字搜尋）
| 關鍵字 | 內容 |
|---|---|
| `const BUILDINGS` | 各建築（名稱、顏色、樓層、標籤位置、說明） |
| `const FLOORS` | 教學大樓 1–4F 的房間 `[x0,y0,x1,y1,type,name,area]`（名稱可在此改） |
| `const POINTS` | 危險角落編號與座標 `[編號, x, z]`（同編號可有多個位置） |
| `const POINT_INFO` | 危險角落資訊卡：`title`、`desc`、`img`/`imgs`（照片）、`floor`、`fzones` |
| `FLOORPTS` | 位於教學大樓樓層內的危險角落編號（只在該樓層顯示） |
| `P(x,z)` | 座標轉換：世界座標 = (x-13, z-8) |

## 介面與行為
- 工具列：重設視角、俯視、教學大樓各樓層、各建築場域、地圖操作、建築名稱、電梯・樓梯・廁所、危險角落、自動旋轉。
- 戶外危險角落永遠顯示；教學大樓樓層內的只在選到該樓層時顯示。
- 面板（樓層、清單、地圖操作、危險角落資訊）可拖曳／縮放／收合，位置存在 localStorage。

## 常見修改
- 改房間名稱：搜尋 `FLOORS` 內的舊名稱後取代。
- 新增／移除危險角落：同時改 `POINTS` 與 `POINT_INFO`；若在樓層內，也加入 `FLOORPTS`。
- 照片需先依 EXIF 轉正、縮到約 900px 寬、JPEG 品質約 72 再轉 base64。

更完整的需求與修改紀錄見 `對話紀錄與修改說明.md`。
