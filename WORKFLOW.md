# 🎓《觀光餐旅業導論》雲端互動闖關與全真檢測系統
## 系統架構、資料流程與完整工作流指引 (Workflow & System Architecture)

---

## 📌 一、系統基本資訊與專案簡介

* **系統全名**：《觀光餐旅業導論》雲端互動闖關與全真檢測系統
* **主要用途**：專為全國高中職（觀光事業科、餐飲管理科、綜合高中觀光餐旅學程）升學衝刺量身打造的跨平台、免安裝雲端自主學習與測驗工具。
* **技術架構**：
  * **前端技術**：純原生 HTML5 / CSS3 (多主題切換、RWD 響應式佈局) / Vanilla JavaScript (ES6+)
  * **雲端後端 (Serverless)**：Google Firebase Realtime Database 即時資料庫 (同步學生帳號、測驗歷程、問題回報)
  * **第三方整合元件**：
    * `html2pdf.js`：學生端「📥 錯題解析 PDF 講義」動態生成與匯出
    * `SheetJS (xlsx.full.min.js)`：教師端「📊 32 關小考成績 Excel 總表」即時匯出
    * `Google Fonts` (Outfit + Noto Sans TC)
  * **代管部署**：GitHub Pages 靜態網站自動持續部署 (CI/CD)
