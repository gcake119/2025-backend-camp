# 前端專案規格書

## 專案概述

**專案名稱：** LiveFit+ 健身教練平台  
**專案類型：** 前端 Web 應用程式  
**技術架構：** Vue 3 + Vite  
**開發日期：** 2025

### 專案簡介

LiveFit+ 是一個線上健身教練服務平台，提供用戶尋找專業健身教練、購買課程方案、預約課程等功能。平台包含三種主要角色：一般用戶、教練和管理員。

---

## 技術棧

### 核心框架與工具
- **Vue 3** (v3.5.22) - 前端框架
- **Vite** (v7.1.7) - 建置工具
- **Vue Router** (v4.6.3) - 路由管理
- **Pinia** (v3.0.4) - 狀態管理

### UI/樣式
- **Tailwind CSS** (v4.1.16) - CSS 框架
- **@tailwindcss/vite** (v4.1.16) - Tailwind CSS Vite 插件

### HTTP 請求
- **Axios** (v1.13.1) - HTTP 客戶端

### 工具函式庫
- **dayjs** (v1.11.19) - 日期時間處理
- **jwt-decode** (v4.0.0) - JWT Token 解碼
- **sweetalert2** (v11.26.3) - 彈窗提示
- **echarts** (v6.0.0) - 圖表視覺化

### 開發工具
- **ESLint** (v9.39.0) - 代碼檢查
- **@vitejs/plugin-vue** (v6.0.1) - Vite Vue 插件

---

## 專案結構

```
frontend/
├── public/                  # 靜態資源
│   ├── avatar-*.png        # 頭像圖片
│   ├── people-*.png        # 人員圖片
│   ├── logo.png            # Logo
│   └── ...
├── src/
│   ├── api/                # API 介面
│   │   ├── admin.js        # 管理員 API
│   │   ├── coaches.js      # 教練 API
│   │   ├── courses.js      # 課程 API
│   │   ├── credit-package.js # 積分套餐 API
│   │   ├── index.js        # API 統一匯出
│   │   ├── request.js      # Axios 設定與攔截器
│   │   └── users.js        # 用戶 API
│   ├── assets/             # 資源文件
│   ├── chart/              # 圖表設定檔
│   │   ├── monthlyIncomeBasicArea.js
│   │   └── purchaseRatioPie.js
│   ├── components/         # 共用組件
│   │   ├── ConfirmWhetherToRegisterModal.vue
│   │   ├── CourseModal.vue
│   │   ├── FitnessPlanSuccessModal.vue
│   │   └── RedirectModal.vue
│   ├── composables/        # 組合式函數
│   │   └── useNow.js
│   ├── config/             # 設定檔
│   │   └── routeTable.js   # 路由權限表
│   ├── layouts/            # 佈局組件
│   │   ├── LayoutHeader.vue
│   │   └── LayoutFooter.vue
│   ├── pages/              # 頁面組件
│   │   ├── admin/          # 管理員頁面
│   │   │   ├── AdminLayout.vue
│   │   │   ├── DashboardView.vue
│   │   │   ├── PromoteTrainer.vue
│   │   │   └── SkillsView.vue
│   │   ├── coach/          # 教練頁面
│   │   │   ├── CoachLayout.vue
│   │   │   ├── CoursesView.vue
│   │   │   ├── EarningsView.vue
│   │   │   └── ProfileView.vue
│   │   ├── public/         # 公開頁面
│   │   │   ├── auth/       # 認證頁面
│   │   │   │   ├── LoginView.vue
│   │   │   │   └── SignupView.vue
│   │   │   ├── CoachDetail.vue
│   │   │   ├── CoachesView.vue
│   │   │   ├── FitnessPlans.vue
│   │   │   └── HomeView.vue
│   │   └── user/           # 用戶頁面
│   │       ├── UserLayout.vue
│   │       ├── DashboardView.vue
│   │       ├── OrdersView.vue
│   │       └── ProfileView.vue
│   ├── router/             # 路由設定
│   │   └── index.js
│   ├── stores/             # Pinia Store
│   │   └── user.js         # 用戶狀態管理
│   ├── utils/              # 工具函數
│   │   ├── cookie.js       # Cookie 操作
│   │   ├── formatCurrency.js # 貨幣格式化
│   │   ├── formatDateTime.js # 日期時間格式化
│   │   ├── swalHandler.js  # SweetAlert 處理
│   │   └── timeComparison.js # 時間比較
│   ├── App.vue             # 根組件
│   ├── main.js             # 應用程式入口
│   └── style.css           # 全域樣式
├── index.html              # HTML 模板
├── package.json            # 專案依賴
├── vite.config.js          # Vite 設定
└── eslint.config.js        # ESLint 設定
```

