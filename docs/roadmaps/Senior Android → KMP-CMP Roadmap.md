# Senior Android → KMP/CMP Roadmap

> **Duration:** 12 Weeks\
> **Goal:** Senior Android / Senior Mobile / KMP Engineer\
> **Projects:** 2\
> **Time Commitment:** \~5–7 hours/week\
> **Product Goal:** ทุก Project ต้องใช้งานได้จริง ไม่ใช่ Demo App

---

# 🎯 Goals

ภายใน 12 สัปดาห์:

- [ ] พร้อมสมัคร **Senior Android Engineer**
- [ ] มี Pure Android application ที่ใช้งานจริง
- [ ] มี KMP + Compose Multiplatform application
- [ ] อธิบาย Architecture และ Trade-offs ได้
- [ ] มี Android + iOS experience ผ่าน KMP/CMP
- [ ] เข้าใจ Kotlin Backend
- [ ] Implement Offline-first Architecture
- [ ] Implement Reliable Data Synchronization
- [ ] Implement Observability ด้วย OpenTelemetry
- [ ] มี GitHub Portfolio พร้อมสำหรับ Interview

Target Roles:

- Senior Android Engineer
- Senior Mobile Engineer
- Kotlin Developer
- KMP Engineer
- Mobile Software Engineer

---

# 🏠 Project #1 — HomeKeep

## Home Inventory & Warranty Manager

> **“จำไม่ได้ว่าของอยู่ไหน ซื้อเมื่อไร หรือประกันหมดเมื่อไร ให้ HomeKeep จำแทน”**

HomeKeep เป็น Android application สำหรับจัดการทรัพย์สินและของใช้ภายในบ้าน

สามารถบันทึก:

- ของชิ้นนี้คืออะไร
- อยู่ที่ไหน
- ซื้อเมื่อไร
- ราคาเท่าไร
- ประกันกี่ปี
- ใบเสร็จอยู่ไหน
- Serial Number
- หมดประกันเมื่อไร
- อยู่ในกล่อง/ตู้ไหน

Application ต้องสามารถใช้งานได้ **100% Offline** โดยไม่ต้อง Login

Cloud Sync เป็น optional feature ในอนาคต

---

# 🎯 Product Principles

HomeKeep ต้อง:

1. ใช้งานได้จริง
2. เปิดแล้วเริ่มใช้ได้ทันที
3. ไม่บังคับ Login
4. Offline-first
5. Privacy-first
6. UI เรียบง่าย
7. เพิ่มของหนึ่งชิ้นได้ภายใน \~30 วินาที
8. Search ของได้รวดเร็ว
9. Architecture รองรับ Cloud Sync ในอนาคต

---

# 📱 Core Features

## Item Management

แต่ละ Item มีข้อมูล:

```text
Item

name
description

category

location
container

quantity

purchaseDate
purchasePrice

warrantyPeriod
warrantyExpiry

serialNumber

photo
receipt

note
```

ตัวอย่าง:

```text
☕ เครื่องชงกาแฟ

Dunches CM4200

📍 ห้องครัว
   └── Coffee Corner

ซื้อเมื่อ
2 Sep 2026

ราคา
฿3,990

ประกัน
1 ปี

หมดประกัน
2 Sep 2027

Serial Number
CM4200-XXXX

🧾 Receipt
📷 Photo
```

---

# 📍 Location System

ของทุกชิ้นสามารถระบุตำแหน่งแบบ hierarchy ได้

```text
Home

├── Living Room
│
├── Kitchen
│   ├── Coffee Corner
│   └── Cabinet
│
├── Bedroom
│
├── Storage Room
│   ├── Cabinet A
│   └── Cabinet B
│
└── Garage
```

รองรับ Container:

```text
Storage Room
      ↓
Cabinet A
      ↓
Box A12
      ↓
HDMI Cable
Adapter
Power Bank
```

---

# 🔎 Search

ค้นหาจาก:

- Item Name
- Category
- Location
- Container
- Serial Number
- Note

ตัวอย่าง:

```text
Search: "HDMI"

HDMI Cable 2m
📍 Storage Room
   → Cabinet A
   → Box A12
```

---

# ⏰ Warranty Reminder

ระบบแจ้งเตือนก่อนประกันหมด

ตัวอย่าง:

```text
⚠️ Warranty Expiring

เครื่องชงกาแฟ
ประกันจะหมดในอีก 30 วัน

2 Sep 2027
```

รองรับ:

- 7 days
- 30 days
- 90 days

---

# 📦 QR Container

Container สามารถมี QR Code ได้

ตัวอย่าง:

```text
Box A12

[ QR CODE ]
```

ติด QR ไว้ที่กล่อง

เมื่อ Scan:

```text
Box A12

12 Items

HDMI Cable
USB Hub
Power Bank
Adapter
LAN Cable
...
```

