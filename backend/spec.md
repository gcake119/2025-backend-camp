# 後端 API 規格文件

## 專案概述

本專案是一個健身教練預約平台的後端系統，使用 Node.js + Express + TypeORM + PostgreSQL 構建。系統支援使用者註冊、教練管理、課程預約、點數購買等功能。

### 技術棧

- **框架**: Express.js 4.18.2
- **ORM**: TypeORM 0.3.20
- **資料庫**: PostgreSQL
- **認證**: JWT (jsonwebtoken)
- **密碼加密**: bcrypt
- **日誌**: pino, pino-http
- **測試**: Jest, Supertest
- **容器化**: Docker, Docker Compose

### 環境配置

```bash
# 主要環境變數
DB_HOST=postgres          # 資料庫主機
DB_PORT=5432             # 資料庫埠
DB_USERNAME=postgres     # 資料庫使用者名稱
DB_PASSWORD=postgres     # 資料庫密碼
DB_DATABASE=fitness      # 資料庫名稱
JWT_SECRET=your-secret   # JWT 密鑰
PORT=3001               # 應用程式埠
```

## 資料庫結構

### Entity 關係圖

```text
User (使用者)
  ├── Coach (教練，一對一)
  ├── Course (課程，一對多)
  ├── CreditPurchase (點數購買記錄，一對多)
  └── CourseBooking (課程預約，一對多)

Coach (教練)
  ├── User (使用者，一對一)
  └── CoachLinkSkill (技能關聯，一對多)

Course (課程)
  ├── User (教練使用者，多對一)
  ├── Skill (技能，多對一)
  └── CourseBooking (預約記錄，一對多)

Skill (技能)
  ├── Course (課程，一對多)
  └── CoachLinkSkill (教練技能關聯，一對多)

CreditPackage (點數包)
  └── CreditPurchase (購買記錄，一對多)

CourseBooking (課程預約)
  ├── User (使用者，多對一)
  └── Course (課程，多對一)

CreditPurchase (點數購買)
  ├── User (使用者，多對一)
  └── CreditPackage (點數包，多對一)
```

### 資料表詳細結構

#### USER (使用者表)

| 欄位 | 類型 | 說明 | 約束 |
| ------ | ------ | ------ | ------ |
| id | UUID | 主鍵 | PRIMARY KEY, AUTO GENERATED |
| name | VARCHAR(50) | 使用者名稱 | NOT NULL |
| email | VARCHAR(320) | 電子信箱 | NOT NULL, UNIQUE |
| role | VARCHAR(20) | 角色 (USER/COACH) | NOT NULL |
| password | VARCHAR(72) | 密碼 (bcrypt 加密) | NOT NULL, SELECT=FALSE |
| created_at | TIMESTAMP | 建立時間 | NOT NULL |
| updated_at | TIMESTAMP | 更新時間 | NOT NULL |

#### COACH (教練表)

| 欄位 | 類型 | 說明 | 約束 |
| ------ | ------ | ------ | ------ |
| id | UUID | 主鍵 | PRIMARY KEY, AUTO GENERATED |
| user_id | UUID | 使用者 ID | NOT NULL, UNIQUE, FK->USER.id |
| experience_years | INTEGER | 經驗年資 | NOT NULL |
| description | TEXT | 教練簡介 | NOT NULL |
| profile_image_url | VARCHAR(2048) | 個人圖片 URL | NULLABLE |
| created_at | TIMESTAMP | 建立時間 | NOT NULL |
| updated_at | TIMESTAMP | 更新時間 | NOT NULL |

#### COURSE (課程表)