---

## 路由架構

### 公開路由（無需認證）

| 路徑 | 組件 | 說明 |
|------|------|------|
| `/` | `HomeView.vue` | 首頁 |
| `/coaches` | `CoachesView.vue` | 教練列表頁 |
| `/coaches/:coachId` | `CoachDetail.vue` | 教練詳細頁 |
| `/fitness-plans` | `FitnessPlans.vue` | 健身方案頁 |
| `/login` | `LoginView.vue` | 登入頁 |
| `/signup` | `SignupView.vue` | 註冊頁 |

### 用戶路由（需 USER 角色）

| 路徑 | 組件 | 說明 |
|------|------|------|
| `/user` | `UserLayout.vue` | 用戶佈局（重定向至 `/user/dashboard`） |
| `/user/dashboard` | `DashboardView.vue` | 課程儀表板 |
| `/user/profile` | `ProfileView.vue` | 個人資料檢視 |
| `/user/orders` | `OrdersView.vue` | 訂單頁面 |

### 教練路由（需 COACH 角色）

| 路徑 | 組件 | 說明 |
|------|------|------|
| `/coach` | `CoachLayout.vue` | 教練佈局（重定向至 `/coach/profile`） |
| `/coach/profile` | `ProfileView.vue` | 教練資料 |
| `/coach/courses` | `CoursesView.vue` | 課程管理 |
| `/coach/earnings` | `EarningsView.vue` | 課程分潤 |

### 管理員路由（需認證）

| 路徑 | 組件 | 說明 |
|------|------|------|
| `/admin` | `AdminLayout.vue` | 管理員佈局（重定向至 `/admin/dashboard`） |
| `/admin/dashboard` | `DashboardView.vue` | 管理員儀表板 |
| `/admin/skills` | `SkillsView.vue` | 技能管理 |
| `/admin/promote-trainer` | `PromoteTrainer.vue` | 提升訓練師 |

### 路由守衛

- **身份驗證**：使用 `beforeEach` 守衛檢查 token
- **角色權限**：根據 `meta.requiredRole` 限制路由訪問
- **自動登入**：從 Cookie 中讀取 token 並自動獲取用戶資料

---

## API 介面

### 基礎設定

- **Base URL：** 從環境變數 `VITE_API_BASE_URL` 讀取
- **請求超時：** 10 秒
- **認證方式：** Bearer Token（從 Cookie 中讀取）

### API 分類

#### 1. 用戶 API (`users.js`)

| 方法 | 端點 | 說明 | 認證 |
|------|------|------|------|
| POST | `/users/login` | 登入 | 否 |
| POST | `/users/signup` | 註冊 | 否 |
| GET | `/users/courses` | 獲取用戶課程 | 是 |
| GET | `/users/credit-package` | 獲取用戶積分套餐 | 是 |
| GET | `/users/profile` | 獲取用戶資料 | 是 |
| PUT | `/users/profile` | 更新用戶資料 | 是 |
| PUT | `/users/password` | 更新密碼 | 是 |

#### 2. 教練 API (`coaches.js`)

| 方法 | 端點 | 說明 | 認證 |
|------|------|------|------|
| GET | `/coaches/?per={per}&page={page}` | 獲取教練列表 | 否 |
| GET | `/coaches/:coachId` | 獲取教練詳細資料 | 否 |
| GET | `/coaches/:coachId/courses` | 獲取教練課程 | 否 |
| GET | `/coaches/skill` | 獲取技能列表 | 否 |
| POST | `/coaches/skill` | 新增技能 | 是 |
| DELETE | `/coaches/skill/:skillId` | 刪除技能 | 是 |

#### 3. 課程 API (`courses.js`)

| 方法 | 端點 | 說明 | 認證 |
|------|------|------|------|
| POST | `/courses/:id` | 預約課程 | 是 |
| DELETE | `/courses/:id` | 取消課程 | 是 |

#### 4. 積分套餐 API (`credit-package.js`)

| 方法 | 端點 | 說明 | 認證 |
|------|------|------|------|
| GET | `/credit-package` | 獲取積分套餐列表 | 否 |
| POST | `/credit-package/:id` | 購買積分套餐 | 是 |

#### 5. 管理員 API (`admin.js`)

