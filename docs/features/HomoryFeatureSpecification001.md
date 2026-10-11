# Homory — Feature Specification 001

## Item Management — Household Item Registration

| Field | Detail |
|---|---|
| Feature ID | HOM-F001 |
| Feature Name | Item Management |
| Version | 1.0 |
| Priority | P0 — Must Have |
| Platform | Android Native |
| Module | `:feature:item` |
| Status | Ready for Development |
| Architecture | MVVM + Clean Architecture |
| Storage | Room (Local-first) |

---

## 1. Overview

Homory เป็นแอปพลิเคชันสำหรับช่วยผู้ใช้จัดเก็บข้อมูลสิ่งของภายในบ้าน เพื่อให้สามารถค้นหา ตรวจสอบ และจัดการข้อมูลสิ่งของได้อย่างสะดวก

**Problem Statement**

ผู้ใช้มีสิ่งของจำนวนมากภายในบ้าน แต่ไม่สามารถจดจำได้ว่าของแต่ละชิ้นอยู่ที่ไหน ซื้อเมื่อไร หรือมีข้อมูลสำคัญอะไรบ้าง

**Solution**

ผู้ใช้สามารถบันทึกสิ่งของลงใน Homory พร้อมรายละเอียดพื้นฐาน โดยข้อมูลจะถูกเก็บไว้ในอุปกรณ์และสามารถใช้งานได้โดยไม่ต้องเชื่อมต่ออินเทอร์เน็ต

### Product Goals

- เพิ่มสิ่งของได้ภายในประมาณ 30 วินาที สำหรับข้อมูลพื้นฐาน
- ใช้งานได้โดยไม่ต้องสมัครสมาชิก
- ข้อมูลคงอยู่หลังปิดและเปิดแอปใหม่
- แสดงรายการสิ่งของที่บันทึกไว้
- สามารถแก้ไขและลบข้อมูลได้
- รองรับการขยายไปยัง Location, Warranty และ QR Container ในอนาคต

---

## 2. Scope

### In Scope — v1.0

- [ ] Add Item
- [ ] Item List
- [ ] Item Detail
- [ ] Edit Item
- [ ] Delete Item
- [ ] Basic Search by Name
- [ ] Empty State
- [ ] Form Validation
- [ ] Local Persistence

### Out of Scope

- Cloud Sync
- Authentication
- Barcode / QR Scanner
- Warranty Reminder
- Location Hierarchy
- Category Management
- Receipt Attachment
- Multiple Photos
- AI Search
- Family Sharing
- Import / Export

**หมายเหตุ:** Feature แรกเน้นให้ Item CRUD สมบูรณ์ก่อน ไม่จำเป็นต้องทำทุกฟีเจอร์ของ Homory ในครั้งเดียว

---

## 3. User Stories

### US-001 — Add Item

As a HomeKeep user (Homory user),

I want to register an item in my home,

So that I can remember and manage it later.

### US-002 — View Items

As a user,

I want to view all registered items,

So that I can find and manage my belongings.

### US-003 — View Item Detail

As a user,

I want to view the details of a selected item,

So that I can inspect its stored information.

### US-004 — Edit Item

As a user,

I want to update an existing item,

So that its information remains accurate.

### US-005 — Delete Item

As a user,

I want to delete an item that I no longer own,

So that my inventory stays relevant.

### US-006 — Search Item

As a user,

I want to search items by name,

So that I can find my belongings quickly.

---

## 4. Functional Requirements

### FR-001 — Create Item

The system shall allow users to register a new item.

**Input Fields**

| Field | Type | Required | Validation |
|---|---|---|---|
| Name | String | Yes | 1–100 characters after trim |
| Description | String | No | Maximum 500 characters |
| Quantity | Integer | Yes | Minimum 1, default 1 |

**Business Rules**

- BR-001: Item name must not be blank.
- BR-002: Leading and trailing whitespace must be removed.
- BR-003: Duplicate item names are allowed.
- BR-004: Quantity must be greater than zero.
- BR-005: Each item must have a unique ID.
- BR-006: Created timestamp must be generated automatically.
- BR-007: Item data must be persisted locally before reporting success.

**Acceptance Criteria**

- AC-001: Given valid item information, when the user taps Save, the system stores the item in Room.
- AC-002: Given an empty name, when the user taps Save, the system displays a validation error.
- AC-003: Given an invalid quantity, the system prevents saving.
- AC-004: Given a successful save, the user returns to Item List and sees the new item.
- AC-005: Given the device is offline, the user can still create an item.
- AC-006: Repeated Save taps while a save is in progress must not create duplicate submissions.