| 欄位 | 類型 | 說明 | 約束 |
| ------ | ------ | ------ | ------ |
| id | UUID | 主鍵 | PRIMARY KEY, AUTO GENERATED |
| user_id | UUID | 教練使用者 ID | NOT NULL, FK->USER.id |
| skill_id | UUID | 技能 ID | NOT NULL, FK->SKILL.id |
| name | VARCHAR(100) | 課程名稱 | NOT NULL |
| description | TEXT | 課程描述 | NOT NULL |
| start_at | TIMESTAMP | 開始時間 | NOT NULL |
| end_at | TIMESTAMP | 結束時間 | NOT NULL |
| max_participants | INTEGER | 最大參與人數 | NOT NULL |
| meeting_url | VARCHAR(2048) | 會議連結 | NOT NULL |
| created_at | TIMESTAMP | 建立時間 | NOT NULL |
| updated_at | TIMESTAMP | 更新時間 | NOT NULL |

#### SKILL (技能表)

| 欄位 | 類型 | 說明 | 約束 |
| ------ | ------ | ------ | ------ |
| id | UUID | 主鍵 | PRIMARY KEY, AUTO GENERATED |
| name | VARCHAR(50) | 技能名稱 | NOT NULL, UNIQUE |
| created_at | TIMESTAMP | 建立時間 | NOT NULL |

#### CREDIT_PACKAGE (點數包表)

| 欄位 | 類型 | 說明 | 約束 |
| ------ | ------ | ------ | ------ |
| id | UUID | 主鍵 | PRIMARY KEY, AUTO GENERATED |
| name | VARCHAR(50) | 點數包名稱 | NOT NULL, UNIQUE |
| credit_amount | INTEGER | 點數數量 | NOT NULL |
| price | NUMERIC(10,2) | 價格 | NOT NULL |
| created_at | TIMESTAMP | 建立時間 | NOT NULL |

#### CREDIT_PURCHASE (點數購買記錄表)

| 欄位 | 類型 | 說明 | 約束 |
| ------ | ------ | ------ | ------ |
| id | UUID | 主鍵 | PRIMARY KEY, AUTO GENERATED |
| user_id | UUID | 使用者 ID | NOT NULL, FK->USER.id |
| credit_package_id | UUID | 點數包 ID | NOT NULL, FK->CREDIT_PACKAGE.id |
| purchased_credits | INTEGER | 購買點數 | NOT NULL |
| price_paid | NUMERIC(10,2) | 支付金額 | NOT NULL |
| created_at | TIMESTAMP | 建立時間 | NOT NULL |
| purchase_at | TIMESTAMP | 購買時間 | NOT NULL |

#### COURSE_BOOKING (課程預約表)

| 欄位 | 類型 | 說明 | 約束 |
| ------ | ------ | ------ | ------ |
| id | UUID | 主鍵 | PRIMARY KEY, AUTO GENERATED |
| user_id | UUID | 使用者 ID | NOT NULL, FK->USER.id |
| course_id | UUID | 課程 ID | NOT NULL, FK->COURSE.id |
| booking_at | TIMESTAMP | 預約時間 | NOT NULL |
| join_at | TIMESTAMP | 加入時間 | NULLABLE |
| leave_at | TIMESTAMP | 離開時間 | NULLABLE |
| cancelled_at | TIMESTAMP | 取消時間 | NULLABLE |
| cancellation_reason | VARCHAR(255) | 取消原因 | NULLABLE |
| created_at | TIMESTAMP | 建立時間 | NOT NULL |

#### COACH_LINK_SKILL (教練技能關聯表)

| 欄位 | 類型 | 說明 | 約束 |
| ------ | ------ | ------ | ------ |
| id | UUID | 主鍵 | PRIMARY KEY, AUTO GENERATED |
| coach_id | UUID | 教練 ID | NOT NULL, FK->COACH.id |
| skill_id | UUID | 技能 ID | NOT NULL, FK->SKILL.id |

## API 端點規格

### 認證機制

大部分 API 需要 JWT 認證，在 HTTP Header 中添加：

```text
Authorization: Bearer <token>
```

### 回應格式

#### 成功回應
```json
{
  "status": "success",
  "data": {
    // 資料內容
  }
}
```

#### 失敗回應
```json
{
  "status": "failed",
  "message": "錯誤訊息"
}
```