| 方法 | 端點 | 說明 | 認證 |
|------|------|------|------|
| GET | `/admin/coaches` | 獲取所有教練 | 是 |
| PUT | `/admin/coaches` | 更新教練資料 | 是 |
| GET | `/admin/coaches/courses` | 獲取所有教練課程 | 是 |
| GET | `/admin/coaches/courses/:id` | 獲取特定課程 | 是 |
| POST | `/admin/coaches/courses` | 新增課程 | 是 |
| PUT | `/admin/coaches/courses/:id` | 更新課程 | 是 |
| GET | `/admin/coaches/revenue?month={month}` | 獲取月收入 | 是 |
| POST | `/admin/coaches/:id` | 提升用戶為教練 | 是 |

### 請求攔截器

- **自動添加 Token**：根據 `routeTable.js` 設定決定是否需要認證
- **統一錯誤處理**：處理 400、401、403、404、500 等錯誤狀態碼

### 公開路由配置

在 `config/routeTable.js` 中定義無需認證的路由：

- `POST /users`: `/signup`, `/login`
- `GET /courses`: 全部
- `GET /credit-package`: 全部
- `POST /credit-package`: 全部
- `DELETE /credit-package/:id`: 全部
- `GET /coaches`: `/skill`, `/:coachId`, `/?per=&page=`, `/:coachId/courses`
- `POST /coaches`: `/skill`
- `DELETE /coaches`: `/skill/:skillId`
- `POST /admin`: `/coaches/:userId`（提升為教練）

---

## 狀態管理 (Pinia)

### User Store (`stores/user.js`)

管理當前登入用戶的狀態：

```javascript
{
  state: {
    name: "",      // 用戶名稱
    role: "",      // 用戶角色 (USER/COACH/ADMIN)
  },
  actions: {
    setCurrentUser({ name, role }),  // 設定用戶資料
    setUserName(name),               // 更新用戶名稱
  }
}
```

---

## 功能模組

### 1. 認證模組

#### 登入功能
- 使用 Email 和密碼登入
- Token 儲存在 Cookie 中
- 自動跳轉至對應角色的首頁

#### 註冊功能
- 用戶註冊新帳號
- 註冊成功後自動登入

#### 登出功能
- 清除 Cookie 中的 Token
- 重置 Store 中的用戶狀態
- 跳轉至首頁

### 2. 首頁模組 (`HomeView.vue`)

包含以下區塊：
- **Hero Banner**：主視覺與 CTA
- **探索課程**：課程特色介紹
- **優勢說明**：平台優勢展示
- **使用流程**：三步驟說明
- **服務介紹**：服務項目展示
- **使命說明**：品牌使命
- **用戶評價**：評價展示
- **教練團隊**：教練輪播展示
- **課程方案**：方案價格展示
- **註冊 CTA**：最後行動呼籲

### 3. 教練模組

#### 教練列表 (`CoachesView.vue`)
- 顯示所有教練
- 支援分頁
- 顯示教練基本資訊與技能

#### 教練詳細頁 (`CoachDetail.vue`)
- 顯示教練詳細資料
- 顯示教練開設的課程
- 預約課程功能

### 4. 用戶模組

#### 課程儀表板 (`DashboardView.vue`)
- 顯示已預約課程數
- 顯示剩餘課程數
- 顯示總課程數統計

#### 個人資料 (`ProfileView.vue`)
- 個人資料編輯
- 密碼修改

#### 訂單頁面 (`OrdersView.vue`)
- 顯示購買的積分套餐記錄
- 訂單詳情查看

### 5. 教練模組（教練端）

#### 教練資料 (`ProfileView.vue`)
- 編輯教練個人資料
- 管理教練技能

#### 課程管理 (`CoursesView.vue`)
- 查看已開設的課程
- 管理課程狀態

#### 課程分潤 (`EarningsView.vue`)
- 查看收入統計
- 圖表視覺化收入數據

### 6. 管理員模組

#### 管理員儀表板 (`DashboardView.vue`)
- 平台數據統計
- 收入圖表展示
- 使用 ECharts 進行數據視覺化

#### 技能管理 (`SkillsView.vue`)
- 新增技能
- 刪除技能
- 技能列表管理

#### 提升訓練師 (`PromoteTrainer.vue`)
- 將一般用戶提升為教練
- 設定教練權限

### 7. 健身方案模組 (`FitnessPlans.vue`)

- 顯示所有可購買的積分套餐
- 套餐詳情與價格
- 購買功能

---

## 共用組件

### 1. LayoutHeader.vue
- 響應式導航欄
- 桌面版與行動版選單
- 根據登入狀態顯示不同內容
- 登出功能