* **線上正式網址**：
  * 🚀 **學生闖關首頁**：[https://abel54575457.github.io/tourism-quiz/index.html](https://abel54575457.github.io/tourism-quiz/index.html)
  * 💼 **教師管理後台**：[https://abel54575457.github.io/tourism-quiz/admin.html](https://abel54575457.github.io/tourism-quiz/admin.html) *(管理密碼：`116`)*

---

## 🧩 二、系統模組與檔案結構

```
tourism-quiz/
├── index.html          # [入口門戶] 學生登入/註冊、全國及校內即時英雄風雲榜、教師後台跳轉
├── game.html           # [闖關核心] 32 關卡地圖、小考答題介面、85分晉級判定、PDF講義匯出、問題回報
├── admin.html          # [教師後台] 學生進度監控、待辦問題回報中心 (呼吸燈警示)、Excel匯出
├── questions.js        # [題庫核心] 32 關卡、640 題次題庫 (含 440 題 100% 零重複核心題與精闢解析)
└── README.md           # 專案說明文件
```

---

## 🔄 三、核心工作流程 (Workflows)

### 3.1 學生端完整學習與測驗工作流 (Student Workflow)

```mermaid
flowchart TD
    Start([學生造訪系統入口 index.html]) --> InputAccount[輸入帳號與密碼]
    InputAccount --> CheckExist{帳號是否存在於 Firebase?}
    
    CheckExist -- 首次登入 --> RegModal[彈出註冊視窗: 選擇北中南東區域 -> 下拉選校 -> 填寫姓名]
    RegModal --> SaveUser[寫入 Firebase users 資料庫並建立 Session]
    
    CheckExist -- 已註冊帳號 --> VerifyPass{密碼是否相符?}
    VerifyPass -- 錯誤 --> AlertWrong[提示密碼錯誤]
    VerifyPass -- 正確 --> SaveUser
    
    SaveUser --> GamePage([進入關卡地圖 game.html])
    GamePage --> ViewMap[檢視 32 個關卡進度地圖: 依目前 unlocked_level 顯示已解鎖/鎖定]
    
    ViewMap --> ClickLevel[點選已解鎖關卡開始測驗]
    ClickLevel --> StartQuiz[載入該關 20 題隨機題序題庫]
    
    StartQuiz --> AnswerQ[閱讀題目與四選一選項 -> 點擊作答]
    AnswerQ --> ImmediateFeedback[即時動畫回饋: 綠色正確 / 紅色答錯並記錄錯題清單]
    ImmediateFeedback --> NextQ{是否已達第 20 題?}
    NextQ -- 否 --> AnswerQ
    
    NextQ -- 是 --> CalcScore[計算總分 (每題 5 分, 滿分 100 分)]
    CalcScore --> SaveScoreDB[非同步將成績寫入 Firebase scores 資料庫]
    
    SaveScoreDB --> ScoreCheck{得分 >= 85 分?}
    ScoreCheck -- 是 (及格通關) --> UnlockNext[解鎖下一關卡 (unlocked_level + 1) 並同步至 Firebase]
    ScoreCheck -- 否 (未達標) --> KeepLevel[保持目前關卡進度, 提示 85 分方可解鎖下一關]
    
    UnlockNext --> ResultView[結算畫面: 呈現分數、評語與錯題精闢解析]
    KeepLevel --> ResultView
    
    ResultView --> ChoiceA[🔄 再次挑戰本關]
    ResultView --> ChoiceB[⭐ 前往下一關]
    ResultView --> ChoiceC[🗺️ 回關卡地圖]
    ResultView --> ChoiceD[📥 匯出錯題與解析 (下載 PDF 講義)]
    ChoiceD --> GenPDF[html2pdf 動態渲染專屬學習評量單並自動下載]
    
    ClickLevel -. 遇到爭議題目 .-> OpenReport[點擊 ⚠️ 問題回報]
    OpenReport --> SendReport[填寫描述並寫入 Firebase reports 資料庫]
```

#### 學生端步驟詳解：
1. **無感註冊與快速登入**：
   * 學生輸入自訂帳號與密碼。
   * 若為首次使用，系統自動彈出註冊視窗，提供「北區 / 中區 / 南區 / 東區及離島」分區下拉選校機制（收錄全國所有設有觀光餐旅類科之高中職），學生只需選擇學校與填寫姓名即可一秒完成註冊。
2. **階梯式闖關地圖**：
   * 系統即時連線雲端資料庫讀取 `unlocked_level`，解鎖對應關卡。
   * 學生必須獲得 **85 分以上（答對 17 題以上）** 才能順利通關並解鎖下一關。
3. **即時反饋與作答體驗**：
   * 每題點選後立即給予視覺回饋（正確顯示綠色呼吸框、錯誤顯示紅色震動框），並自動累計錯誤題目的題幹、學生作答、正確答案與考點解析。
4. **📥 下載錯題解析 PDF 講義**：
   * 測驗結束後，學生可一鍵點擊下載 PDF 講義。
   * 講義自動排版「學校、姓名、測驗單元、得分、錯題對照（含學生答案 ❌ 與標準答案 ✔️）、老師精闢考點解析」，支援離線複習與列印。
5. **即時排行榜榮譽激勵**：
   * 每筆作答完成後，首頁排行榜秒級自動更新，呈現全台總排行與校內排名。

---

### 3.2 教師端/管理者工作流 (Teacher/Admin Workflow)

```mermaid
flowchart TD
    TeacherStart([教師點擊入口後台按鈕]) --> PwdModal[彈出密碼驗證視窗]
    PwdModal --> CheckPwd{輸入密碼 === 116?}
    CheckPwd -- 錯誤 --> PwdErr[提示密碼錯誤拒絕存取]
    CheckPwd -- 正確 --> AdminDashboard([進入教師管理後台 admin.html])
    
    AdminDashboard --> RealtimeSync[Firebase 即時監聽 users / scores / reports]
    RealtimeSync --> UpdateStats[即時刷新三項指標看板: 學生總數 / 累積測驗次數 / 待審查問題回報]
    
    UpdateStats --> CheckReports{是否有待處理之題目回報?}
    CheckReports -- 是 (Count > 0) --> ShowBanner[🚨 頂部亮起呼吸燈警示橫幅: 提示有 X 筆待處理題目回報]
    CheckReports -- 否 (Count = 0) --> HideBanner[隱藏頂部警示橫幅]
    
    ShowBanner --> ClickBanner[點擊橫幅跳轉至「題目問題回報」分頁]
    ClickBanner --> ReviewReportCard[檢視回報卡片: 關卡單元 / 題目內容 / 學生建議]
    ReviewReportCard --> CopyBtn[點擊 📋 複製題目資訊: 一鍵複製題號、題幹與描述至剪貼簿]
    CopyBtn --> FixQuestion[教師至題庫進行文字/答案修改]
    FixQuestion --> ResolveBtn[點擊 ✓ 標記已修正並結案: 從 Firebase 移除該筆回報]
    
    AdminDashboard --> StudentProgressTab[「學生進度管理」分頁]
    StudentProgressTab --> SearchBox[即時關鍵字搜尋: 依學校、學生姓名或帳號即時過濾]
    StudentProgressTab --> ViewTable[瀏覽每位學生之解鎖進度與通關百分比進度條]
    StudentProgressTab --> ExportExcel[點擊 📊 匯出 Excel 成績表]
    ExportExcel --> GenExcel[SheetJS 即時彙整全體學生各關最高分並自動下載 .xlsx 檔案]
```

#### 教師端步驟詳解：
1. **安全認證進入**：
   * 教師於首頁點選「💼 教師/管理員後台」，輸入管理密碼 `116` 即可進入後端儀表板。
2. **🚨 題目問題回報即時提醒中心**：
   * 若有學生在測驗中提交題目疑義或錯字回報，後台頂部會自動亮起**紅色動態呼吸燈警示橫幅**。
   * 點擊橫幅可一鍵查看回報明細。
   * 每筆卡片提供「**📋 複製題目資訊**」按鈕，方便老師一鍵將題目與學生建議複製至剪貼簿進行核對。
   * 修正完畢後點擊「**✓ 標記已修正並結案**」即可清除紀錄並即時扣減待辦計數。
3. **全台學生進度即時看板**：
   * 實時列出所有已註冊學生之學校名稱、帳號、姓名、已解鎖關卡數與視覺化進度百分比條。
   * 支援即時搜尋與校名篩選。
4. **📊 32 關成績 Excel 報表匯出**：
   * 一鍵將全體學生的「學校、帳號、姓名、目前解鎖進度、第 1 關 ～ 第 32 關每關最高得分」自動產出為標準 Excel 試算表（`.xlsx`），方便匯入校內成績系統或教學檔案。

---

## 🗄️ 四、雲端資料庫架構 (Firebase Realtime Database Schema)

系統採用 Google Firebase Realtime Database（東南亞新加坡節點），資料庫結構規範如下：

```json
{
  "users": {
    "student_username_01": {
      "name": "陳小明",
      "school": "國立台南大學附中",
      "password": "user_password",
      "unlocked_level": 3,
      "registered_at": 1725178900000
    }
  },
  "scores": {
    "-P0Ppsn0Uqbx50agiiGf": {
      "username": "student_username_01",
      "name": "陳小明",
      "school": "國立台南大學附中",
      "level": 1,
      "score": 100,
      "completed_at": 1725179200000
    }
  },
  "reports": {
    "-P0Qabc123xyz456": {
      "username": "student_username_01",
      "name": "陳小明",
      "school": "國立台南大學附中",
      "level": 2,
      "level_title": "第一章第2節：觀光餐旅業的行業統計與標準分類",
      "question": "有關行政院主計總處對觀光餐旅業的分類，下列何者錯誤？",
      "choices": ["住宿及餐飲業屬I大類", "住宿業又可分成短期住宿業與長期住宿業", "六福村屬S大類", "臺灣高鐵屬H大類"],
      "answer": "B",
      "error_desc": "題目解析建議補充官方分類無長期住宿業之詳細說明",
      "reported_at": 1725179500000
    }
  }
}
```

---

## 📚 五、題庫架構與最新課綱規範 (Question Bank Design)

題庫（`questions.js`）完全符合教育部 108 課綱與最新統測試題趨勢，共規劃 **32 個挑戰關卡、640 道挑戰題次**：

### 5.1 關卡分佈表

| 關卡編號 | 關卡名稱 / 評量單元 | 題數 | 特色與規範 |
| :---: | :--- | :---: | :--- |
| **第 1 關** | 第一章第1節：觀光與餐旅的定義與本質 | 20 題 | UNWTO 定義、Hospitality 語源、觀光三要素 |
| **第 2 關** | 第一章第2節：觀光餐旅業的行業統計與標準分類 | 20 題 | **最新主計總處標準（支援服務業=O大類、休閒娛樂=S大類）** |
| **第 3 關** | 第一章第3節：觀光要素與觀光系統理論 | 20 題 | Neil Leiper 觀光地理模型、觀光吸引物分類 |
| **第 4 關** | 第一章第4節：觀光發展的經濟、社會與環境衝擊 | 20 題 | 觀光乘數效應、Doxey 刺激指數、過度旅遊、文化商品化 |
| **第 5 關** | 第一章第5節：國內外觀光組織與推動機構 | 20 題 | 觀光署、TVA、UNWTO、WTTC、PATA、ICAO、IATA |
| **第 6 關** | 第二章第1節：餐旅從業人員基本特質與專業素養 | 20 題 | 職場核心能力、從業人格特質、情緒勞務 |
| **第 7 關** | 第二章第2節：餐旅職場職業道德與國際禮儀 | 20 題 | LAST 客訴黃金法則、乘車尊卑席次、西餐禮儀、性平職場 |
| **第 8 關** | 第三章第1節：餐飲業的定義、特徵與發展史 | 20 題 | 餐廳起源 Boulanger、餐飲業特性與演進史 |
| **第 9 關** | 第三章第2節：餐飲業類別與現代連鎖經營型態 | 20 題 | 直營/加盟/委託經營、雲端廚房、外送平台經濟 |
| **第 10 關** | 第三章第3節：菜單設計原則與菜單工程獲利矩陣 | 20 題 | 明星/工馬/難題/瘦狗矩陣分析、菜單定價策略 |
| **第 11 關** | 第三章第4節：廚房組織架構與法語專業職掌 | 20 題 | Brigade 廚房編組、行政主廚與各線廚師法語職掌 |
| **第 12 關** | 第三章第5節：法俄美英四大西餐服務方式與酒水 | 20 題 | 法式桌邊推車、俄式銀盤、美式盤餐、英式家庭式服務 |
| **第 13 關** | 第三章第6節：餐飲成本控制、衛生安全與HACCP | 20 題 | 食品良好衛生規範(GHP)、HACCP 核心溫度 75℃ 管制點 |
| **第 14 關** | 第三章第7節：宴會會議規劃與餐飲採購倉儲實務 | 20 題 | 宴會席次安排、驗收標準、先進先出 (FIFO) |
| **第 15 關** | 第四章第1節：旅館業的定義、歷史與現代分類 | 20 題 | César Ritz 現代飯店之父、Statler 商務連鎖先驅、Motel 起源 |
| **第 16 關** | 第四章第2節：星級評鑑制度與民宿管理辦法 | 20 題 | 星級旅館評鑑要點、民宿客房 5 間/150㎡ 法規標準 |
| **第 17 關** | 第四章第3節：客房類型與國際房價計價制度 | 20 題 | EP、AP、MAP、CP、BP 五大計價制度、房型標準 |
| **第 18 關** | 第四章第4節：客務部組織職掌與夜間稽核作業 | 20 題 | 櫃檯、行李房、金鑰匙 Concierge、夜間稽核流程 |
| **第 19 關** | 第四章第5節：房務部清潔流程、開夜床與備品管理 | 20 題 | 房況代碼 (Clean/Dirty/OOO/DND)、開夜床 (Turndown) 禮儀 |
| **第 20 關** | 第五章第1節：旅行業的定義、發展史與產業特性 | 20 題 | Thomas Cook 近代旅行業之父、OTA 平台、Inbound/Outbound |
| **第 21 關** | 第五章第2節：旅行業法規分類、業務權限與資本額 | 20 題 | 綜合/甲種/乙種旅行業業務範圍與資本額規定、責任險 |
| **第 22 關** | 第五章第3節：國際航權理論、票務規則與GDS系統 | 20 題 | 1~9 大國際航權、Open Jaw (OJ)、GDS/BSP 票務清算系統 |
| **第 23~27 關** | 🏆 第 1 章 ～ 第 5 章 章節總整理大考 (共 5 關) | 各 20 題 | 該章節全範圍跨節次統整抽考（含精闢複習解析） |
| **第 28~32 關** | 🎯 全真模擬考 第 1 回 ～ 第 5 回 (共 5 關) | 各 20 題 | 跨全冊 1~5 章統測級高擬真綜合模考 |

### 5.2 核心題庫零重複保證 (Zero-Duplicate Algorithm)
* **第 1 關至第 22 關之 440 道核心考題**經過後端演算法嚴格查核，確認每一道題目的題幹與選項皆為 **100% 獨立原創，絕無重複出題**。

---

## 🚀 六、維護與部署作業流程 (CI/CD Maintenance)

### 6.1 題庫修改與重新編譯
若需更新或增修題庫題目，請依照下列步驟：
1. 編輯題庫生成腳本：`scratch/build_perfect_database.py`。
2. 執行腳本生成最新的 `questions.js`：
   ```powershell
   python build_perfect_database.py
   ```
3. 執行重複性校驗：
   ```powershell
   python analyze_stats.py
   # 輸出應顯示：Distinct unique core questions: 440 / 440 (ZERO duplicates!)
   ```

### 6.2 推送發布至 GitHub Pages
```powershell
$git = "C:\Users\user\AppData\Local\GitHubDesktop\app-3.6.4\resources\app\git\cmd\git.exe"
& $git add index.html game.html admin.html questions.js
& $git commit -m "Update system features and question explanations"
& $git push origin main
```
* 推送至 `main` 分支後，GitHub Pages 將於 20~30 秒內自動完成全域線上更新！

---

## 📞 七、支援與技術聯絡
* **技術架構維護**：Google DeepMind Antigravity Pair-Programming Agent
* **系統授權**：供全國高中職觀光餐旅相關科系教學與學生升學模擬測驗免費使用。