#### 錯誤回應
```json
{
  "status": "error",
  "message": "伺服器錯誤"
}
```

---

## 1. 使用者相關 API (`/api/users`)

### 1.1 使用者註冊

**端點**: `POST /api/users/signup`

**說明**: 註冊新使用者帳號

**請求 Body**:

```json
{
  "name": "使用者名稱",
  "email": "user@example.com",
  "password": "Password123"
}
```

**密碼規則**:

- 長度 8-16 字元
- 必須包含大寫字母
- 必須包含小寫字母
- 必須包含數字

**成功回應** (201):

```json
```json
{
  "status": "success",
  "data": {
    "user": {
      "id": "uuid",
      "name": "使用者名稱"
    }
  }
}
```

**錯誤回應**:
- `400`: 欄位未填寫正確
- `400`: 密碼不符合規則
- `409`: Email 已被使用

---

### 1.2 使用者登入

**端點**: `POST /api/users/login`

**說明**: 使用者登入並取得 JWT token

**請求 Body**:

```json
{
  "email": "user@example.com",
  "password": "Password123"
}
```

**成功回應** (200):

```json
```json
{
  "status": "success",
  "data": {
    "user": {
      "id": "uuid",
      "name": "使用者名稱",
      "role": "USER"
    },
    "token": "jwt-token-string"
  }
}
```

**錯誤回應**:

- `400`: 欄位未填寫正確
- `400`: 密碼不符合規則
- `401`: Email 或密碼錯誤

---

### 1.3 取得使用者個人資料

**端點**: `GET /api/users/profile`

**認證**: Required

**成功回應** (200):
```json
{
  "status": "success",
  "data": {
    "id": "uuid",
    "name": "使用者名稱",
    "email": "user@example.com",
    "role": "USER"
  }
}
```

---

### 1.4 更新使用者個人資料

**端點**: `PUT /api/users/profile`

**認證**: Required

**請求 Body**:

```json
{
  "name": "新名稱"
}
```

**成功回應** (200):

```json
```json
{
  "status": "success",
  "data": {
    "user": {
      "id": "uuid",
      "name": "新名稱",
      "email": "user@example.com"
    }
  }
}
```

**錯誤回應**:

- `400`: 欄位未填寫正確

---

### 1.5 更新密碼

**端點**: `PUT /api/users/password`

**認證**: Required

**請求 Body**:

```json
{
  "oldPassword": "OldPassword123",
  "newPassword": "NewPassword123"
}
```

**成功回應** (200):

```json
```json
{
  "status": "success",
  "data": {
    "message": "密碼更新成功"
  }
}
```

**錯誤回應**:

- `400`: 欄位未填寫正確
- `400`: 密碼不符合規則
- `401`: 舊密碼錯誤

---

### 1.6 取得使用者點數包記錄

**端點**: `GET /api/users/credit-package`

**認證**: Required

**成功回應** (200):
```json
{
  "status": "success",
  "data": {
    "total_credits": 100,
    "purchases": [
      {
        "id": "uuid",
        "credit_package_id": "uuid",
        "purchased_credits": 50,
        "price_paid": "500.00",
        "purchase_at": "2026-01-18T10:00:00.000Z",
        "package_name": "基礎方案"
      }
    ]
  }
}
```

---

### 1.7 取得使用者課程預約記錄

**端點**: `GET /api/users/courses`

**認證**: Required

**成功回應** (200):
```json
{
  "status": "success",
  "data": [
    {
      "id": "uuid",
      "course_id": "uuid",
      "booking_at": "2026-01-18T10:00:00.000Z",
      "cancelled_at": null,
      "course": {
        "id": "uuid",
        "name": "瑜伽入門",
        "start_at": "2026-01-20T10:00:00.000Z",
        "end_at": "2026-01-20T11:00:00.000Z",
        "meeting_url": "https://meet.example.com/room"
      }
    }
  ]
}
```

