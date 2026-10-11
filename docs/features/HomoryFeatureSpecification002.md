# Homory — Feature Specification 002

## Item Photo & Attachment

| Field | Detail |
|---|---|
| Feature ID | HOM-F002 |
| Feature Name | Item Photo & Attachment |
| Version | 1.0 |
| Priority | P0 — Must Have |
| Platform | Android Native |
| Dependencies | HOM-F001 — Item Management |
| Primary Module | `:feature:item` |
| Architecture | MVVM + Clean Architecture |
| Storage | App-private File Storage + Room |
| Status | Ready for Development |

---

## 1. Overview

### Problem Statement

การบันทึกชื่อและรายละเอียดสิ่งของอย่างเดียวอาจไม่เพียงพอ ผู้ใช้ต้องการเห็นรูปภาพเพื่อระบุสิ่งของได้รวดเร็ว และต้องการเก็บหลักฐานการซื้อหรือเอกสารที่เกี่ยวข้องไว้กับสิ่งของนั้น

### Solution

Homory สามารถถ่ายภาพหรือเลือกรูปจากอุปกรณ์มาแนบกับ Item รวมถึงแนบรูปใบเสร็จหรือไฟล์ PDF ได้

ไฟล์ถูกคัดลอกมาเก็บในพื้นที่ส่วนตัวของแอป และสามารถเปิดดูได้โดยไม่ต้องเชื่อมต่ออินเทอร์เน็ต

### Goals

- เพิ่มรูปภาพให้ Item ได้
- เปลี่ยนและลบรูปภาพได้
- แนบใบเสร็จหรือเอกสารได้
- เปิดดูรูปและเอกสารได้
- ไม่สูญเสียไฟล์เมื่อ URI ต้นทางไม่สามารถเข้าถึงได้
- จัดการไฟล์ที่ไม่ใช้งานแล้วอย่างปลอดภัย

---

## 2. Scope

### In Scope — v1.0

- [ ] Capture Item Photo
- [ ] Select Photo from Gallery
- [ ] Display Item Thumbnail
- [ ] View Full-size Photo
- [ ] Replace Item Photo
- [ ] Remove Item Photo
- [ ] Attach Receipt Image
- [ ] Attach Receipt PDF
- [ ] View Receipt
- [ ] Remove Receipt
- [ ] Delete Associated Files When Item Is Deleted
- [ ] File Validation
- [ ] Local File Persistence
- [ ] Error Handling

### Out of Scope

- Multiple item photos
- Multiple receipts
- OCR
- Cloud Storage
- Cloud Sync
- Image Editing / Cropping
- PDF Editing
- AI Image Recognition
- Document Scanning
- File Sharing
- Export / Backup

**v1 limitation:** Item หนึ่งชิ้นมีรูปหลักได้ 1 รูป และเอกสารใบเสร็จได้ 1 ไฟล์

---

## 3. User Stories

### US-001 — Capture Photo

As a user, I want to take a photo of an item so that I can recognize it later.

### US-002 — Select Photo

As a user, I want to select an existing photo so that I do not need to take a new one.

### US-003 — Manage Photo

As a user, I want to replace or remove an item's photo so that its information stays accurate.

### US-004 — Attach Receipt

As a user, I want to attach a receipt image or PDF so that I can retrieve proof of purchase when needed.

### US-005 — View Attachment

As a user, I want to open an attached receipt so that I can review its contents.

### US-006 — Delete Attachment

As a user, I want to remove an attachment without deleting the item itself.

---

## 4. Functional Requirements

### FR-001 — Capture Item Photo

The system shall allow users to capture a photo using the device camera.

**Business Rules**

- BR-001: A photo must be associated with an existing Item ID.
- BR-002: Only one primary item photo is supported.
- BR-003: A captured photo must be copied or saved into app-private storage.
- BR-004: Camera cancellation must not change the current photo.
- BR-005: The system must not save an incomplete or unreadable image.
- BR-006: A newly captured image must replace the existing photo only after successful processing and persistence.

