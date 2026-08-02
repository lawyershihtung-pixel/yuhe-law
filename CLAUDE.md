# 禹河法律事務所 網站 — 專案筆記

給未來協助修改此網站的 AI / 開發者的快速上手說明。**任何新對話請先讀這份。**

## 一、這是什麼

禹河法律事務所(主持律師:張世東律師)的官方網站。純靜態網站(HTML + 內嵌 CSS/JS),
託管於 **GitHub Pages**,以 **Firebase Firestore** 存文章、**Firebase Authentication** 做後台登入。
定位:主打「全台」法律服務(民事、家事、刑事、不動產、行政),辦公室在桃園中壢。

## 二、網址與託管

- 正式網域:**https://yuhe.tw**(向 HiNet 註冊,使用 HiNet DNS 代管)
  - DNS:apex `yuhe.tw` 四筆 A 記錄指向 GitHub Pages(185.199.108–111.153);`www` CNAME → `lawyershihtung-pixel.github.io`
  - repo 根目錄有 `CNAME` 檔(內容 `yuhe.tw`);GitHub Pages 已啟用 Enforce HTTPS
- GitHub repo:`lawyershihtung-pixel/yuhe-law`
- **GitHub Pages 從 `main` 分支根目錄發布**。推上 main 後約 1–2 分鐘生效。
- 舊網址 `lawyershihtung-pixel.github.io/yuhe-law/` 會自動轉到 yuhe.tw。

## 三、頁面

前台(對外):
- `index.html` — 首頁(hero、服務項目、關於、流程、聯絡表單、地圖)
- `about.html` — 關於我們 / 律師簡介
- `news.html` — 最新消息(文章列表,讀 Firestore `news` 集合)
- `knowledge.html` — 法律知識(文章列表,讀 Firestore `knowledge` 集合)

後台(管理,需登入):
- `news-admin.html` — 管理「最新消息」(寫入 `news` 集合)
- `knowledge-admin.html` — 管理「法律知識」(寫入 `knowledge` 集合)

其他:`sitemap.xml`、`robots.txt`、`CNAME`

## 四、Firebase

- 專案 ID:**`yuhe-law-website`**(Spark 免費方案)。與事務所「案件管理系統」是**不同**專案,刻意隔離。
- `firebaseConfig` 直接寫在各頁 `<script>` 裡(apiKey 公開屬正常,安全靠 Firestore 規則)。
- **Firestore 集合**:`news`、`knowledge`。文件欄位:
  `title, category, date(Timestamp), summary, content, published(bool), pinned(bool), order(number), cover(base64字串或''), images([{token,data(base64)}]), createdAt, updatedAt`
- **Firestore 安全規則**(存在 `firestore.rules`,但**必須手動在 Firebase 主控台發布**才生效):
  news、knowledge 皆為「已發布(published==true)公開可讀、草稿僅登入者可讀、僅登入者可寫」,其餘集合一律拒絕。
  ⚠️ 規則的兩條 `allow read` 要**分開寫**(published 與 auth 不可併成同一式,否則未登入列表查詢會被拒)。
- **Authentication**:Email/密碼登入。管理員帳號建於 Firebase Console(Email/密碼)。
  授權網域已含 `yuhe.tw`、`www.yuhe.tw`(email/密碼登入其實不受此限,屬預留)。
- ⚠️ 若新增集合或欄位用途,記得同步更新 `firestore.rules` 並到主控台重新發布。

## 五、已實作的功能(改動時注意別弄壞)

- **視覺主題**:SORATO「天空藍漸層」。以各頁 `</head>` 前一段覆蓋 `<style>` 重定義 CSS 變數
  (`--gold`→天空藍 #1f8fe0、`--ink`→深夜藍 #0f1e33、`--cream`→白)達成。後台也套同色系。
- **文章圖片**:後台可上傳封面圖 + 內文插圖;選圖後用 canvas 壓縮成 base64(約 1000px/JPEG 0.72),
  存進 Firestore(單篇總量需 < 約 950KB,因 Firestore 單文件上限 1MB)。內文用 `[[圖N]]` 標記,前台渲染成圖。
- **置頂 + 排序**:後台每篇可「置頂(pinned)」;列表有 ▲▼ 上下移動(交換 `order` 值)。
  排序規則(前後台一致):pinned 優先 → order 由大到小(無 order 則用日期)。
- **搜尋**:前台有關鍵字搜尋框,過濾標題/摘要/內文/分類,可與分類篩選並用。
- **文章專屬連結**:前台文章可用 `?id=文章ID` 直接開啟;彈窗有「複製連結」;支援瀏覽器上/下一頁。
- **XSS 防護**:前台渲染文章文字都經 `escapeHtml()`;圖片來源為自家 base64。**新增顯示使用者內容處務必比照跳脫**。
- **地圖**:首頁嵌入 Google 官方商家嵌入碼(place ID),「查看地圖」連到 Google 商家短連結。
- **SEO**:各頁有 title/description/canonical/robots、Google 驗證 meta;sitemap.xml、robots.txt 指向 yuhe.tw。
  Google Search Console 已驗證 `https://yuhe.tw/`。

## 六、部署流程(每次修改都照這個)

1. 開發分支:`claude/webpage-edit-permissions-3kjuu1`(在此提交)。
2. commit 後推到該分支,再 fast-forward 併入 `main` 並推送(GitHub Pages 發布 main)。
3. 使用者以強制重整(Ctrl+Shift+R)確認。若動到 Firestore 規則,提醒使用者到主控台重新發布規則。

## 七、常見注意事項

- 前台文章查詢用 `where('published','==',true)`(不加 orderBy,改前端排序)以**避免需要複合索引**。
- 檔案很大(內含 base64 圖片),編輯時用精準字串替換,勿整檔重寫。
- 站內連結為相對路徑;絕對網址(canonical/sitemap/og)用 `https://yuhe.tw/`(根目錄,無 /yuhe-law/ 子路徑)。
- 不要把管理員密碼、任何機密寫進 repo。