---

## 2. 點數包相關 API (`/api/credit-package`)

### 2.1 取得所有點數包

**端點**: `GET /api/credit-package`

**說明**: 取得所有可購買的點數包列表

**成功回應** (200):
```json
{
  "status": "success",
  "data": [
    {
      "id": "uuid",
      "name": "基礎方案",
      "credit_amount": 50,
      "price": "500.00",
      "created_at": "2026-01-01T00:00:00.000Z"
    }
  ]
}
```

---

### 2.2 新增點數包

**端點**: `POST /api/credit-package`

**說明**: 建立新的點數包 (管理功能)

**請求 Body**:
```json
{
  "name": "進階方案",
  "credit_amount": 100,
  "price": 900
}
```

**成功回應** (201):
```json
{
  "status": "success",
  "data": {
    "id": "uuid",
    "name": "進階方案",
    "credit_amount": 100,
    "price": "900.00"
  }
}
```

**錯誤回應**:
- `400`: 欄位未填寫正確
- `409`: 點數包名稱已存在

---

### 2.3 購買點數包

**端點**: `POST /api/credit-package/:creditPackageId`

**認證**: Required

**說明**: 使用者購買指定的點數包

**成功回應** (201):
```json
{
  "status": "success",
  "data": {
    "purchase": {
      "id": "uuid",
      "user_id": "uuid",
      "credit_package_id": "uuid",
      "purchased_credits": 100,
      "price_paid": "900.00",
      "purchase_at": "2026-01-18T10:00:00.000Z"
    }
  }
}
```

**錯誤回應**:
- `400`: 找不到該點數包

---

### 2.4 刪除點數包

**端點**: `DELETE /api/credit-package/:creditPackageId`

**說明**: 刪除指定的點數包 (管理功能)

**成功回應** (200):
```json
{
  "status": "success",
  "data": {
    "message": "刪除成功"
  }
}
```

**錯誤回應**:
- `400`: 找不到該點數包

---

## 3. 技能相關 API (`/api/coaches/skill`)

### 3.1 取得所有技能

**端點**: `GET /api/coaches/skill`

**成功回應** (200):
```json
{
  "status": "success",
  "data": [
    {
      "id": "uuid",
      "name": "瑜伽",
      "created_at": "2026-01-01T00:00:00.000Z"
    }
  ]
}
```

---

### 3.2 新增技能

**端點**: `POST /api/coaches/skill`

**請求 Body**:
```json
{
  "name": "皮拉提斯"
}
```

**成功回應** (201):
```json
{
  "status": "success",
  "data": {
    "id": "uuid",
    "name": "皮拉提斯"
  }
}
```

**錯誤回應**:
- `400`: 欄位未填寫正確
- `409`: 技能名稱已存在

---

### 3.3 刪除技能

**端點**: `DELETE /api/coaches/skill/:skillId`

**成功回應** (200):
```json
{
  "status": "success",
  "data": {
    "message": "刪除成功"
  }
}
```

**錯誤回應**:
- `400`: 找不到該技能

---

## 4. 教練相關 API (`/api/coaches`)

### 4.1 取得教練列表

**端點**: `GET /api/coaches`

**Query Parameters**:
- `per`: 每頁筆數 (整數)
- `page`: 頁碼 (整數)

**範例**: `GET /api/coaches?per=10&page=1`

**成功回應** (200):
```json
{
  "status": "success",
  "data": [
    {
      "id": "uuid",
      "user_id": "uuid",
      "name": "教練名稱"
    }
  ]
}
```

**錯誤回應**:
- `400`: 欄位未填寫正確

---

### 4.2 取得教練詳細資料

**端點**: `GET /api/coaches/:coachId`