**Acceptance Criteria**

- AC-001: User can open the camera from Item Detail or Edit Item.
- AC-002: Captured photo is displayed after a successful save.
- AC-003: Canceling the camera preserves the previous state.
- AC-004: Photo remains available after app restart.
- AC-005: Capture failure displays an error without corrupting the Item.

### FR-002 — Select Photo from Gallery

The system shall support selecting an image using Android Photo Picker.

**Business Rules**

- BR-007: Accept JPEG, PNG and WebP images.
- BR-008: Unsupported or unreadable files must be rejected.
- BR-009: Selected images must be copied to app-private storage.
- BR-010: The app must not depend on continued access to the original content URI.

**Acceptance Criteria**

- AC-006: User can select an existing photo.
- AC-007: The selected photo becomes the Item's primary photo.
- AC-008: Canceling the picker makes no changes.
- AC-009: Photo remains accessible when the original file is moved or removed, provided the app's private copy remains intact.
- AC-010: Unsupported files produce an understandable error.

### FR-003 — View Item Photo

The system shall display the item's primary photo.

**Acceptance Criteria**

- AC-011: Item List displays a thumbnail when a photo exists.
- AC-012: Item Detail displays the saved photo.
- AC-013: Tapping the photo opens a full-size viewer.
- AC-014: Items without photos display a placeholder.
- AC-015: Large images do not block the main thread during loading.

### FR-004 — Replace / Remove Photo

The system shall allow users to replace or remove an item's photo.

**Business Rules**

- BR-011: Replacement must not remove the previous file until the new reference is persisted successfully.
- BR-012: Removing a photo must not delete the Item.
- BR-013: Unreferenced files should be cleaned up safely.

**Acceptance Criteria**

- AC-016: User can replace an existing photo.
- AC-017: After replacement, the new photo is displayed.
- AC-018: User can remove the current photo.
- AC-019: Item information remains unchanged after photo removal.
- AC-020: Failed replacement preserves the previous photo.

### FR-005 — Attach Receipt

The system shall support attaching a receipt image or PDF.

**Supported Types**

| Type | MIME |
|---|---|
| JPEG | `image/jpeg` |
| PNG | `image/png` |
| WebP | `image/webp` |
| PDF | `application/pdf` |

**Business Rules**

- BR-014: One receipt attachment per Item.
- BR-015: File size must not exceed 10 MB.
- BR-016: The system must validate file content, not only the filename extension or declared MIME type.
- BR-017: Receipt must be stored in app-private storage.
- BR-018: Receipt replacement must preserve the old attachment if the new operation fails.

**Acceptance Criteria**

- AC-021: User can select an image receipt.
- AC-022: User can select a PDF receipt.
- AC-023: Selected receipt is linked to the correct Item.
- AC-024: File larger than 10 MB is rejected.
- AC-025: Unsupported or invalid file is rejected.
- AC-026: Receipt remains available after app restart.

### FR-006 — View / Remove Receipt

**Acceptance Criteria**

- AC-027: User can open an attached image receipt.
- AC-028: User can open an attached PDF using a compatible installed viewer.
- AC-029: User can remove an attached receipt.
- AC-030: Removing the receipt does not remove the Item.
- AC-031: A missing or corrupted attachment displays an error instead of crashing.

For external PDF viewing, use a temporary `content://` URI through `FileProvider` with read permission granted only to the receiving app.

### FR-007 — Delete Associated Files

When an Item is deleted, its photo and receipt must also be scheduled for cleanup.

**Acceptance Criteria**

- AC-032: Deleting an Item removes its database record.
- AC-033: The system cleans up the associated files after the Item deletion succeeds.
- AC-034: If cleanup fails, the app records the failure and retries cleanup later.
- AC-035: Failure to delete a file must not restore a previously deleted Item or cause an application crash.

---

## 5. UI/UX Specification

### Item Detail