### FR-002 — Item List

The system shall display all registered items.

**Requirements**

- Display item name
- Display quantity
- Display description preview when available
- Sort by creation date, newest first
- Automatically update when local data changes

**Acceptance Criteria**

- AC-007: Given no items exist, display an Empty State.
- AC-008: Given items exist, display all saved items.
- AC-009: Given a new item is saved, the list updates without manual refresh.
- AC-010: Given an item is deleted, it disappears from the list.
- AC-011: Given the app restarts, previously saved items remain available.

### FR-003 — Item Detail

The system shall allow users to view an item's details.

**Displayed Fields**

- Name
- Description
- Quantity
- Created At
- Updated At

**Acceptance Criteria**

- AC-012: Tapping an item opens its detail screen.
- AC-013: The correct item is loaded using its ID.
- AC-014: If the item no longer exists, the system displays an appropriate error or not-found state.

### FR-004 — Edit Item

The system shall allow users to modify an existing item.

**Business Rules**

- Existing item ID must remain unchanged.
- Created timestamp must remain unchanged.
- Updated timestamp must change after a successful update.
- The same validation rules as Create Item apply.

**Acceptance Criteria**

- AC-015: Editing an item displays its existing information.
- AC-016: Saving valid changes updates the existing record.
- AC-017: Invalid information cannot be saved.
- AC-018: Changes appear immediately in Item Detail and Item List.

### FR-005 — Delete Item

The system shall allow users to delete an existing item.

**Business Rules**

- Deletion requires confirmation.
- Canceling deletion must not modify data.
- Deletion is permanent in v1.0.
- No undo functionality in this version.

**Acceptance Criteria**

- AC-019: Tapping Delete displays a confirmation dialog.
- AC-020: Confirming deletion removes the item from Room.
- AC-021: Canceling deletion preserves the item.
- AC-022: After deletion, the user returns to Item List.

### FR-006 — Search Items

The system shall support basic item name search.

**Business Rules**

- Search is case-insensitive.
- Leading/trailing query whitespace is ignored.
- Empty search displays all items.
- Search must operate entirely on local data.

**Acceptance Criteria**

- AC-023: Entering a search query filters matching items.
- AC-024: Clearing the query restores all items.
- AC-025: A query with no matches displays a No Results state.
- AC-026: Search works without internet connectivity.

---

## 5. UI/UX Specification

### Screen 1 — Item List

```text
┌─────────────────────────────┐
│ Homory                      │
│ My Items                    │
├─────────────────────────────┤
│ 🔍 Search items...          │
├─────────────────────────────┤
│                             │
│ 📦 Coffee Machine           │
│    Quantity: 1              │
│                             │
│ 💻 MacBook                  │
│    Quantity: 1              │
│                             │
│ 🔌 HDMI Cable               │
│    Quantity: 3              │
│                             │
│                         [+] │
└─────────────────────────────┘
```

### Screen 2 — Add / Edit Item

```text
┌─────────────────────────────┐
│ ← Add Item                  │
├─────────────────────────────┤
│                             │
│ Item Name *                 │
│ [ Coffee Machine          ] │
│                             │
│ Description                 │
│ [ Dunches CM4200          ] │
│                             │
│ Quantity                    │
│ [ - ]       1       [ + ]   │
│                             │
│                             │
│ [       Save Item         ] │
└─────────────────────────────┘
```

### Screen 3 — Item Detail

```text
┌─────────────────────────────┐
│ ← Item Detail          Edit │
├─────────────────────────────┤
│                             │
│ ☕ Coffee Machine           │
│                             │
│ Dunches CM4200              │
│                             │
│ Quantity                    │
│ 1                           │
│                             │
│ Created                     │
│ 09 Oct 2026                 │
│                             │
│ [ Edit Item ]               │
│ [ Delete Item ]             │
└─────────────────────────────┘
```

### UI States

- Loading
- Success
- Empty
- Error
- Saving
- Deleting
- No Search Results

---

## 6. Data Model

### Domain Model

Module: `:core:model`

```kotlin
data class Item(
    val id: String,
    val name: String,
    val description: String?,
    val quantity: Int,
    val createdAt: Long,
    val updatedAt: Long,
)
```

Timestamps are stored as Unix epoch milliseconds.

### Database Entity

Module: `:core:database`