**成功回應** (200):
```json
{
  "status": "success",
  "data": {
    "user": {
      "name": "教練名稱",
      "role": "COACH"
    },
    "coach": {
      "id": "uuid",
      "user_id": "uuid",
      "experience_years": 5,
      "description": "專業瑜伽教練",
      "profile_image_url": "https://example.com/image.jpg",
      "created_at": "2026-01-01T00:00:00.000Z",
      "updated_at": "2026-01-01T00:00:00.000Z",
      "skills": ["瑜伽", "皮拉提斯"]
    }
  }
}
```

**錯誤回應**:
- `400`: 欄位未填寫正確
- `400`: 找不到該教練

---

### 4.3 取得教練的課程列表

**端點**: `GET /api/coaches/:coachId/courses`

**說明**: 取得指定教練的未來課程

**成功回應** (200):
```json
{
  "status": "success",
  "data": [
    {
      "id": "uuid",
      "name": "瑜伽入門",
      "description": "適合初學者的瑜伽課程",
      "start_at": "2026-01-20T10:00:00.000Z",
      "end_at": "2026-01-20T11:00:00.000Z",
      "max_participants": 10,
      "skill_name": "瑜伽"
    }
  ]
}
```

**錯誤回應**:
- `400`: 找不到該教練

---

## 5. 課程相關 API (`/api/courses`)

### 5.1 取得所有課程

**端點**: `GET /api/courses`

**說明**: 取得所有未來的課程列表

**成功回應** (200):
```json
{
  "status": "success",
  "data": [
    {
      "id": "uuid",
      "name": "瑜伽入門",
      "description": "適合初學者的瑜伽課程",
      "start_at": "2026-01-20T10:00:00.000Z",
      "end_at": "2026-01-20T11:00:00.000Z",
      "max_participants": 10,
      "meeting_url": "https://meet.example.com/room",
      "skill": {
        "name": "瑜伽"
      },
      "coach": {
        "name": "教練名稱"
      }
    }
  ]
}
```

---

### 5.2 預約課程

**端點**: `POST /api/courses/:courseId`

**認證**: Required

**說明**: 預約指定的課程，會扣除 1 點點數

**成功回應** (201):
```json
{
  "status": "success",
  "data": {
    "booking": {
      "id": "uuid",
      "user_id": "uuid",
      "course_id": "uuid",
      "booking_at": "2026-01-18T10:00:00.000Z"
    }
  }
}
```

**錯誤回應**:
- `400`: 找不到該課程
- `400`: 課程已額滿
- `400`: 點數不足
- `400`: 已經預約過此課程
- `400`: 無法預約自己的課程

---

### 5.3 取消課程預約

**端點**: `DELETE /api/courses/:courseId`

**認證**: Required

**請求 Body**:
```json
{
  "cancellation_reason": "臨時有事"
}
```

**成功回應** (200):
```json
{
  "status": "success",
  "data": {
    "message": "取消預約成功",
    "refunded_credits": 1
  }
}
```

**錯誤回應**:
- `400`: 找不到預約記錄
- `400`: 課程已開始，無法取消

---

## 6. 管理員/教練相關 API (`/api/admin`)

### 6.1 將使用者升級為教練

**端點**: `POST /api/admin/coaches/:userId`

**說明**: 將現有使用者升級為教練身份

**請求 Body**:
```json
{
  "experience_years": 5,
  "description": "專業瑜伽教練，擁有多年教學經驗",
  "profile_image_url": "https://example.com/image.jpg",
  "skills": ["skill-uuid-1", "skill-uuid-2"]
}
```

**成功回應** (201):
```json
{
  "status": "success",
  "data": {
    "coach": {
      "id": "uuid",
      "user_id": "uuid",
      "experience_years": 5,
      "description": "專業瑜伽教練",
      "profile_image_url": "https://example.com/image.jpg"
    }
  }
}
```

**錯誤回應**:
- `400`: 欄位未填寫正確
- `400`: 使用者不存在
- `409`: 該使用者已經是教練

---

### 6.2 教練建立課程

**端點**: `POST /api/admin/coaches/courses`

**認證**: Required (必須是教練身份)