```text
┌──────────────────────────────┐
│ ← Item Detail           Edit │
├──────────────────────────────┤
│                              │
│    ┌──────────────────┐      │
│    │                  │      │
│    │    ITEM PHOTO    │      │
│    │                  │      │
│    └──────────────────┘      │
│                              │
│    [ Change ] [ Remove ]     │
│                              │
│ Coffee Machine               │
│ Dunches CM4200               │
│ Quantity: 1                  │
│                              │
│ Receipt                      │
│ ┌──────────────────────────┐ │
│ │ 📄 receipt.pdf           │ │
│ │ View              Remove │ │
│ └──────────────────────────┘ │
│                              │
└──────────────────────────────┘
```

### Photo Source Selection

```text
Add Photo

[ Take Photo ]

[ Choose from Gallery ]

[ Cancel ]
```

### Empty Photo State

```text
┌──────────────────────────┐
│                          │
│       No Photo           │
│                          │
│     [ Add Photo ]        │
│                          │
└──────────────────────────┘
```

### UI States

- Idle
- Picking
- Processing
- Saving
- Success
- Error
- Missing File

All long-running operations must show appropriate progress or disable conflicting actions.

---

## 6. Data Model

### Design Decision

**ใช้ตาราง `item_attachments` แยกจาก `items`**

เหตุผล:

- ไม่ต้องเพิ่มคอลัมน์รูปภาพลงใน ItemEntity
- รองรับ attachment types ในอนาคต
- ลด coupling ระหว่าง Item Management และ File Storage
- สามารถขยายไปยัง multiple attachments ภายหลัง

### AttachmentType

Module: `:core:model`

```kotlin
enum class AttachmentType {
    ITEM_PHOTO,
    RECEIPT
}
```

### Domain Model

```kotlin
data class ItemAttachment(
    val id: String,
    val itemId: String,
    val type: AttachmentType,
    val relativePath: String,
    val mimeType: String,
    val fileSize: Long,
    val createdAt: Long,
)
```

### Room Entity

Module: `:core:database`

```kotlin
@Entity(
    tableName = "item_attachments",
    foreignKeys = [
        ForeignKey(
            entity = ItemEntity::class,
            parentColumns = ["id"],
            childColumns = ["itemId"],
            onDelete = ForeignKey.CASCADE
        )
    ],
    indices = [
        Index(value = ["itemId"]),
        Index(
            value = ["itemId", "type"],
            unique = true
        )
    ]
)
data class ItemAttachmentEntity(
    @PrimaryKey
    val id: String,
    val itemId: String,
    val type: AttachmentType,
    val relativePath: String,
    val mimeType: String,
    val fileSize: Long,
    val createdAt: Long,
)
```

Room requires a `TypeConverter` for the enum unless the selected Room configuration provides an appropriate conversion.

### Storage Structure

```text
files/
└── attachments/
    └── items/
        └── {itemId}/
            ├── {photoAttachmentId}.jpg
            └── {receiptAttachmentId}.pdf
```

Use generated filenames instead of user-provided filenames.

Store only relative paths in Room.

---

## 7. Architecture

### Modules

| Module | Responsibility |
|---|---|
| `:core:model` | Attachment model and type |
| `:core:database` | Attachment entity, DAO, migration |
| `:domain:inventory` | Attachment repository contract and use cases |
| `:data:inventory` | Repository implementation and file storage |
| `:feature:item` | Photo/receipt UI and ViewModels |
| `:app` | Navigation, Hilt integration, FileProvider configuration |

**Decision:** Do not create a new Gradle module for this feature yet. Add the functionality to the existing Item feature and Inventory data layer.

### Dependency Flow

```text
ItemDetailScreen
       ↓
ItemDetailViewModel
       ↓
AttachmentUseCase
       ↓
AttachmentRepository
       ↓
AttachmentRepositoryImpl
       ├── AttachmentDao
       └── LocalFileStorage
```

### Import Flow

```text
User selects file
       ↓
Photo Picker / Document Picker
       ↓
Validate content and size
       ↓
Copy to temporary private file
       ↓
Process / Compress if needed
       ↓
Commit private file
       ↓
Save attachment metadata in Room
       ↓
Display attachment
       ↓
Cleanup old or unused file
```