```kotlin
@Entity(
    tableName = "items",
    indices = [
        Index(value = ["createdAt"])
    ]
)
data class ItemEntity(
    @PrimaryKey
    val id: String,
    val name: String,
    val description: String?,
    val quantity: Int,
    val createdAt: Long,
    val updatedAt: Long,
)
```

### DAO Contract

```kotlin
@Dao
interface ItemDao {

    @Query(
        "SELECT * FROM items ORDER BY createdAt DESC, id DESC"
    )
    fun observeItems(): Flow<List<ItemEntity>>

    @Query(
        "SELECT * FROM items WHERE id = :id LIMIT 1"
    )
    fun observeItem(id: String): Flow<ItemEntity?>

    @Insert(onConflict = OnConflictStrategy.ABORT)
    suspend fun insert(item: ItemEntity)

    @Update
    suspend fun update(item: ItemEntity): Int

    @Query("DELETE FROM items WHERE id = :id")
    suspend fun deleteById(id: String): Int
}
```

---

## 7. Architecture Specification

### Module Responsibilities

| Module | Responsibility |
|---|---|
| `:app` | Application entry, navigation, DI integration |
| `:core:model` | Pure Kotlin item model |
| `:core:database` | Room entity, DAO, database |
| `:domain:inventory` | Repository contract, use cases |
| `:data:inventory` | Repository implementation, entity mapping |
| `:feature:item` | Compose screens, ViewModels, UI states |
| `:core:designsystem` | Shared Compose theme and components |

### Dependency Flow

```text
:app
  │
  ▼
:feature:item
  │
  ▼
:domain:inventory
  │
  ▼
:core:model

:data:inventory
  ├── :domain:inventory
  ├── :core:model
  └── :core:database
```

The application module wires the repository implementation into the dependency injection graph.

### Write Flow

```text
User
 ↓
ItemEditorScreen
 ↓
ItemEditorViewModel
 ↓
AddItemUseCase
 ↓
ItemRepository
 ↓
ItemRepositoryImpl
 ↓
ItemDao
 ↓
Room Database
```

### Read Flow

```text
Room Database
 ↓
ItemDao (Flow)
 ↓
ItemRepositoryImpl
 ↓
ObserveItemsUseCase
 ↓
ItemListViewModel
 ↓
StateFlow
 ↓
ItemListScreen
```

---

## 8. Use Case Specification

Required use cases:

```text
AddItemUseCase
UpdateItemUseCase
DeleteItemUseCase
GetItemUseCase
ObserveItemsUseCase
SearchItemsUseCase
```

### AddItemUseCase

Responsibilities:

1. Validate input
2. Normalize item name
3. Generate unique ID
4. Generate timestamps
5. Create domain model
6. Persist through repository
7. Return success or domain error

### UpdateItemUseCase

Responsibilities:

1. Validate input
2. Verify item exists
3. Preserve ID and creation timestamp
4. Update editable fields
5. Update modification timestamp
6. Persist changes

### DeleteItemUseCase

Responsibilities:

1. Receive item ID
2. Verify item exists
3. Delete through repository
4. Return success or not-found result

### SearchItemsUseCase

Responsibilities:

1. Normalize query
2. Search item names
3. Return observable results
4. Support empty query
5. Avoid unnecessary database queries during rapid typing

Recommended UI debounce: 250–300 ms.

---

## 9. Error Handling

| Scenario | Expected Behavior |
|---|---|
| Empty name | Display inline validation error |
| Name too long | Display validation error |
| Invalid quantity | Prevent saving |
| Database insert failure | Show error; preserve form |
| Database update failure | Show error; preserve changes |
| Item not found | Show not-found state |
| Database read failure | Show error with retry |
| Rapid Save taps | Prevent duplicate submission |
| Offline device | All features remain available |

Errors must not cause an unhandled application crash.

---

## 10. Testing Specification

### Unit Tests

- [ ] Add valid item
- [ ] Reject empty name
- [ ] Trim item name
- [ ] Reject quantity less than 1
- [ ] Allow duplicate names
- [ ] Generate unique item ID
- [ ] Preserve creation timestamp on update
- [ ] Update modification timestamp
- [ ] Delete existing item
- [ ] Handle missing item
- [ ] Search case-insensitively

### Database Tests

- [ ] Insert item
- [ ] Observe inserted item
- [ ] Update item
- [ ] Delete item
- [ ] Verify persistence
- [ ] Verify sorting
- [ ] Verify search query

### ViewModel Tests