**請求 Body**:
```json
{
  "skill_id": "uuid",
  "name": "瑜伽入門",
  "description": "適合初學者的瑜伽課程",
  "start_at": "2026-01-20T10:00:00.000Z",
  "end_at": "2026-01-20T11:00:00.000Z",
  "max_participants": 10,
  "meeting_url": "https://meet.example.com/room"
}
```

**成功回應** (201):
```json
{
  "status": "success",
  "data": {
    "course": {
      "id": "uuid",
      "user_id": "uuid",
      "skill_id": "uuid",
      "name": "瑜伽入門",
      "description": "適合初學者的瑜伽課程",
      "start_at": "2026-01-20T10:00:00.000Z",
      "end_at": "2026-01-20T11:00:00.000Z",
      "max_participants": 10,
      "meeting_url": "https://meet.example.com/room"
    }
  }
}
```

**錯誤回應**:
- `400`: 欄位未填寫正確
- `401`: 必須是教練身份

---

### 6.3 取得教練個人資料

**端點**: `GET /api/admin/coaches`

**認證**: Required (必須是教練身份)

**成功回應** (200):
```json
{
  "status": "success",
  "data": {
    "id": "uuid",
    "user_id": "uuid",
    "experience_years": 5,
    "description": "專業瑜伽教練",
    "profile_image_url": "https://example.com/image.jpg",
    "skills": [
      {
        "id": "uuid",
        "name": "瑜伽"
      }
    ]
  }
}
```

---

### 6.4 更新教練個人資料

**端點**: `PUT /api/admin/coaches`

**認證**: Required (必須是教練身份)

**請求 Body**:
```json
{
  "experience_years": 6,
  "description": "更新的教練簡介",
  "profile_image_url": "https://example.com/new-image.jpg",
  "skills": ["skill-uuid-1", "skill-uuid-2"]
}
```

**成功回應** (200):
```json
{
  "status": "success",
  "data": {
    "coach": {
      "id": "uuid",
      "experience_years": 6,
      "description": "更新的教練簡介",
      "profile_image_url": "https://example.com/new-image.jpg"
    }
  }
}
```

---

### 6.5 取得教練的課程列表

**端點**: `GET /api/admin/coaches/courses`

**認證**: Required (必須是教練身份)

**成功回應** (200):
```json
{
  "status": "success",
  "data": [
    {
      "id": "uuid",
      "name": "瑜伽入門",
      "description": "適合初學者的瑜伽課程",
      "start_at": "2026-01-20T10:00:00.000Z",
      "end_at": "2026-01-20T11:00:00.000Z",
      "max_participants": 10,
      "meeting_url": "https://meet.example.com/room",
      "skill_name": "瑜伽",
      "current_participants": 5
    }
  ]
}
```

---

### 6.6 取得課程詳細資料

**端點**: `GET /api/admin/coaches/courses/:courseId`

**認證**: Required

**成功回應** (200):
```json
{
  "status": "success",
  "data": {
    "course": {
      "id": "uuid",
      "name": "瑜伽入門",
      "description": "適合初學者的瑜伽課程",
      "start_at": "2026-01-20T10:00:00.000Z",
      "end_at": "2026-01-20T11:00:00.000Z",
      "max_participants": 10,
      "meeting_url": "https://meet.example.com/room"
    },
    "bookings": [
      {
        "id": "uuid",
        "user": {
          "name": "學員名稱"
        },
        "booking_at": "2026-01-18T10:00:00.000Z",
        "cancelled_at": null
      }
    ]
  }
}
```

---

### 6.7 更新課程資料

**端點**: `PUT /api/admin/coaches/courses/:courseId`

**認證**: Required

**請求 Body**:
```json
{
  "name": "進階瑜伽",
  "description": "適合有基礎的學員",
  "start_at": "2026-01-21T10:00:00.000Z",
  "end_at": "2026-01-21T11:30:00.000Z",
  "max_participants": 15,
  "meeting_url": "https://meet.example.com/new-room"
}
```

