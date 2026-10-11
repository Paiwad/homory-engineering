# Homory — Module Structure

> **Architecture:** Multi-Module + Clean Architecture\
> **Platform:** Pure Android\
> **UI:** Jetpack Compose\
> **Package Root:** `dev.panglab.homory`

---

# 📦 Initial Module Structure

เริ่มต้นด้วย **7 modules** ก่อน เพื่อไม่ให้เกิด over-engineering

```text
homory/
│
├── app/
│
├── core/
│   ├── model/
│   ├── database/
│   └── designsystem/
│
├── domain/
│   └── inventory/
│
├── data/
│   └── inventory/
│
└── feature/
    ├── home/
    └── item/
```

Gradle modules:

```text
:app

:core:model
:core:database
:core:designsystem

:domain:inventory
:data:inventory

:feature:home
:feature:item
```

---

# Module Types

| Module               | Type                | Android Dependency |
| -------------------- | ------------------- | ------------------ |
| `:app`               | Android Application | ✅                  |
| `:core:model`        | Kotlin/JVM Library  | ❌                  |
| `:core:database`     | Android Library     | ✅                  |
| `:core:designsystem` | Android Library     | ✅                  |
| `:domain:inventory`  | Kotlin/JVM Library  | ❌                  |
| `:data:inventory`    | Android Library     | ✅                  |
| `:feature:home`      | Android Library     | ✅                  |
| `:feature:item`      | Android Library     | ✅                  |

> `:core:model` และ `:domain:inventory` ควรเป็น **pure Kotlin** เพื่อไม่ให้ business logic ผูกกับ Android Framework

---

# Package Structure

Package root:

```text
dev.panglab.homory
```

แต่ละ module:

```text
:app
dev.panglab.homory

:core:model
dev.panglab.homory.core.model

:core:database
dev.panglab.homory.core.database

:core:designsystem
dev.panglab.homory.core.designsystem

:domain:inventory
dev.panglab.homory.domain.inventory

:data:inventory
dev.panglab.homory.data.inventory

:feature:home
dev.panglab.homory.feature.home

:feature:item
dev.panglab.homory.feature.item
```

---

# `:app`

**Type:** Android Application

Application entry point และ integration layer

```text
app/

└── src/main/java/dev/panglab/homory/

    ├── HomoryApplication.kt
    ├── MainActivity.kt
    ├── HomoryApp.kt
    │
    └── navigation/
        └── HomoryNavHost.kt
```

Responsibilities:

- Application lifecycle
- MainActivity
- Root Compose UI
- Navigation graph
- Dependency wiring
- Feature integration

ไม่ควรมี:

- Business Logic
- Room DAO
- Repository Implementation
- Feature-specific UI

Dependencies:

```text
:app
 │
 ├── :feature:home
 ├── :feature:item
 └── :core:designsystem
```

---

# `:core:model`

**Type:** Kotlin/JVM Library

เก็บ shared models ที่ไม่ขึ้นกับ Android Framework

```text
core/model/

└── src/main/kotlin/dev/panglab/homory/core/model/

    ├── Item.kt
    ├── Category.kt
    ├── Location.kt
    ├── Container.kt
    └── Warranty.kt
```

ตัวอย่าง:

```kotlin
data class Item(
    val id: String,
    val name: String,
    val description: String?,
    val categoryId: String?,
    val locationId: String?,
    val containerId: String?,
)
```

ห้ามมี:

```kotlin
@Entity
@Parcelize
@Composable
```

Model ใน module นี้ต้องเป็น **Plain Kotlin Object**

---

# `:core:database`

**Type:** Android Library

รับผิดชอบ Local Persistence ด้วย Room

```text
core/database/

└── src/main/java/dev/panglab/homory/core/database/

    ├── HomoryDatabase.kt
    │
    ├── entity/
    │   ├── ItemEntity.kt
    │   ├── CategoryEntity.kt
    │   ├── LocationEntity.kt
    │   └── ContainerEntity.kt
    │
    ├── dao/
    │   ├── ItemDao.kt
    │   ├── CategoryDao.kt
    │   ├── LocationDao.kt
    │   └── ContainerDao.kt
    │
    ├── migration/
    │
    └── di/
        └── DatabaseModule.kt
```

Responsibilities:

- Room Database
- Entity
- DAO
- Migration
- Database configuration

Example:

```kotlin
@Entity(tableName = "items")
data class ItemEntity(
    @PrimaryKey
    val id: String,
    val name: String,
)
```

ไม่ควร expose `ItemEntity` ไปถึง UI

---

# `:core:designsystem`

**Type:** Android Library + Compose

Design System กลางของ Homory

```text
core/designsystem/

└── src/main/java/dev/panglab/homory/core/designsystem/

    ├── theme/
    │   ├── Color.kt
    │   ├── Theme.kt
    │   ├── Typography.kt
    │   └── Shape.kt
    │
    ├── component/
    │   ├── HomoryButton.kt
    │   ├── HomoryCard.kt
    │   ├── HomoryTextField.kt
    │   ├── HomoryTopAppBar.kt
    │   └── HomoryEmptyState.kt
    │
    └── icon/
```

Responsibilities:

- Theme
- Typography
- Colors
- Reusable primitive UI
- Common Compose components

---

# `:domain:inventory`

**Type:** Kotlin/JVM Library

เป็น Business Layer หลักของ Homory

```text
domain/inventory/

└── src/main/kotlin/dev/panglab/homory/domain/inventory/

    ├── repository/
    │   ├── ItemRepository.kt
    │   ├── LocationRepository.kt
    │   └── CategoryRepository.kt
    │
    └── usecase/
        ├── AddItemUseCase.kt
        ├── UpdateItemUseCase.kt
        ├── DeleteItemUseCase.kt
        ├── GetItemUseCase.kt
        ├── GetItemsUseCase.kt
        ├── SearchItemsUseCase.kt
        └── GetExpiringWarrantyUseCase.kt
```

Repository Contract:

```kotlin
interface ItemRepository {

    fun observeItems(): Flow<List<Item>>

    fun searchItems(
        query: String
    ): Flow<List<Item>>

    suspend fun getItem(
        id: String
    ): Item?

    suspend fun saveItem(
        item: Item
    )

    suspend fun deleteItem(
        id: String
    )
}
```

Responsibilities:

- Business rules
- Repository contracts
- Use cases

ห้าม depend on:

- Android
- Room
- Compose
- ViewModel
- Activity/Context

---

# `:data:inventory`

**Type:** Android Library

Implementation ของ Domain Repository

```text
data/inventory/

└── src/main/java/dev/panglab/homory/data/inventory/

    ├── repository/
    │   ├── ItemRepositoryImpl.kt
    │   ├── LocationRepositoryImpl.kt
    │   └── CategoryRepositoryImpl.kt
    │
    ├── mapper/
    │   ├── ItemMapper.kt
    │   ├── LocationMapper.kt
    │   └── CategoryMapper.kt
    │
    └── di/
        └── InventoryDataModule.kt
```

Flow:

```text
Room
 │
 ▼
ItemEntity
 │
 ▼
Mapper
 │
 ▼
Item
 │
 ▼
Repository
```

Dependencies:

```text
:data:inventory
      │
      ├── :core:database
      ├── :core:model
      └── :domain:inventory
```

อนาคตสามารถเพิ่ม:

```text
data/inventory/

├── local/
├── remote/
├── sync/
├── mapper/
└── repository/
```

โดยไม่กระทบ Feature Layer

---

# `:feature:home`

**Type:** Android Library + Compose

Home Dashboard

```text
feature/home/

└── src/main/java/dev/panglab/homory/feature/home/

    ├── HomeScreen.kt
    ├── HomeViewModel.kt
    ├── HomeUiState.kt
    │
    ├── component/
    │   ├── RecentItemCard.kt
    │   ├── WarrantyCard.kt
    │   └── LocationCard.kt
    │
    └── navigation/
        └── HomeNavigation.kt
```

Responsibilities:

- Home UI
- Home ViewModel
- UI State
- User interaction

---

# `:feature:item`

**Type:** Android Library + Compose

จัดการ Item ทั้งหมด

```text
feature/item/

└── src/main/java/dev/panglab/homory/feature/item/

    ├── list/
    │   ├── ItemListScreen.kt
    │   ├── ItemListViewModel.kt
    │   └── ItemListUiState.kt
    │
    ├── detail/
    │   ├── ItemDetailScreen.kt
    │   ├── ItemDetailViewModel.kt
    │   └── ItemDetailUiState.kt
    │
    ├── editor/
    │   ├── ItemEditorScreen.kt
    │   ├── ItemEditorViewModel.kt
    │   └── ItemEditorUiState.kt
    │
    ├── component/
    │   ├── ItemCard.kt
    │   └── ItemImage.kt
    │
    └── navigation/
        └── ItemNavigation.kt
```

