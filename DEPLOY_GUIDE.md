# 觀光餐旅概論雲端闖關遊戲 - Firebase & GitHub 部署指南

本系統採用完全無伺服器 (Serverless) 架構。您不需要購買伺服器或網域，只需按照以下兩個步驟，即可將遊戲部署上網，供全國各校的學生同時連線遊玩！

---

## 🚨 疑難排解：為什麼會出現「註冊寫入失敗」？
如果您的學生在註冊時看到 **「註冊寫入失敗，請重試！」** 的紅色錯誤訊息，通常是因為：
1. **Firebase 資料庫的安全規則 (Rules) 尚未開放**，因此資料庫拒絕了前端寫入帳號資料的要求。
2. 請務必按照下方 **「設定安全規則 (Rules)」** 的說明，在 Firebase 後台將規則更新並點擊「發佈」。

---

## 🛠️ 第一步：建立 Google Firebase 即時資料庫

### 1. 建立專案
1. 前往 **[Firebase Console 官網](https://console.firebase.google.com/)**。
2. 點選 **「新增專案」**，輸入專案名稱（例如：`tourism-quiz-game`），並完成建立。
3. 專案建立完成後，在首頁點擊 **`</>`** (Web) 圖示以註冊網頁應用程式。
4. 註冊後，Firebase 會提供一組 **`firebaseConfig` 金鑰設定值**。請複製這段設定值，它長得像這樣：
   ```javascript
   const firebaseConfig = {
       apiKey: "YOUR_API_KEY",
       authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
       databaseURL: "https://YOUR_PROJECT_ID-default-rtdb.firebaseio.com",
       projectId: "YOUR_PROJECT_ID",
       storageBucket: "YOUR_PROJECT_ID.appspot.com",
       messagingSenderId: "...",
       appId: "..."
   };
   ```
5. 請分別打開您的 `index.html`、`game.html` 與 `admin.html`，將檔案內對應的 `const firebaseConfig` 替換成您自己的設定值。

### 2. 建立 Realtime Database (即時資料庫)
1. 在 Firebase Console 左側選單點選 **建置 (Build) > Realtime Database**。
2. 點擊 **「建立資料庫」**。
3. **資料庫位置**：強烈建議選擇 **「新加坡」(Singapore / asia-southeast1)**，以確保亞洲地區學生的連線存取速度最快。
4. **安全規則**：初始選擇「鎖定模式」或「測試模式」皆可。

### 3. 設定安全規則 (Rules) 【非常重要：解決無法註冊問題】
1. 進入 Realtime Database 後，點選上方的 **「規則」(Rules)** 分頁。
2. 將裡面的內容完全替換成以下規則（此規則極度精簡，可確保註冊與成績記錄順暢寫入且不發生權限錯誤）：
   ```json
   {
     "rules": {
       ".read": true,
       "users": {
         "$username": {
           ".write": true
         }
       },
       "scores": {
         "$score_id": {
           ".write": true
         }
       },
       "reports": {
         "$report_id": {
           ".write": true
         }
       }
     }
   }
   ```
3. 點擊右下角的 **「發佈」(Publish)** 即可生效。

---

## 🌐 第二步：使用 GitHub Pages 免費部署上網

### 1. 建立公開專案
1. 註冊或登入 **[GitHub](https://github.com/)**。
2. 點選右上角的 **「+」 > New repository**。
3. 填入專案名稱（例如：`tourism-quiz`），並確保將權限設為 **Public (公開)**。

### 2. 上傳網頁檔案
1. 使用 **GitHub Desktop** 軟體，或在 GitHub 專案網頁點擊 **「uploading an existing file」**。
2. 將以下四個檔案上傳到專案的根目錄中：
   * 📄 `index.html` (登入與榮譽榜首頁)
   * 📄 `game.html` (學生端作答與錯題解析)
   * 📄 `admin.html` (教師成績監控與問題回報管理後台)
   * 📄 `questions.js` (1060題包含解析的核心題庫)
3. 完成上傳並進行 **Commit** (提交) 與 **Push** (推送)。

### 3. 開啟 GitHub Pages 網頁託管
1. 在您的 GitHub 專案網頁，點選上方的 **Settings** (設定) 分頁。
2. 在左側選單點選 **Pages**。
3. 在 **Build and deployment** 下的 Branch，將預設的 `None` 改選為 **`main`** (或 `master`)，後方路徑維持 `/ (root)`。
4. 點選 **Save** 儲存。
5. 稍等約 1 分鐘，重新整理該頁面，您就會在上方看到您的網站專屬網址！
   * 學生入口網址：`https://[您的GitHub帳號].github.io/[專案名稱]/index.html`

---

## 🔑 常用密碼資訊
* **教師/管理員登入後台密碼**：`116`
  *(如欲修改此密碼，請直接使用文字編輯器打開 `index.html` 與 `admin.html`，搜尋 `116` 並修改為您的自訂密碼。)*