**成功回應** (200):
```json
{
  "status": "success",
  "data": {
    "course": {
      "id": "uuid",
      "name": "進階瑜伽",
      "description": "適合有基礎的學員",
      "start_at": "2026-01-21T10:00:00.000Z",
      "end_at": "2026-01-21T11:30:00.000Z",
      "max_participants": 15,
      "meeting_url": "https://meet.example.com/new-room"
    }
  }
}
```

---

### 6.8 取得教練收益統計

**端點**: `GET /api/admin/coaches/revenue`

**認證**: Required (必須是教練身份)

**Query Parameters**:
- `month`: 月份名稱 (january, february, march, april, may, june, july, august, september, october, november, december)

**範例**: `GET /api/admin/coaches/revenue?month=january`

**成功回應** (200):
```json
{
  "status": "success",
  "data": {
    "total": {
      "revenue": 5000,
      "participants": 50,
      "course_count": 10
    }
  }
}
```

**說明**:
- `revenue`: 該月份總收益
- `participants`: 該月份不重複參與學員數
- `course_count`: 該月份總預約次數 (未取消)

**錯誤回應**:
- `400`: 欄位未填寫正確

---

## 7. 檔案上傳 API (`/api/upload`)

### 7.1 上傳檔案

**端點**: `POST /api/upload`

**Content-Type**: `multipart/form-data`

**請求 Body**:
- `file`: 檔案 (form-data)

**成功回應** (200):
```json
{
  "status": "success",
  "data": {
    "url": "https://example.com/uploads/filename.jpg"
  }
}
```

---

## 8. 健康檢查 API

### 8.1 健康檢查

**端點**: `GET /healthcheck`

**說明**: 檢查服務是否正常運行

**成功回應** (200):
```
OK
```

---

## 中介軟體 (Middlewares)

### 認證中介軟體 (auth.js)

**功能**: 驗證 JWT token 並從資料庫取得使用者資訊

**使用方式**:
```javascript
const auth = require('./middlewares/auth')({
  secret: jwtSecret,
  userRepository: userRepository,
  logger: logger
})

router.get('/protected', auth, controller.method)
```

**驗證流程**:
1. 檢查 `Authorization` header 是否存在且格式為 `Bearer <token>`
2. 解析並驗證 JWT token
3. 從資料庫查詢使用者
4. 將使用者資訊附加到 `req.user`

**錯誤訊息**:
- `401`: Token 已過期
- `401`: 無效的 token
- `401`: 請先登入

---

### 教練身份檢查中介軟體 (isCoach.js)

**功能**: 驗證使用者是否具有教練身份

**使用方式**:
```javascript
const isCoach = require('./middlewares/isCoach')

router.post('/coaches/courses', auth, isCoach, controller.method)
```

**驗證流程**:
1. 檢查 `req.user.role` 是否為 `COACH`
2. 若不是教練，返回 403 錯誤

**錯誤訊息**:
- `403`: 無權限執行此操作

---

## 工具函式 (Utils)

### JWT 生成 (generateJWT.js)

**功能**: 生成 JWT token

**使用方式**:
```javascript
const generateJWT = require('./utils/generateJWT')

const token = generateJWT({
  id: user.id,
  role: user.role
}, secret, expiresIn)
```

---

### 日誌記錄器 (logger.js)

**功能**: 使用 pino 記錄應用程式日誌

**使用方式**:
```javascript
const logger = require('./utils/logger')('ModuleName')

logger.info('Info message')
logger.warn('Warning message')
logger.error('Error message')
```

---

## 錯誤處理

### 全域錯誤處理

所有路由都應該使用 `try-catch` 包裹，並將錯誤傳遞給 `next(error)`：

```javascript
try {
  // 業務邏輯
} catch (error) {
  logger.error('錯誤訊息:', error)
  next(error)
}
```

### 錯誤狀態碼