### Consistency Rules

Because Room and filesystem operations do not share one transaction:

1. Copy and validate the new file before changing database metadata.
2. Never overwrite the existing attachment file in place.
3. Persist the new file reference only after the new file is ready.
4. Delete the old file only after the new metadata is committed.
5. On failure, retain the previous attachment.
6. Provide a cleanup mechanism for orphaned files.

---

## 8. Use Cases

Required use cases:

```text
SaveItemPhotoUseCase
RemoveItemPhotoUseCase
GetItemPhotoUseCase

SaveReceiptUseCase
RemoveReceiptUseCase
GetReceiptUseCase
```

### Repository Contract

```kotlin
interface ItemAttachmentRepository {

    fun observeAttachments(
        itemId: String
    ): Flow<List<ItemAttachment>>

    suspend fun saveAttachment(
        itemId: String,
        type: AttachmentType,
        source: AttachmentSource
    ): Result<ItemAttachment>

    suspend fun removeAttachment(
        itemId: String,
        type: AttachmentType
    ): Result<Unit>
}
```

`AttachmentSource` should be a platform-neutral input abstraction. The Android data layer resolves the selected URI or staged file without exposing `Context`, `Uri`, or `ContentResolver` to the pure Kotlin domain module.

---

## 9. Android Implementation Guidelines

### Photo Selection

Use Android Photo Picker through Activity Result APIs.

- Prefer `PickVisualMedia`
- No broad storage permission required for supported picker flows
- Copy the selected content into app-private storage

### Camera Capture

Use `ActivityResultContracts.TakePicture` with a pre-created temporary URI.

- Use `FileProvider` for the camera output URI
- Verify the resulting image before importing
- Delete temporary files on cancellation or failure
- No in-app camera implementation required for v1

### Receipt Selection

Use `ActivityResultContracts.OpenDocument` or `GetContent` with supported MIME types.

- Validate the actual file content
- Copy into private storage immediately
- Avoid depending on persisted access to the original URI

### Image Loading

Use Coil for Compose.

- Load scaled thumbnails in Item List
- Avoid decoding full-resolution images for small UI elements
- Handle missing and corrupted files

### Storage

Use:

```kotlin
context.filesDir
```

Do not use external public storage for managed attachments.

Avoid storing image or PDF binary content directly in Room.

---

## 10. Database Migration

HOM-F002 introduces a new Room table.

Implementation tasks:

- [ ] Add `ItemAttachmentEntity`
- [ ] Add `ItemAttachmentDao`
- [ ] Add foreign key to `ItemEntity`
- [ ] Add unique index on `(itemId, type)`
- [ ] Register enum converter if required
- [ ] Increment database version
- [ ] Implement migration from existing schema
- [ ] Export Room schema
- [ ] Add migration tests

**Important:** Do not use destructive migration for users who already have HOM-F001 data.

---

## 11. Error Handling

| Scenario | Expected Behavior |
|---|---|
| User cancels picker | No changes |
| User cancels camera | Delete temporary file |
| Unsupported file | Show validation error |
| File exceeds size limit | Reject attachment |
| Insufficient storage | Preserve old attachment |
| Corrupted image | Reject or show error |
| Database save fails | Preserve previous metadata |
| File deletion fails | Retry cleanup later |
| Attachment file missing | Show missing-file state |
| No PDF viewer installed | Show understandable error |

No file-processing exception should crash the application.

---

## 12. Testing Specification

### Unit Tests

- [ ] Accept supported image formats
- [ ] Reject unsupported formats
- [ ] Reject oversized receipt
- [ ] Replace photo successfully
- [ ] Preserve old photo when replacement fails
- [ ] Remove photo without deleting Item
- [ ] Associate receipt with correct Item
- [ ] Handle missing attachment
- [ ] Prevent duplicate attachment type per Item

### Database Tests

