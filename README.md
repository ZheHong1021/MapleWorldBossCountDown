# 🍁 楓之谷世界王倒數計時器

一個專為楓之谷 (MapleStory) 玩家設計的世界王倒數計時器，幫助你追蹤各個頻道的 BOSS 重生時間。

## ✨ 功能特色

- 📋 **預設 BOSS 資料**：內建常見世界王的重生時間範圍
- ⏰ **智能狀態顯示**：
  - ⏳ 等待重生（藍色）- 未到最短重生時間
  - ✨ 準備重生（綠色閃爍）- 在重生時間範圍內
  - 🔥 已超時（紅色警示）- 超過最長重生時間
- 🔄 **一鍵重置**：擊殺 BOSS 後可立即重新計時
- 📍 **多頻道支援**：可同時追蹤多個頻道的 BOSS
- 🗺️ **多位置顯示**：支援 BOSS 出現在多個位置的情況
- 🕒 **自訂時間**：可設定過去的擊殺時間或使用當前時間
- 🎨 **主題切換**：可在深色與淺色主題間切換，記住使用者偏好
- 💾 **本地儲存**：資料保存在瀏覽器，重新整理不丟失
- 🖼️ **BOSS 圖片**：每個世界王都有對應圖片
- 📱 **響應式設計**：完美支援手機和電腦
- 🌐 **現代化介面**：美觀的按鈕設計，清楚的狀態指示

## 🚀 線上版本

👉 **[立即使用](https://ZheHong1021.github.io/MapleWorldBossCountDown/)**

## 🎮 支援的 BOSS

以下為目前支援的 BOSS 名單（依 `public/data.json` 配置自動同步，若有異動請參考該檔案）：
- 雪毛怪人
- 瘋狂喵z客
- 黑輪王
- 艾利傑
- 巴洛古
- 肯德熊
- 九尾妖狐
- 葛雷金剛
- 殭屍蘑菇王
- 蘑菇王
- 書生幽靈
- 厄運死神
- 沼澤巨鱷
- 雪山魔女
- 巨大深山人蔘
- 喵怪仙人
- 仙人娃娃
- 咕咕鐘
- 冥界幽靈
- 巨居蟹
- 殭屍猴王
- 樹妖王
- 紅寶王

如需新增或修改 BOSS，請直接編輯 `public/data.json`。

## 📖 使用說明

### 1. 新增計時器
- 選擇要追蹤的 BOSS
- 輸入頻道名稱（例如：1168頻道）
- **選擇時間設定**：
  - 不勾選「自訂開始時間」：使用當前時間作為 BOSS 擊殺時間
  - 勾選「自訂開始時間」：設定 BOSS 實際被擊殺的時間
  - 可使用快速按鈕：5分鐘前、10分鐘前、30分鐘前、現在
- 點擊「開始倒數計時」

### 2. 管理計時器
- 🔄 **重新計時**：擊殺 BOSS 後點擊此按鈕重新開始倒數
- ❌ **移除**：刪除不需要的計時器

### 3. 主題切換
- 點擊右上角的 🌙/☀️ 按鈕切換深色/淺色主題
- 應用程式會記住您的主題偏好
- 首次使用時會根據系統偏好自動設定

### 4. 查看資訊
- 倒數時間以 `時:分:秒` 格式顯示
- 顯示開始時間和預計重生時間範圍
- 右上角顯示當前系統時間

## 🛠️ 本地開發

```bash
# 安裝依賴
npm install

# 啟動開發伺服器
npm run dev

# 建置生產版本
npm run build

# 預覽建置結果
npm run preview

# 部署到 GitHub Pages
npm run deploy
```

## 🛠️ 技術架構

- **前端框架**：Vue 3 (Composition API)
- **建構工具**：Vite
- **樣式**：CSS3 (支援深色/淺色主題)
- **資料格式**：JSON
- **部署平台**：GitHub Pages
- **儲存方式**：localStorage (瀏覽器本地儲存)

## 📁 專案結構

```
├── .github/
│   └── workflows/
│       └── deploy.yml         # GitHub Actions 自動部署
├── public/
│   └── data.json             # BOSS 資料配置
├── src/
│   ├── components/
│   │   ├── TimerCard.vue     # 計時器卡片組件
│   │   ├── TimerForm.vue     # 新增計時器表單
│   │   ├── TimerList.vue     # 計時器列表
│   │   ├── TopControls.vue   # 頂部控制列
│   │   └── AppFooter.vue     # 頁面底部
│   ├── assets/
│   │   └── images/           # BOSS 圖片資源
│   ├── App.vue              # 主應用程式組件
│   ├── main.js              # 應用程式入口點
│   └── style.css            # 全域樣式
├── index.html               # HTML 模板
├── package.json             # 專案配置
├── vite.config.js          # Vite 配置
└── README.md               # 專案說明文件
```

## 🔧 自訂 BOSS 資料

編輯 `public/data.json` 檔案來新增或修改 BOSS 資訊：

```json
{
  "bosses": [
    {
      "id": "unique_id",
      "name": "BOSS名稱",
      "minRespawnMinutes": 45,
      "maxRespawnMinutes": 68,
      "location": "單一位置說明"
    },
    {
      "id": "multi_location_boss",
      "name": "多位置BOSS",
      "minRespawnMinutes": 60,
      "maxRespawnMinutes": 120,
      "location": [
        "位置一",
        "位置二",
        "位置三"
      ]
    }
  ]
}
```

### 位置格式說明
- **單一位置**：使用字串格式 `"location": "位置名稱"`
- **多個位置**：使用陣列格式 `"location": ["位置1", "位置2", "位置3"]`

## 📊 資料來源

感謝 [中文 Artale Monster Database](https://a2983456456.github.io/artale-drop/) 提供的 BOSS 資料。

## 🚀 部署說明

此專案使用 GitHub Actions 自動部署到 GitHub Pages：

1. 推送程式碼到 main 分支
2. GitHub Actions 自動建置和部署
3. 約 2-3 分鐘後網站就會更新

## 🤝 貢獻

歡迎提交 Issue 或 Pull Request 來改善這個專案！

### 貢獻指南
1. Fork 此專案
2. 建立功能分支 (`git checkout -b feature/AmazingFeature`)
3. 提交變更 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 開啟 Pull Request

## 📄 授權

MIT License

## 🔗 相關連結

- [線上版本](https://YOUR_USERNAME.github.io/MapleWorldBossCountDown/)
- [GitHub 倉庫](https://github.com/YOUR_USERNAME/MapleWorldBossCountDown)
- [Issue 回報](https://github.com/YOUR_USERNAME/MapleWorldBossCountDown/issues)
- [資料來源](https://a2983456456.github.io/artale-drop/)

---


Made with ❤️ for MapleStory players
