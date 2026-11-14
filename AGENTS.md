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
| `const ACC=` | 無障礙設施圖層：`icons` 為 `[類型, x, z, 所屬建築id, 是否在樓內]`，`routes` 為無障礙通路折線；`img` 為圖示 base64。由 `access-plan/` 的學校無障礙平面圖以相似轉換（縮放約 0.0977、旋轉約 -88.3°）換算而來 |
| `調整模式` / `campus3d-acc-draft` | 無障礙設施面板的「調整位置」：使用者拖曳、新增、刪除圖示後按「完成並產生文字」，得到 `[["類型",x,z],...]`（x、z 為 P 座標）。收到這段文字時，用它取代 `ACC.icons`（每筆補上所屬建築 id 與是否在樓內），草稿暫存在 localStorage `campus3d-acc-draft` |
| `const ACC_RAMPS` / `makeRamp` | 斜坡道 3D 模型：每座 `[x, z, 角度(度), 長, 寬, 高低對調(0/1)]`（P 座標，預設長 0.6、寬 0.2）。調整模式按「＋ 新增斜坡道」放置、拖曳、旋轉、改長寬；產生的文字第二段就是這個陣列，直接取代 `ACC_RAMPS` 即可。底部高度由 `accSurfaceY` 依地面自動決定 |
| 司令台模型（`buildPodium`） | `ACC_RAMPS` 第 7 欄為 `"podium"` 時畫成「司令台＋斜坡道」：平台約 1.1×0.7、高 0.24，斜坡接在平台側邊（長、寬用第 4、5 欄，`高低對調` 換到另一側），有扶手。調整模式按「＋ 新增司令台（含斜坡道）」放置，刪除方式同斜坡道 |
| `accFloorIcons` / `accSyncFloors` | 教學大樓與行政大樓：外觀模式時，樓內的 `ACC.icons` 顯示在屋頂；選樓層或分層展開時，該棟只保留電梯（永遠顯示在屋頂）＋所選樓層 `FLOORS`／`ADMIN_PLAN` 中 `acc` 房間的無障礙廁所圖示。電梯一律放屋頂（`accRoofNear`）並畫在最上層。`ACC.icons` 第 5 欄 `-1` 代表依位置自動判斷 |
| 樓層專用圖示 | `ACC.icons` 可有第 6 欄樓層標記，如 `"admin:1"`、`"teach:3"`：該圖示放在那一層的 `inner` 群組，只在選到該樓層（或分層展開）時顯示。調整模式下先選樓層再新增，產生的文字會是 `["wc",x,z,"admin:1"]`；套用時把它寫成 `["wc",x,z,"admin",-1,"admin:1"]`。電梯不分樓層，一律在屋頂 |
| `const ACC_ROUTES` / `makeRoute` | 無障礙通路：每條是依行進方向排列的點 `[[x,z],...]`（P 座標），終點畫箭頭。調整模式可選取路線、拖曳控制點、加點／刪點、反轉方向、刪除或新增路線；產生的文字第三段就是這個陣列，直接取代 `ACC_ROUTES` |

## 介面與行為
- 工具列：重設視角、俯視、教學大樓各樓層、各建築場域、地圖操作、建築名稱、電梯・樓梯・廁所、危險角落、自動旋轉。
- 戶外危險角落永遠顯示；教學大樓樓層內的只在選到該樓層時顯示。
- 面板（樓層、清單、地圖操作、危險角落資訊）可拖曳／縮放／收合，位置存在 localStorage。

## 常見修改
- 改房間名稱：搜尋 `FLOORS` 內的舊名稱後取代。
- 新增／移除危險角落：同時改 `POINTS` 與 `POINT_INFO`；若在樓層內，也加入 `FLOORPTS`。
- 照片需先依 EXIF 轉正、縮到約 900px 寬、JPEG 品質約 72 再轉 base64。

更完整的需求與修改紀錄見 `對話紀錄與修改說明.md`。

## 清單與浮動面板的標準（所有新增的清單都必須符合）
網站內每一個清單或浮動面板都要同時具備：
1. **拖曳移動**：按標題列拖曳，雙擊標題列回到原位（參考 `plhead` 的寫法）。
2. **收合按鈕**：標題列有「收合／展開」，收合後只留標題列。
3. **可調整大小**：右下角有 `.presize` 拖曳把手，寬高記在 localStorage。
4. **位置與大小記憶**：用 localStorage 保存，失敗時要 try/catch。

實作方式：新增面板時直接呼叫現成的 helper：
- `floatResizable(box, key, minW, minH)`：加右下角拖曳把手並記住大小。
- `floatCollapsible(box, btn, key)`：綁定收合按鈕並記住收合狀態；收合的 CSS 是 `.fmin`。
- 拖曳移動仿照 `plist`／`dpanel` 的 pointer 事件寫法。

目前符合標準的：無障礙設施（`acclist`）、各建築場域（`blist`）、教學大樓各樓層（`panel`）、地圖操作（`ctrl`）、危險角落資訊（`dpanel`）、校園危險角落清單（`plist`）。

## 行政大樓模型
- 平面圖資料：`admin-plan/`；參考照片：`admin-photo/`。
- 程式關鍵字：`ADMIN_PLAN`、`admSetMode`、`admTileMat`、`admExt`。
- 詳細說明見 `3D地圖重建說明.md` 第 8、9 節。