Use Case:

> ไม่ต้องเปิดทุกกล่องเพื่อดูว่าของอยู่ไหน

---

# 🏗 Tech Stack

## Android

- Kotlin
- Jetpack Compose
- Material 3
- Coroutines
- Flow / StateFlow
- Clean Architecture
- MVVM
- Hilt
- Room
- WorkManager
- Navigation Compose
- CameraX
- Barcode / QR Scanner
- JUnit
- MockK / Fake
- Compose UI Test

---

# 🏛 Architecture

```text
Compose UI
     ↓
ViewModel
     ↓
UseCase
     ↓
Repository
     ↓
Room
```

Room เป็น **Single Source of Truth**

```text
Room
 ↓
Flow
 ↓
Repository
 ↓
UseCase
 ↓
ViewModel
 ↓
Compose
```

Architecture ต้องเตรียมพร้อมสำหรับอนาคต:

```text
                Repository
                    │
           ┌────────┴────────┐
           │                 │
        Local              Remote
        Room                API
           │                 │
           └──── Sync ───────┘
```

แต่ **Remote/Cloud ยังไม่ทำใน v1**

---

# 🗓 Month 1 — HomeKeep / Senior Android

# Week 1 — Product Foundation

## Product

ก่อนเขียน Feature:

- [ ] Define MVP
- [ ] Define User Flow
- [ ] Define Item Model
- [ ] Define Location Model
- [ ] Define Category Model
- [ ] Define Container Model
- [ ] Define Navigation
- [ ] Sketch 5 main screens

Main screens:

```text
Home
 ↓
Items
 ↓
Item Detail
 ↓
Add/Edit Item

+

Locations
```

## Kotlin Review

- [ ] Coroutines
- [ ] CoroutineScope
- [ ] Structured Concurrency
- [ ] Cancellation
- [ ] Exception Handling
- [ ] Flow
- [ ] StateFlow
- [ ] SharedFlow
- [ ] Dispatcher

## Project Setup

- [ ] Create Git repository
- [ ] Gradle Version Catalog
- [ ] Compose
- [ ] Material 3
- [ ] Hilt
- [ ] Room
- [ ] Navigation
- [ ] Detekt / Ktlint
- [ ] Basic CI

Suggested structure:

```text
app

core
├── database
├── model
├── designsystem
└── common

data
├── local
├── mapper
└── repository

domain
├── model
├── repository
└── usecase

feature
├── home
├── item
├── location
├── category
└── settings
```

## Week 1 Checkpoint

Application สามารถ:

```text
Open App
   ↓
Home
   ↓
Items
   ↓
Add Item
```

ได้

---

# Week 2 — Make It Usable

เป้าหมาย:

> **สิ้น Week 2 ต้องติดตั้งบนมือถือและเริ่มใช้กับของในบ้านจริง**

## Item

Implement:

- [ ] Add Item
- [ ] Edit Item
- [ ] Delete Item
- [ ] Item Detail
- [ ] Item List

## Data

Implement:

- [ ] Room Database
- [ ] Item Entity
- [ ] Location Entity
- [ ] Category Entity
- [ ] Container Entity
- [ ] DAO
- [ ] Repository
- [ ] Flow

## Location

Implement:

```text
Home
 ↓
Room
 ↓
Cabinet
 ↓
Container
```

## Photo

- [ ] Take Photo
- [ ] Select Photo
- [ ] Store photo reference
- [ ] Display photo

## Search

Implement basic search:

```text
Item Name
Category
Location
Container
```

## Week 2 Checkpoint

เพิ่มของจริงในบ้านอย่างน้อย:

**20 Items**

ตัวอย่าง:

```text
Coffee Machine
Laptop
Monitor
Car Tools
Power Bank
Baby Items
Documents
Electronics
```

Success criteria:

> ใช้ HomeKeep แทนการจำเองได้จริง

---

# Week 3 — Product Value

สัปดาห์นี้เน้น Feature ที่ทำให้ HomeKeep ไม่ใช่ CRUD App

## Warranty

Implement:

- [ ] Purchase Date
- [ ] Purchase Price
- [ ] Warranty Period
- [ ] Warranty Expiry
- [ ] Warranty Status

Status:

```text
ACTIVE
EXPIRING
EXPIRED
```

## Reminder

ใช้ WorkManager / Notification

```text
Warranty
   ↓
30 days remaining
   ↓
WorkManager
   ↓
Notification
```

รองรับ:

- [ ] 7 days
- [ ] 30 days
- [ ] 90 days

## Receipt

- [ ] Attach Receipt
- [ ] View Receipt
- [ ] Replace Receipt

## QR Container

Implement:

```text
Container
   ↓
Generate QR
   ↓
Print / Save
   ↓
Attach to Box
```