- [ ] Insert attachment metadata
- [ ] Query attachments by Item ID
- [ ] Replace attachment metadata
- [ ] Enforce one attachment per type
- [ ] Verify foreign key cascade
- [ ] Verify database migration

### Integration Tests

- [ ] Import photo from picker URI
- [ ] Capture photo and persist it
- [ ] Open receipt PDF
- [ ] Remove attachment file
- [ ] Recover from file write failure
- [ ] Recover from database write failure
- [ ] Clean up orphaned files
- [ ] App restart retains attachments

### UI Tests

- [ ] Add photo
- [ ] Replace photo
- [ ] Remove photo
- [ ] Display photo placeholder
- [ ] Attach receipt
- [ ] View receipt
- [ ] Remove receipt
- [ ] Display import failure

### Manual Regression

- [ ] Existing Item CRUD still works
- [ ] Search still works
- [ ] Airplane mode
- [ ] App restart
- [ ] Process recreation
- [ ] Device rotation
- [ ] Camera cancellation
- [ ] Low storage condition
- [ ] Large image
- [ ] Thai filename
- [ ] App permission denial where applicable

---

## 13. Non-Functional Requirements

### Performance

- File I/O and image processing must run off the main thread.
- Large photos should be downsampled before display.
- Item List scrolling must remain smooth.
- File size limits must be enforced while reading to avoid unbounded memory usage.

### Reliability

- Failed replacement must preserve the previous attachment.
- File references must remain valid after app restart.
- Orphaned files must be recoverable through cleanup.
- Deleting an Item must not leave unmanaged files indefinitely.

### Privacy

- Files stored in app-private storage.
- No cloud upload.
- No broad media-library permission.
- No internet requirement.
- Attachments are deleted when the app is uninstalled unless separately backed up.

---

## 14. Implementation Plan

### Phase 1 — Attachment Foundation

- [ ] Define `AttachmentType`
- [ ] Define `ItemAttachment`
- [ ] Create Room entity and DAO
- [ ] Implement Room migration
- [ ] Create file storage abstraction
- [ ] Implement local file storage
- [ ] Implement attachment repository

### Phase 2 — Photo Management

- [ ] Integrate Android Photo Picker
- [ ] Integrate camera capture
- [ ] Implement photo import
- [ ] Implement photo replacement
- [ ] Implement photo removal
- [ ] Display Item thumbnail
- [ ] Display full-size photo

### Phase 3 — Receipt Management

- [ ] Integrate document picker
- [ ] Support receipt images
- [ ] Support PDF receipts
- [ ] Implement receipt replacement
- [ ] Implement receipt removal
- [ ] Implement receipt viewer

### Phase 4 — Reliability & Testing

- [ ] File validation tests
- [ ] Database migration tests
- [ ] Repository tests
- [ ] File consistency tests
- [ ] UI tests
- [ ] Item deletion cleanup
- [ ] Regression tests

---

## 15. Definition of Done

HOM-F002 is complete when:

- [ ] User can take a photo of an Item
- [ ] User can choose an existing photo
- [ ] User can view, replace and remove a photo
- [ ] Item List displays thumbnails
- [ ] User can attach an image receipt
- [ ] User can attach a PDF receipt
- [ ] User can view and remove receipts
- [ ] Attachments work without internet
- [ ] Attachments survive app restart
- [ ] Failed operations preserve existing attachments
- [ ] Item deletion triggers attachment cleanup
- [ ] Room migration preserves HOM-F001 data
- [ ] Unit and integration tests pass
- [ ] No critical regression in Item Management

---

## 16. Success Criteria

The user can:

1. Open an existing Item.
2. Take or select a photo.
3. View the photo in Item Detail.
4. Attach a receipt image or PDF.
5. Close and reopen Homory.
6. Retrieve both attachments without internet.
7. Replace or remove attachments safely.

**Release Milestone: HOM-F002 — Item Photo & Attachment MVP**

The feature is considered successful when at least 10 real household Items can be enriched with photos or receipts and remain usable offline.