| 狀態碼 | 說明 |
| ------ | ------ |
| 200 | 成功 |
| 201 | 建立成功 |
| 400 | 請求錯誤 (欄位驗證失敗) |
| 401 | 未授權 (需要登入或 token 無效) |
| 403 | 禁止存取 (權限不足) |
| 409 | 衝突 (資源已存在) |
| 500 | 伺服器錯誤 |

---

## 測試

### 測試架構

- **單元測試**: `test/unit/`
- **整合測試**: `test/integration/`

### 執行測試

```bash
# 執行所有測試
npm test

# 執行單元測試
npm run test:unit

# 執行整合測試
npm run test:integration
```

### 測試設定

- **全域設定**: `test/globalSetup.js`
- **全域清理**: `test/globalTeardown.js`
- **測試排序**: `test/sequencer.js`

---

## 部署

### Docker 部署

```bash
# 啟動服務 (建立容器並啟動)
npm run start

# 重新啟動服務
npm run restart

# 停止服務
npm run stop

# 清除服務 (刪除容器和資料)
npm run clean
```

### 初始化資料庫結構

```bash
npm run init:schema
```

### 本地開發

```bash
# 啟動開發伺服器
npm run dev
```

---

## 開發規範

### 程式碼風格

- 使用 ESLint (Standard 配置)
- 使用 2 空格縮排
- 使用單引號

### 命名規範

- **變數/函式**: camelCase (例: `userName`, `getUserProfile`)
- **類別**: PascalCase (例: `UserController`)
- **常數**: UPPER_SNAKE_CASE (例: `JWT_SECRET`)
- **資料表**: UPPER_SNAKE_CASE (例: `USER`, `COURSE_BOOKING`)
- **欄位**: snake_case (例: `user_id`, `created_at`)

### Git 提交訊息

建議使用 Conventional Commits 格式：

```text
<type>: <subject>

<body>
```

類型 (type):

- `feat`: 新功能
- `fix`: 錯誤修復
- `docs`: 文件更新
- `refactor`: 重構
- `test`: 測試相關
- `chore`: 維護任務

---

## 安全性考量

### 密碼安全

- 使用 bcrypt 加密密碼 (salt rounds = 10)
- 密碼必須符合強度要求 (8-16 字元，包含大小寫字母和數字)
- 密碼欄位在查詢時預設不返回 (`select: false`)

### JWT 安全

- Token 包含使用者 ID 和角色資訊
- 建議設定合理的過期時間
- 敏感操作需要重新驗證

### SQL 注入防護

- 使用 TypeORM 參數化查詢
- 避免字串拼接 SQL

### CORS 設定

- 啟用 CORS 中介軟體
- 生產環境應限制允許的來源

---

## 效能優化

### 資料庫連線池

- 使用 TypeORM 連線池 (poolSize: 10)

### 查詢優化

- 使用 `select` 指定需要的欄位
- 適當使用 `relations` 載入關聯資料
- 避免 N+1 查詢問題

### 日誌優化

- 使用 pino (高效能日誌套件)
- 生產環境可使用 pino-pretty 格式化輸出

---

## 附錄

### 常用 SQL 操作範例

#### 查詢使用者總點數

```javascript
const totalCredits = await dataSource
  .getRepository('CreditPurchase')
  .createQueryBuilder('cp')
  .select('SUM(cp.purchased_credits)', 'total')
  .where('cp.user_id = :userId', { userId })
  .getRawOne()
```

#### 查詢課程預約數

```javascript
const bookingCount = await dataSource
  .getRepository('CourseBooking')
  .count({
    where: {
      course_id: courseId,
      cancelled_at: IsNull()
    }
  })
```

#### 查詢未來的課程

```javascript
const courses = await dataSource
  .getRepository('Course')
  .find({
    where: {
      end_at: MoreThan(new Date())
    }
  })
```

---

## 聯絡資訊

如有任何問題或建議，請聯絡開發團隊。

---

**文件版本**: 1.0.0  
**最後更新**: 2026-01-18