Scan:

```text
Camera
 ↓
QR
 ↓
Container ID
 ↓
Container Detail
 ↓
Items
```

## Week 3 Checkpoint

ใช้จริง:

- [ ] สร้าง Container อย่างน้อย 3 กล่อง
- [ ] Generate QR
- [ ] ติด QR จริงอย่างน้อย 1 กล่อง
- [ ] Scan แล้วเปิดรายการของได้
- [ ] มี Item ที่ตั้ง Warranty Reminder จริง

---

# Week 4 — Production Quality

## Testing

- [ ] DAO tests
- [ ] Repository tests
- [ ] UseCase tests
- [ ] Coroutine tests
- [ ] Flow tests
- [ ] ViewModel tests
- [ ] Compose UI tests

## Performance

ทดสอบ:

```text
100 Items
500 Items
1,000 Items
5,000 Items
```

ตรวจสอบ:

- [ ] Search performance
- [ ] Room query performance
- [ ] Compose recomposition
- [ ] Image loading
- [ ] Memory usage

## UX

ตรวจสอบ:

- [ ] Empty State
- [ ] Loading
- [ ] Error
- [ ] Delete confirmation
- [ ] Form validation
- [ ] Accessibility
- [ ] Dark Mode

## Backup

เพิ่ม:

- [ ] Export local backup
- [ ] Import backup

อย่างน้อยควรสามารถ Backup database/data ของผู้ใช้ได้ก่อนนำไปใช้จริง

---

# 📦 HomeKeep v1.0

สิ้น Month 1 ต้องสามารถ:

- [ ] Add Item
- [ ] Edit Item
- [ ] Delete Item
- [ ] Search
- [ ] Category
- [ ] Location
- [ ] Container
- [ ] Photo
- [ ] Receipt
- [ ] Warranty
- [ ] Reminder
- [ ] QR Container
- [ ] Backup / Restore
- [ ] Fully Offline
- [ ] No Login

และต้องมีข้อมูลจริงในเครื่องอย่างน้อย:

> **30–50 Items**

---

# 🎯 Month 1 Milestone

HomeKeep ต้องผ่านคำถามนี้:

> **“ถ้าหยุดพัฒนาแอปวันนี้ ฉันยังอยากเก็บมันไว้ในมือถือไหม?”**

ถ้าคำตอบคือ **Yes** → Product มี Value

ถ้าคำตอบคือ **No** → หาสาเหตุก่อนเพิ่ม Feature

---

# 🚀 HomeKeep Future Roadmap

สิ่งเหล่านี้ **ห้ามทำใน Month 1**

## v1.1

```text
Cloud Backup
Multi-device
Sync Engine
```

## v1.2

```text
Family Sharing
Shared Home
Roles
```

## v1.3

```text
Consumables
Low Stock
Shopping List
```

## v1.4

```text
OCR Receipt
Auto Warranty Detection
AI Search
```

ตัวอย่าง:

```text
"เครื่องชงกาแฟอยู่ไหน?"

HomeKeep AI

→ Kitchen
→ Coffee Corner
```

---

# 🌐 Project #2 — FamilyFlow

**Kotlin Multiplatform Family Finance App**

Cross-platform application สำหรับจัดการรายรับ รายจ่าย และงบประมาณครอบครัว

Platforms:

```text
Android
   +
iOS
```

## Tech Stack

- Kotlin Multiplatform
- Compose Multiplatform
- Coroutines
- Flow
- Ktor
- kotlinx.serialization
- Koin
- Room KMP / SQLDelight
- Kotlin Test

Optional Backend:

- Kotlin
- Spring Boot
- PostgreSQL
- Docker
- OpenTelemetry

---

# 🗓 Month 2 — Kotlin Multiplatform

## Week 5 — KMP Fundamentals

Learn:

- [ ] KMP project structure
- [ ] `commonMain`
- [ ] `androidMain`
- [ ] `iosMain`
- [ ] `expect / actual`
- [ ] Platform abstraction

Build:

```text
Android ─┐
         ├── Shared Kotlin
iOS ─────┘
```

Checkpoint:

- [ ] Android target works
- [ ] iOS target works
- [ ] Shared Kotlin logic works

---

# Week 6 — Shared Architecture

Move into `commonMain`:

- [ ] Models
- [ ] Repository interfaces
- [ ] UseCases
- [ ] Validation
- [ ] State management

Architecture:

```text
commonMain

├── domain
│   ├── model
│   ├── repository
│   └── usecase
│
├── data
│   ├── local
│   ├── remote
│   └── repository
│
└── presentation
```

---

# Week 7 — Networking & Database

Networking:

- [ ] Ktor
- [ ] kotlinx.serialization
- [ ] Error mapping
- [ ] Timeout
- [ ] Retry