### 2. LayoutFooter.vue
- 頁尾資訊
- 社群媒體連結
- 聯絡資訊

### 3. Modal 組件
- `ConfirmWhetherToRegisterModal.vue` - 確認註冊提示
- `CourseModal.vue` - 課程詳情彈窗
- `FitnessPlanSuccessModal.vue` - 購買成功提示
- `RedirectModal.vue` - 重定向提示

---

## 工具函數

### Cookie 操作 (`utils/cookie.js`)
- `getDataFromCookieByKey(key)` - 讀取 Cookie
- `setKeyFromCookie(name, token, exp)` - 設定 Cookie
- `removeCookie(name)` - 刪除 Cookie

### 格式化工具
- `formatCurrency.js` - 貨幣格式化（千分位）
- `formatDateTime.js` - 日期時間格式化

### 其他工具
- `swalHandler.js` - SweetAlert2 統一處理
- `timeComparison.js` - 時間比較邏輯

---

## 環境變數

需要在 `.env` 中設定的變數：

```env
VITE_API_BASE_URL=http://your-api-url.com
```

---

## 開發指令

```bash
# 安裝依賴
npm install

# 開發模式
npm run dev

# 建置生產版本
npm run build

# 預覽生產版本
npm run preview
```

---

## 權限控制

### 路由層級權限
- **公開路由**：任何人都可訪問
- **需認證路由**：需要登入（`meta.requiresAuth: true`）
- **角色路由**：需要特定角色（`meta.requiredRole: "USER" | "COACH"`）

### API 層級權限
- 根據 `routeTable.js` 設定決定 API 請求是否需要 Token
- 請求攔截器自動添加 Token（如需認證）

### 頁面層級權限
- 在 `beforeEach` 守衛中檢查用戶角色
- 不符合權限時自動重定向

---

## 響應式設計

- 使用 Tailwind CSS 的響應式斷點
- 支援桌面版、平板、手機版
- 行動版導航使用下拉選單

### 主要斷點
- `sm:` - 640px 以上
- `md:` - 768px 以上
- `lg:` - 1024px 以上
- `xl:` - 1280px 以上
- `2xl:` - 1536px 以上

---

## 顏色系統

專案使用 Tailwind CSS 自訂顏色系統：

- **primary-0 ~ primary-900** - 主色調（深色系）
- **secondary-800** - 強調色（橙色系）
- **success-600** - 成功色
- **neutral-200/300/400** - 中性色

詳細顏色規範請參考 `COLOR_GUIDE.md`

---

## 錯誤處理

### HTTP 錯誤處理
在 `request.js` 的響應攔截器中統一處理：
- **400** - 請求錯誤
- **401** - 未授權，需重新登入
- **403** - 沒有權限
- **404** - 資源不存在
- **500** - 伺服器錯誤

### 用戶提示
- 使用 SweetAlert2 顯示錯誤訊息
- 使用全局 `$swal` 屬性

---

## 性能優化

1. **路由懶加載**：使用動態 import 載入頁面組件
2. **組件按需載入**：大型組件使用異步載入
3. **圖片優化**：使用適當的圖片格式和尺寸
4. **代碼分割**：Vite 自動進行代碼分割

---

## 瀏覽器支援

- Chrome（最新版）
- Firefox（最新版）
- Safari（最新版）
- Edge（最新版）

---

## 安全性考量

1. **Token 儲存**：使用 HttpOnly Cookie（建議後端設定）
2. **XSS 防護**：Vue 自動轉義用戶輸入
3. **CSRF 防護**：依賴後端實現
4. **路由權限**：前端與後端雙重驗證

---

## 未來擴展建議

1. **國際化 (i18n)**：支援多語言
2. **深色模式**：支援主題切換
3. **PWA 支援**：離線使用
4. **即時通訊**：用戶與教練即時溝通
5. **推送通知**：課程提醒等功能
6. **單元測試**：使用 Vitest 進行測試
7. **E2E 測試**：使用 Playwright 或 Cypress

---

## 專案維護

### 代碼規範
- 遵循 ESLint 規則
- 使用 Vue 3 Composition API
- 組件命名使用 PascalCase
- 文件命名使用 PascalCase（組件）或 camelCase（工具函數）

### 版本控制
- 使用 Git 進行版本控制
- 遵循語義化版本控制

---

## 聯絡資訊

如有任何問題或建議，請聯繫開發團隊。

---

**文件版本：** 1.0  
**最後更新：** 2025年
