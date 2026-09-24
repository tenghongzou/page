# page

部署在 GitHub Pages 的純前端小工具集，網址：<https://page.pervive.cc/>

## 工具

### PDF 轉圖片（JPG / PNG）

路徑：[`Converter/pdftojpg.html`](Converter/pdftojpg.html)

- 點擊或拖放選擇本機 PDF，逐頁轉成 JPG 或 PNG，顯示縮圖格線與進度，可隨時取消
- 可調解析度（1x / 2x / 3x，約 72 / 144 / 216 DPI）與 JPG 品質；改設定後可一鍵重新轉換
- 每頁可單獨下載，也可勾選多頁打包成 ZIP 下載
- 單頁超過約 1600 萬像素時自動降低解析度（避免 iOS Safari 的 canvas 上限）
- 支援深色模式、鍵盤操作與螢幕閱讀器
- **檔案只在瀏覽器中處理，不會上傳到任何伺服器**

使用的函式庫（透過 jsDelivr CDN 載入）：

- [PDF.js](https://github.com/mozilla/pdf.js) `pdfjs-dist@4.10.38`：渲染 PDF。使用 `intent: 'print'`，切換到其他分頁時轉換不會暫停
- [fflate](https://github.com/101arrowz/fflate) `0.8.3`：打包 ZIP（不壓縮，速度最快）

## 專案結構

```
.
├── index.html                  # 首頁，工具卡片
├── assets/
│   └── style.css               # 共用樣式與 design tokens（含深色模式）
├── Converter/
│   └── pdftojpg.html           # PDF 轉圖片
├── CNAME                       # 自訂網域 page.pervive.cc
├── dist/                       # 部署時組裝的輸出目錄（不進版控）
└── .github/workflows/deploy.yml
```

## 本機開發

不需要建置，用任意靜態伺服器即可：

```bash
python3 -m http.server 8000
# 開啟 http://localhost:8000/
```

> 直接用 `file://` 開啟可能因瀏覽器安全限制無法載入 PDF.js worker，請使用本機伺服器。

## 部署

使用 GitHub Actions 官方 Pages 流程（`upload-pages-artifact` + `deploy-pages`），以下情況會觸發部署：

| 觸發方式 | 範例 |
|---|---|
| 推送版本 tag（`v*`） | `git tag v1.1.0 && git push origin v1.1.0` |
| 推送到 `prod` 分支 | `git push origin master:prod` |
| 手動執行 | GitHub → Actions → Deploy to GitHub Pages → Run workflow |

推送到 `master` **不會**部署，`master` 為開發分支。

部署時會把 `index.html`、`CNAME`、`assets/`、`Converter/` 複製到 `dist/` 後發佈。**新增頁面或目錄時，記得同步修改 `deploy.yml` 的 Build 步驟。**

`github-pages` environment 只允許 `prod` 分支與 `v*` tag 部署。

### 網域與 HTTPS

- DNS 由 Cloudflare 管理：`page` 為 CNAME → `tenghongzou.github.io`，**必須維持「僅 DNS」（灰色雲朵）**
- HTTPS 憑證由 GitHub Pages 自動申請（Let's Encrypt）並續期，已開啟「Enforce HTTPS」
- 若改回 Cloudflare Proxy，GitHub 將無法續期憑證
