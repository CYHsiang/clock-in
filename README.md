# 彈性上下班打卡助手 (Vue 3 + Vite + Tailwind CSS)

專為彈性上下班設計的打卡紀錄與下班倒數小工具，支援手機桌面捷徑 (PWA) 與近期打卡紀錄。

## 一、 直接上傳 GitHub 自動部署 (免指令)
1. 在 GitHub 建立一個公開 (Public) 的 Repository。
2. 將本專案解壓縮後的**所有檔案與資料夾**（包含 `.github`、`public`、`src`、`package.json` 等）上傳或 Push 到 GitHub。
3. 進入 GitHub 專案的 **Settings** -> **Pages**。
4. 在 **Build and deployment** 下方的 **Source** 選擇 **GitHub Actions**。
5. GitHub 會自動編譯 Vue 專案並產生專屬網址！

## 二、 在本機 (Mac / Windows) 開發與修改
```bash
# 1. 安裝套件
npm install

# 2. 啟動開發伺服器
npm run dev

# 3. 打包編譯
npm run build
```