Database:

- [ ] Room KMP / SQLDelight
- [ ] Repository
- [ ] Local Cache

Architecture:

```text
Repository
   │
 ┌─┴─────────┐
 ↓           ↓
Local       Remote
 DB          API
```

---

# Week 8 — Compose Multiplatform

Screens:

- [ ] Dashboard
- [ ] Transactions
- [ ] Add Expense
- [ ] Categories
- [ ] Budget
- [ ] Settings

Support:

- [ ] Android
- [ ] iOS

Handle:

- [ ] Loading
- [ ] Error
- [ ] Empty State
- [ ] Offline State

# 🎯 Month 2 Milestone

**FamilyFlow v1.0**

Working on:

```text
Android
+
iOS
```

---

# 🗓 Month 3 — Backend & Production Engineering

# Week 9 — Kotlin Spring Boot

Build:

```text
Android ─┐
         │
iOS ─────┼── API
         │
         ▼
    Spring Boot
         │
         ▼
    PostgreSQL
```

Implement:

- [ ] Transaction API
- [ ] Budget API
- [ ] Category API
- [ ] Validation
- [ ] Error Handling

---

# Week 10 — Production Backend

Implement:

- [ ] Authentication
- [ ] JWT
- [ ] Database Migration
- [ ] Unit Tests
- [ ] Integration Tests
- [ ] Docker
- [ ] Docker Compose

Goal:

```bash
docker compose up
```

starts the complete environment.

---

# Week 11 — Observability

Add OpenTelemetry:

- [ ] Traces
- [ ] Metrics
- [ ] Logs
- [ ] HTTP instrumentation
- [ ] Database instrumentation
- [ ] Error tracking
- [ ] Latency tracking

Flow:

```text
Mobile
   ↓
API
   ↓
Spring Boot
   ↓
Database

   │
   ▼

OpenTelemetry
```

Test:

- [ ] Timeout
- [ ] HTTP 500
- [ ] Slow API
- [ ] Database Error
- [ ] Network interruption

---

# Week 12 — Portfolio & Interview

**No new technology this week.**

## GitHub

### HomeKeep

- [ ] README
- [ ] Screenshots
- [ ] Architecture Diagram
- [ ] Offline-first explanation
- [ ] QR workflow
- [ ] Warranty workflow
- [ ] Testing strategy

### FamilyFlow

- [ ] README
- [ ] Android screenshots
- [ ] iOS screenshots
- [ ] KMP architecture
- [ ] Backend architecture
- [ ] Observability example

---

# 💼 Interview Stories

Prepare at least 5:

1. Offline-first architecture
2. Local data modeling
3. Background processing
4. KMP architecture
5. Observability / reliability

Explain using:

```text
Problem
   ↓
Constraints
   ↓
Options
   ↓
Decision
   ↓
Implementation
   ↓
Measurement
   ↓
Result
```

---

# 🏁 Final Portfolio

## Project #1

### HomeKeep

**Production-oriented Native Android Application**

Demonstrates:

```text
Kotlin
Jetpack Compose
Coroutines / Flow
Clean Architecture
Room
WorkManager
Camera / QR
Offline-first
Notifications
Testing
Performance
```

And most importantly:

> **Real Product used in daily life**

---

## Project #2

### FamilyFlow

**KMP + CMP Application**

Demonstrates:

```text
Kotlin Multiplatform
Compose Multiplatform
Android
iOS
Ktor
Shared Architecture
Spring Boot
PostgreSQL
Docker
OpenTelemetry
```

---

# 📌 Rules

1. **HomeKeep v1 must finish within 4 weeks**
2. HomeKeep must be used on a real phone from Week 2
3. Use real household data
4. Architecture must support future Sync — but **do not build Sync yet**
5. No AI in Month 1
6. No Cloud in Month 1
7. No Authentication in Month 1
8. Product Value > Feature Count
9. Architecture > Fancy UI
10. Tests > unnecessary features
11. Start interviewing after Month 1
12. If Layoff becomes imminent:

```text
CV
 ↓
Interview Preparation
 ↓
Job Applications
 ↓
Roadmap
```

---

# 🚀 90-Day Success Criteria

- [ ] HomeKeep installed and actively used
- [ ] 30–50 real Items stored
- [ ] QR attached to a real container
- [ ] Warranty Reminder working
- [ ] HomeKeep GitHub portfolio ready
- [ ] FamilyFlow works on Android
- [ ] FamilyFlow works on iOS
- [ ] Kotlin Backend working
- [ ] OpenTelemetry implemented
- [ ] Unit / Integration / UI tests
- [ ] Architecture documentation
- [ ] CV updated
- [ ] Senior Android interview ready
- [ ] KMP interview ready

**Target:** Senior Android / Senior Mobile / KMP Engineer