Dependencies:

```text
:feature:item
      │
      ├── :domain:inventory
      ├── :core:model
      └── :core:designsystem
```

Feature ห้ามเรียก:

```text
ItemDao ❌
HomoryDatabase ❌
ItemRepositoryImpl ❌
```

Feature รู้จักเฉพาะ:

```text
UseCase
Repository Interface
Domain Model
```

---

# Dependency Direction

Dependency ต้องไหลเข้า Domain ไม่ใช่ย้อนออกมา

```text
                         :app
                           │
             ┌─────────────┴─────────────┐
             │                           │
             ▼                           ▼
      :feature:home               :feature:item
             │                           │
             └─────────────┬─────────────┘
                           │
                           ▼
                  :domain:inventory
                           │
                           ▼
                     :core:model
                           ▲
                           │
                   :data:inventory
                           │
                           ▼
                   :core:database
```

UI dependencies:

```text
:feature:*
     │
     ▼
:core:designsystem
```

---

# Data Flow

อ่านข้อมูล:

```text
Room
 ↓
DAO
 ↓
RepositoryImpl
 ↓
Repository
 ↓
UseCase
 ↓
ViewModel
 ↓
StateFlow
 ↓
Compose UI
```

เขียนข้อมูล:

```text
User Action
 ↓
Compose
 ↓
ViewModel
 ↓
UseCase
 ↓
Repository
 ↓
RepositoryImpl
 ↓
DAO
 ↓
Room
```

---

# Gradle Includes

`settings.gradle.kts`

```kotlin
include(":app")

include(":core:model")
include(":core:database")
include(":core:designsystem")

include(":domain:inventory")
include(":data:inventory")

include(":feature:home")
include(":feature:item")
```

---

# Future Modules

**อย่าเพิ่งสร้างตอนเริ่ม Project**

เพิ่มเมื่อ Feature เริ่มมี complexity จริง

```text
core/
├── common/
├── ui/
├── testing/
└── notifications/

feature/
├── location/
├── scanner/
├── warranty/
└── settings/
```

อนาคตเมื่อมี Cloud:

```text
core/
└── network/

data/
└── sync/

sync/
├── worker/
├── queue/
└── conflict/
```

Full future structure:

```text
homory/

├── app/
│
├── build-logic/
│
├── core/
│   ├── common/
│   ├── model/
│   ├── database/
│   ├── network/
│   ├── designsystem/
│   ├── ui/
│   ├── notifications/
│   └── testing/
│
├── domain/
│   └── inventory/
│
├── data/
│   ├── inventory/
│   └── sync/
│
└── feature/
    ├── home/
    ├── item/
    ├── location/
    ├── scanner/
    ├── warranty/
    └── settings/
```

---

# Module Creation Guide

สร้างผ่าน Android Studio:

### Android Application

```text
:app
```

### Java or Kotlin Library

```text
:core:model
:domain:inventory
```

### Android Library

```text
:core:database
:core:designsystem

:data:inventory

:feature:home
:feature:item
```

### ❌ Do Not Use

```text
Dynamic Feature
```

Homory ไม่จำเป็นต้องใช้ Dynamic Feature Module ในตอนนี้

---

# Architecture Rules

1. Feature Module ห้ามเรียก DAO โดยตรง
2. Feature Module ห้ามรู้จัก Room Entity
3. Domain ห้าม depend on Android Framework
4. Domain ห้าม depend on Data Layer
5. Database ห้าม contain Business Logic
6. `ItemEntity` ≠ `Item`
7. Mapping เกิดใน Data Layer
8. Room เป็น Single Source of Truth สำหรับ v1
9. Remote API ยังไม่ต้องสร้างใน v1
10. Module ใหม่ต้องมีเหตุผลทาง Architecture ไม่ใช่สร้างเพื่อให้ Project ดูใหญ่

---

# Initial Setup

เริ่มจาก:

```text
:app

:core:model
:core:database
:core:designsystem

:domain:inventory
:data:inventory

:feature:home
:feature:item
```

**เป้าหมายคือให้ Homory ใช้งานได้จริงก่อน แล้วค่อยแตก Module ตาม Complexity ของ Product**