- [ ] Initial loading
- [ ] Load success
- [ ] Empty state
- [ ] Validation error
- [ ] Save success
- [ ] Save failure
- [ ] Delete success
- [ ] Search query update

### UI Tests

- [ ] Navigate from list to editor
- [ ] Create item
- [ ] Edit item
- [ ] Delete confirmation
- [ ] Cancel deletion
- [ ] Search item
- [ ] Empty state rendering

### Manual Regression

- [ ] Airplane mode
- [ ] App restart
- [ ] Device rotation
- [ ] Background/foreground
- [ ] Process recreation
- [ ] Rapid button taps
- [ ] Thai item names
- [ ] Long item names

---

## 11. Non-Functional Requirements

### Performance

- Basic CRUD operations should feel responsive.
- Item List must support at least 1,000 records without noticeable UI blocking on the target test device.
- Database operations must not block the main thread.
- Search must not freeze the UI.

### Reliability

- No data loss after a confirmed successful save.
- UI state must remain consistent with persisted data.
- Failed operations must not report success.
- Database writes should be atomic.

### Privacy

- All item data stored locally.
- No account required.
- No analytics or third-party data transmission in v1.
- No internet permission required for this feature.

### Maintainability

- UI must not depend directly on Room.
- Domain must not depend on Android Framework.
- Repository contracts must be testable with fakes.
- Business rules must not be duplicated across ViewModels.

---

## 12. Implementation Tasks

### Phase 1 — Data Foundation

- [ ] Create Item domain model
- [ ] Create ItemEntity
- [ ] Create ItemDao
- [ ] Create HomoryDatabase
- [ ] Implement ItemMapper
- [ ] Create ItemRepository contract
- [ ] Implement ItemRepositoryImpl
- [ ] Configure Hilt dependency bindings

### Phase 2 — Business Logic

- [ ] AddItemUseCase
- [ ] UpdateItemUseCase
- [ ] DeleteItemUseCase
- [ ] GetItemUseCase
- [ ] ObserveItemsUseCase
- [ ] SearchItemsUseCase
- [ ] Add unit tests

### Phase 3 — Presentation

- [ ] ItemListScreen
- [ ] ItemListViewModel
- [ ] ItemEditorScreen
- [ ] ItemEditorViewModel
- [ ] ItemDetailScreen
- [ ] ItemDetailViewModel
- [ ] Navigation
- [ ] Form validation
- [ ] Error states

### Phase 4 — Quality

- [ ] DAO tests
- [ ] ViewModel tests
- [ ] Compose UI tests
- [ ] Offline verification
- [ ] Process recreation testing
- [ ] Regression testing

---

## 13. Definition of Done

Feature HOM-F001 is considered complete when:

- [ ] All P0 functional requirements implemented
- [ ] All acceptance criteria AC-001 to AC-026 pass
- [ ] Item CRUD works offline
- [ ] Data survives application restart
- [ ] Search works correctly
- [ ] Unit and database tests pass
- [ ] Critical UI flows tested
- [ ] No known critical defects
- [ ] Architecture boundaries respected
- [ ] Code reviewed
- [ ] README updated
- [ ] Feature demonstrated on a physical Android device

---

## 14. Future Extensions

HOM-F001 should be designed to support the following without requiring a major rewrite:

| Feature ID | Future Feature |
|---|---|
| HOM-F002 | Item Photo & Attachment |
| HOM-F003 | Location & Container Management |
| HOM-F004 | Warranty & Receipt |
| HOM-F005 | Warranty Notifications |
| HOM-F006 | QR Container |
| HOM-F007 | Backup & Restore |
| HOM-F008 | Cloud Synchronization |

These features are not included in HOM-F001.

---

## 15. Open Decisions

Before implementation, confirm:

- [ ] Whether quantity represents identical units or a single physical asset
- [ ] Whether deleted items require recovery in a future version
- [ ] Whether item IDs will be UUID strings
- [ ] Whether search should expand to descriptions later
- [ ] Whether photos will be stored in app-private storage

**Recommended defaults:** UUID identifiers, permanent delete for v1, app-private photo storage in the next feature, and quantity representing identical units of one item.

---

## 16. Release Milestone

**HOM-F001 — Item Management MVP**

The feature is successful when a user can:

1. Open Homory.
2. Register an item.
3. View the saved item.
4. Edit its information.
5. Search for it.
6. Delete it with confirmation.
7. Close and reopen the app without losing saved data.
8. Complete all operations without internet connectivity.

**Primary success metric:** A user can register at least 10 real household items and retrieve their details later without needing another tool.